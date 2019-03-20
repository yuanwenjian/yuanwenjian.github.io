---
title: Redis 笔记
description:  Redis笔记
date: 2019-02-03 14:40:00
comments: true
tags: 
    - Redis
categories:
    - Redis
---

# Redis 5.0 + 安装，启动及配置
## 安装
```bash
brew install redis 
```

## 设置密码
修改redis.conf 配置文件添加密码 路径为/usr/local/etc/redis.conf 
修改requirepass 配置

## redis单点启动

```bash
redis-server usr/local/etc/redis.conf 
```

# Redis集群搭建
## 多节点设置
```bash
mkdir -p /usr/local/etc/redis/6380
mkdir -p /usr/local/etc/redis/6381
mkdir -p /usr/local/etc/redis/6382

将redis.conf 拷贝到对象文件
```
## 修改配置
```bash
daemonize yes  
port 7001 #分别对每个机器的端口号进行设置  
dir /usr/local/redis-cluster/6380/ #指定数据文件存放位置，必须要指定不同的目录位置，不然会丢失数据
cluster-enabled yes #启动集群模式 
cluster-config-file nodes-6380.conf #集群节点信息文件，这里 700* 和port对应上  
cluster-node-timeout 5000   
 # bind 127.0.0.1 #去掉bind绑定访问ip信息
protected-mode  no  #关闭保护模式  
appendonly yes  #如果要设置密码需要增加如下配置： 
requirepass 123456  #设置redis访问密码
masterauth 123456  #设置集群节点间访问密码，跟上面一致
```

## 创建集群并启动
```bash
redis-cli -a 123456 --cluster create  127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382
# 可以使用redis-cli --cluster help获取帮助文档
```

## 登录集群并查看信息
```bash
redis-cli -a password -c -h host -p port # 登录

# 连接后查看
cluster nodes # 查看节点信息

cluster info # 


```

## 添加节点

### 新增节点
按**节点设置** 配置新增一节点，并启动，使用一下命令添加到集群
```bash
redis-cli -a password --cluster add-node ip1:port1  ip:port # ip1:port1为新增的节点，ip:port为之前存在的节点
```
添加后连接集群查看信息如下所示

```bash
127.0.0.1:6382> CLuster nodes
03952bb13258cb20996a9f45d08345b514bbc294 127.0.0.1:6381@16381 master - 0 1552402298000 3 connected 8192-12287
756f3036cd3231d2325b323750fadb17e2ee2354 127.0.0.1:6384@16384 master - 0 1552402301469 5 connected
3f3e07dd1850b4be8e22185d383725a7726511b1 127.0.0.1:6382@16382 myself,master - 0 1552402300000 4 connected 12288-16383
e88a3896016b13599988d99cbe157de53cbbcdcf 127.0.0.1:6383@16383 master - 0 1552402300434 0 connected
ae11b4462a529d4d8e17117cd384c8de22462afc 127.0.0.1:6379@16379 master - 0 1552402298376 1 connected 0-4095
70e6b35a208d18eb51bfcd328e577cc0c68ac73c 127.0.0.1:6380@16380 master - 0 1552402300000 2 connected 4096-8191
```
显示新增节点没有分配哈希槽，这时需要手动分配

### 重新分片
使用以下命令分配

1. 重新分配哈希槽
```bash
redis-cli -a password --cluster reshard ip:port # 之前有的节点
```

2. 输入哈希槽个数 出现以下提示 表示你想要为新增的节点添加多少哈希槽 16384/节点个数 输入
```bash
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)?
```

3. 为哪个节点分配, 查看节点信息，输入对应的hash值
```
What is the receiving node ID?
```

4. 选取分配的源节点，就是从哪个节点移动 ，平均分配的话输入all
```bash
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1:
```
5. 接下来输入确认移动即可

6. 完成后查看节点信息
```bash
127.0.0.1:6382> CLuster nodes
03952bb13258cb20996a9f45d08345b514bbc294 127.0.0.1:6381@16381 master - 0 1552402706415 3 connected 9443-12287
756f3036cd3231d2325b323750fadb17e2ee2354 127.0.0.1:6384@16384 master - 0 1552402705394 7 connected 0-453 683-1250 4778-5345 8874-9442 12970-13537
3f3e07dd1850b4be8e22185d383725a7726511b1 127.0.0.1:6382@16382 myself,master - 0 1552402706000 4 connected 13538-16383
e88a3896016b13599988d99cbe157de53cbbcdcf 127.0.0.1:6383@16383 master - 0 1552402706000 6 connected 454-682 4096-4777 8192-8873 12288-12969
ae11b4462a529d4d8e17117cd384c8de22462afc 127.0.0.1:6379@16379 master - 0 1552402707449 1 connected 1251-4095
70e6b35a208d18eb51bfcd328e577cc0c68ac73c 127.0.0.1:6380@16380 master - 0 1552402706000 2 connected 5346-8191
```

## 删除节点

1. 删除前先重新分片，因为节点包含数据，否则会报以下错误
```bash
redis-cli -a 123456 --cluster del-node 127.0.0.1:6383 756f3036cd3231d2325b323750fadb17e2ee2354  # ip:port 集群上ip
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Removing node 756f3036cd3231d2325b323750fadb17e2ee2354 from cluster 127.0.0.1:6383
[ERR] Node 127.0.0.1:6384 is not empty! Reshard data away and try again.
```

2. 按之前分片重新配置

3. 删除 出现以下表示节点删除成功
```bash
redis-cli -a 123456 --cluster del-node 127.0.0.1:6383 756f3036cd3231d2325b323750fadb17e2ee2354
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Removing node 756f3036cd3231d2325b323750fadb17e2ee2354 from cluster 127.0.0.1:6383
>>> Sending CLUSTER FORGET messages to the cluster...
>>> SHUTDOWN the node.
```

## 搭建集群报错处理
新增节点时报以下错误，原因为之前配置已经有缓存文件等，需要删除文件，dump.rdb ，配置文件cluster-config-file 配置的，然后重启，重新添加
```bash
[ERR] Node 127.0.0.16379 is not empty. Either the nodealready knows other nodes (check with CLUSTER NODES) or contains some key in database 0
```

# Redis 主从配置
## 主从配置
1. 主服务器正常启动
2. 从服务器配置redis.conf文件一下即可
```bash
slaveof ip port # 从服务器配置一下即可
```
## 查看主从配置信息

redis-cli 连接
```bash
 info replication
```

## 哨兵Sentinel 配置

1. 修改哨兵服务器配置

```bash
port 26379
sentinel monitor master ip 端口  quorum #  quorum 数字，表示master 下线所需的sentinel个数 master自定义监控主节点的名称， ip只能配置主节点
sentinel auth-pass master 123456   # 密码
```

2. 启动Sentinel服务
```bash
redis-sentinel redis-sentinel.conf配置文件路径
```

3. 启动后 关闭master节点 从节点会默认生成master，当之前主节点重启后会默认成为之后节点的从节点,可根据 info replication查看信息

---
title: Docker搭建mysql集群
description: 学习Mycat的读写分离及通过Mycat实现多租户,需要多台数据库,本地mysql有些问题,所以学习一下通过Docker安装mysql,并成功搭建mysql集群,集群架构为一主两副,本篇记录一下安装过程
date: 2020-05-27 20:40:00
comments: true
tags: 
    - Docker
    - Mysql  
categories:
    - Docker
    - Mysql
---

# 宿主主机创建文件及及配置文件,用来存放数据
```bash
# master主节点配置文件
mkdir -p docker/mysql-cluster/master/conf/
# master主节点数据目录
mkdir -p docker/mysql-cluster/master/data/
# slave1 节点配置文件
mkdir -p docker/mysql-cluster/slave1/conf/
# slave1 节点数据目录
mkdir -p docker/mysql-cluster/slave1/data/
# slave2 配置文件
mkdir -p docker/mysql-cluster/slave2/conf/
# slave2 数据目录
mkdir -p docker/mysql-cluster/slave2/data/
```

# 配置主从节点配置文件
## 主节点配置

```bash
cd docker/mysql-cluster/master/conf/
vim my.cnf 添加以下内容

# [mysqld]
# 用于标识不同的数据库服务器，而且唯一
server_id = 1
# 需要启用二进制日志
log-bin= mysql-bin
read-only=0
# 指定同步的数据库
binlog-do-db=user
# 忽略记录二进制日志的数据库
replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

```

## slave1 从节点1配置
```bash
cd docker/mysql-cluster/slave1/conf/
vim my.cnf 添加以下内容

[mysqld]
server_id = 2
log-bin= mysql-bin
read-only=1
# 指定同步的数据库
replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```
## slave2 从节点2配置
```bash
cd docker/mysql-cluster/slave2/conf/
vim my.cnf 添加以下内容

[mysqld]
server_id = 3
log-bin= mysql-bin
read-only=1
# 指定同步的数据库
replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

# 使用docker创建mysql,并且使用上述配置
```bash
# 创建一mysql容器,名称为cluster-master .外射端口为43306,密码为12345 ,数据目录映使用~/docker/mysql-cluster/master/data,配置文件使用~/docker/mysql-cluster/master/conf/my.cnf:/etc/mysql/my.cnf ,下面两个同理,启动两个从节点
docker run --name cluster-master  -d -p 43306:3306 -e MYSQL_ROOT_PASSWORD=12345 -v ~/docker/mysql-cluster/master/data:/var/lib/mysql -v ~/docker/mysql-cluster/master/conf/my.cnf:/etc/mysql/my.cnf  mysql:5.7.22

docker run --name cluster-slave1  -d -p 43307:3306 -e MYSQL_ROOT_PASSWORD=12345 -v ~/docker/mysql-cluster/slave1/data:/var/lib/mysql -v ~/docker/mysql-cluster/slave1/conf/my.cnf:/etc/mysql/my.cnf  mysql:5.7.22

docker run --name cluster-slave2  -d -p 43308:3306 -e MYSQL_ROOT_PASSWORD=12345 -v ~/docker/mysql-cluster/slave2/data:/var/lib/mysql -v ~/docker/mysql-cluster/slave2/conf/my.cnf:/etc/mysql/my.cnf  mysql:5.7.22

```

# 主节点创建主从备份的账号
```sql
-- 创建repl ,登录没有限制,密码为12345
grant replication slave ,replication client on *.* to repl@'%' identified by '12345' ;

```

# 从节点配置
```sql
-- 配置从节点,master_host因为是容器mysql,无法直接使用ip访问宿主网络,Mac可以配置ocker.for.mac.host.internal,
-- master_port 映射到宿主端口,user,password使用上面主节点配置,
-- master_log_file与master_log_pos 意思是读取哪个文件,读取位置多少.可以在主节点通过show master status 查看,
change master to master_host='docker.for.mac.host.internal', master_port=43306,master_user ='repl',master_password='12345',master_log_file='mysql-bin.000004',master_log_pos= 1869;
-- 开启主从备份
start slave 

-- 查看状态
show slave status 
```

配置完成,主节点新增删除会同步到从节点


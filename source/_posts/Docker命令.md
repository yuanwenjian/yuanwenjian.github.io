---
title: Docker命令
description: Docker是一个开源的引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在笔记本上编译测试通过的容器可以批量地在生产环境中部署，包括VMs（虚拟机）、 bare metal、OpenStack 集群和其他的基础应用平台。 
date: 2019-03-21 20:40:00
comments: true
tags: 
    - Docker
    - tool  
categories:
    - Docker
    - tool
---

# 命令

## 镜像常用命令
```bash
docker version # 查看版本
docker info  # 查看信息
docker images # 查看镜像 与docker image ls 类似
docker image ls # 查看镜像
docker ps # 查看运行的容器 与 docker container ls 类似
docker ps -l # 显示所有容器

```

## 容器常用命令
```bash
docker ps -l 显示所有容器，
docker container ls -a 显示所有容器包含终止的容器
docker container rm containerID # 删除容器
docker container prune # 清除所有终止的容器
docker exec container # 进入容器  参数 -i :即使没有附加也保持STDIN 打开 -t :分配一个伪终端
```
## docker mysql 导入数据命令
```bash
docker exec -i  db001 mysql -uroot -p12345  < /Users/yuanwenjian/github/xxl-job/doc/db/tables_xxl_job.sql #db001 容器名 
```
## docker 启动mysql 
```bash
docker run --name db001 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=12345 -d mysql:5.7.22
```

## docker 进入容器
```bash
docker exec -it db001 bash # db001 容器名
```

## docker 配置Redis主从复制 挂载外部配置文件
```bash

# 主节点 配置文件dir注掉
docker run -d -p 6379:6379 -v /Users/yuanwenjian/docker/redis/redis-slave/6379/redis.conf:/etc/redis/redis.conf -v /Users/yuanwenjian/docker/redis/redis-slave/6379/:/data --name redis-master redis:5.0.3 redis-server /etc/redis/redis.conf

# 从节点1 注意 从节点配置文件replicaof 属性不能使用ip,通过link参数使用容器名称代替，--link 详解 redis-master 表示主节点容器名，：master别名与配置文件对应
docker run -d -p 6380:6380 --link redis-master:master -v /Users/yuanwenjian/docker/redis/redis-slave/6380/redis.conf:/etc/redis/redis.conf -v /Users/yuanwenjian/docker/redis/redis-slave/6380/:/data --name redis-slave-1 redis:5.0.3 redis-server /etc/redis/redis.conf

#从节点2 
docker run -d -p 6381:6381 --link redis-master:master -v /Users/yuanwenjian/docker/redis/redis-slave/6381/redis.conf:/etc/redis/redis.conf -v /Users/yuanwenjian/docker/redis/redis-slave/6381/:/data --name redis-slave-2 redis:5.0.3 redis-server /etc/redis/redis.conf

```

## docker 配置Sentinel挂载外部配置
```bash  
docker run -d -p 26379:26379 -v /Users/yuanwenjian/docker/redis/redis-slave/6379/redis-sentinel.conf:/etc/redis/redis-sentinel.conf --link redis-master:docker-redis --name redis-sentinel-1 redis:5.0.3 redis-sentinel /etc/redis/redis-sentinel.conf

docker run -d -p 26380:26380 -v /Users/yuanwenjian/docker/redis/redis-slave/6380/redis-sentinel.conf:/etc/redis/redis-sentinel.conf --link redis-master:docker-redis --name redis-sentinel-2 redis:5.0.3 redis-sentinel /etc/redis/redis-sentinel.conf


docker run -d -p 26381:26381 -v /Users/yuanwenjian/docker/redis/redis-slave/6381/redis-sentinel.conf:/etc/redis/redis-sentinel.conf --link redis-master:docker-redis --name redis-sentinel-3 redis:5.0.3 redis-sentinel /etc/redis/redis-sentinel.conf
```

### sentinel配置注意事项
docker中配置sentine，容器内各个sentinel会无法通信，
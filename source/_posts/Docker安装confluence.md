---
title: Docker安装Confluence
description: Confluence是一个专业的企业知识管理与协同软件，也可以用于构建企业wiki。使用简单，但它强大的编辑和站点管理特征能够帮助团队成员之间共享信息、文档协作、集体讨论，信息推送。
date: 2020-05-27 20:40:00
comments: true
tags: 
    - Docker
categories:
    - Docker
---

1. 安装数据库
docker run --name mysql-confluence  -d -p 53306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v ~/docker/mysql-confluence/data:/var/lib/mysql  mysql:5.7.22

2. 下载镜像
docker pull cptactionhank/atlassian-confluence:7.3.1

3. 启动confluence 
docker run -d --name confluence -p 18090:8090  -v ~/docker/confluence/data:/var/atlassian/confluence --link mysql-confluence:db --user root:root cptactionhank/atlassian-confluence:7.3.1

4. 下载替换文件

wget https://gitee.com/bzone/app-repo/raw/master/atlassian-extras-decoder-v2-3.2.jar
wget https://gitee.com/bzone/app-repo/raw/master/atlassian-universal-plugin-manager-plugin-2.22.jar


5. 备份文件
cd /opt/atlassian/confluence/confluence/WEB-INF/lib 
mv atlassian-extras-decoder-v2-3.4.1.jar atlassian-extras-decoder-v2-3.4.1.jar.bak

cd /opt/atlassian/confluence/confluence/WEB-INF/atlassian-bundled-plugins 
mv atlassian-universal-plugin-manager-plugin-4.0.6.jar atlassian-universal-plugin-manager-plugin-4.0.6.jar.bak

6. 下载破解工具

https://files-cdn.cnblogs.com/files/Javame/confluence%E7%A0%B4%E8%A7%A3%E5%B7%A5%E5%85%B7.zip

解压,输入server-Id 

7. 配置数据库
jdbc:mysql://mysql-confluence:3306/confluence?sessionVariables=tx_isolation='READ-COMMITTED'&useUnicode=true&characterEncoding=utf8



6. 将下载文件copy到confluence
docker cp atlassian-extras-decoder-v2-3.2.jar confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar

docker cp atlassian-extras-2.4.jar confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar
docker cp atlassian-universal-plugin-manager-plugin-2.22.jar  confluence:/opt/atlassian/confluence/confluence/WEB-INF/atlassian-bundled-plugins/atlassian-universal-plugin-manager-plugin-4.0.6.jar

7. 启动

docker run 



confluence 
账号: admin 
密码: 123456 
邮箱: yuanwj0929@163.com



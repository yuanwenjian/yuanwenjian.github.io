---
title: Mycat实现读取分离
description:  Mycat实现读取分离配置
date: 2020-05-13 20:40:00
comments: true
tags: 
    - Java  
    - Mycat 
categories:
    - Mycat
    - Java 
---
# 安装数据库
使用docker安装三个数据库分别为db001,db002,db003
```java
docker run --name db001 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=12345 -d mysql:5.7.22
docker run --name db002 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=12345 -d mysql:5.7.22
docker run --name db003 -p 3308:3306 -e MYSQL_ROOT_PASSWORD=12345 -d mysql:5.7.22
```
# Mycat配置
```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

	<schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100" randomDataNode="dn1">
		<table name="wms_phone_imei"  dataNode="dn1" /> <!-- 表名数据库表 dataNode与<dataNode>节点对应 -->
	</schema>
	<dataNode name="dn1" dataHost="localhost1" database="user" /> <!-- dataHost与<dataHost对应> database物理数据库 -->

    <!-- balance 可选值为0,1,2,3 
    0表示禁止读写分离,所有读写请求落在主节点,
    1全部的readHost与stand by writeHost参与select语句的负载均衡，简单的说，当双主双从模式(M1->S1，M2->S2，并且M1与 M2互为主备)，正常情况下，M2,S1,S2都参与select语句的负载均衡。
    2表示所有读操作都随机的在 writeHost、readhost 上分发。
    3表示所有读操作落在readhost -->
	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="3"
			  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<!-- can have multi write hosts -->
		<writeHost host="db001" url="127.0.0.1:3306" user="root" password="12345"> <!-- 主数据库配置 -->
            <readHost host="db002" url="127.0.0.1:3307" user="root" password="12345" weight="1"/> <!-- 从节点配置 -->
            <readHost host="db003" url="127.0.0.1:3308" user="root" password="12345" weight="1"/>

		</writeHost>
		<!-- <writeHost host="hostM2" url="localhost:3316" user="root" password="123456"/> -->
	</dataHost>

	
</mycat:schema>
```
# 应用层SpringBoot配置

springboot 使用时与正常数据库连接一样,只是建数据库连接替换为Mycat连接即可

```yml
    url: jdbc:mysql://127.0.0.1:8066/TESTDB?useSSL=false
    username: root
    password: 123456
```


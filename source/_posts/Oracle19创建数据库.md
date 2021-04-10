---
title: Oracle19c 创建数据库
description:  Oracle19c 创建数据库记录
date: 2010-11-20 13:25:00
comments: true
tags: 
    - Oracle
categories:
    - Tools
---
# 创建docker容器
docker run --name oracle19c -p 1521:1521 -p 5500:5500 -v /Users/yuanwenjian/docker/oracle-19c: /opt/oracle/oradata 7b5eb4597688

ORCLCDB
system

123456

进入容器内,./passwordSet

https://mhl.xyz/Oracle/ORA-65096.html

查看PDB名称
select con_id,dbid,NAME,OPEN_MODE from v$pdbs;
打开PDB模式
alter pluggable database ORCLPDB1 open;


6、切换容器到pdb模式
alter session set container=ORCLPDB1;

创建表空间
create tablespace test  logging  datafile '/opt/oracle/oradata/ORCLCDB/test.dbf' size 200m  autoextend on  next 20m maxsize 20480m   ;


create user c##test identified by 123456  default tablespace test;
create user TEST identified by 123456  default tablespace test;

create user c##username identified by password;

授权
grant connect to c##test;
grant create session to c##test;
grant resource to c##test;
grant create table to c##test;
GRANT UNLIMITED TABLESPACE to  c##test;

原因是在CDB内创建用户分配表空间时，所分配的表空间必须在PDB和CDB中同时存在，否则会报错。
如果是在PDB与CDB有相同表空间的情况下给CDB用户分配表空间，则会分配CDB的表空间，
给用户PDB的表空间并不受影响。所以要在PDB内创建相同的表空间，然后再回CDB创建用户
1. 查询当前数据库名称
show con_name

年龄
2. 查询PDB数据库名称
select name,open_mode from v$pdbs;

3. 切换数据库
alter session set container=ORCLPDB1;

4. 创建表空间
create tablespace test  logging  datafile '/opt/oracle/oradata/ORCLCDB/ORCLPDB1/test.dbf' size 200m  autoextend on  next 20m maxsize 20480m   ;



5. 切换数据库CDB

alter session set container=CDB$ROOT;

6. 创建用户

# 创建实例

进入容器
cd 

## sqlplus 登录Oracle

sqlplus system/123456 as sysdba

## 创建用户及表空间
```sql
1：创建临时表空间
create temporary tablespace admin_temp  
tempfile '/opt/oracle/oradata/WMS/admin_temp.dbf' 
size 50m  
autoextend on  
next 50m maxsize 20480m  
extent management local;  
 
2：创建数据表空间
create tablespace admin_data   
datafile '/opt/oracle/oradata/WMS/admin_data.dbf' 
size 50m  
autoextend on  
next 50m maxsize unlimited  
extent management local
segment space management auto;
 
第3步：创建用户并指定表空间
create user admin identified by 123456 
default tablespace admin_data  
temporary tablespace admin_temp;  
 
第4步：给用户授予权限
grant connect,resource,dba to admin;



grant connect,resource,dba to admin;
```


# 创建数据库
```sql
CREATE DATABASE mynewdb
   USER SYS IDENTIFIED BY sys_password
   USER SYSTEM IDENTIFIED BY system_password
   LOGFILE GROUP 1 ('/u01/logs/my/redo01a.log','/u02/logs/my/redo01b.log') SIZE 100M BLOCKSIZE 512,
           GROUP 2 ('/u01/logs/my/redo02a.log','/u02/logs/my/redo02b.log') SIZE 100M BLOCKSIZE 512,
           GROUP 3 ('/u01/logs/my/redo03a.log','/u02/logs/my/redo03b.log') SIZE 100M BLOCKSIZE 512
   MAXLOGHISTORY 1
   MAXLOGFILES 16
   MAXLOGMEMBERS 3
   MAXDATAFILES 1024
   CHARACTER SET AL32UTF8
   NATIONAL CHARACTER SET AL16UTF16
   EXTENT MANAGEMENT LOCAL
   DATAFILE '/u01/app/oracle/oradata/mynewdb/system01.dbf'
     SIZE 700M REUSE AUTOEXTEND ON NEXT 10240K MAXSIZE UNLIMITED
   SYSAUX DATAFILE '/u01/app/oracle/oradata/mynewdb/sysaux01.dbf'
     SIZE 550M REUSE AUTOEXTEND ON NEXT 10240K MAXSIZE UNLIMITED
   DEFAULT TABLESPACE users
      DATAFILE '/u01/app/oracle/oradata/mynewdb/users01.dbf'
      SIZE 500M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED
   DEFAULT TEMPORARY TABLESPACE tempts1
      TEMPFILE '/u01/app/oracle/oradata/mynewdb/temp01.dbf'
      SIZE 20M REUSE AUTOEXTEND ON NEXT 640K MAXSIZE UNLIMITED
   UNDO TABLESPACE undotbs1
      DATAFILE '/u01/app/oracle/oradata/mynewdb/undotbs01.dbf'
      SIZE 200M REUSE AUTOEXTEND ON NEXT 5120K MAXSIZE UNLIMITED
   USER_DATA TABLESPACE usertbs
      DATAFILE '/u01/app/oracle/oradata/mynewdb/usertbs01.dbf'
      SIZE 200M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED;
```

# 运行脚本来构建数据字典视图

```sql
@?/rdbms/admin/catalog.sql
@?/rdbms/admin/catproc.sql
@?/rdbms/admin/utlrp.sql
@?/sqlplus/admin/pupbld.sql
catpcat.sql
```

catalog.sql
创建数据库字典表, 这是一个动态性能视图，同义词存放的视图视图，. 授予对同义词的公共访问权。

catproc.sql
运行PL/SQL所需的或与PL/SQL一起使用的所有脚本。

utlrp.sql
重新编译处于无效状态的所有PL/SQL模式，包、过程和类型。

pupbld.sql
需要 SQL*Plus. 启用或关闭用户命令.

catpcat.sql
构建数据字典。该脚本使用catctl.pl程序运行(不使用SQL*Plus)，并在内部运行catalog.sql和catproc.sql具有并行进程，从而提高了构建数据字典的性能。


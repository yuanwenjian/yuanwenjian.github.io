---
title: 服务器部署
description: 服务器部署
date: 2018-03-26 22:48:00
comments: true
tags: 
    - 笔记  
categories:
    - 笔记
---

# 安装必备软件
```bash
yum install -y lrzsz # 安装rz,sz上传下载文件

yum install -y wget #安装wget

yum install -y vim #安装vim

yum install -y unzip zip #安装压缩、解压缩

```
# 上传软件包至服务器 ~/tools目录

# 安装配置JDK
```bash
tar -xzvf jdk-7u79-linux-x64.tar.gz -C /usr/lib/   # 解压

#配置JDK
vim /etc/profile.d/java.sh

#!/bin/bash
export JAVA_HOME=/usr/lib/jdk1.7.0_79
export PATH=$PATH:$JAVA_HOME/bin

# 加载环境变量
/etc/profile.d/
source java.sh
```

# 安装配置tomcat

1. tar -zxvf apache-tomcat-7.0.68.tar.gz -C /var/lib/
2. cd /var/lib/
3. mv apache-tomcat-7.0.68/ tomcat7

4. 上传配置文件tomcat到/etc/init.d/目录下

4. cd /etc/init.d/
5. chmod a+x tomcat

chkconfig --add tomcat


# 安装配置redis

1. tar -zxvf redis-3.0.7.tar.gz -C /usr/local/ 

2. ~~可能需要安装gcc  
yum -y install gcc~~

3. 
``` bash
cd /usr/local/redis-3.0.7
make
make  MALLOC=libc
cd src/
make  test  #(make test遇到You need tcl 8.5 or newer in order to run the Redis test错误，需要安装tcl
# 安装tcl ， 
yum install tcl
```

4. 错误解决 如果遇到以下错误
```c#
如果装完redis， 运行make test 报了这样的一个错误：
!!! WARNING The following tests failed:

*** [err]: Test replication partial resync: ok psync (diskless: yes, reconnect: 1) in tests/integration/replication-psync.tcl
Expected condition '[s -1 sync_partial_ok] > 0' to be true ([s -1 sync_partial_ok] > 0)
Cleanup: may take some time... OK
make[1]: *** [test] Error 1
make[1]: Leaving directory `/usr/local/src/redis-3.2.1/src'
make: *** [test] Error 2

```
![redisError1][redisError1]

![redisErrro2][redisError2]
更改 tests/integration/replication-psync.tcl 文件

vi tests/integration/replication-psync.tcl
把对应报错的那段代码中的 after后面的数字，从100改成 500。我个人觉得，这个参数貌似是等待的毫秒数。
![fixError][fixError]

4. make install

5. 添加service服务

 - 复制

```bash
# 将redis_init_script拷贝到init.d/redis下
cp ./utils/redis_init_script /etc/rc.d/init.d/redis
```
 - 修改部分脚本 

```bash  
vim /etc/rc.d/init.d/redis
原文件是没有以下第2行的内容的，添加(带#号)
#chkconfig: 2345 80 90 
redis开启的命令，以后台运行的方式执行。
$EXEC $CONF &  
原文件EXEC、CLIEXEC参数，也是有所更改。 
EXEC=/usr/local/redis/bin/redis-server   
CLIEXEC=/usr/local/redis/bin/redis-cli 

新建目录存放配置文件
mkdir /etc/redis
mkdir /var/redis
mkdir /var/redis/log
mkdir /var/redis/run
mkdir /var/redis/6379

```
 
6. 添加redis配置文件
```bash 
cp  /usr/local/redis-3.0.7/redis.conf  /etc/redis/6379.conf
# 修改配置
requirepass  123456  #设置redis数据库连接密码
daemonize yes    #以后台daemon方式运行redis
pidfile /var/redis/run/redis_6379.pid   #redis后台运行，默认pid文件路径/var/run/redis.pid
logfile /var/redis/log/redis_6379.log  #可以指定日志文件路径
dir /var/redis/6379    #本地快照数据库存放目录
```

7. 添加服务
```bash
chkconfig  --add  redis
# 启动服务
service redis start
```

# 安装rabbit-mq
1.  安装软件
```bash
su -c 'rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm'
yum  install erlang
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.0.2/rabbitmq-server-3.0.2-1.noarch.rpm
yum localinstall rabbitmq-server-3.0.2-1.noarch.rpm
```

2.  修改配置文件
```bash
cd /usr/sbin/
vim /etc/rabbitmq/rabbitmq.config
# 再rabbitmq.config添加
[ 
{rabbit, [{tcp_listeners, [5672]}, {loopback_users, ["mquser"]}]} 
].

vim /etc/rabbitmq/rabbitmq-env.conf
# 添加以下内容
NODENAME=rabbitmq@localhost

```
3. 启动rabbit服务
```bash
service rabbitmq-server start
```

4. 添加用户,并赋予权限
```bash 
cd /usr/sbin/
service rabbitmq-server start
#创建管理员用户，负责整个MQ的运维
sudo ./rabbitmqctl add_user mquser 123456
#赋予其administrator角色
sudo ./rabbitmqctl set_user_tags mquser administrator 
# 该命令使用户mquser具有/这个virtual host中所有资源的配置、写、读权限以便#管理其中的资源
sudo ./rabbitmqctl set_permissions -p / mquser ".*" ".*" ".*"
sudo ./rabbitmq-server start & 
```
5. 设置开机自启动
```bash 
chkconfig rabbitmq-server on
```

# 安装zookeeper

1. 解压到 /usr/local下
```bash 
tar -zxvf zookeeper-3.4.9.tar.gz  -C /usr/local/
```
2. 生成配置
```bash
conf/目录下有个zoo_sample.cfg，是样板配置文件
复制一份成zoo.cfg在当前目录下
cd /usr/local/zookeeper-3.4.9/conf/
cp zoo_sample.cfg  zoo.cfg
里面有两个比较重要的配置：
dataDir=/var/lib/zookeeper # 数据存放位置，可根据需要修改
clientPort=2181 # 服务监听端口，可根据需要修改
```

3. 上传配置文件zookeeper到/etc/init.d/，修改配置路径和参数
```bash
vim /etc/init.d/zookeeper
修改配置文件中的参数  bin/zkServer.sh start &

修改执行权限 chmod a+x zookeeper
添加到开机启动项chkconfig  --add  zookeeper

服务器重启之后需要重启一下zookeeper服务
service zookeeper stop 
service zookeeper start
```


#安装Mysql
```bash
需要首先删除系统自动的mysql
rpm -qa|grep mysql

rpm -e mysql-libs --nodeps

解压缩文件
cd /root/tools
tar -xvf mysql-5.7.11-1.el6.x86_64.rpm-bundle.tar

本地安装
yum localinstall mysql-community-libs-5.7.11-1.el6.x86_64.rpm mysql-community-common-5.7.11-1.el6.x86_64.rpm mysql-community-client-5.7.11-1.el6.x86_64.rpm mysql-community-server-5.7.11-1.el6.x86_64.rpm 


启动mysql服务
service mysqld start

生成log文件，包含临时密码
grep "password" /var/log/mysqld.log
记录临时密码4o(frDfe*I&k
用临时密码登录后修改密码
step 1: SET PASSWORD = PASSWORD('Ss-wms123456');
设置密码永不过期
step 2: ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
step 3: flush privileges;


替换 /etc/my.cnf
修改中间的路径参数

```

# 挂载磁盘修改配置文件
```bash
1. 使用 fdisk -l  查看哪个磁盘没有挂载
fdisk –l
df -h就是查看磁盘容量的使用情况。


4.将未挂载磁盘挂载到mysql数据路径下
挂载命令
mkfs.ext4 /dev/vdb 将/dev/vdb格式化为ext4类型  （不明确类型，可能报：mount: you must specify the filesystem type的错误）

mount /dev/vdb /data

5. 开机自动挂载
在/etc/fstab 添加以下命令
/dev/vdb              /home                  ext4    defaults        0 0


mysql挂载后：对应修改my.cnf信息：

[mysqld]
datadir=/data/lib/mysql
socket=/data/lib/mysql/mysql.sock
[client]
port		= 3306
socket	= /data/lib/mysql/mysql.sock
对服务器和客户均指定路径名，使得它们都使用同一个套接字文件。如果你只为服务器设置路径，客户程序将仍然期望在原位置执行套接字，在修改后重启服务器，使它在新位置创建套接字

```

# 上传shell
上传两个配置文件dubboconfig.zip    shells.zip到 /root/tools/ 下
该shell，可能具体内部路径需要根据实际路径修改，如备份文件放置目录等。
需要修改dbconfig文件，配置当前数据库的host、用户名、密码。

# 防止误删除操作
```bash 
mkdir –p /root/.trash/
cd /root/shells/
cp rmcmd.sh /etc/profile.d/
source /etc/profile.d/rmcmd.sh

```

# 文件夹创建
```bash
mkdir -p /root/setup
mkdir -p /bootapps/logs/
```

# 配置防火墙
防火墙（谨慎设置，容易把自己挡在系统外）
1. 修改 vim /etc/sysconfig/iptables,在22端口配置下，加入开放80,8080端口

```bash
vim /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT

```
2. 重启防火墙
```bash
service iptables restart
```
# 自定义快捷命令
1. 修改  /root/.bashrc 文件
```bash
vim root/.bashrc
alias tailtomcat='tail -n 100 -f /var/lib/tomcat7/logs/catalina.out'
alias cdtomcatc='cd /var/lib/tomcat7/webapps/ecloud-wms/WEB-INF/classes'
alias cdtomcat='cd /var/lib/tomcat7/bin'
```
2. 加载配置  
```bash
source /root/.bashrc
```
# tomcat相关配置

1. 修改server.xml
在server.xml 文件中添加\</Host>之前添加
```xml
<Context docBase="/var/resources/wmsfile"  reloadable="true"  debug="0" path="/wmsfile"/>
```
2. 修改启动参数

```bash
cd /var/lib/tomcat7/bin/
vim catalina.sh
# 添加以下内容
JAVA_OPTS="-server -Xms4048m -Xmx4048m -Xss512k -XX:PermSize=256m -XX:MaxPermSize=512m"
```

# 15、服务启动
启动服务:shells 文件夹下执行
```bash 
./dubboDeploy.sh 将项目全部部署
./dubboStartup.sh 重新编译edb_wms.war 主业务
tomcat下，/bin  ./startup.sh 启动tomcat
正常情况，操作启动tomcat即可
```



[redisError1]:../images/redisError1.png
[redisError2]:../images/redisError2.png
[fixError]:../images/fixError.png
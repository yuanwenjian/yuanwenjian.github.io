---
title: Mac开发环境搭建记录
description:  记录Mac开发环境搭建,以免忘记 
date: 2018-07-13 00：31:00
tags: 
    - Mac
categories:
    - Mac
---

# 描述
记录Mac软件下载地址机及安装方法，方便以后下载

# #JDK

## jdk下载
下载之前需要注册Oracle账号
[jdk7下载地址][jdk7下载地址]
[jdk8下载地址][jdk8下载地址]


[jdk7下载地址]:http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html
[jdk8下载地址]:http://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html

## 安装
下载完成点击安装

## 环境变量配置
进入家目录 cd ~/
vim .bash_profile
加入以下配置
```bash
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_162.jdk/Contents/Home  //java目录，默认为这个
PATH=$JAVA_HOME/bin:$PATH:.
export JAVA_HOME
export PATH
```

## 测试
java -version 

## office安装
## office 官网下载地址
https://officecdn-microsoft-com.akamaized.net/pr/C1297A47-86C4-4C1F-97FA-950631F94777/OfficeMac/Microsoft_Office_2016_16.15.18070902_Installer.pkg

## 破解软件
[破解链接参考][office破解]
## Navicat破解版安装

[Navicate下载目录][Navicat]

下载后安装即可

[Navicat]:http://160721.16.unicom.data.tv002.com:443/down/c04e6a65c4e70f6eca654e26a49476d5-101555179/NavicatPremium1128.dmg?cts=wt-f-D125A34A83A225F24ea3&ctp=125A34A83A225&ctt=1531538800&limit=1&spd=85000&ctk=d94cacb99e2db3acc1a95f66a5e300aa&chk=c04e6a65c4e70f6eca654e26a49476d5-101555179&mtd=1

## Mysql安装

[Mysql下载目录][Mysql]

### 配置mysql环境变量
将mysql添加到环境变量，以便使用命令行
```bash
cd ~/
vim .bash_profile
# 添加环境变量
PATH=$PATH:/usr/local/mysql/bin
```

### 重置密码
1. 关闭mysql 
    - 使用命令关闭 sudo /usr/local/mysql/support-files/mysql.server stop
    - 系统设置 -》mysql -> Stop mysql server

2. 使用root进入mysql目录并重启
```bash
sudo -i 
cd /usr/local/mysql/bin
./mysqld_safe --skip-grant-tables &
```

3. 新开终端

4. 登录mysql 
```bash
mysql -uroot -p
```
密码随便输入

5. 获取权限
flush privileges;

6. 设置新密码
```bash
set password for 'root'@'localhost'=password('新密码');
```

## Maven配置

### maven下载
[maven下载地址][maven]

### 配置
将下载解压，配置到环境变量中

## oh my zsh安装

## 安装
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

## 改默认shell
echo $SHELL   查看当前终端

chsh -s /bin/zsh  修改
重启终端


## 安装redis
```bash
brew install redis 
```
修改redis密码
```bash
onfig set requirepass passwords
```

[maven]:http://mirrors.shu.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
[Mysql]:https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.22-macos10.13-x86_64.dmg
[office破解]:https://www.jianshu.com/p/3c5dc4f3c96e
[jdk7下载地址]:http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html
[jdk8下载地址]:http://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html

## 安装zookeeper
```
brew install zookeeper
```bash 

修改zookeeper配置
```bash 
vim /usr/local/etc/zookeeper/zoo.cfg
```

zookeeper 启动
```bash
zkServer start 
```

安装目录
/usr/local/Cellar/zookeeper/

配置目录
/usr/local/etc/zookeeper

## 安装nginx

```bash 
brew install nginx
```
安装目录
/usr/local/var/www

配置文件
/usr/local/etc/nginx/nginx.conf

日志目录
/usr/local/var/log/nginx



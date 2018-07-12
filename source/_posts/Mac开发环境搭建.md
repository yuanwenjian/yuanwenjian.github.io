---
title: Mac开发环境搭建记录
description:  记录Mac开发环境搭建,以免忘记 
date: 2018-07-13 00：31:00
tags: 
    - Mac
categories:
    - Mac
---

# JDK

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

# office安装
## office 官网下载地址
https://officecdn-microsoft-com.akamaized.net/pr/C1297A47-86C4-4C1F-97FA-950631F94777/OfficeMac/Microsoft_Office_2016_16.15.18070902_Installer.pkg

## 破解软件
[破解链接参考][office破解]

[office破解]:https://www.jianshu.com/p/3c5dc4f3c96e
[jdk7下载地址]:http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html
[jdk8下载地址]:http://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html

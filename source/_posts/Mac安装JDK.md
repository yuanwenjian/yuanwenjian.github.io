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
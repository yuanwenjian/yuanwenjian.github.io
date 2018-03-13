---
title: linux 命令
description:  linux 命令记录
date: 2018-02-02 20:40:00
comments: true
tags: 
    - Linux
    - Shell  
categories:
    - Linux
---
# linux 自用命令记录

## awk 常用命令
### awk与if合用
```bash
awk -F '   ' {if($3="aa")print $)} inFile> outFile
```
## 获取文件列数
```bash
awk -F '    ' '{print $NF}'  filename //统计fileName列数
```
## awk内置常用变量

|变量名|说明 |
|-----|------|
|  $0 |正列|
|$1~$n| 第几个字段|
|$NR| 已经读到的行|
|$NF| 列数|
|FINENAME|当前文件名|

## 查看端口号占用的进程
```bash
lsof -i:port
```

## 压缩解压命令
### tar解压tar.zx文件
```shell
1. 先用xz -d file.tar.zx
2. tar -xvf file.tar  -z对应的gzip文件 -j对应的是bzip2
```
### 压缩成tar.zx文件
``` bash
1.xz -z file
2.tar cvf file.zx
```

## unzip命令

### 解压指定目录

```
    unzip -d
```

### zip压缩指定文件名

```
 zip -j fileName file
```

### rar 解压
```
unrar v file 查看文件
unrar x file.rar
unrar x file.rar -w path 解压到指定目录
```

### rar 压缩

```bash
rar a file.rar path 将path目录压缩
rar a -df file.rar path 压缩并删除原始文件
```

将/etc/passwd 的内容列出并且打印行号，同时，请将2-5行删除

在第二行后（即第三行）加上“drink tea?”

插入两行呢？

整行取代，我想将2-5行的内容取代为“No 2-5 number”

仅列出第5-7行

取出本机ip地址：

删除注释
sed '/^#/d' f
删除空白行
sed '/^$/d' file
在regular_express.txt文件末尾加入一行# hello world
sed '$a #helloword' 在regular_express.txt

处理/etc/passwd 为一个新的文件，方式为：删除第四行，第六行则替换成“no six line”

sed '4d' | sed '6c no six line' passwd > fileName
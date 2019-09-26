---
title: 服务器SpringBoot修改单个class
description: springboot以jar包启动,当修改服务器bug时,需要将整个jar包上传,网速慢的话太耗时,本文为修改单个class,就能重新编译,避免整个jar上传
date: 2019-09-31 13:59:00
comments: true
tags: 
    - tool  
    - SpringBoot
categories:
    - tool
    - SpringBoot
---

# 修改class

1. 将jar复制为zip文件
```bash
cp  xxx.jar xxx.zip
```

2. 解析压缩文件
```bash
unzip xxx.zip
```
3. 进入解压后的文件,替换对应的class文件
```
cd BOOT-INF/classes/

# 上传单个文件,替换原始class
```

4. 重新打包成jar文件
```bash
jar -cvfM0 xxx.jar BOOT-INF/ META-INF/ org/  # c 创建jar压缩文件 v输出详细内容 M 不创建MET-INF 0不压缩

```
# jar命令详解
```bash
用法: jar {ctxui}[vfmn0PMe] [jar-file] [manifest-file] [entry-point] [-C dir] files ...
选项:
    -c  创建新档案
    -t  列出档案目录
    -x  从档案中提取指定的 (或所有) 文件
    -u  更新现有档案
    -v  在标准输出中生成详细输出
    -f  指定档案文件名
    -m  包含指定清单文件中的清单信息
    -n  创建新档案后执行 Pack200 规范化
    -e  为捆绑到可执行 jar 文件的独立应用程序
        指定应用程序入口点
    -0  仅存储; 不使用任何 ZIP 压缩
    -P  保留文件名中的前导 '/' (绝对路径) 和 ".." (父目录) 组件
    -M  不创建条目的清单文件
    -i  为指定的 jar 文件生成索引信息
    -C  更改为指定的目录并包含以下文件
如果任何文件为目录, 则对其进行递归处理。
清单文件名, 档案文件名和入口点名称的指定顺序
与 'm', 'f' 和 'e' 标记的指定顺序相同。

```
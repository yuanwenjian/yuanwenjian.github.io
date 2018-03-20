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
### sed常用命令
命令格式
> sed [options] 'command' file(s)

选项
>1. -e或--expression：以选项中的指定的script来处理输入的文本文件；
>2. -f或--file：以选项中指定的script文件来处理输入的文本文件；
>3. -h或--help：显示帮助；
>4. -n或--quiet或——silent：仅显示script处理后的结果；
>5. -V或--version：显示版本信息。

命令
>1. a\ 在当前行下面插入文本。
>1. i\ 在当前行上面插入文本。
>1. c\ 把选定的行改为新的文本。
>1. d 删除，删除选择的行。
>1. D 删除模板块的第一行。
>1. s 替换指定字符
>1. h 拷贝模板块的内容到内存中的缓冲区。
>1. H 追加模板块的内容到内存中的缓冲区。
>1. g 获得内存缓冲区的内容，并替代当前模板块中的文本。
>1. G 获得内存缓冲区的内容，并追加到当前模板块文本的后面。
>1. l 列表不能打印字符的清单。
>1. n 读取下一个输入行，用下一个命令处理新的行而不是用第一个命令。
>1. N 追加下一个输入行到模板块后面并在二者间嵌入一个新行，改变当前行号码。
>1. p 打印模板块的行。
>1. P(大写) 打印模板块的第一行。
>1. q 退出Sed。
>1. b lable 分支到脚本中带有标记的地方，如果分支不存在则分支到脚本的末尾。
>1. r file 从file中读行。
>1. t label if分支，从最后一行开始，条件一旦满足或者T，t命令，将导致分支到带有标>1. 号的命令处，或者到脚本的末尾。
>1. T label 错误分支，从最后一行开始，一旦发生错误或者T，t命令，将导致分支到带有>1. 标号的命令处，或者到脚本的末尾。
>1. w file 写并追加模板块到file末尾。  
>1. W file 写并追加模板块的第一行到file末尾。  
>1. ! 表示后面的命令对所有没有被选定的行发生作用。  
>1. = 打印当前行号码。  

示例
```bash
删除包含某个单词的行
sed 's/^*word*$/d'
在第二行后（即第三行）加上“drink tea?”
sed '2a drink tea'
整行取代，我想将2-5行的内容取代为“No 2-5 number”
仅列出第5-7行
sed -n '5,7p' fileName
删除注释
sed '/^#.*/d' fileName
删除空白行
sed '/'
在regular_express.txt文件末尾加入一行# hello world
sed '$a hello world' fileName
处理/etc/passwd 为一个新的文件，方式为：删除第四行，第六行则替换成“no six line”
sed '4d'  |sed '6c noSix'  passwd > fileName
```
### sort命令

|参数| 描述|
|-|-|
|-t|指定分隔符|
|-n|按照数值排序|
|-r|倒序|
|-k|指定按照第几列|
|-u| 去重| 
|-f|忽略大小写|
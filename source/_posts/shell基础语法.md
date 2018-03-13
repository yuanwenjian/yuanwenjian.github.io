---
title: shell基础语法
description:  shell基础语法
date: 2018-02-26 20:40:00
comments: true
tags: 
    - Linux
    - Shell  
categories:
    - Linux
---

# linxu 自用脚本

## linux 运算符
### 关系运算符
以下参数a为10,b为20,加减乘除省略,

|运算符| 说明 | 示例 |
|-|-|-|
|%| 取余 | `expr $a % $b` 结果10| 
|-eq| 判断相等,相等返回true | [ $a -eq $b ] false|
|-ne| 判断相等,不相等返回true | [$a -ne $b] true|
|-le| 小于或等于| [ $a -le $b ]  true|
|-lt| 小于 | [ $a -lt $b ] true|
|-ge| 大于或等于 | [$b -ge $a ] true |
|-gt| 大于 | [ $b -gt $a ] true|
### 布尔运算符
|运算符| 说明 | 示例 |
|-|-|-|
|!| 非| [ !false ] true|
|-a| 与运算 | [ $a -lt 20 -a $b -gt 100 ] false|
|-o| 或运算| [ $a -lt 20 -o $b -gt 100 ] true|

### 逻辑运算符
&& and || 

|运算符| 说明 | 示例 |
|-|-|-|
|&&| 逻辑与 |[[ $a -lt 100 && $b -gt 100 ]] 返回 false|
| 双竖线 |逻辑或 |[[ $a -lt 100 双竖线 $b -gt 100 ]] 返回 true|

### 字符运算符
|运算符| 说明 | 示例 |
|-|-|-|
|=	|检测两个字符串是否相等，相等返回 true。|	[ $a = $b ] 返回 false。|
|!=|	检测两个字符串是否相等，不相等返回 true。|	[ $a != $b ] 返回 true。|
|-z|	检测字符串长度是否为0，为0返回 true。|	[ -z $a ] 返回 false。|
|-n|	检测字符串长度是否为0，不为0返回 true。|	[ -n $a ] 返回 true。|
|str|	检测字符串是否为空，不为空返回 true。	|[ $a ] 返回 true。|

### 文件测试运算符
|运算符| 说明 | 示例 |
|-|-|-|
|-b file|	检测文件是否是块设备文件，如果是，则返回 true。|	[ -b $file ] 返回 false。|
|-c file|	检测文件是否是字符设备文件，如果是，则返回 true。|	[ -c $file ] 返回 false。|
|-d file|	检测文件是否是目录，如果是，则返回 true。	|[ -d $file ] 返回 false。|
|-f file|	检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。|	[ -f $file ] 返回 true。|
|-g file|	检测文件是否设置了 SGID 位，如果是，则返回 true。|	[ -g $file ] 返回 false。|
|-k file|	检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。|	[ -k $file ] 返回 false。|
|-p file|	检测文件是否是有名管道，如果是，则返回 true。|	[ -p $file ] 返回 false。|
|-u file|	检测文件是否设置了 SUID 位，如果是，则返回 true。|	[ -u $file ] 返回 false。|
|-r file|	检测文件是否可读，如果是，则返回 true。|	[ -r $file ] 返回 true。|
|-w file|	检测文件是否可写，如果是，则返回 true。|	[ -w $file ] 返回 true。|
|-x file|	检测文件是否可执行，如果是，则返回 true。|	[ -x $file ] 返回 true。|
|-s file|	检测文件是否为空（文件大小是否大于0），不为空返回 true。|	[ -s $file ] 返回 true。|
|-e file|	检测文件（包括目录）是否存在，如果是，则返回 true。|	[ -e $file ] 返回 true。|
## linux if语法
```bash
if []
then
    echo 'aaa'
fi 
```

## linux if..else语法
```bash
if [ $1 == $2 ]
then
    echo '两个参数相等'
else 
    echo '两个参数不等'
fi
```

## linux if..elseif 语法
```bash
if [ $1 -gt $2 ]
then
    echo '参数1大于参数2'
elif [ $1 -eq $2 ]
then 
    echo '参数相等'
else 
    echo '参数1小于参数2'
fi
```
## linux 批量处理文件夹
```bash
for file in `ls $path`;
do
    echo $file;
done
```


---
title: linux常用脚本
description:  linux常用脚本
date: 2018-02-02 20:40:00
comments: true
tags: 
    - Linux
    - Shell  
categories:
    - Linux
---

# linxu 自用脚本

## linux 批量处理文件夹
```bash
for file in `ls $path`;
do
    echo $file;
done
```
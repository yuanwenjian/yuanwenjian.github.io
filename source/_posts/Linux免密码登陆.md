---
title: Linux免密码登陆
description: 经常登陆服务器，如果每次输入都需要输入密码会很浪费时间，而且有时会在脚本中访问服务器，此时无法手动输入密码
date: 2017-11-10 20:40:00
comments: true
tags: 
    - Linux
categories:
    - Linux
    - 黑科技
---
# 第一步:在本地机器上使用ssh-keygen产生公钥私钥对
```bash
ssh-keygen  //一路回车
```
执行完会在~/.ssh/目录下生成两个文件 id_rsa.pub和id_rsa

## 第二步： 用ssh-copy-id将公钥复制到远程机器中
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub remote-host
```
输入密码，配置完成


# 清除服务端秘钥
```bash
ssh-keygen -R 149.28.120.236 ## 远程ip
```
189 1145 3870 sy
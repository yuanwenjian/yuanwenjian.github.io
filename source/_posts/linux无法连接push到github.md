---
title: linux 无法push 代码到github解决方案
description:  浏览器可正常访问github,提交代码时timeout，使用ssh -Tv git@github.com 卡在expecting SSH2_MSG_KEX_DH_GEX_GROUP 
date: 2018-01-23 21:43:00
tags: 
    - ssh 
    - gitub 
categories:
    - 问题
---

# 问题描述
git突然无法push代码了，通过google推测是ssh问题，具体原因未知，通过ssh -TV git@github.com 卡在expecting SSH2_MSG_KEX_DH_GEX_GROUP位置不动了， 经过好几个小时google找到解决方案，希望可以为有同样问题的老哥提供帮助

# 解决问题
## 方案一
找到/etc/ssh/ssh_config 保证下面几行代码位注释，没有的可以手动添加(tip:最好先备份之前配置，该方案对我无用)
```
Host *
SendEnv LANG LC_*
HashKnownHosts yes
GSSAPIAuthentication yes
GSSAPIDelegateCredentials no
Ciphers aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128,aes128-cbc,3des-cbc
HostKeyAlgorithms ssh-rsa,ssh-dss
MACs hmac-md5,hmac-sha1,hmac-ripemd160
```

## 方案二

运行以下命令
```bash
echo 2 > /proc/sys/net/ipv4/tcp_mtu_probing
```

tip:可以先运行 cat /proc/sys/net/ipv4/tcp_mtu_probing 查看结果
该方案解决了我的问题

## 参考链接

[无法访问特定链接](https://fitzcarraldoblog.wordpress.com/tag/mtu/)

[ssh无法登陆](https://superuser.com/questions/568891/ssh-works-in-putty-but-not-terminal)




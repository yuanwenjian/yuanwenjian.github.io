---
title: 使用linux 命令行发送邮件
description:  使用linux 命令行发送邮件
date: 2018-02-02 20:40:00
comments: true
tags: 
    - Linux
    - Shell  
categories:
    - Linux
---

# 安装发送邮件
```bash
yum install mailx
```

# 配置发送
在 /etc/mail.rc  添加
```bash
set from=xxx@qq.cn

set smtp=smtp.exmail.qq.com

set smtp-auth-user=xxx@qq.cn

set smtp-auth-password=xxxx

set smtp-auth=login

```

# 测试
```
echo 'test' | mail -s 'test' xxxx@qq.cn
```

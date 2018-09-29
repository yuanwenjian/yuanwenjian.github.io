---
title: CentOS crontab安装与配置
description: crontab命令被用来提交和管理用户的需要周期性执行的任务，与windows下的计划任务类似，当安装完成操作系统后，默认会安装此服务工具，并且会自动启动crond进程，crond进程每分钟会定期检查是否有要执行的任务，如果有要执行的任务，则自动执行该任务。
date: 2018-09-10 20:40:00
comments: true
tags: 
    - Linux
    - Shell  
categories:
    - Linux
---

## 描述

crontab命令被用来提交和管理用户的需要周期性执行的任务，与windows下的计划任务类似，当安装完成操作系统后，默认会安装此服务工具，并且会自动启动crond进程，crond进程每分钟会定期检查是否有要执行的任务，如果有要执行的任务，则自动执行该任务。

## CentOS crontab安装与配置

### 安装
```bash
yum -y install vixie-cron
yum -y install crontabs
```

### 配置
```bash
service crond start //启动服务
service crond stop //关闭服务
service crond restart //重启服务
service crond reload //重新载入配置
service crond status //查看crontab服务状态
```

### 添加开机自启动
```bash
chkconfig –level 345 crond on
```

#### 查看开机自启动服务
```bash
chkconfig --list 

crond          	0:off	1:off	2:on	3:on	4:on	5:on	6:off 
```
- 等级0表示：表示关机
- 等级1表示：单用户模式
- 等级2表示：无网络连接的多用户命令行模式
- 等级3表示：有网络连接的多用户命令行模式
- 等级4表示：不可用
- 等级5表示：带图形界面的多用户模式
- 等级6表示：重新启动

### crontab 添加定时任务
```
crontab -e

minute   hour   day   month   week command //顺序为 分 时 日 月 周
```

### 可选值
1. minute： 表示分钟，可以是从0到59之间的任何整数。
2. hour：表示小时，可以是从0到23之间的任何整数。
3. day：表示日期，可以是从1到31之间的任何整数。
4. month：表示月份，可以是从1到12之间的任何整数。
5. week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。
6. command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。

其他可选值
1. 星号（*）：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
2. 逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”
3. 中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”
4. 正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。

### 示例
```bash

* * * * * command  //每1分钟执行一次command

3,15 * * * * command //每小时的第3和第15分钟执行

3,15 8-11 * * * command //在上午8点到11点的第3和第15分钟执行

3,15 8-11 */2 * * command //每隔两天的上午8点到11点的第3和第15分钟执行

3,15 8-11 * * 1 command //每个星期一的上午8点到11点的第3和第15分钟执行

30 21 * * * /etc/init.d/smb restart //每晚的21:30重启smb 

45 4 1,10,22 * * /etc/init.d/smb restart //每月1、10、22日的4 : 45重启smb 

10 1 * * 6,0 /etc/init.d/smb restart //每周六、周日的1:10重启smb

0,30 18-23 * * * /etc/init.d/smb restart //每天18 : 00至23 : 00之间每隔30分钟重启smb  

0 23 * * 6 /etc/init.d/smb restart //每星期六的晚上11:00 pm重启smb 

* */1 * * * /etc/init.d/smb restart //每一小时重启smb

* 23-7/1 * * * /etc/init.d/smb restart //晚上11点到早上7点之间，每隔一小时重启smb 

0 11 4 * mon-wed /etc/init.d/smb restart //每月的4号与每周一到周三的11点重启smb

0 4 1 jan * /etc/init.d/smb restart //一月一号的4点重启smb

01 * * * * root run-parts /etc/cron.hourly //每小时执行/etc/cron.hourly目录内的脚本
```

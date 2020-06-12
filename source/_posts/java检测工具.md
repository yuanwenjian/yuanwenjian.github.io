---
title: JAVA JDK内置检测工具
description: 
date: 2018-04-24 20:40:00
comments: true
tags: 
    - Java
categories:
    - Java
---

# JAVA查看JVM 

## JDK内置命令
1. jps -mlv  显示PID,完整类名,运行参数
效果如图：
![jps_mlv][jps]

2. jps -p //只显示PID

3. jmap -heap PID //查看堆信息

4. jcmd PID VM.uptime //查看JVM运行时间

5. jcmd PID VM.version  //查看jvm版本

6. jcmd process_id VM.system_properties //查看JVM系统属性

7. jstat -gc <pid> <统计间隔时间 单位毫秒>  <统计次数> //jstat 是 JVM 自带命令行工具，可用于统计内存分配速率、GC 次数，GC 耗时。

显示结果中文含义
```
S0C：年轻代中第一个survivor（幸存区）的容量 (字节)
S1C：年轻代中第二个survivor（幸存区）的容量 (字节)
S0U：年轻代中第一个survivor（幸存区）目前已使用空间 (字节)
S1U：年轻代中第二个survivor（幸存区）目前已使用空间 (字节)
EC：年轻代中Eden（伊甸园）的容量 (字节)
EU：年轻代中Eden（伊甸园）目前已使用空间 (字节)
OC：Old代的容量 (字节)
OU：Old代目前已使用空间 (字节)
PC：Perm(持久代)的容量 (字节)
PU：Perm(持久代)目前已使用空间 (字节)
YGC：从应用程序启动到采样时年轻代中gc次数
YGCT：从应用程序启动到采样时年轻代中gc所用时间(s)
FGC：从应用程序启动到采样时old代(全gc)gc次数
FGCT：从应用程序启动到采样时old代(全gc)gc所用时间(s)
GCT：从应用程序启动到采样时gc用的总时间(s)
NGCMN：年轻代(young)中初始化(最小)的大小 (字节)
NGCMX：年轻代(young)的最大容量 (字节)
NGC：年轻代(young)中当前的容量 (字节)
OGCMN：old代中初始化(最小)的大小 (字节)
OGCMX：old代的最大容量 (字节)
OGC：old代当前新生成的容量 (字节)
PGCMN：perm代中初始化(最小)的大小 (字节)
PGCMX：perm代的最大容量 (字节)
PGC：perm代当前新生成的容量 (字节)
S0：年轻代中第一个survivor（幸存区）已使用的占当前容量百分比
S1：年轻代中第二个survivor（幸存区）已使用的占当前容量百分比
E：年轻代中Eden（伊甸园）已使用的占当前容量百分比
O：old代已使用的占当前容量百分比
P：perm代已使用的占当前容量百分比
S0CMX：年轻代中第一个survivor（幸存区）的最大容量 (字节)
S1CMX ：年轻代中第二个survivor（幸存区）的最大容量 (字节)
ECMX：年轻代中Eden（伊甸园）的最大容量 (字节)
DSS：当前需要survivor（幸存区）的容量 (字节)（Eden区已满）
TT： 持有次数限制
MTT ： 最大持有次数限制
```
效果如图
![jstat][jstat]


## 参考

[Java 内置监控工具][内置工具]
[importNew][import]

[内置工具]: https://www.ibm.com/developerworks/cn/java/j-lo-performance-analysissy-tools2/index.html
[jps]: /images/jdk/jps_mlv.jpg
[import]:http://www.importnew.com/17308.html
[jstat]: ../images/jdk/jstat.png
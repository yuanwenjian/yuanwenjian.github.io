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
## 参考

[Java 内置监控工具][内置工具]
[importNew][import]

[内置工具]: https://www.ibm.com/developerworks/cn/java/j-lo-performance-analysissy-tools2/index.html
[jps]: /images/jdk/jps_mlv.jpg
[import]:http://www.importnew.com/17308.html
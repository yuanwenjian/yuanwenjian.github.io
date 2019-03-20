---
title: Arthas常用排查命令
description:  Arthas 是Alibaba开源的Java诊断工具，深受开发者喜爱
date: 2019-03-08 20:40:00
comments: true
tags: 
    - Java
categories:
    - Java
---
# Arthas简介

Arthas 是Alibaba开源的Java诊断工具，深受开发者喜爱。

当你遇到以下类似问题而束手无策时，Arthas可以帮助你解决：

1. 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
2. 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
3. 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
4. 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
5. 是否有一个全局视角来查看系统的运行状况？
6. 有什么办法可以监控到JVM的实时运行状态？

# artha使用教程
## 安装并启动
```bash
wget https://alibaba.github.io/arthas/arthas-boot.jar
java -jar arthas-boot.jar
java -jar arthas-boot.jar --repo-mirror aliyun --use-http  # 如果下载速度慢，可以使用阿里镜像
```
## watch命令
1. demo 
```bash 
$ watch demo.MathGame primeFactors "{params,throwExp}" -x 2  # 查看报错行
```
2. 参数详解

- watch 命令定义了4个观察事件点，即 -b 方法调用前，-e 方法异常后，-s 方法返回后，-f 方法结束后
- 4个观察事件点 -b、-e、-s 默认关闭，-f 默认打开，当指定观察点被打开后，在相应事件点会对观察表达式进行求值并输出
- 这里要注意方法入参和方法出参的区别，有可能在中间被修改导致前后不一致，除了 -b 事件点 params 代表方法入参外，其余事件都代表方法出参
- 当使用 -b 时，由于观察事件点是在方法调用前，此时返回值或异常均不存在

3. 表达式
[表达式使用参考][表达式参考]


## thread 查看线程
```bash
thread -n 3 # 查看当前最忙的N个线程，并打印堆栈信息
```

# 官方文档
[arthas官方文档][arthas官方文档]
[arthas github 地址][github地址]


[表达式参考]:https://alibaba.github.io/arthas/advice-class.html 
[arthas官方文档]:https://alibaba.github.io/arthas/
[github地址]:https://github.com/alibaba/arthas
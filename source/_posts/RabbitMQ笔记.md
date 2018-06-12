---
title: RabbitMQ笔记
description: RabbitMQ是一个消息代理 - 一个消息系统的媒介。它可以为你的应用提供一个通用的消息发送和接收平台，并且保证消息在传输过程中的安全。
date: 2018-05-08 12:30:00
comments: true
tags: 
    - RabbitMQ
    - 后端  
categories:
    - 后端
---

# RabbitMq简介

## 简介
消息系统允许软件、应用相互连接和扩展．这些应用可以相互链接起来组成一个更大的应用，或者将用户设备和数据进行连接．消息系统通过将消息的发送和接收分离来实现应用程序的异步和解偶．

或许你正在考虑进行数据投递，非阻塞操作或推送通知。或许你想要实现发布／订阅，异步处理，或者工作队列。所有这些都可以通过消息系统实现。

RabbitMQ是一个消息代理 - 一个消息系统的媒介。它可以为你的应用提供一个通用的消息发送和接收平台，并且保证消息在传输过程中的安全。

## 技术亮点
1. 可靠性
RabbitMQ提供了多种技术可以让你在性能和可靠性之间进行权衡。这些技术包括持久性机制、投递确认、发布者证实和高可用性机制。

2. 灵活的路由
消息在到达队列前是通过交换机进行路由的。RabbitMQ为典型的路由逻辑提供了多种内置交换机类型。如果你有更复杂的路由需求，可以将这些交换机组合起来使用，你甚至可以实现自己的交换机类型，并且当做RabbitMQ的插件来使用。

3. 集群
在相同局域网中的多个RabbitMQ服务器可以聚合在一起，作为一个独立的逻辑代理来使用。

4. 联合
对于服务器来说，它比集群需要更多的松散和非可靠链接。为此RabbitMQ提供了联合模型。

5. 高可用的队列
在同一个集群里，队列可以被镜像到多个机器中，以确保当其中某些硬件出现故障后，你的消息仍然安全。

6. 多协议
RabbitMQ 支持多种消息协议的消息传递。

7. 广泛的客户端
只要是你能想到的编程语言几乎都有与其相适配的RabbitMQ客户端。

8. 可视化管理工具
RabbitMQ附带了一个易于使用的可视化管理工具，它可以帮助你监控消息代理的每一个环节。

9. 追踪
如果你的消息系统有异常行为，RabbitMQ还提供了追踪的支持，让你能够发现问题所在。

10. 插件系统
RabbitMQ附带了各种各样的插件来对自己进行扩展。你甚至也可以写自己的插件来使用。

# RabbitMQ架构图

![RabbitMQ][rabbitMq]

# RabbitMQ概念介绍
1. Producing , 生产者, 产生消息的角色. 对应上图P
2. Exchange , 交换机,用于转发消息，但是它不会做存储_ ，如果没有 Queue bind 到 Exchange 的话，它会直接丢弃掉 Producer 发送过来的消息。
    - **路由键**: 消息到交换机的时候，交互机会转发到对应的队列中，那么究竟转发到哪个队列，就要根据该路由键。

3. Queue , 队列, 消息暂时呆的地方. 对应上图红色队列
4. Consuming , 消费者, 把消息从队列中取出的角色. 对应上图C
5. Message , RabbitMQ 中的消息有自己的一系列属性, 某些属性对信息流有直接影响.
6.  绑定：也就是交换机需要和队列相绑定，这其中如上图所示，是多对多的关系。

## 交换机详解
rabbitmq的message model实际上消息不直接发送到queue中，中间有一个exchange是做消息分发，producer甚至不知道消息发送到那个队列中去。因此，当exchange收到message时，必须准确知道该如何分发。是append到一定规则的queue，还是append到多个queue中，还是被丢弃？这些规则都是通过exchagne的4种type去定义的。

### Directed Exchange
路由键exchange，该交换机收到消息后会把消息发送到指定routing-key的queue中。那消息交换机是怎么知道的呢？其实，producer deliver消息的时候会把routing-key add到 message header中。routing-key只是一个messgae的attribute。

### Topic Exchange
通配符交换机，exchange会把消息发送到一个或者多个满足通配符规则的routing-key的queue。其中_表号匹配一个word，#匹配多个word和路径，路径之间通过.隔开。如满足a._.c的routing-key有a.hello.c；满足#.hello的routing-key有a.b.c.helo。

### Fanout Exchange
扇形交换机，该交换机会把消息发送到所有binding到该交换机上的queue。这种是publisher/subcribe模式。用来做广播最好。
所有该exchagne上指定的routing-key都会被ignore掉。

### Header Exchange
设置header attribute参数类型的交换机

# SpringBoot与RabbitMQ整合Demo

[SpringBoot整合MQ Demo][github]
# 参考
[RabbitMQ中文文档][rabbit官网]
[Spring 集成 RabbitMQ 与其概念，消息持久化，ACK机制等][Spring整合RabbitMQ]

[rabbitMq]:/images/rabbitMQ.jpg

[rabbit官网]: http://rabbitmq.mr-ping.com/description.html
[Spring整合RabbitMQ]:https://github.com/401Studio/WeekLearn/issues/2
[github]: https://github.com/yuanwenjian/spring-mq
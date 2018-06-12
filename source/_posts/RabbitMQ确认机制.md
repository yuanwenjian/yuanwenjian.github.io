---
title: RabbitMQ消息确认
description: RabbitMQ是一个消息代理 - 一个消息系统的媒介。它可以为你的应用提供一个通用的消息发送和接收平台，并且保证消息在传输过程中的安全。
date: 2018-05-08 12:30:00
comments: true
tags: 
    - RabbitMQ
    - 后端  
categories:
    - 后端
---
# 介绍
从消费者到RabbitMQ的递送处理确认被称为消费确认;borker对生产者的确认称为生产者确认。这两个功能都基于相同的想法，并受TCP的启发。它们对于从发行商到RabbitMQ节点以及从RabbitMQ节点到消费者的可靠传送都是至关重要的



ReturnCallback 确认到队列
# 消费端确认

## Delivery Tags
在我们继续讨论其他主题之前，重要的是解释如何识别交付（并且确认指示它们各自的交付），当消费者或者订阅者被注册时，消息将由RabbitMQ的basic.deliver方法传递（推送）,该方法会携带一个channel唯一标识的deliberyTag，

## 消费端确认
一条消息发布会在头部生成deliberyuTag,减少网络流量可以使用批量确认，通过设置mutiple=true完成

当多字段设置为true时，RabbitMQ将确认这个序列号之前的所有消息都已经得到了处理。，直至包括确认中指定的标签。与其他所有与确认相关的内容一样，这是每个channel的范围,比如ch信道中有5,6,7,8tag,当确认8时，mutiple设为true,该信道里所有tag状态全部变为acknowledged，如果设为false,5，6,7状态未unacknowledged
channel.basicAck(long deliveryTag, boolean multiple);

# 生产端确认
ConfirmCallback 确认到交换器



# 参考
[RabbitMQ确认机制][确认机制]

[确认机制]:https://www.rabbitmq.com/confirms.html
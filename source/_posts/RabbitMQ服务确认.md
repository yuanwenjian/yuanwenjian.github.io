---
title: RabbitMQ确认机制
description: RabbitMQ提供了transaction、confirm两种消息确认机制。transaction即事务机制，手动提交和回滚；confirm机制提供了Confirmlistener和waitForConfirms两种方式。confirm机制效率明显会高于transaction机制，但后者的优势在于强一致性。建议使用conrim机制。 。
date: 2018-06-12 12:30:00
comments: true
tags: 
    - RabbitMQ
    - 后端  
categories:
    - 后端
---
# RabbitMQ消息发送流程
RabbMQ里面有几个重要概念 producer(生产者),binding(绑定),exchange(交换机),queue(队列),routing key(路由键),Consumer(消费者) 下面mq发送流程及各个组件是如何关联的

1. producer发送消息
2. 发送消息至queue,mq发送消息至exchange,发送消息由exchange完成,mq根据routing key 将queue绑定到exchange ,这个关系称为为binding.消息发送会携带routing key(包括空的key).然后mq根据bingings 匹配routing key ,如果匹配成功发送至queue中
3. 消费者从queue获取消息消费

# 消息保证准确性
从上面可知消息发送过程中存在丢失的隐患,比如消息发送exchange失败,exchange发送queue失败,消费者消费时失败.所以需要保证消息不丢失
消费者确认在MQ叫做 acknowlegements 机制；生产确认则叫做 publisher confirms。
# 生产者确认


# 消费者确认
---
title: RabbitMQ确认机制
description: RabbitMQ提供了transaction、confirm两种消息确认机制。transaction即事务机制，手动提交和回滚；confirm机制提供了Confirmlistener和waitForConfirms两种方式。confirm机制效率明显会高于transaction机制，但后者的优势在于强一致性。建议使用conrim机制。 。
date: 2018-06-12 12:30:00
comments: true
tags: 
    - RabbitMQ
    - SpringBoot
    - 后端  
categories:
    - 后端
    - SpringBoot
    - RabbitMQ
---
# RabbitMQ消息发送流程
RabbMQ里面有几个重要概念 producer(生产者),binding(绑定),exchange(交换机),queue(队列),routing key(路由键),Consumer(消费者) 下面mq发送流程及各个组件是如何关联的

1. producer发送消息
2. 发送消息至queue,mq发送消息至exchange,发送消息由exchange完成,mq根据routing key 将queue绑定到exchange ,这个关系称为为binding.消息发送会携带routing key(包括空的key).然后mq根据bingings 匹配routing key ,如果匹配成功发送至queue中
3. 消费者从queue获取消息消费

# 消息保证准确性
从上面可知消息发送过程中存在丢失的隐患,比如消息发送exchange失败,exchange发送queue失败,消费者消费时失败.所以需要保证消息不丢失
消费者确认在MQ叫做 acknowlegements 机制；生产确认则叫做 publisher confirms。
只包括confrim 机制

# 生产者确认
生产者将channel设置成confirm模式，一旦channel进入confirm模式，所有在该channel上面发布的消息都会被指派一个唯一的ID(Correlation Id从1开始)。一旦消息被投递到所匹配的队列之后，broker就会发送一个确认给生产者（包含消息的唯一ID）。这就使得生产者知道消息已经正确到达目的队列了，如果消息和队列是可持久化的，那么确认消息会将消息写入磁盘之后发出。broker回传给生产者的确认消息中deliver-tag域包含了确认消息的序列号，此外broker也可以设置basic.ack的multiple域，表示到这个序列号之前的所有消息都已经得到了处理。


confirm模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等channel返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，如果RabbitMQ因为自身内部错误导致消息丢失，就会发送一条nack消息，生产者应用程序同样可以在回调方法中处理该nack消息。

在channel 被设置成 confirm 模式之后，所有被 publish 的后续消息都将被 confirm（即 ack） 或者被nack一次。但是没有对消息被 confirm 的快慢做任何保证，并且同一条消息不会既被 confirm又被nack （注：已经在transaction事务模式的channel是不能再设置成confirm模式的，即这两种模式是不能共存的）

confrim 分为两部分，一种情况为生产者到exchange,另一种为exchange到队列，以下为SpringBoot 代码
```


```

# 消费者确认

为了确保消息或者任务不会丢失，RabbitMQ支持消息确认–ACK。ACK机制是消费端从RabbitMQ收到消息并处理完成后，反馈给RabbitMQ，RabbitMQ收到反馈后才将此消息从队列中删除。如果一个消费者在处理消息时挂掉（网络不稳定、服务器异常、网站故障等原因导致频道、连接关闭或者TCP连接丢失等），那么他就不会有ACK反馈，RabbitMQ会认为这个消息没有正常消费，会将此消息重新放入队列中。如果有其他消费者同时在线，RabbitMQ会立即将这个消息推送给这个在线的消费者。这种机制保证了在消费者服务器故障的时候，能不丢失任何消息和任务。


忘记通过basicAck返回确认信息是常见的错误。这个错误非常严重，将导致消费者客户端退出或者关闭后，消息会被退回RabbitMQ服务器，这会使RabbitMQ服务器内存爆满，而且RabbitMQ也不会主动删除这些被退回的消息。如果要监控这种错误，可以使用rabbitmqctl messages_unacknowledged命令打印出出相关的信息。

## 消费者忘记确认
当autoAck参数置为false，对于RabbitMQ服务端而言，队列中的消息分成了两个部分：一部分是等待投递给消费者的消息；一部分是已经投递给消费者，但是还没有收到消费者ack的消息。如果RabbitMQ一直没有收到消费者的确认信号，并且消费此消息的消费者已经断开连接，则RabbitMQ会安排该消息重新进入队列，等待投递给下一个消费者，当然也有可能还是原来的那个消费者。


RabbitMQ不会为未确认的消息设置过期时间，它判断此消息是否需要重新投递给消费者的唯一依据是消费该消息的消费者连接是否已经断开，这么设计的原因是RabbitMQ允许消费者消费一条消息的时间可以很久很久。


如果消息消费失败，也可以调用Basic.Reject或者Basic.Nack来拒绝当前消息而不是确认，如果只是简单的拒绝那么消息会丢失，需要将相应的requeue参数设置为true，那么RabbitMQ会重新将这条消息存入队列，以便可以发送给下一个订阅的消费者。如果requeue参数设置为false的话，RabbitMQ立即会把消息从队列中移除，而不会把它发送给新的消费者。
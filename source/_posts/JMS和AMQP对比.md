---
title: JMS和AMQP对比
date: 2016-11-24 18:28:48
tags: [JMS,AMQP]
categories: [JAVA]
---

### JMS和AMQP对比

@(技术学习)[JMS, AMQP]


[TOC]

#### JMS
通常而言提到JMS（Java MessageService）实际上是指JMS API。JMS是由Sun公司早期提出的消息标准，旨在为java应用提供统一的消息操作，包括create、send、receive等。JMS已经成为Java Enterprise Edition的一部分。从使用角度看，JMS和JDBC担任差不多的角色，用户都是根据相应的接口可以和实现了JMS的服务进行通信，进行相关的操作。

JMS通常包含如下一些角色：                  


![Alt text](./1477113998576.png)


JMS提供了两种消息模型，peer-2-peer(点对点)以及publish-subscribe（发布订阅）模型。当采用点对点模型时，消息将发送到一个队列，该队列的消息只能被一个消费者消费。而采用发布订阅模型时，消息可以被多个消费者消费。在发布订阅模型中，生产者和消费者完全独立，不需要感知对方的存在。

消息如何从producer端达到consumer端由message-routing来决定。在JMS中，消息路由非常简单，由producer和consumer链接到同一个queue（p2p）或者topic（pub/sub）来实现消息的路由。JMSconsumer同时支持message selector（消息选择器），通过消息选择器，consumer可以只消费那些通过了selector筛选的消息。在JMS兄中，消息路由机制的图示如下：
![Alt text](./1477114193676.png)

常见的消息队列，大部分都实现了JMS API，可以担任JMS provider的角色，如ActiveMQ，Redis以及RabbitMQ等。

#### AMQP
AMQP（advanced message queuing protocol）是一种协议，更准确的说是一种binary wire-level protocol（链接协议）。这是其和JMS的本质差别，AMQP不从API层进行限定，而是直接定义网络交换的数据格式。这使得实现了AMQP的provider天然性就是跨平台的。意味着我们可以使用Java的AMQP provider，同时使用一个python的producer加一个rubby的consumer。从这一点看，AQMP可以用http来进行类比，不关心实现的语言，只要大家都按照相应的数据格式去发送报文请求，不同语言的client均可以和不同语言的server链接。

在AMQP中，消息路由（messagerouting）和JMS存在一些差别，在AMQP中增加了Exchange和binding的角色。producer将消息发送给Exchange，binding决定Exchange的消息应该发送到那个queue，而consumer直接从queue中消费消息。queue和exchange的bind有consumer来决定。AMQP的routing scheme图示过程如下：
![Alt text](./1477114174634.png)


 目前AMQP逐渐成为消息队列的一个标准协议，当前比较流行的rabbitmq、stormmq都使用了AMQP实现。

#### JMS和AMQP对比


 ![Alt text](./1477114397156.png)

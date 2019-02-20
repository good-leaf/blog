---
title: rabbitmq知识
date: 2019-02-13 17:23:39
categories: 组件
tags: rabbitmq
---

tags: rabbitmq
---

### RabbitMQ简介

RabbitMQ是实现AMQP（高级消息队列协议）的消息中间件的一种，最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。RabbitMQ主要是为了实现系统之间的双向解耦而实现的。当生产者大量产生数据时，消费者无法快速消费，那么需要一个中间层。保存这个数据。

AMQP，即Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。

RabbitMQ是一个开源的AMQP实现，服务器端用Erlang语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持AJAX。用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。

<!--more-->

### 交换机(Exchange)

RabbitMQ常用的Exchange Type有fanout、direct、topic、headers这四种（AMQP规范里还提到两种Exchange Type，分别为system与自定义，这里不予以描述）

- fanout

转发消息到所有绑定队列。

- direct

把消息路由到那些binding key与routing key完全匹配的Queue中。

- headers

headers类型的Exchange不依赖于routing key与binding key的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配。 在绑定Queue与Exchange时指定一组键值对；当消息发送到Exchange时，RabbitMQ会取到该消息的headers（也是一个键值对的形式），对比其中的键值对是否完全匹配Queue与Exchange绑定时指定的键值对；如果完全匹配则消息会路由到该Queue，否则不会路由到该Queue。

- topic

topic类型的Exchange在匹配规则上进行了扩展。

routing key为一个句点号“. ”分隔的字符串（我们将被句点号“. ”分隔开的每一段独立的字符串称为一个单词），如“stock.usd.nyse”、“nyse.vmw”、“quick.orange.rabbit”。

binding key与routing key一样也是句点号“. ”分隔的字符串。

binding key中可以存在两种特殊字符“*”与“#”，用于做模糊匹配，其中“*”用于匹配一个单词，“#”用于匹配多个单词（可以是零个）。

### RabbitMQ RPC

![](https://cdn.www.sojson.com/file/doc/6121174952)

[RabbitMQ](http://www.sojson.com/tag_rabbitmq.html "RabbitMQ") 中实现`RPC`的机制是：

- 客户端发送请求（消息）时，在消息的属性（`MessageProperties`，在`AMQP`协议中定义了14中`properties`，这些属性会随着消息一起发送）中设置两个值`replyTo`（一个`Queue`名称，用于告诉服务器处理完成后将通知我的消息发送到这个`Queue`中）和`correlationId`（此次请求的标识号，服务器处理完成后需要将此属性返还，客户端将根据这个id了解哪条请求被成功执行了或执行失败）

- 服务器端收到消息并处理

- 服务器端处理完消息后，将生成一条应答消息到`replyTo`指定的`Queue`，同时带上`correlationId`属性

- 客户端之前已订阅`replyTo`指定的`Queue`，从中收到服务器的应答消息后，根据其中的`correlationId`属性分析哪条请求被执行了，根据执行结果进行后续业务处理

### RabbitMQ 选型和对比

1.从社区活跃度

按照目前网络上的资料，`RabbitMQ`、`activeM`、`ZeroMQ`三者中，综合来看，`RabbitMQ`是首选。

2.持久化消息比较

`ZeroMq`不支持，`ActiveMq`和`RabbitMq`都支持。持久化消息主要是指我们机器在不可抗力因素等情况下挂掉了，消息不会丢失的机制。

3.综合技术实现

可靠性、灵活的路由、集群、事务、高可用的队列、消息排序、问题追踪、可视化管理工具、插件系统等等。

`RabbitMq`/`Kafka`最好，`ActiveMq`次之，`ZeroMq`最差。当然`ZeroMq`也可以做到，不过自己必须手动写代码实现，代码量不小。尤其是可靠性中的：持久性、投递确认、发布者证实和高可用性。

 4.高并发

毋庸置疑，`RabbitMQ`最高，原因是它的实现语言是天生具备高并发高可用的`erlang`语言。

 5.比较关注的比较，RabbitMQ和 Kafka

`RabbitMq`比`Kafka`成熟，在可用性上，稳定性上，可靠性上， [RabbitMq](http://www.sojson.com/tag_rabbitmq.html "RabbitMq") 胜于 [Kafka](http://www.sojson.com/tag_kafka.html "Kafka") （理论上）。

另外，`Kafka`的定位主要在日志等方面， 因为`Kafka`设计的初衷就是处理日志的，可以看做是一个日志（消息）系统一个重要组件，针对性很强，所以 如果业务方面还是建议选择`RabbitMq`。

### RabbitMQ Erlang Client

[Amqp Erlang Client](http://www.rabbitmq.com/erlang-client.html "Amqp Erlang Client")

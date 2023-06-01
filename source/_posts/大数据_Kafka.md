---
title: Kafka 分布式事件流平台
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(54).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(54).jpg
tags:
  - 大数据
  - Kafka
category:
  - - 编程
    - 大数据
 
date: 2020-01-27 20:58:59
---

Kafka是一个开源的`分布式事件流平台`，被数千家公司用于`高性能数据管道`、`流分析`、`数据集成`和`关键任务应用`。

是由Apache软件基金会开发的一个开源流处理平台，由`Scala`和`Java`编写。

是一种`高吞吐量`的分`布式发布订阅消息`系统，它可以处理消费者在网站中的所有动作流数据。 

这种动作（网页浏览，搜索和其他用户的行动）是在现代网络上的许多社会功能的一个关键因素。 

这些数据通常是由于吞吐量的要求而通过`处理日志`和`日志聚合`来解决。 

对于像Hadoop一样的日志数据和离线分析系统，但又要求`实时处理`的限制，这是一个可行的解决方案。

Kafka的目的是通过Hadoop的并行加载机制来`统一线上和离线的消息处`理，也是为了`通过集群来提供实时的消息`。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 企业级消息系统

### 1.1 为什么需要消息系统/消息队列

> “消息队列”(Message Queue , MQ)从字面来理解，是一个队列，拥有先进先出(First Input First Output , FIFO)的特性。

> 那么为什么需要消息系统/消息队列呢?
>>在高并发的应用场景中，由于来不及同步处理请求，接收到的请求往往会发生阻塞。
> 
>>例如，大量的插入、更新请求同时到达数据库，这会导致行或表被锁住，最后会因为请求堆积过多而触发“连接数过多的异常” ( Too Many Connections)错误。
> 
>>因此，在高并发的应用场景中需要一个缓冲机制，而消息队列则可以很好地充当这样一个角色

### 1.2 消息系统/消息队列的作用

> 在实际的应用中，消息队列主要有以下作用。
>> 1.`应用解耦`：多个应用可通过消息队列对消息进行处理，应用之间相互独立，互不影响；
> 
>> 2.`异步处理`：相比于串行和井行处理，异步处理可以减少处理的时间；
> 
>> 3.`流量削峰/数据限流`：流量高峰期，可通过消息队列来控制流量， 避免流量过大而引起应用系统崩溃；
> 
>> 4.`消息通信`：实现点对点消息队列或聊天室等。

#### 1.2.1 应用解耦

> 如果模块之间不存在直接调用，那么新增模块或者修改模块就对其他模块影响最小，这样系统的可扩展性无疑更好一些！

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/1.png)

> 由于消息与平台和语言无关，并且在语法上也不再是函数之间的调用，因此，消息队列允许应用接口独立地进行扩展，只用应用接口遵守同样的接口约束。

> 举例，用户使用客户端上传一张个人图片，具体流程如图1-2 所示。
>> (1)图片上传系统将图片信息（如唯一ID 、图片类型、图片尺寸等）批量写入消息队列，写入成功后会将结果直接返回给客户端。
> 
>> (2)人脸识别系统定时从消息队列中读取数据，完成对新增图片的识别。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/2.png)

> 图片上传系统无须关心人脸识别系统是否对上传的图片进行了处理，它只需要关心是否成功将图片信息写入消息队列。

> 由于用户无须立即知晓人脸识别的结果，因此人脸识别系统可选择不同的调度策略来处理消息队列中的图片信息。

#### 1.2.2 异步处理

> 在不使用消息队列的情况下,用户的请求数据直接写入数据库,在高并发的情况下,会对数据库造成巨大的压力,同时也使得响应延迟加剧.在使用消息队列后,用户请求的数据发送给消息队列后立即返回,再消息队列的消费者进程(通常该进程独立部署在专门的服务器上)从消息中获取数据,异步写入数据库.由于消息队列服务器处理速度远远快于数据库,因此用户的响应延迟得到有效改善.显著提高服务器性能

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/3.png)

> 用户在注册账号时，服务程序需要给用户发送邮件注册信息和短信注册信息。比较传统的做法是－一通过串行和并行的方式来实现。
>>(1)串行方式： 先将用户注册信息写入数据库，然后发送短信注册信息，再发送邮件注册信息。以上三个任务全部完成后，才会将结果返回给用户。具体流程如图1 - 3 所示。假设这三个阶段的耗时均为20 ms ， 不考虑网络等其他消耗，则整个过程需耗时60ms 。
> 
>>(2)并行方式：先将用户注册信息写入数据库， 然后在发送短信注册信息的同时还发送邮件注册信息。以上任务全部完成后才会将结果返回给用户。具体流程如图1 - 4 所示。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/4.png)

> 假设这三个阶段的耗时均为20ms ， 不考虑网络等其他消耗，则整个过程需耗时40 ms 。<br/>
> 与串行的不同之处是，并行处理提高了处理效率，减少了处理时间。<br/>
> 针对上述应用场景，采传统方式时，系统的性能（如并发量、吞吐量、响应时间等）会产生瓶颈。此时需要引入消息队列异步处理非必要业务环节。具体架构如图1-5 所示。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/5.png)

> 用户将注册信息写入数据库约耗时20ms (和串行和并行的处理时间相同） 。短信和邮件注册信息写入消息队列后会直接将结果返回给用户。由于写入消息队列的速度非常快，基本可以忽略。<br/>
> 另外， “通过异步读取消息队列中的短信注册信息”过程和“邮件注册信息”过程相当于同时进行的，那么整个过程约耗时20ms 。<br/>
> 从上面的分析可以看出，在调整架构后，系统的整体处理时间异步是串行方式的1/3 ，是并行方式的1/2 。

#### 1.2.3 流量削峰/数据限流

> 消息队列还具有很好的削峰作用,通过异步请求,将短时间高并发产生的事务消息存储在消息队列中,从而削平高峰期的并发事务.在电商网站促销,秒杀活动中,合理使用消息队列,可有效抵御促销活动刚开始大量涌入的订单对系统造成的冲击

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/6.png)

> 数据限流也是消息队列的常用场景之一， 一般在促销和“秒杀”活动中使用得较为广泛。<br/>
> 例如，在电商的“双11 ＂活动中， 由于瞬间的数据访问量过大，服务器接收到的数据请求过大，则导致服务器上的应用服务无法处理请求而崩溃。<br/>
> 为了解决这类问题， 一般需要先将用户请求写入消息队列(相当于用消息队列做一次缓冲），然后服务器上的应用服务再从消息队列中读取数据。具体流程如图1 - 6 所示。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/7.png)

> 数据限流具有以下优点：
>> 用户请求写数据到消息队列时，不与应用业务服务直接接触，中间存在一次缓冲。这极大地减少了应用服务处理用户请求的压力。
> 
>> 可以设置队列的长度，用户请求遵循FIFO 原则。后来的用户请求处于队列之外时，是无法秒杀到商品的，这些请求会直接被舍弃， 返给用户“商品己售完”的结果。

> FIFO ( First Input First Output ，先进先出）是一种较为传统的执行方法，按照请求的进入顺序依次进行处理。

#### 1.2.4 消息通信

> 消息队列具有高效的通信机制，所以其在点对点通信和聊天室通信中被广泛应用。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/8.png)

### 1.3 消息系统的分类

> 1.`点对点模式`（一对一，消费者主动拉取数据，消息收到后消息清除）
>>点对点模型通常是一个基于拉取或者轮询的消息传送模型，这种模型从队列中请求信息，而不是将消息推送到客户端。这个模型的特点是发送到队列的消息被一个且只有一个接收者接收处理，即使有多个消息监听者也是如此。

> 2.`发布/订阅模式`（一对多，数据生产后，推送给所有订阅者）
>> 发布订阅模型则是一个基于推送的消息传送模型。发布订阅模型可以有多种不同的订阅者，临时订阅者只在主动监听主题时才接收消息，而持久订阅者则监听主题的所有消息，即使当前订阅者不可用，处于离线状态。和点对点方式不同，发布到topic的消息会被所有订阅者消费。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/9.png)

### 1.4 常见的消息系统

> `RabbitMQ`
>> RabbitMQ 2007年发布，是一个在AMQP(高级消息队列协议)基础上完成的，可复用的企业消息系统，是当前最主流的消息中间件之一。

> `ActiveMQ`
>> ActiveMQ是由Apache出品，ActiveMQ 是一个完全支持JMS1.1和J2EE 1.4规范的 JMS Provider实现。它非常快速，支持多种语言的客户端和协议，而且可以非常容易的嵌入到企业的应用环境中，并有许多高级功能

> `RocketMQ`
>> RocketMQ出自 阿里公司的开源产品，用 Java 语言实现，在设计时参考了 Kafka，并做出了自己的一些改进，消息可靠性上比 Kafka 更好。RocketMQ在阿里集团被广泛应用在订单，交易，充值，流计算，消息推送，日志流式处理等

> `Kafka`
>> Apache Kafka是一个分布式消息发布订阅系统。它最初由LinkedIn公司基于独特的设计实现为一个分布式的提交日志系统( a distributed commit log)，之后成为Apache项目的一部分。Kafka系统快速、可扩展并且可持久化。它的分区特性，可复制和可容错都是其不错的特性。

> `其他MQ`
>> 1..NET消息中间件 DotNetMQ<br/>
>> 2.基于HBase的消息队列 HQueue<br/>
>> 3.Go 的 MQ 框架 KiteQ<br/>
>> 4.MemcacheQ 是一个基于 MemcacheDB 的消息队列服务器。<br/>
>> 5.ZeroMQ：号称最快的消息队列系统，尤其针对大吞吐量的需求场景，擅长的高级/复杂的队列，但是技术也复杂，并且只提供非持久性的队列。<br/>
>> 6.Redis：是一个key-Value的NOSql数据库，但也支持MQ功能，数据量较小，性能优于RabbitMQ，数据超过1w时就慢的无法忍受

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/10.png)

## 2 Kafka简介

### 2.1 前世今生

> Kafka 起源于Linkedln领英 公司。起初， Linkedln 需要收集各个业务系统和应用的指标数据来进行数据分析，原先是使用“自定义开发”系统来实现的。但这期间需要采集的数据量非常大，且内容很复杂。除要采集操作系统的基础指标（例如：内存、CPU 、磁盘、网络等）外，还要采集很多和业务相关的数据指标。

> 随着数据量的增长、业务需求的复杂度提高，这个“自定义开发”系统的问题也越来越多。例如，在处理一个HTTP 请求数据时，由于数据内容是以XML 数据格式进行传输的， 需要先对这部分数据做解析处理，然后才能拿来做离线分析。由于这样一个自定义开发系统不够稳定，且XML 数据格式的解析过程也非常复杂，所以系统经常出现问题。出现问题后，定位分析也比较麻烦，需要很长的处理时间，所以无法做到实时服务。

> 之后， Linkedln 想寻找一种可支持大数据实时服务并且支持水平扩展的解决方案。尝试过使用ActiveMQ ，但是它不支持水平扩展，并且ActiveMQ 内部有很多Bug。

> 于是， Linkedln 团队开发了一个既满足实时处理需求，又可支持水平拓展的消息系统---Kafka，它还拥有高吞吐量特性。

> 2010 年， Kafka 项目被托管到Github 开源社区。一时间，大量开发者被这个项目所吸引。

> 2011 年， Kafka 成为Apache 项目基金会的一个开源项目。

> 2012 年， Apache 项目基金会开始对Kafka 项目进行孵化。

> 之后，不断有Linkedln 员工和社区成员来维护和改善Kafka 项目， Kafka项目得到持续不断地改进。

> 如今， Kafka 项目成为Apache 项目基金会的顶级项目之一。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/11.png)

> 官网：
>> [`http://kafka.apache.org/`](http://kafka.apache.org/)

> 官方文档
>> [`http://kafka.apache.org/documentation/#design`](http://kafka.apache.org/documentation/#design)
>> [`http://kafka.apache.org/documentation/#implementation`](http://kafka.apache.org/documentation/#implementation)
>> [`http://kafka.apache.org/documentation/#operations`](http://kafka.apache.org/documentation/#operations)
>> [`http://kafka.apache.org/documentation/#security`](http://kafka.apache.org/documentation/#security)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/12.png)

### 2.2 设计初衷

> Kafka 雏形由LinkedIn 开发，设计之初被LinkedIn 用来处理活动流数据和运营数据。

> 活动流数据，是指浏览器访问记录、页面搜索记录、查看网页详细记录等站点内容。

> 运营数据，是指服务器的基本指标，例如CPU 、磁盘I/O 、网络、内存等。

> 在后续版本法代中， Kafka 被设计成一个统一的平台，可用来处理大公司所有的实时数据。需要它能够满足以下需求。
>> 1.`高吞吐量`
>>> 日常生活中所使用的支付宝、微信、QQ 这类软件的用户量非常庞大，每秒产生的数据流量也非常巨大。面对这类场景，若要实时地聚合消息日志，必须具有高吞吐量才能支持高容量事件流。
>>
>> 2.`高可用队列`
>>> 分布式消息队列系统都具有异步处理机制。另外，分布式消息队列系统一般都拥有处理大量数据积压能力， 以便支持其他离线系统的定期数据加载。
>>
>> 3.`低延时`
>>> 实时应用场景对时延的要求极为严格。耗时越少，则结果越理想。这意味着，设计出来的系统必须拥有低延迟处理能力。
>>
>> 4.`分布式机制`
>>> 系统还需具有支持分区、分布式、能实时处理消息等特点，井能在机器出现故障时保证数据不丢失。

> 为满足这些需求， Kafka 拥有了许多独特的特性，这使得它更类似于数据库日志，而不是传统的消息传递系统。

### 2.3 应用场景

> 在实际的使用场景中， Kafka 有着广泛的应用。例如，`日志收集`、`消息系统`、`活动追踪`、`运营指标`、`流式处理`、`事件源`等。

> 1.`日志收集`
>> 在实际工作中， 系统和应用程序都会产生大量的日志。为了方便管理这些日志，可以利用Kafka 将这些零散的日志收集到Kafka 集群中，然后通过Kafka 的统一接口将这些数据开放给不同的消费者(Consumer) 。统一接口包括： Hadoop 的应用接口、HBase 的应用接口、ElasticSearch的应用接口等。

> 2.`消息系统`
>> 线上业务流量很大的应用，可以使用Kafka 作为缓冲， 以减少服务端的压力。这样能够有效地解耦生产者(Producer)和消费者(Consumer)，以及缓冲消息数据。

> 3.`用户轨迹`
>> 可使用Kafka 记录浏览器用户或者手机App 用户产生的各种记录，例如浏览的网页、搜索的内容、点击的内容等。
>> 这些用户活动信息会被服务器收集到Kafka 集群中进行存储，然后消费者通过“消费”这些活动数据来做实时分析，或者加载到Hive 数据仓库做离线数据分析与挖掘。

> 4.`记录运营监控数据`
>> Kafka 也可用来记录运营监控数据，包括收集各种分布式应用系统的数据（如Hadoop 系统、Hive 系统、HBase 系统等）。

> 5.`实现流处理`
>> Kafka 是一个流处理平台，所以在实际应用场景中也会与其他大数据套件结合使用，例如Spark Streaming 、Storm 、Flink 等。

> 6.`事件源`
>> 事件源是一种应用程序的设计风格，其中状态更改会产生一条带有时间戳的记录，然后将这条以时间序列产生的记录进行保存。在面对非常大的存储数据时，可以使用这种方式来构建非常优秀的后端程序。

### 2.4 在大数据项目中所处的位置

> 在流式计算中，Kafka一般用来缓存数据，Storm、Spark Streaming、Flink通过消费Kafka的数据进行计算。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/13.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/14.png)

## 3 Kafka安装

### 3.1 准备工作 

> .Kafka的源代码是利用Scala 语言编写的，它需要运行在Java 虚拟机Java Virtual MachinN JVM）上。
>> 因此，在安装Kafka 之前需要先安装JDK。

> 2.Kafka是一个分布式消息中间件系统，它依赖ZooKeeper 管理和协调Kafka 集群的各个代理CBroker)节点。
>> 因此，在安装Kafka 集群之前需要先安装ZooKeeper 集群。

> 3.`准备如下目录`
>> 安装包存放的目录：/export/software
> 
>> 安装程序存放的目录：/export/servers
> 
>> 数据目录：/export/data
> 
>> 日志目录：/export/logs
> 
>> 如果没有需要创建:
> 
>> mkdir -p /export/servers/
> 
>> mkdir -p /export/software/
> 
>> mkdir -p /export/data/
> 
>> mkdir -p /export/logs/

> 4.`下载kafka安装压缩包`
>> http://archive.apache.org/dist/kafka/
> 
>> https://www.apache.org/dyn/closer.cgi?path=/kafka/1.0.0/kafka_2.11-1.0.0.tgz
> 
>> 由于kafka是scala语言编写的，基于scala的多个版本，kafka发布了多个版本。其中2.11是推荐版本

### 3.2 上传压缩包并解压

> 这里统一使用  kafka_2.11-1.0.0.tgz 这个版本

```
tar -zxvf kafka_2.11-1.0.0.tgz -C  /export/servers/
cd /export/servers/
mv kafka_2.11-1.0.0 kafka
```

### 3.3 配置环境变量

```
vim /etc/profile
#KAFKA_HOME
export KAFKA_HOME=/export/servers/kafka
export PATH=$PATH:$KAFKA_HOME/bin
source /etc/profile
```

### 3.4 分发安装包

```
scp -r /export/servers/kafka  node2:/export/servers
scp -r /export/servers/kafka  node3:/export/servers

scp /etc/profile node2:/etc/profile
scp /etc/profile node3:/etc/profile

source /etc/profile
```

### 3.5 修改Kafka配置文件

```
cp   /export/servers/kafka/config/server.properties  /export/servers/kafka/config/server.properties.bak
vim  /export/servers/kafka/config/server.properties
```

> 主要修改以下6个地方:
>> 1) `broker.id`            需要保证每一台kafka都有一个独立的broker
>> 2) `log.dirs`             数据存放的目录  
>> 3) `zookeeper.connect`    zookeeper的连接地址信息  
>> 4) `delete.topic.enable`  是否直接删除topic
>> 5) `host.name`            主机的名称
>> 6) 修改: `listeners=PLAINTEXT://node1:9092`

> 第一台机器修改kafka配置文件server.properties

```
vim  /export/servers/kafka/config/server.properties
```

删除所有

```
ggdG或者:%d
```

```
broker.id=0
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/export/data/kafka/kafka-logs
num.partitions=2
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.flush.interval.messages=10000
log.flush.interval.ms=1000
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=node1:2181,node2:2181,node3:2181
zookeeper.connection.timeout.ms=6000
group.initial.rebalance.delay.ms=0
delete.topic.enable=true
host.name=node1
```

> 第二台机器修改kafka配置文件server.properties

```
vim  /export/servers/kafka/config/server.properties
```

删除所有

```
ggdG或者:%d
```

```
broker.id=1
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/export/data/kafka/kafka-logs
num.partitions=2
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.flush.interval.messages=10000
log.flush.interval.ms=1000
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=node1:2181,node2:2181,node3:2181
zookeeper.connection.timeout.ms=6000
group.initial.rebalance.delay.ms=0
delete.topic.enable=true
host.name=node2
```

> 第三台机器修改kafka配置文件server.properties

```
vim  /export/servers/kafka/config/server.properties
```

删除所有

```
ggdG或者:%d
```

```
broker.id=2
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/export/data/kafka/kafka-logs
num.partitions=2
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.flush.interval.messages=10000
log.flush.interval.ms=1000
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=node1:2181,node2:2181,node3:2181
zookeeper.connection.timeout.ms=6000
group.initial.rebalance.delay.ms=0
delete.topic.enable=true
host.name=node3
```

> 配置详解

```
＃设置Kafka 节点唯一ID
broker.id=O
＃ 开启删除Kafka 主题属性
delete.topic.enable=true
＃设置网络请求处理线程数
num.network.threads=10
＃设置磁盘IO 请求线程数
num.io.threads=20
＃设置发送buffer字节数
socket.send.buffer.bytes=1024000
＃设置收到buffer字节数
socket.receive.buffer.bytes=l024000
＃设置最大请求字节数
socket.request.max.bytes=l048576000
＃设置消息记录存储路径
log.dirs=/export/data/kafka/kafka-logs
＃设置Kafka 的主题分区数
num.partitions=2
＃设置主题保留时间
log.retention.hours=l68
＃设置Zookeeper 的连接地址
zookeeper.connect=node1:2181,node2:2181,node3:2181
＃设置Zookeeper连接起时时间
zookeeper.connection.timeout.ms=60000
```

### 3.6 启动集群

> 先启动zk

> 再在三台机器上分别启动

>> 前台启动
```
/export/servers/kafka/bin/kafka-server-start.sh /export/servers/kafka/config/server.properties
```

>> 后台启动

```
nohup /export/servers/kafka/bin/kafka-server-start.sh /export/servers/kafka/config/server.properties >/dev/null 2>&1 &
```

> `nohup` 加在一个命令的最前面，表示不挂断的运行命令<br/>
> `&`加在一个命令的最后面，表示这个命令放在后台执行

> 验证kafka正常启动

```
jps
```

> 查看在zk上的注册信息
> 登录zookeeper:

```
/export/servers/zookeeper/bin/zkCli.sh
```

> 执行:

```
ls /brokers/ids
```

### 3.7 关闭集群

```
/export/servers/kafka/bin/kafka-server-stop.sh stop
```

> 或者使用

```
kafkacmd.sh stop
```

## 4 Kafka常用命令

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/33.png)

### 4.1 主题管理

#### 4.1.1 创建主题

##### 4.1.1.1 自动创建

> 可以通过`auto.create.topics.enable` 属性来自动创建主题。默认情况下，该属性值为`true`。

> 因此，生产者应用程序向Kafka 集群中一个不存在的主题写数据时，会自动创建一个默认分区和默认副本系数的主题。

> 默认分区的数值由`$KAFKA_HOME/config/server.properties`文件中的属性`num.partitions`控制，

> 默认副本系数值由`$KAFKA_HOME/config/server.properties`文件中的属性`default.replication.factor` 控制

##### 4.1.1.2 手动创建

> 可以通过`kafka-topics.sh` 脚本于动创建主题。

> `kafka-topics.sh` 脚本是在`kafka-run-class.sh` 脚本的基础上进行了封装。

> 在执行管理主题的脚本时，通过调用一个`kafka.admin.TopicCommand` 类来实现一系列的管理操作


> 创建一个名为“ order ”的主题，该主题拥有3 个副本和6 个分区

```
cd /export/servers/kafka/bin
kafka-topics.sh --zookeeper node1:2181 --create --topic order --replication-factor 3 --partitions 6
```

> 查看所有的主题

```
kafka-topics.sh  --zookeeper node1:2181 --list
```

> 进入Kafka 系统消息数据存储目录/export/data/kafka/kafka-logs 中查看

```
cd /export/data/kafka/kafka-logs
ll
```

> 也可以通过zkCli.sh 脚本连接到Zookeepr 去访问主题分区信息和元数据信息

```
/export/servers/zookeeper/bin/zkCli.sh
ls /brokers/topics/order/partitions
get /brokers/topics/order
quit
```

#### 4.1.2 查看主题

> `kafka-topics.sh` 脚本提供了两个查看主题信息的命令。
>> `list` ：用来展示所有的主题名。
>
>> `describe` ： 用来查看指定主题或全部主题的详细信息；
>>> 如果没有指定`topic` 参数， 则查看的是全部的主题信息。
>> 
>>> 如果指定了`topic` 参数，则查看的是某一特定的主题信息；

> 查看所有主题名

```
kafka-topics.sh --zookeeper  node1:2181 --list
```

> 查看全部主题信息。

```
kafka-topics.sh --zookeeper node1:2181 --describe
```

> 查看单个主题信息。

```
kafka-topics.sh --zookeeper node1:2181 --describe  --topic order
```

#### 4.1.3 修改主题

> 在Kafka 集群中创建一个主题后，后期维护该主题时可以通过alter 命令来进行修改，修改内容包含主题的配置信息。

> 创建一个新的主题， 1 个分区， 1 个副本

```
kafka-topics.sh  --zookeeper node1:2181 --create --topic user --replication-factor 1 --partitions 1  --config max.message.bytes=102400
```

> 查看覆盖的配置参数

```
kafka-topics.sh --zookeeper node1:2181 --describe --topic user --topics-with-overrides  
```

> 修改大小

```
kafka-topics.sh  --zookeeper node1:2181  --alter --topic user --config max.message.bytes=204800
```

> 再次查看

```
kafka-topics.sh --zookeeper node1:2181 --describe --topic user --topics-with-overrides
```

#### 4.1.4 删除主题

> 如想删除主题，则需要在启动Kafka 集群之前开启删除主题的开关

> 在$KAFKA_ HOME/config/server.properties 文件中添加属性delete.topic.enable ＝true 。该属性默认是false。

> 我们在之前已经配置过了

> 创建主题

 ```
kafka-topics.sh  --zookeeper node1:2181 --create --topic test_delete --replication-factor 1 --partitions 1  
```

> 删除主题

```
kafka-topics.sh  --zookeeper node1:2181  --delete --topic test_delete
```

> 查看主题

```
kafka-topics.sh  --zookeeper node1:2181  --list
```

### 4.2 分区与副本管理

> 在实际应用场景中， 由于前期考虑不周到或者是业务数据量增加，后期可能需要扩展主题的分区和副本， 这时可以通过Kafka 提供的脚本工具来完成。

> `注意:一般在实际开发中,我们会在topic创建的时候指定分区和副本数,一般不会后期轻易更改,除非后期数据量剧增,需要提高并发读写,那么可能会增加分区数`

#### 4.2.1 分区和副本的背景和作用

> 在Kafka系统中，为何要在主题中加入分区和副本的概念呢？“主题”是一个逻辑概念，而“分区”则是一个物理概念。

> `1.谁关注分区?`
>> 用户在调用生产者接口时，只需要关心将消息数据发送到哪个主题。而用户在调用消费者接口时，也只需要关心订阅哪个主题。整个流程下来，用户并不关心每条消息数据存储在Kafka集群哪个代理节点上。

> `2.分区的作用--水平扩展、并发读写、提高吞吐量`
>> 从性能方面来说，如果主题内消息数据只存储在一个代理节点， 那该节点将很快会成为Kafka 集群的瓶颈， 无法实现水平扩展。因此， 把主题内的消息数据分布到整个Kafka 集群就是一件很重要的事情， 而分区的引入则很好地解决了水平扩展的问题。
>> 主题上的每个分区可以被认为是一个无限长度的数组，新来的消息数据可以有序地追加到该数组上。从物理意义上讲，每个分区对应一个文件夹。一个Kafka 代理节点上可以存放多个分区。这样， “生产者”可以将消息数据发送到多个代理节点上的多个分区， “消费者”也可以并行地从多个代理节点上的不同分区获取数据，实现水平扩展。

> `3.副本的背景`
>> 在大数据场景中，企业的业务数据是非常宝贵的， 数据存储的要求非常严格，不允许有数据丢失的情况出现。因此，需要有一种机制来保证数据的高可用。

> `4.副本的作用--高可用、保证数据安全`
>> 为了保证消息数据的高可用性，主题中引入副本机制也是很有必要的。一个主题拥有多个副本，可以很好地避免数据丢失的风险。

#### 4.2.2 修改分区

> 注意:在Kafka中主题的分区数只能增加不能减少

> 查看

```
kafka-topics.sh  --zookeeper node1:2181 --describe --topic user
```

> 修改分区数(只能增加)

```
kafka-topics.sh  --zookeeper node1:2181 --alter --topic user --partitions 3
```

> 查看

```
kafka-topics.sh  --zookeeper node1:2181 --describe --topic user
```

#### 4.2.3 修改副本

> 创建主题

```
kafka-topics.sh --zookeeper node1:2181 --create --topic user2 --replication-factor 1 --partitions  6  --config max.message.bytes=102400
```

> 查看

```
kafka-topics.sh -describe -zookeeper node1:2181 --topic user2
```

`下面的了解`

> 编写json

```
vim user2_replicas.json
{
    "version": 1,
    "partitions": [{
        "topic": "user2",
        "partition": 0,
        "replicas": [2, 0, 1]
    }, {
        "topic": "user2",
        "partition": 1,
        "replicas": [0, 1, 2]
    }, {
        "topic": "user2",
        "partition": 2,
        "replicas": [1, 2, 0]
    }, {
        "topic": "user2",
        "partition": 3,
        "replicas": [2, 1, 0]
    }, {
        "topic": "user2",
        "partition": 4,
        "replicas": [0, 2, 1]
    }, {
        "topic": "user2",
        "partition": 5,
        "replicas": [1, 0, 2]
    }]
}
```

> 加载脚本修改副本数

```
kafka-reassign-partitions.sh --zookeeper node1:2181 --reassignment-json-file user2_replicas.json --execute
```

> 查看执行结果

```
kafka-reassign-partitions.sh --zookeeper node1:2181 --reassignment-json-file user2_replicas.json --verify
```

> 查看分区结果

```
kafka-topics.sh -describe -zookeeper node1:2181 --topic user2
```

### 4.3 生产--将消息数据写入Kafka 系统


> 在Kafka 系统中写入数据的应用一般被称为“生产者”，而读取数据的应用一般被称为“消费者” 。

> Kafka 集群中的数据均由生产者提供。生产者实时读取原始数据(例如日志数据、数据库记录、系统日志等〉，在代码结构中进行业务逻辑处理，然后调用Kafka 的生产者接口将处理后的消息记录写入Kafka 集群中。

> Kafka 生产者交互流程如图所示。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/34.png)

> Kafka 系统提供了一系列的操作脚本， 这些脚本放置在$KAFKA HOME/bin 目录中。

> 其中，kafka-console-producer.sh 脚本可用来作为生产者客户端。

> 在安装Kafka 集群后， 可以执行kafka-console-producer.sh 脚本快速地做一些简单的功能验证。

#### 4.3.1 使用脚本操作生产者

```
kafka-console-producer.sh --broker-list node1:9092 --topic test_topic
```

> test_topic主题会被自动创建

#### 4.3.2 启动消费者程序，并查看消息

```
kafka-console-consumer.sh --bootstrap-server node1:9092  --topic test_topic --from-beginning   (推荐)
```
或者
```
kafka-console-consumer.sh --zookeeper node1:2181  --topic test_topic --from-beginning   (已过期)
```

### 4.4 消费--从Kafka系统中读取消息数据

#### 4.4.1 消费者和消费者组

##### 4.4.1.1 消费者和消费者组的区别

> 一个消费者组，可以有一个或者多个消费者程序,消费者主要作用是读取消息数据。

> 消费者组名(GroupId)通常由一个字符串表示，具有唯一性；

> 如果一个消费者组订阅了主题，那么该主题中的每个分区只能分配给某一个消费者组中的某一个消费者程序。所以一般我们让`分区数 = 消费者数`

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/35.png)

##### 4.4.1.2 为什么需要消费者组

> 当生产者向Kafka 系统主题写消息数据的速度比消费者读取的速度要快时，随着时间的增长，主题中的消息数据将出现越来越严重的堆积现象。面对这类情况，通常可以增加多个消费者程序来水平扩展，从而解决这种堆积现象。

> `消费者组是Kafka 系统提供的一种可扩展、高容错的消费者机制。`

##### 4.4.1.3 消费者和分区的对应关系

> Kafka 消费者是消费者组中的一部分。

> 当一个消费者组中存在多个消费者程序来消费主题中的消息数据时，每个消费者程序会读取不同分区(Partition)上的消息数据。

> 1 个消费者程序，读取主题中6 个分区的数据(消费者压力很大)
>> 例如，现在有一个业务主题IP_Login ，它有6 个分区。而消费者组IP_Login_Group 中只有一个消费者程序IP_Login_ Consumer1 消费者程序Consumer1读取6 个分区的消息数据，如图所示。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/36.png)

> 3 个消费者程序，读取主题中6 个分区的数据(消费者压力还是比较大)
>> 如果消费者组中的消费者程序增加到3 个，此时每个消费者程序将读取两个分区中的消息数据， 如图

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/37.png)

> 6 个消费者程序，读取主题中6 个分区的数据(最佳状态: 消费者程序的数量 = 最大分区数)
>> 如果消费者组中的消费者程序增加到6 个，此时， 每个消费者程序将分别读取l 个分区的消息数据，如图

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/38.png)

> 7 个消费者程序，读取主题中6 个分区的数据(会有消费者处于空闲/浪费)
>> 如果消费者组中的消费者程序增加到7 个，此时， 每个消费者程序将分别读取1 个分区的消息数据，剩余的1 个消费者程序会处于空闲状态，如图

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/39.png)

> 总结
>> 总之，消费者客户端可以通过增加消费者组中消费者程序的个数来进行水平扩展，提升读取主题消息数据的能力。
> 
>> 因此， 在Kafka 系统生产环境中， 建议在创建主题时给主题分配多个分区，这样可以提高读取的性能。
> 
>> `消费者程序的数量尽量不要超过主题的最大分区数，即需要消费者程序的数量 = 最大分区数`
> 
>> `小于的话,个别消费者压力较大,`
> 
>> `大的话,多出来的消费者程序是空闲的,浪费系统资源。`

#### 4.4.2 操作消费者的方式

> `脚本`
>> Kafka 系统提供了一系列的可操作脚本， 这些脚本放置在$KAFKA HOME /bin 目录下。
> 
>> 其中，有一个脚本可用来作为消费者客户端，即kafka-console-consumer.sh 。
> 
>> Kafka 系统的消费者通过拉取的方式来获取主题(Topic)中的消息数据， 同时采用消费者组机制让每个消费者程序属于一个消费者组。
> 
>> 在创建一个消费者程序时，如果没有指定消费者组ID ， 则该消费者程序会被分配到一个默认的消费者组。
> 
>> 在Kafka 系统中，消费者组是一个全局概念，具有唯一性。

> `API`
>> Kafka 系统中， 消费者的实现方式分为`新／旧API` 。
>>>
>>> `新的`:在Kafka 0.10.0.x 之后的版本中，Kafka 系统默认将消费实例产生的元数据信息存储到一个名为“ __consumer_offsets”的内部主题中。
>>>
>>>> 使用--bootstrap-server
>>> 
>>>` 旧的`:在Kafka 0.10.0.x 之前的版本中，Kafka 系统默认的消费方式是将消费实例产生的元数据信息存储到Zookeeper 集群。
>>>
>>>> 使用--zookeeper

#### 4.4.3 脚本演示

##### 4.4.3.1 用新接口启动消费者程序

> Kafka 系统提供的kafka-console-consumer.sh 脚本对kafka-run-class .sh 脚本进行了二次封装，并引用了kafka.tools.Console.Consumer 工具类。该工具类会根据输入的参数类型，来判断运行的是Kafka 新版本消费者接口还是Kafka 旧版本消费者接口。

> 执行kafka-console-consumer.sh 脚本去“消费” 一个主题

```
kafka-console-consumer.sh --bootstrap-server node1:9092 --from-beginning  group.id=test_topic_group --topic test_topic
```

> 生产消息:

```
kafka-console-producer.sh --broker-list node1:9092 --topic test_topic
```

##### 4.4.3.2 用旧接口启动消费者程序--过期了

> 启动一个旧消费者程序

```
kafka-console-consumer.sh --zookeeper node1:2181  group.id=test_topic_group --topic test_topic --from-beginning
```

> 旧版本消费者元数据的存储结构
>> 在使用老版本消费者程序“消费”数据时， 每个消费者程序在被创建时都会往Zookeeper集群中写入元数据信息。
>> 如果消费者程序所属的消费者组在Zookeeper 集群中不存在，则会在Zookeeper 集群上的/consumers 目录中创建一个以消费者组名命名的目录，并在该目录下创建3 个子目录ids 、owners 、offsets

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/40.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/41.png)

## 5 Kafka-JavaAPI操作

### 5.1 生产者

#### 5.1.1 生产者接口

> Kafka 0.10.0.0 及以后的版本，对生产者代码的底层实现进行了重构。

> katka.producer.Producer类被org.apache.kafka.clients.producer.KafkaProducer类替换。

> Kafka 系统支持两种不同的发送方式--同步模式(Sync)和异步模式(ASync)

#### 5.1.2 生产模式

##### 5.1.2.1 异步模式

> 在Kafka 0.10.0.0 及以后的版本中，客户端应用程序调用生产者应用接口，默认使用异步的方式发送消息。

> 生产者客户端在通过异步模式发送消息时， 通常会调用回调函数的send()方法发送消息。

> 生产者端收到Kafka 代理节点的响应后会触发回调函数。

> 1 .什么场景下需要使用异步模式
>> 假如生产者客户端与Kafka 集群节点间存在网络延时（1OOms），此时发送10 条消息记录，则延时将达到1s 。而大数据场景下有着海量的消息记录， 发送的消息记录是远不止10 条，延时将非常严重。大数据场景下，如果采用异步模式发送消息记录，几乎没有任何耗时，通过回调函数可以知道消息发送的结果。

> 2 .异步模式数据写入流程
>> 例如，一个业务主题（ip_login）有6 个分区。生产者客户端写入一条消息记录时， 消息记录会先写入某个缓冲区，生产者客户端直接得到结果（这时，缓冲区里的数据并没有写到Kafka代理节点中主题的某个分区）。之后， 缓冲区中的数据会通过异步模式发送到Kafka 代理节点中主题的某个分区中。具体数据写入流程如图所示。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/42.png)

> 消息记录提交给send()方法后，实际上该消息记录被放入一个缓冲区的发送队列，然后通过后台线程将其从缓冲区队列中取出井进行发送； 发送成功后会触发send 方法的回调函数Callback 。

##### 5.1.2.2 同步模式

> 生产者客户端通过send（）方法实现同步模式发送消息，并返回一个Future 对象，同时调用get方法等待Future 对象， 看send方法是否发送成功。

> 1.什么场景下使用同步模式
>> 如果要在写数据到Kafka 集群代理节点时需要立即知道消息是否写入成功，此时应使用同步模式。

> 2.同步模式的数据写入流程
>> 例如，在一个业务主题ip_login 中有6 个分区。生产者客户端写入一条消息记录到生产者服务端，生产者服务端接收到数据后会立马将其发送到主题ip_login 的某个分区去，然后才将结果返给生产者客户端。具体流程如图4- 8 所示。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/43.png)

> 这里通过调用Future 接口中的get（）方法等待Kafka 集群代理节点(Broker)的状态返回。如果Producer 发送消息记录成功了， 则返回RecordMetadata 对象，该对象可用来查看消息记录的偏移量(Offset)。

> 采用同步模式发送消息记录，系统的性能会下降很多，因为需要等待写入结果。

> 如果生产者客户端和Kafka 集群间的网络出现异常，或者Kafka 集群处理消息请求过慢，则消息的延时将很严重。

> `所以， 一般不建议这样使用。`

##### 5.1.2.3 代码演示

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/44.png)

> `pom`

```xml
<dependencies>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.2</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

> `同步模式`

```
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Properties;
import java.util.concurrent.ExecutionException;

/**
 * Author xiaoma
 * Date 2020/10/8 16:46
 * Desc 演示Kafka的生产者API-同步模式
 */
public class MyKafkaProducer_Sync {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //1.准备Properties参数
        Properties props = new Properties();
        //Kafka集群地址
        //props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");//node1:9092,node2:9092,node3:9092
        props.put("bootstrap.servers", "node1:9092");//node1:9092,node2:9092,node3:9092
        //消息确认机制
        //acks=0，意思就是KafkaProducer客户端，只要把消息发送出去，不管那条数据有没有在Partition Leader上落到磁盘，都不管他了，直接就认为这个消息发送成功了
        //acks=1，只要Partition Leader接收到消息，就认为成功了，不管他其他的Follower有没有同步过去这条消息了。
        //acks=all/-1，意思就是说Partition Leader接收到消息之后，还必须要求ISR列表里跟Leader保持同步的那些Follower都要把消息同步过去，才能认为这条消息是写入成功了。
        //all即所有副本都同步到数据时send方法才返回, 以此来完全判断数据是否发送成功, 理论上来讲数据不会丢失
        props.put("acks", "all");
        //重试次数
        props.put("retries", 1);
        //消息发送的批次大小,单位:byte,注意:消息发到Kafka集群是以批次的形式发送的
        props.put("batch.size", 16384);
        //消息发送的时间间隔,单位:ms, 注意:batch.size和linger.ms满足一个就会发送!
        props.put("linger.ms", 100);
        //内存缓冲区大小
        props.put("buffer.memory", 33554432);
        //k-v序列化类型
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");


        //2.创建KafkaProducer
        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(props);

        //3.发送消息
        for (int i = 0; i < 10; i++) {//循环发送10条消息到Kafka
            //将需要发送到kafka的消息封装为record对象
            ProducerRecord<String, String> record = new ProducerRecord<>("order", "key_"+i, "value_" + i);
            //同步发送消息,并返回消息的元数据,如消息发送到哪个partation了,offset是多少?
            RecordMetadata metadata = kafkaProducer.send(record).get();
            System.out.println("消息发送到"+metadata.partition()+"号partation,offset为:"+metadata.offset());
        }
        System.out.println("同步消息已发送完毕");

        //4.关闭资源
        kafkaProducer.close();
    }
}
```

> `异步模式`

```
import org.apache.kafka.clients.producer.*;

import java.util.Properties;
import java.util.concurrent.ExecutionException;

/**
 * Author xiaoma
 * Date 2020/10/8 16:46
 * Desc 演示Kafka的生产者API-异步模式
 */
public class MyKafkaProducer_ASync {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //1.准备Properties参数
        Properties props = new Properties();
        //Kafka集群地址
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");//node1:9092,node2:9092,node3:9092
        props.put("bootstrap.servers", "node1:9092");//node1:9092,node2:9092,node3:9092
        //消息确认机制
        //acks=0，意思就是KafkaProducer客户端，只要把消息发送出去，不管那条数据有没有在Partition Leader上落到磁盘，都不管他了，直接就认为这个消息发送成功了
        //acks=1，只要Partition Leader接收到消息，就认为成功了，不管他其他的Follower有没有同步过去这条消息了。
        //acks=all/-1，意思就是说Partition Leader接收到消息之后，还必须要求ISR列表里跟Leader保持同步的那些Follower都要把消息同步过去，才能认为这条消息是写入成功了。
        //all即所有副本都同步到数据时send方法才返回, 以此来完全判断数据是否发送成功, 理论上来讲数据不会丢失
        props.put("acks", "all");
        //重试次数
        props.put("retries", 1);
        //消息发送的批次大小,单位:byte,注意:消费发到Kafka集群是以批次的形式发送的
        props.put("batch.size", 16384);
        //消息发送的时间间隔,单位:ms, 注意:batch.size和linger.ms满足一个就会发送!
        props.put("linger.ms", 100);
        //内存缓冲区大小
        props.put("buffer.memory", 33554432);
        //k-v序列化类型
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");


        //2.创建KafkaProducer
        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(props);

        //3.发送消息--异步
        for (int i = 0; i < 100; i++) {//循环发送10条消息到Kafka
            //将需要发送到kafka的消息封装为record对象
            ProducerRecord<String, String> record = new ProducerRecord<>("order", "key_" + i, "value_" + i);
            /*//同步发送消息,并返回消息的元数据,如消息发送到哪个partation了,offset是多少?
            RecordMetadata metadata = kafkaProducer.send(record).get();
            System.out.println("消息发送到"+metadata.partition()+"号partation,offset为:"+metadata.offset());*/
            //异步发送消息,传入需要发送的record,和该record真正发送成功后的需要执行回调函数!
            kafkaProducer.send(record, new Callback() {
                //onCompletion方法会在record真正发送成功后执行
                @Override
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    //record真正发送成功后才会执行该方法.所以可以在该方法里面获取到metadata
                    System.out.println("消息发送到"+metadata.partition()+"号partation,offset为:"+metadata.offset());
                }
            });
        }
        System.out.println("异步消息已发送完毕");

        //4.关闭资源
        kafkaProducer.close();
    }
}
```

#### 5.1.3 分区策略

##### 5.1.3.1 默认分区策略

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/45.png)

> `第1种：如果指定了分区号，那么数据就会全部进入到指定的分区里面去`
>> //producer.send(new ProducerRecord<String, String>("test",1,"1", "hello world"+i));
> 
>> //如果给了分区号，也给了key值，那么优先使用指定的分区号

> `第2种：如果没有给定分区号，但是给了数据的key，那么通过key的hash取值来决定数据到哪一个分区里面去`
>> producer.send(new ProducerRecord<String, String>("test","101", "hello world"+i));
> 
>> //如果没有指定分区号，给定了我们的key，那么就会通过key的hash取值进行分区，实际工作当中，如果通过这种方式进行分区一定要注意，key的值一定要变化

> `第3种：没有给定分区号，也没有给定key值，通过轮询的方式来决定数据去哪一个分区`
>> producer.send(new ProducerRecord<String, String>("test", "hello world"+i));
> 
>> //没有给定分区，也没有给数据的key值，那么就会使用轮循的方式实现分区

> 总结:
>> 如果指定partition，就用partition
> 
>> 如果指定key，使用key进行hash取模。
> 
>> 如果没有指定key，使用轮询的方式。

##### 5.1.3.2 自定义分区策略

> 在分布式应用场景中， Kafka 系统默认的分区策略并不能很好地满足业务需求，这时需根据Kafka 系统提供的应用接口来自定义主题分区， 以满足具体的业务场景需求。

> 实现一个自定义主题分区的基本步骤如下：
>> 实现Partitioner 接口， 并重写partition()方法， 在该方法中实现自定义主题分区的算法；
> 
>> 在生产者应用程序中，设置partitioner.class 属性为自定义主题分区类。

```
import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;

import java.util.Map;

/**
 * Author xiaoma
 * Date 2020/10/9 9:58
 * Desc
 */
public class MyPartition implements Partitioner {
    //该方法返回的分区编号就是该key对应的分区编号
    //根据key/手机号的前三位的hash值对数据进行分区
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        String phoneNum = (String) key;
        String threeNum = phoneNum.substring(0, 3);//前三位
        int partitionCount = cluster.partitionsForTopic(topic).size();//分区数
        int partitionNum = Math.abs(threeNum.hashCode()) % partitionCount;
        System.out.println("手机号前三位:"+threeNum+" 分区编号:"+partitionNum);
        return partitionNum;
    }

    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}
```

> 程序里指定自定义分区策略

```
props.put("partitioner.class", "cn.xiaoma.MyPartitioner");// 指定自定义分区类
```

### 5.2 消费者

#### 5.2.1 消费者接口

> `新版` 消费者API把旧版的高阶API和旧版的低阶API者整合到一起了，对应KafkaConsumer类的subscribe和assign方法

`注意:`

> `新版` API 是在 kafka 0.9 版本后增加的，推荐使用新版 API


> [http://kafka.apache.org/0100/documentation.html#impl_consumer](http://kafka.apache.org/0100/documentation.html#impl_consumer) <br/>
> [https://blog.csdn.net/Simon_09010817/article/details/83748974](https://blog.csdn.net/Simon_09010817/article/details/83748974) <br/>
> [https://blog.csdn.net/Simon_09010817/article/details/83750115](https://blog.csdn.net/Simon_09010817/article/details/83750115)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/46.png)

> 消费者新接口的实现原理
>> 在Kafka 0.10.0.x 及之后版本中， 消费者实现的原理并不复杂， 它利用Kafka 系统的内部主题，以消费者组(Group) 、主题(Topic) 和分区(Partition)作为组合主键，所有消费者程序产生的偏移量(Offsets)都会提交到该内部主题__consumer_offsets中进行存储。
> 
>> 由于消费者程序产生的这部分数据非常重要，不能丢失，所以将消息数据的应答(Acks )级别设置为all(-1)
> 
>> Kafka 系统又在内存中构建了一个三元组:Group 、Topic)和Partition来维护最新的偏移量信息。
> 
>> 消费者程序可以直接从内存中获取最新的偏移量值。

#### 5.2.2 如何指定从哪个偏移量开始消费

> `auto.offset.reset`
>> kafka-0.10.1.X版本之前: (offest保存在zk中)；
>> 
>>> `auto.offset.reset 的值为smallest和largest`.
>>
>> kafka-0.10.1.X版本之后: (offest保存在kafka的名为__consumer_offsets的topic里面)；
>> 
>>> `auto.offset.reset 的值更改为:earliest,latest,和none`
>>
>>> 1.`earliest` ：当各分区下有已提交的 Offset 时，从提交的 Offset开始消费；无提交的Offset 时，从头开始消费；
>> 
>>> 2.`latest` ： 当各分区下有已提交的 Offset 时，从提交的 Offset 开始消费；无提交的 Offset时，消费新产生的该分区下的数据
>> 
>>> 3.`none` ： Topic 各分区都存在已提交的 Offset 时，从 Offset 后开始消费；只要有一个分区不存在已提交的 Offset，则抛出异常。

#### 5.2.3 如何提交消息的偏移量

> 在Kafka 0.10.0. x 之前
>> 消费者程序会将“消费”的偏移量(Offsets)提交到Zookeeper系统的／consumers 目录，例如，消费者组名为test_topic_group，主题名为test_topic,分区数为l ，那么运行老版本消费者程序后，在Zookeeper 系统中，偏移量提交的路径是/test_topic_group/offsets/test_ topic/O	使用Zookeeper 集群来存储元数据信息是存在比较大的风险的：虽然Java 虚拟机帮助系统能完成一些优化操作，但是消费者程序频繁地与Zookeeper集群发生写交互，不仅性能比较低，而且后期水平扩展也比较困难；如果写元数据期间Zookeeper 集群的性能降低，则Kafka 集群的吞吐量也跟着受影响。
>> Zookeeper系统并不适合频繁地进行读写操作，因为Zookeeper 系统性能降低会严重影响Kafka 集群的吞吐量。

> 所以，在Kafka新版本消费者程序中，对偏移量的提交进行了重构，将其保存到Kafka 系统内部主题中，消费者程序产生的偏移量会持续追加到该内部主题的分区中。Kafka 系统提供了两种方式来提交偏移量，它们分别是自动提交和手动提交。

##### 5.2.3.1 自动提交

> 使用KafkaConsumer 自动提交偏移量时，

> 需要在配置属性中将“`enable.auto.commit`”设置为`true` ，

> 另外可以设置“`auto.commit.interval.ms`＂ 属性来控制自动提交的时间间隔。

> Kafka 系统自动提交偏移量的底层实现调用了`ConsumerCoordinator` 的`commitOffsetsSync()`函数来进行同步提交，或者·commitOffsetsAsync()·函数来进行异步提交。自动提交的流程如图

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/47.png)

```
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.util.Arrays;
import java.util.Properties;

/**
 * Author xiaoma
 * Date 2020/10/9 10:50
 * Desc 演示Kafka消费者API-自动提交offset
 */
public class MyKafkaConsumer_AutoCommit {
    public static void main(String[] args) {
        //1.准备参数
        Properties props = new Properties();
        //kafka集群地址
        props.put("bootstrap.servers", "node1:9092");//node1:9092,node2:9092,node3:9092
        //消费者组名称(如果不指定,会自动生成一个,但一个都指定,方便管理)
        props.put("group.id", "myGroup");
        //是否自动提交offset,true表示自动提交
        props.put("enable.auto.commit", "true");
        //自动提交偏移量时的时间间隔ms值
        props.put("auto.commit.interval.ms", "1000");
        //配置offset重置位置
        //如果有offset记录就从记录的位置开始消费
        //如果没有记录offset,earliest表示从最开始的数据,latest表示从最新的数据,none报错
        props.put("auto.offset.reset", "earliest");

        //反序列化kv类型
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        //2.创建Kafka消费者对象
        KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(props);

        //3.订阅主题
        kafkaConsumer.subscribe(Arrays.asList("order"));

        //4.开始消费
        while (true){//消费者可以一直运行并订阅主题消费其中的消息!
            //poll表示从Kafka获得消息
            ConsumerRecords<String, String> Records = kafkaConsumer.poll(100);
            for (ConsumerRecord<String, String> record : Records) {
                System.out.println("消费到的数据,分区:"+record.partition()+" offset:"+record.offset()+" value:" + record.value());
            }
        }
    }
}
```

##### 5.2.3.2 手动提交

> 在编写消费者程序代码时，

> 将配置属性“`enable.auto.commit`”的值设为“`false`”，则可以通过手动模式来提交偏移量。

> Kafka Consumer 消费者程序类提供了两种手动提交偏移量的方式--同步提交`commitSync()`函数和异步提交`commitAsync()`函数。

> 同步提交和异步提交的区别在于：同步提交需要等待响应结果，会造成阻塞现象；异步提交不会被阻塞。

> 在实际应用场景中，会采用异步提交的方式来管理偏移量，这样有助于提升消费者程序的吞吐量。

```
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Properties;

/**
 * Author xiaoma
 * Date 2020/10/9 10:50
 * Desc 演示Kafka消费者API-手动提交offset
 */
public class MyKafkaConsumer_ManualCommit {
    public static void main(String[] args) {
        //1.准备参数
        Properties props = new Properties();
        //kafka集群地址
        props.put("bootstrap.servers", "node1:9092");//node1:9092,node2:9092,node3:9092
        //消费者组名称(如果不指定,会自动生成一个,但一个都指定,方便管理)
        props.put("group.id", "myGroup");
        //是否自动提交offset,true表示自动提交,false表示使用手动提交
        props.put("enable.auto.commit", "false");
        //自动提交偏移量时的时间间隔ms值,手动提交时不需要指定时间间隔
        //props.put("auto.commit.interval.ms", "1000");
        //配置offset重置位置
        //如果有offset记录就从记录的位置开始消费
        //如果没有记录offset,earliest表示从最开始的数据,latest表示从最新的数据,none报错
        props.put("auto.offset.reset", "earliest");

        //反序列化kv类型
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        //2.创建Kafka消费者对象
        KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(props);

        //3.订阅主题
        kafkaConsumer.subscribe(Arrays.asList("order"));

        //4.开始消费
        //手动提交:
        //什么时候提交?
        //--1.指定时间间隔,每隔xxms提交一次,这就和自动提交一样了,还不如直接使用自动提交!
        //--2.每消费一条就提交一次!可以,但是可能会影响性能!
        //--3.每消费一小批就提交一次,如:每消费5条就提交一次!
        List<ConsumerRecord> list = new ArrayList<>();
        while (true){//消费者可以一直运行并订阅主题消费其中的消息!
            //poll表示从Kafka获得消息
            ConsumerRecords<String, String> Records = kafkaConsumer.poll(100);
            for (ConsumerRecord<String, String> record : Records) {
                System.out.println("消费到的数据,分区:" + record.partition() + " offset:" + record.offset() + " value:" + record.value());
                //kafkaConsumer.commitAsync();//每消费一条就提交一次
                list.add(record);
                if (list.size() >= 5) {
                    kafkaConsumer.commitSync();//每消费5条就提交一次!//同步
                    //kafkaConsumer.commitAsync();//每消费5条就提交一次!//异步
                    list.clear();
                    System.out.println("offset已提交");
                }
            }
        }
    }
}

```

## 6 Kafka拦截器

### 6.1 拦截器原理

> Producer拦截器(interceptor)是在Kafka 0.10版本被引入的，主要用于实现clients端的定制化控制逻辑。

> 对于producer而言，interceptor使得用户在消息发送前以及producer回调逻辑前有机会对消息做一些定制化需求，比如`修改消息`等。

> 同时，producer允许用户指定多个interceptor按序作用于同一条消息从而形成一个拦截链(interceptor chain)。

> Intercetpor的接口是`org.apache.kafka.clients.producer.ProducerInterceptor`，其定义的方法包括：

> 1.`configure(configs)`
>> 获取配置信息和初始化数据时调用。

> 2.`onSend(ProducerRecord)`
>> 该方法封装进KafkaProducer.send方法中，即它运行在用户主线程中。Producer确保在消息被序列化以及计算分区前调用该方法。
>> 用户可以在该方法中对消息做任何操作，但最好保证不要修改消息所属的topic和分区，否则会影响目标分区的计算

> 3.`onAcknowledgement(RecordMetadata, Exception)`
>> 该方法会在消息被应答或消息发送失败时调用，并且通常都是在producer回调逻辑触发之前。onAcknowledgement运行在producer的IO线程中，因此不要在该方法中放入很重的逻辑，否则会拖慢producer的消息发送效率

> 4.`close`：
>> 关闭interceptor，主要用于执行一些资源清理工作
>> 如前所述，interceptor可能被运行在多个线程中，因此在具体实现时用户需要自行确保线程安全。另外倘若指定了多个interceptor，则producer将按照指定顺序调用它们，并仅仅是捕获每个interceptor可能抛出的异常记录到错误日志中而非在向上传递。这在使用过程中要特别留意。

### 6.2 拦截器案例

#### 6.2.1 需求

> 实现一个简单的双interceptor组成的拦截链。

> 第1个interceptor将消息中的手机号打码,如13888888888--> 138****8888

> 第2个interceptor在消息发送后统计成功发送消息数或失败发送消息数

#### 6.2.2 代码实现

> 1 增加打码拦截器

```
import org.apache.kafka.clients.producer.ProducerInterceptor;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Map;

/**
 * Author xiaoma
 * Date 2020/10/9 11:45
 * Desc 用来在消息发送到Kafka之前对消息进行打码处理
 * 如:13888888888 -> 138****8888
 */
public class MyInterceptor1_Mosaic implements ProducerInterceptor<String,String> {
    //onSend会在消息真正发送到Kafka之前被调用执行!
    @Override
    public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
        String phoneNum = record.value();
        String mosaicPhoneNum = phoneNum.substring(0, 3) + "****" + phoneNum.substring(7);
        System.out.println("打码前:"+phoneNum+" 打码后:"+mosaicPhoneNum);
        ProducerRecord<String, String> mosaicRecord = new ProducerRecord<>(
                record.topic(),
                record.partition(),
                record.timestamp(),
                record.key(),
                mosaicPhoneNum,//注意:把原来的手机号换为打码后的!
                record.headers()
        );
        return mosaicRecord;//将打码之后的ProducerRecord返回/发送给Kafka集群
    }

    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {

    }

    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}
```

> 2 统计发送消息成功和发送失败消息数，并在producer关闭时打印这两个计数器

```
import org.apache.kafka.clients.producer.ProducerInterceptor;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Map;

/**
 * Author xiaoma
 * Date 2020/10/9 11:45
 * Desc 统计发送消息成功和失败的次数并输出
 */
public class MyInterceptor2_Statistics  implements ProducerInterceptor<String,String> {
    private Integer successCount = 0;//成功次数
    private Integer failCount = 0;//失败次数

    @Override
    public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
        return record;//直接发
    }

    //消息发送之后会执行,也就是ack之后执行
    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
            if(exception == null){
                //发送成功
                successCount++;
            }else{
                //发送失败
                failCount++;
            }
    }

    //拦截器执行完之后执行
    @Override
    public void close() {
        //输出发送成功和发送失败的次数
        System.out.println("发送成功的次数"+successCount+",发送失败的次数"+failCount);
    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}
```

> 3 producer主程序

```
//指定使用自定义的拦截器
props.put("interceptor.classes", Arrays.asList(
        "cn.itcast.interceptor.MyInterceptor1_Mosaic",
        "cn.itcast.interceptor.MyInterceptor2_Statistics"
));
```

> 测试

>> 在kafka上启动消费者，然后运行客户端java程序。

```
kafka-console-consumer.sh --bootstrap-server node1:9092 --topic test_topic
```

>> 观察java平台控制台输出数据如下

```
发送成功的消息数: 10
发送失败的消息数: 0
```

## 7 Kafka Streams

### 7.1 Kafka Streams

> Kafka在0.10.0.0版本以前的定位是分布式，分区化的，带备份机制的日志提交服务。

> 在这之前kafka也没有提供数据处理的服务。

> 大家的流处理计算主要是还是依赖于Spark Streaming，Flink等流式处理框架。

> 但是他们都离不开Kafka的消息中转，所以Kafka于0.10.0.0版本推出了自己的流处理框架，Kafka Streams。

> Kafka的定位也正式变成为了Apache Kafka® is a distributed streaming platform，分布式流处理平台。

### 7.2 Kafka Streams特点

> `轻量级`
>> 无需专门的集群
> 
>> 一个库，而不是框架

> `完全集成`
>> 100%的Kafka 0.10.0版本兼容
> 
>> 易于集成到现有的应用程序

> `实时性`
>> 毫秒级延迟
> 
>> 并非微批处理
> 
>> 窗口允许乱序数据
> 
>> 允许迟到数据
> 
>> Record级别的处理

### 7.3 Kafka Streams案例

[https://kafka.apache.org/documentation/streams/](https://kafka.apache.org/documentation/streams/)

> 1 需求
>> 从order主题消费消息

> 2 案例代码

```
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.common.utils.Bytes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KTable;
import org.apache.kafka.streams.kstream.Materialized;
import org.apache.kafka.streams.state.KeyValueStore;

import java.util.Arrays;
import java.util.Properties;

public class MyKafkaStreamsTest {
    public static void main(final String[] args) throws Exception {
        //1.准备参数
        Properties props = new Properties();
        //应用名称
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-application");
        //集群地址
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "node1:9092");
        //kv序列化类型
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

        //2.创建流处理对象
        StreamsBuilder builder = new StreamsBuilder();
        //3.订阅主题并消费消息
        //textLines就是从kafka中消费到的消息,如:kafka kafka flink flink flink
        KStream<String, String> textLines = builder.stream("order");
        //4.处理数据
        KTable<String, Long> wordCounts = textLines
                .flatMapValues(textLine -> Arrays.asList(textLine.toLowerCase().split("\\W+")))//切割单词
                .groupBy((key, word) -> word)//按照单词进行分组
                .count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as("counts-store"));//计数,得到:(kafka,2),(flink,3)
        //5.输出结果
        wordCounts.foreach((k,v)-> System.out.println(k+":"+v));

        //6.构建并启动流处理
        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();
    }

}

```

> 3 运行程序

>> 启动生产者

```
kafka-console-producer.sh --broker-list node1:9092 --topic test_topic
```

> 启动程序

## 8 Kafka原理

### 8.1 架构图解

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/15.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/16.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/17.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/18.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/19.png)

### 8.2 核心概念

#### 8.2.1 代理、生产者、消费者、消费者组

##### 8.2.1.1 代理

> 在Kafka 集群中，一个Kafka 进程/实例被称为一个Broker代理节点。

##### 8.2.1.2 生产者

> 在Kafka 系统中，向kafka broker发消息的客户端通常被称为Producer生产者

##### 8.2.1.3 消费者

> Consumer消费者从Kafka 集群指定的Topic主题中读取消息记录。

> 在读取主题数据时， 需要设置消费组名(Groupid)。如果不设置，则Kafka 消费者会默认生成一个消费组名称。

##### 8.2.1.4 消费者组

> 消费者程序在读取Kafka系统Topic主题中的数据时， 通常会使用多个线程来执行。

> 一个消费者组可以包含一个或多个消费者程序，使用多分区和多线程模式可以极大提高读取数据的效率。

> `注意`:
>> 一般而言， 一个消费者对应一个线程。在给应用程序设直线程数量时，遵循“线程数<=分区数”原则。
>> 如果线程数大于分区数，则多余的线程不会消费分区中的数据，这样会造成资源浪费。

#### 8.2.2 主题、分区、副本、记录

##### 8.2.2.1 主题

> 通过主题来区分不同业务类型

> Kafka 系统通过主题来区分不同业务类型的消息记录。

> 如图所示:不同类型的消息记录进行分类后写入到对应的主题中。

> 例如充值记录、登录记录、交易记录、聊天记录，分别写入到充值主题、登录主题、交易主题、聊天主题中进行存储。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/20.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/21.png)

##### 8.2.2.2 分区

> 通过分区(Partition)来支持物理层面上的并发读写， 以提高Kafka 集群的吞吐量。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/22.png)

> 每一个主题(Topic)中可以有一个或者多个分区(Partition)。

> 在Kafka 系统的设计思想中，分区是基于物理层面上的，不同的分区对应着不同的数据文件夹。

> 每个分区(Partition)内部的消息记录是有序的，每个消息都有一个偏移量序号(Offset)。

> 一个分区只对应一个代理节点(Broker )，一个代理节点可以管理多个分区。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/23.png)

> Kafka 通过分区(Partition)来支持物理层面上的并发读写， 以提高Kafka 集群的吞吐量。如下图:

> Consumer group A 有两个消费者来读取4个partition中数据；Consumer group B有四个消费者来读取4个 partition中的数据

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/24.png)

##### 8.2.2.3 副本

> Kafka通过副本(Replication) 机制来保证集群数据的高可用性，并保证数据的安全（容错）。

> 在Kafka 系统中， 每个主题(Topic)在创建时会要求指定它的副本数(以Partition为单位对Message进行冗余备份)，默认是1 。

> 通过副本(Replication) 机制来保证Kafka 分布式集群数据的高可用性，并保证数据的安全（容错）。

> `注意`:
>> 一般设置为2或3，不可设置太大，因为数据要同步到不同机器上，副本数太大的话要大量的网络传输和磁盘占用

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/25.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/26.png)

##### 8.2.2.4 记录

> 被实际写入到Kafka 集群并且可以被消费者应用程序读取的数据，被称为记录(Record)。

> 每条记录包含一个键(Key)、值(Value)和时间戳(Times tamp) 。

#### 8.2.3 Offset、Segment、Leader、ISR

##### 8.2.3.1 Offset(偏移量)

> offset偏移量是分区内每条消息的id/唯一标记，是一个long型数字/8个字节的数字,用于定位位于segment段里的唯一消息。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/27.png)

> Kafka offset管理
>> 消费者在消费的过程中需要记录自己消费了多少数据，即消费 Offset。
> 
>> Kafka Offset 是Consumer Position，与 Broker 和 Producer 都无关。
> 
>> 每个 Consumer Group、每个 Topic 的每个Partition 都有各自的 Offset，如下图所示。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/28.png)

> 通常由如下几种 Kafka Offset 的管理方式：
>> 1.Spark Checkpoint：在 Spark Streaming 执行Checkpoint 操作时，将 Kafka Offset 一并保存到 HDFS 中。这种方式的问题在于：当 Spark Streaming 应用升级或更新时，以及当Spark 本身更新时，Checkpoint 可能无法恢复。因而，不推荐采用这种方式。
> 
>> 2.HBASE、Redis 等外部 NOSQL 数据库：这一方式可以支持大吞吐量的 Offset 更新，但它最大的问题在于：用户需要自行编写 HBASE 或 Redis 的读写程序，并且需要维护一个额外的组件。
> 
>> 3.ZOOKEEPER：老版本的位移offset是提交到zookeeper中的，目录结构是 ：/consumers/<group.id>/offsets/ <topic>/<partitionId> ，但是由于 ZOOKEEPER 的写入能力并不会随着 ZOOKEEPER 节点数量的增加而扩大，因而，当存在频繁的 Offset 更新时，ZOOKEEPER 集群本身可能成为瓶颈。因而，不推荐采用这种方式。
> 
>> 4.KAFKA 自身的一个特殊 Topic（__consumer_offsets）中：这种方式支持大吞吐量的Offset 更新，又不需要手动编写 Offset 管理程序或者维护一套额外的集群，因而是迄今为止最为理想的一种实现方式。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/29.png)

##### 8.2.3.2 Segment(分段)

> partition、segment、offset都是为topic服务的，每个topic可以分为多个partition，一个partition相当于一个大目录，每个partition下面有多个大小相等的segment文件，这个segment是由message组成的。segment大小及生命周期在server.properties文件中配置。

> 分区在存储层面是逻辑append log文件，包含多个segment文件

> 每个partiiton有多个segment，segment又包含了两个同名文件:
>> `xxx.log`：存放我们的日志文件，即所有的数据最后都以日志文件的形式存放到了kafka集群当中
> 
>> `xxx.index` ：其实就是一个索引，记录了一条消息在log文件中的位置，查找消息的时候先从index获取位置，然后就可以定位到消息在log文件具体哪个地方，这样查找消息的速度更快。
> 
>> segment file的命名为:起始offset.log.例如"00000000000.log";

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/30.png)

> Kafka解决查询效率的手段之一是将数据文件分段，比如有100条Message，它们的offset是从0到99。假设将数据文件分成5段，第一段为0-19，第二段为20-39，以此类推，每段放在一个单独的数据文件里面，数据文件以该段中最小的offset命名。这样在查找指定offset的Message的时候，用二分查找就可以定位到该Message在哪个段中。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/31.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/32.png)

> 比如：要查找绝对offset为7的Message：
>> 1.首先是用二分查找确定它是在哪个LogSegment中，自然是在第一个Segment中。
> 
>> 2.打开这个Segment的index文件，也是用二分查找找到offset小于或者等于指定offset的索引条目中最大的那个offset。自然offset为6的那个索引是我们要找的，通过索引文件我们知道offset为6的Message在数据文件中的位置为9807。
> 
>> 3.打开数据文件，从位置为9807的那个地方开始顺序扫描直到找到offset为7的那条Message。

> 这套机制是建立在offset是有序的。索引文件被映射到内存中，所以查找的速度还是很快的。

> 一句话，Kafka的Message存储采用了分区(partition)，分段(LogSegment)和稀疏索引这几个手段来达到了高效性。

> `注意`:
> 
>> 1.当某个segment上的消息条数达到配置值或消息发布时间超过阈值时，segment上的消息会被flush到磁盘，只有flush到磁盘上的消息订阅者才能订阅到
> 
>> 2.segment达到一定的大小（可以通过配置文件设定,默认1G）后将不会再往该segment写数据，broker会创建新的segment。

##### 8.2.3.3 Leader

> 每个Replication集合中的Partition都会选出一个唯一的Leader，所有的读写请求都由Leader处理。

> 其他Replicas(follower)从Leader处把数据更新同步到本地。

> `注意`:
>>每个Cluster中会选举出一个Broker来担任Controller，负责处理Partition的Leader选举，协调Partition迁移等工作。
> 
>>partition的leader和follower之间的监控通过ZK完成。
> 
>>写是都往leader上写，读也只在leader上读，flower只是数据的一个备份，保证leader被挂掉后顶上来，并不往外提供服务。

##### 8.2.3.4 ISR(In-Sync Replica)

> ISR是Replicas的一个子集，表示目前Alive且与Leader能够“Catch-up”跟得上的Replicas(follower)集合。

> 由于读写都是首先落到Leader上，所以一般来说通过同步机制从Leader上拉取数据的Replica都会和Leader有一些延迟，leader会维护一个与其基本保持同步的Replica列表，该列表称为ISR(in-sync Replica)，每个Partition都会有一个ISR，由leader动态维护

> 如果一个flower比一个leader落后太多，或者超过一定时间未发起数据复制请求，则leader将其重ISR中移除 ,也就是延迟时间和延迟条数任意一个超过阈值都会把该Replica踢出ISR。

> 每个Partition都有它自己独立的ISR。

> Kafka是同步还是异步?
>> kafka不是完全同步，也不是完全异步，是一种ISR机制
> 
>> 同步还是异步取决于何时Commit？
> 
>> 同步复制： 只有所有的ISR把数据拿过去后才commit，一致性好，可用性不高。
> 
>> 异步复制： 只要leader拿到数据立即commit，等ISR慢慢去复制，可用性高，立即返回，一致性差一些。
> 
>> Commit：是指leader告诉客户端，这条数据写成功了。


> `commit策略`：<br/>
>> `server配置`
>>> rerplica.lag.time.max.ms=10000
>>> /# 如果leader发现flower超过10秒没有向它发起fech请求，那么leader考虑这个flower是不是程序出了点问题
>>> /# 或者资源紧张调度不过来，它太慢了，不希望它拖慢后面的进度，就把它从ISR中移除。
>>> rerplica.lag.max.messages=4000 # 相差4000条就移除
>>> /# flower慢的时候，保证高可用性，同时满足这两个条件后又加入ISR中，
>>> /# 在可用性与一致性做了动态平衡
>>
>> `topic配置`
>>>min.insync.replicas=1 # 需要保证ISR中至少有多少个replica
>>
>> `Producer配置`
>>> request.required.asks=0
>>> /# 1：当leader接收到消息之后发送ack，丢会重发，丢的概率很小
>>> /# 0: 相当于异步的，不需要leader给予回复，producer立即返回，发送就是成功, 可能会丢失消息
>>> /# -1：当所有的follower都同步消息成功后发送ack.  几乎不会丢失消息

### 8.3 Kafka生产过程分析

#### 8.3.1 写入方式

> producer采用推（push）模式将消息发布到broker，每条消息都被追加（append）到分区（patition）中，属于顺序写磁盘（顺序写磁盘效率比随机写内存要高，保障kafka吞吐率）。

#### 8.3.2 写入流程

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/48.png)

> producer写入消息流程如下：
> 1、总体流程
> 
>> Producer连接任意活着的Broker，请求指定Topic，Partion的Leader元数据信息，然后直接与对应的Broker直接连接，发布数据
> 
> 2、开放分区接口(生产者数据分发策略)
> 
>> 2.1、用户可以指定分区函数，使得消息可以根据key，发送到指定的Partition中。
> 
>> 2.2、kafka在数据生产的时候，有一个数据分发策略。默认的情况使用DefaultPartitioner.class类。

#### 8.3.3 分区（Partition）

> Kafka集群有多个消息代理服务器（broker-server）组成，发布到Kafka集群的每条消息都有一个类别，用主题（topic）来表示。

> 通常，不同应用产生不同类型的数据，可以设置不同的主题。

> 一个主题一般会有多个消息的订阅者，当生产者发布消息到某个主题时，订阅了这个主题的消费者都可以接收到生成者写入的新消息。

> Kafka集群为每个主题维护了分布式的分区（partition）日志文件，物理意义上可以把主题（topic）看作进行了分区的日志文件（partition log）。

> 主题的每个分区都是一个有序的、不可变的记录序列，新的消息会不断追加到日志中。

> 分区中的每条消息都会按照时间顺序分配到一个单调递增的顺序编号，叫做偏移量（offset），这个偏移量能够唯一地定位当前分区中的每一条消息。

> 消息发送时都被发送到一个topic，其本质就是多个分区(多个目录)中的某一个，而topic是由一些Partition Logs(分区日志)组成，其组织结构如下图所示：

> 下图中的topic有3个分区，每个分区的偏移量都从0开始，不同分区之间的偏移量都是独立的，不会相互影响。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/49.png)

> 我们可以看到，每个Partition中的消息都是有序的，生产的消息被不断追加到Partition log上，其中的每一个消息都被赋予了一个唯一的offset值。

> 发布到Kafka主题的每条消息包括键值和时间戳。消息到达服务器端的指定分区后，都会分配到一个自增的偏移量。

> 原始的消息内容和分配的偏移量以及其他一些元数据信息最后都会存储到分区日志文件中。消息的键也可以不用设置，这种情况下消息会均衡地分布到不同的分区。

> 1）分区的原因
>> （1）方便在集群中扩展，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个Partition组成，因此整个集群就可以适应任意大小的数据了；
> 
>> （2）可以提高并发，因为可以以Partition为单位读写了。

> 2）分区的原则
> 
>> （1）指定了patition，则直接使用；
> 
>> （2）未指定patition但指定key，通过对key的value进行hash出一个patition
> 
>> （3）patition和key都未指定，使用轮询选出一个patition。

> DefaultPartitioner类源码

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/50.png)

> 3）消息有序性
>> 传统消息系统在服务端保持消息的顺序，如果有多个消费者消费同一个消息队列，服务端会以消费存储的顺序依次发送给消费者。
但由于消息是异步发送给消费者的，消息到达消费者的顺序可能是无序的，这就意味着在并行消费时，传统消息系统无法很好地保证消息被顺序处理。
虽然我们可以设置一个专用的消费者只消费一个队列，以此来解决消息顺序的问题，但是这就使得消费处理无法真正执行。
Kafka比传统消息系统有更强的顺序性保证，它使用主题的分区作为消息处理的并行单元。
Kafka以分区作为最小的粒度，将每个分区分配给消费者组中不同的而且是唯一的消费者，并确保一个分区只属于一个消费者，即这个消费者就是这个分区的唯一读取线程。
那么，只要分区的消息是有序的，消费者处理的消息顺序就有保证。每个主题有多个分区，不同的消费者处理不同的分区，所以Kafka不仅保证了消息的局部有序性，也做到了消费者的负载均衡。

#### 8.3.4 副本（Replication）

> 同一个partition可能会有多个replication（对应 server.properties 配置中的 default.replication.factor=N）。没有replication的情况下，一旦broker 宕机，其上所有 patition 的数据都不可被消费，同时producer也不能再将数据存于其上的patition。引入replication之后，同一个partition可能会有多个replication，而这时需要在这些replication之间选出一个leader，producer和consumer只与这个leader交互，其它replication作为follower从leader 中复制数据。

### 8.4 Broker保存信息

#### 8.4.1 存储方式

> 物理上把topic分成一个或多个patition（对应 server.properties 中的num.partitions=3配置），每个patition物理上对应一个文件夹（该文件夹存储该patition的所有数据segment文件,包括消息文件.log和索引文件.index），如下：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/51.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/52.png)

#### 8.4.2 存储策略

> 无论消息是否被消费，kafka都会保留所有消息。有两种策略可以删除旧数据：
>> 1）基于时间：log.retention.hours=168
> 
>> 2）基于大小：log.retention.bytes=1073741824

> 需要注意的是，因为Kafka读取特定消息的时间复杂度为O(1)，即与文件大小无关，所以这里删除过期文件与提高 Kafka 性能无关。

#### 8.4.3 Zookeeper存储结构

> 元数据存储在ZK

> 注意:新版本中逐渐的将部分数据存在Kafka的自己的主题中!如__consumer_offsets

> 目的是为了减少Kafka对于三方系统的依赖(侧面看出kafka的野心很大)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/53.png)

### 8.5 Kafka消费过程分析

#### 8.5.1 消费方式

> 消息由生产者发布到Kafka集群后，会被消费者消费。

> 消息的消费模型有两种：推送模型（push）和拉取模型（pull）。

> 基于推送模型（push）的消息系统，由消息代理记录消费者的消费状态。消息代理在将消息推送到消费者后，标记这条消息为已消费，但这种方式无法很好地保证消息被处理。比如，消息代理把消息发送出去后，当消费进程挂掉或者由于网络原因没有收到这条消息时，就有可能造成消息丢失（因为消息代理已经把这条消息标记为已消费了，但实际上这条消息并没有被实际处理）。如果要保证消息被处理，消息代理发送完消息后，要设置状态为“已发送”，只有收到消费者的确认请求后才更新为“已消费”，这就需要消息代理中记录所有的消费状态，这种做法显然是不可取的。

> `Kafka采用拉取模型，由消费者自己记录消费状态`，每个消费者互相独立地顺序读取每个分区的消息。

> 如下图所示，有两个消费者（不同消费者组）拉取同一个主题的消息，消费者A的消费进度是3，消费者B的消费进度是6。

> 消费者拉取的最大上限通过`最高水位（watermark）`控制，生产者最新写入的消息如果还没有达到备份数量，对消费者是不可见的。

> `这种由消费者控制偏移量的优点是：消费者可以按照任意的顺序消费消息`。比如，消费者可以重置到旧的偏移量，重新处理之前已经消费过的消息；或者直接跳到最近的位置，从当前的时刻开始消费。

> （从0.8.2开始同时支持将offset存于Zookeeper中和专用的Kafka Topic中__consumer_offsets(默认)或者存到其他的文件系统中,或者存到Redis）。

> 在一些消息系统中，消息代理会在消息被消费之后立即删除消息。如果有不同类型的消费者订阅同一个主题，消息代理可能需要冗余地存储同一消息；或者等所有消费者都消费完才删除，这就需要消息代理跟踪每个消费者的消费状态，这种设计很大程度上限制了消息系统的整体吞吐量和处理延迟。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/54.png)

> `Kafka的做法是生产者发布的所有消息会一致保存在Kafka集群中，不管消息有没有被消费。用户可以通过设置保留时间或文件大小来清理过期的数据`，比如，设置保留策略为两天。那么，在消息发布之后，它可以被不同的消费者消费，在两天之后，过期的消息就会自动清理掉。
>> 1）基于时间：log.retention.hours=168   # 24*7
> 
>> 2）基于大小：log.retention.bytes=1073741824

#### 8.5.2 消费流程

> Consumer连接指定的Topic partition所在leader broker，采用pull方式从kafkalogs中获取消息。对于不同的消费模式，会将offset保存在不同的地方

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/55.png)

#### 8.5.3 消费模式

> `自动提交oﬀset值`
```
//以下两行配置设置消费者自动提交offset
props.put("enable.auto.commit", "true");
props.put("auto.commit.interval.ms",  "1000");
```

> `手动提交oﬀset`<br/>
> 如果Consumer在获取数据后，需要加入处理，数据完毕后才确认oﬀset，需要程序来控制oﬀset的确认？ 关闭自动提交确认选项
```
//以下两行配置和代码设置消费者手动提交offset
props.put("enable.auto.commit",  "false");
kafkaConsumer.commitAsync();
```

> `完成处理每个分区中的记录后提交偏移量`
>> 在某些情况下，可能希望通过明确指定偏移量来更好地控制已提交的记录。
> 
>> 在下面的示例中，我们在完成处理每个分区中的记录后提交偏移量。

```
try {
    while(running) {
        ConsumerRecords<String, String> records = consumer.poll(Long.MAX_VALUE);
        for (TopicPartition partition : records.partitions()) {
            List<ConsumerRecord<String, String>> partitionRecords = records.records(partition);
            for (ConsumerRecord<String, String> record : partitionRecords) {
                System.out.println(record.offset() + ": " + record.value());
            }
            long lastOffset = partitionRecords.get(partitionRecords.size() -1).offset();
            consumer.commitSync(Collections.singletonMap(partition, new OffsetAndMetadata(lastOffset + 1)));
        }
    }
}
```

`注意事项`：
> 提交的偏移量应始终是应用程序将读取的下一条消息的偏移量。<br/>
> 因此，在调用commitSync（偏移量）时，应该 在最后处理的消息的偏移量中添加一个

> `使用消费者消费指定分区的数据`
```
//consumer.subscribe(Arrays.asList("foo"));
//手动指定 消费指定分区的数据---start
String topic = "foo";
TopicPartition partition0 = new TopicPartition(topic, 0);
TopicPartition partition1 = new TopicPartition(topic, 1);
consumer.assign(Arrays.asList(partition0, partition1));
//手动指定 消费指定分区的数据---end
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(100);
    for (ConsumerRecord<String, String> record : records){
        System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
    }
}
```

`注意事项：`
> 主题与分区订阅只能二选一

#### 8.5.4 消费者组

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/56.png)

> 消费者是以consumer group消费者组的方式工作，由一个或者多个消费者组成一个组，共同消费一个topic。

> 每个分区在同一时间只能由group中的一个消费者读取，但是多个group可以同时消费这个partition。

> 在图中，有一个由三个消费者组成的group，有一个消费者读取主题中的两个分区，另外两个分别读取一个分区。

> 某个消费者读取某个分区，也可以叫做某个消费者是某个分区的拥有者。

> 在这种情况下，消费者可以通过水平扩展的方式同时读取大量的消息。

> 另外，如果一个消费者失败了，那么其他的group成员会自动负载均衡读取之前失败的消费者读取的分区。

## 9 kafka监控及运维_kafkaManager

> 在开发工作中，消费在Kafka集群中消息，数据变化是我们关注的问题，当业务不复杂时，我们可以使用Kafka 命令工具轻松完成我们的工作。<br/>
> 随着业务复杂性的增加，那么我们使用Kafka提供命令工具，已经感到无能为力，那么Kafka监控系统目前尤为重要。

> kafkaManager雅虎开源的一个工具，可直接使用资料中编译好的

> 上传解压

```
kafka-manager-1.3.3.7.tar.gz
tar -zxvf kafka-manager-1.3.3.7.tar.gz
```

> 更改application.conf

```
vim /export/servers/kafka-manager-1.3.3.7/conf/application.conf
kafka-manager.zkhosts="node1:2181,node2:2181,node3:2181"
```

> 添加执行权限

```
cd /export/servers/kafka-manager-1.3.3.7/bin
chmod u+x kafka-manager
```

> 启动

```
./kafka-manager  -Dconfig.file=/export/servers/kafka-manager-1.3.3.7/conf/application.conf -Dhttp.port=8070 &
```
```
jps
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/58.png)

> web页面访问：

```
http://node1:8070
```

> 添加kafka集群

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/59.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/60.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/61.png)

> 浏览kafka的概况

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/62.png)

> 创建主题

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/63.png)

## 10 KafkaTool

> kafkatool是一款客户端 Kafka 可视化工具,可以直接安装在window上! 也可以装在其他系统

> [下载地址](https://www.kafkatool.com/download.html)

```
https://www.kafkatool.com/download.html
```

> 安装：傻瓜式

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/64.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/65.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/66.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/67.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kafka/68.png)

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

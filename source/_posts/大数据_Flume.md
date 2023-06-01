---
title: Flume 流式数据采集
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(58).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(58).jpg
tags:
  - 大数据
  - Flume
category:
  - - 编程
    - 大数据
 
date: 2020-03-14 19:11:30
---

Flume是一个`分布式`的、`可靠`的、`可用`的服务，用于`高效地收集、聚合和移动大量的日志数据`。

它有一个简单而灵活的`基于流数据流的架构`。

它具有`鲁棒性`和`容错性`，具有可调的`可靠性机制`和多种`故障转移和恢复机制`。

它使用一个简单的可扩展数据模型，允许`在线分析应用程序`。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 Flume概述

> 官网：http://flume.apache.org/

### 1.1 Flume介绍

> Flume 是 Cloudera 提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的工具。基于流式架构，容错性强，也很灵活简单。

### 1.2 Flume核心流程

> Flume 的核心是把数据从source数据源收集过来，再将收集到的数据送到指定的sink目的地/下沉地。为了保证输送的过程一定成功，在送到目的地sink之前，会先使用channel通道缓存数据,待数据真正到达目的地sink后，flume 再删除自己缓存的数据。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flume/1.png)

> Flume 支持定制各类数据发送方，用于收集各类型数据；同时，Flume 支持定制各种数据接受方，用于最终存储数据。也就是支持各种数据源和目的地

> 一般的采集需求，通过对 flume 的简单配置即可实现。针对特殊场景也具备良好的自定义扩展能力。因此，flume 可以适用于大部分的日常数据采集场景。

### 1.3 Flume角色详解

> Flume 系统中核心的角色是agent，agent 本身是一个 Java 进程，一般运行在日志收集节点。

#### 1.3.1 Agent

> 每一个 Agent 相当于一个数据传递员，内部有三个组件：`Source`、`Channel`、`Sink`

#### 1.3.2 Source

> 数据源/采集源，用于跟数据源对接，以获取数据；同时Source会将收集到的数据流传输到Channel，常用的Source有:
>> 1)`exec`：可通过tail -f命令去tail一个文件，然后实时同步日志到sink
> 
>> 2)`spooldir`：可监听一个目录，同步目录中的新文件到sink,被同步完的文件可被立即删除或被打上标记。适合用于同步新文件，但不适合对实时追加日志的文件进行监听并同步。
> 
>> 3)`taildir`：可实时监控一批文件，并记录每个文件最新消费位置，agent进程重启后不会有重复消费的问题
> 
>> 4)`支持自定义`

#### 1.3.3 Channel

> Channel是Agent 内部的数据传输通道，用于从 source 将数据传递到 sink；用于桥接Sources和Sinks，类似于一个队列/缓存。Channel分为:
>> 1)`Memory Channel是基于内存的,速度快`
> 
>> 2)File Channel是基于文件的,速度慢,因为会将所有事件写到磁盘,但数据更安全

#### 1.3.4 Sink

> Sink是下沉地/目的地，采集数据的传送目的地，用于从Channel收集数据，将数据写到目标源,Sink分为:
>> 1)可以是`HDFS`、`HBase`、`Kafka`等
> 
>> 2)也可以是下一个 FlumeAgent的Source
> 
>> 3)支持自定义

#### 1.3.5 Event

> 在整个数据的传输的过程中，数据的流动用是Event表示从 source，流向 channel，再到 sink，是Flume 内部数据传输的最基本单元,本质就是一个字节数组。

> Event 将传输的数据进行封装并可携带 headers(头信息)信息。

### 1.4 Flume复杂流程

> http://flume.apache.org/releases/content/1.9.0/FlumeUserGuide.html

> Flume的一般流程是这样的:
>> source监控某个文件或数据流，数据源产生新的数据，拿到该数据后，将数据封装在一个Event中，并发送到channel后提交，channel队列先进先出，sink去channel队列中拉取数据，然后写入到HDFS/Kafka/或者其他的数据源,甚至是下一个Agent的Source。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flume/2.png)

> 但是也支持如下的复杂流程:
>
>> `串联`
>> ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flume/3.png)    
>
>> `并联`
>> ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flume/4.png)
> 
>> `多sink`
>> ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flume/5.png)


## 2 Flume入门案例

### 2.1 模拟实时数据

> 1.准备一个数据生成脚本

```
cd /export/servers/tmp/log/
vim logGenerate.sh
for((i=0;i<=100;i++));
do echo "file-flume-kafka-test-"+$i>>/export/servers/tmp/log/test.log;
sleep 1
done
```

> 2.赋予执行权限

```
chmod 755 logGenerate.sh
```

> 3.执行

```
./logGenerate.sh
```

> 4.测试

```
tail -f /export/servers/tmp/log/test.log
```

### 2.2 编写配置文件

```
cd /export/servers/flume-1.6.0-cdh5.14.0-bin/conf
vim file-flume-kafka.conf
```

```
client.sources = f1
client.channels = ch1
client.sinks = s1

#配置source
client.sources.f1.type=exec                                                                                                           
client.sources.f1.command=tail -F /export/servers/tmp/log/test.log                                                                                 
client.sources.f1.channels=ch1

#channel采用内存模式
client.channels.ch1.type = memory
client.channels.ch1.capacity = 1000
client.channels.ch1.transactionCapacity = 1000

#channel采用文件模式
#client.channels.ch1.type = file
#client.channels.ch1.capacity = 1000000
#client.channels.ch1.write-timeout = 1
#client.channels.ch1.transactionCapacity = 612000
#client.channels.ch1.maxFileSize = 2146435071
#client.channels.ch1.minimumRequiredSpace = 524288000
#client.channels.ch1.dataDirs = /tmp/dataDirs
#client.channels.ch1.checkpointDir = /tmp/checkpoint

#sinks
client.sinks.s1.channel = ch1
#写入kafka的类型
client.sinks.s1.type = org.apache.flume.sink.kafka.KafkaSink
#kafka topic
client.sinks.s1.kafka.topic = flume_kafka
#broker地址和端口号
client.sinks.s1.kafka.bootstrap.servers = node01:9092
#序列化类型
client.sinks.s1.serializer.class=kafka.serializer.StringEncoder
#ack机制
client.sinks.s1.kafka.producer.acks = 1
```

### 2.3 测试

> 1.启动ZK和Kafka并准备主题
>> 启动

```
zkServer.sh start
cd /export/servers/kafka_2.11-1.0.0/
bin/kafka-server-start.sh config/server.properties >/dev/null 2>& 1 &
```
>> 查看主题

```
bin/kafka-topics.sh --zookeeper node01:2181 --list
```

>> 创建主题

```
bin/kafka-topics.sh --zookeeper node01:2181 --create --partitions 1 --replication-factor 1 --topic flume_kafka
```

>> 启动消费者

```
bin/kafka-console-consumer.sh --bootstrap-server node01:9092 --topic flume_kafka
```

> 2.启动Flume

```
cd /export/servers/flume-1.6.0-cdh5.14.0-bin
bin/flume-ng agent -c conf -f conf/file-flume-kafka.conf -n client -Dflume.root.logger=INFO,console
```

> 3.执行数据模拟脚本

```
cd /export/servers/tmp/log/
./logGenerate.sh
```

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
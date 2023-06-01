---
title: Flink 数据流上的有状态计算
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(55).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(55).jpg
tags:
  - 大数据
  - Flink
category:
  - - 编程
    - 大数据
 
date: 2020-02-03 22:12:42
---

`Apache Flink`是一个`框架和分布式处理引擎`，用于在`无界和有界数据流上进行有状态计算`。

`Flink`被设计成可以在所有常见的集群环境中运行，以内存速度和任何规模执行计算。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 Flink概述

### 1.1 实时即未来

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/1.png)

> 如今的我们正生活在新一次的信息革命浪潮中，5G、物联网、智慧城市、工业4.0、新基建……等新名词层出不穷，唯一不变的就是变化！对于我们所学习的大数据来说更是这样：数据产生的越来越快、数据量越来越大，数据的来源越来越千变万化，数据中隐藏的价值规律更是越来越被重视！数字化时代的未来正在被我们创造！

> 历史的发展从来不会一帆风顺，随着大数据时代的发展，海量数据和多种业务的实时处理需求激增，比如：实时监控报警系统、实时风控系统、实时推荐系统等，传统的批处理方式和早期的流式处理框架因其自身的局限性，难以在延迟性、吞吐量、容错能力，以及使用便捷性等方面满足业务日益苛刻的要求。在这种形势下，Flink 以其独特的天然流式计算特性和更为先进的架构设计，极大地改善了以前的流式处理框架所存在的问题。

> 扩展阅读:为什么说流处理即未来？

```
https://news.qudong.com/article/562521.shtml
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/2.png)

### 1.2 一切从Apache开始

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/3.png)

> Flink 诞生于欧洲的一个大数据研究项目 StratoSphere。该项目是柏林工业大学的一个研究性项目。早期， Flink 是做 Batch 计算的，但是在 2014 年， StratoSphere 里面的核心成员孵化出 Flink，同年将 Flink 捐赠 Apache，并在后来成为 Apache 的顶级大数据项目，同时 `Flink 计算的主流方向被定位为 Streaming`， 即用流式计算来做所有大数据的计算，这就是 Flink 技术诞生的背景。

> 2014 年 Flink 作为主攻流计算的大数据引擎开始在开源大数据行业内崭露头角。区别于 Storm、Spark Streaming 以及其他流式计算引擎的是：它不仅是一个高吞吐、低延迟的计算引擎，同时还提供很多高级的功能。比如它提供了有状态的计算，支持状态管理，支持强一致性的数据语义以及支持 基于Event Time的WaterMark对延迟或乱序的数据进行处理等

### 1.3 富二代Flink

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/4.png)

> https://blog.csdn.net/dQCFKyQDXYm3F8rB0/article/details/86117374

> 随着人工智能时代的降临，数据量的爆发，在典型的大数据的业务场景下数据业务最通用的做法是：选用批处理的技术处理全量数据，采用流式计算处理实时增量数据。在绝大多数的业务场景之下，用户的业务逻辑在批处理和流处理之中往往是相同的。但是，用户用于批处理和流处理的两套计算引擎是不同的。因此，用户通常需要写两套代码。毫无疑问，这带来了一些额外的负担和成本。阿里巴巴的商品数据处理就经常需要面对增量和全量两套不同的业务流程问题，所以阿里就在想，我们能不能有一套统一的大数据引擎技术，用户只需要根据自己的业务逻辑开发一套代码。这样在各种不同的场景下，不管是全量数据还是增量数据，亦或者实时处理，一套方案即可全部支持，这就是阿里选择 Flink 的背景和初衷。

> 2015 年阿里巴巴开始使用 Flink 并持续贡献社区(阿里内部还基于Flink做了一套Blink)，2019年1月8日，阿里巴巴以 9000 万欧元(7亿元人民币)收购了创业公司 Data Artisans。从此Flink开始了新一轮的乘风破浪！

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/5.png)

### 1.4 Flink官方介绍

> 官网地址:

[https://flink.apache.org/](https://flink.apache.org/)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/6.png)

### 1.5 Flink组件栈

> 一个计算框架要有长远的发展，必须打造一个完整的 Stack。只有上层有了具体的应用，并能很好的发挥计算框架本身的优势，那么这个计算框架才能吸引更多的资源，才会更快的进步。所以 Flink 也在努力构建自己的 Stack。

> Flink分层的组件栈如下图所示：每一层所包含的组件都提供了特定的抽象，用来服务于上层组件。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/7.png)

> 各层详细介绍：
>> `物理部署层`：Flink 支持本地运行、能在独立集群或者在被 YARN 管理的集群上运行， 也能部署在云上，该层主要涉及Flink的部署模式，目前Flink支持多种部署模式：本地、集群(Standalone、YARN)、云(GCE/EC2)、Kubenetes。Flink能够通过该层能够支持不同平台的部署，用户可以根据需要选择使用对应的部署模式。
> 
>> `Runtime核心层`：Runtime层提供了支持Flink计算的全部核心实现，为上层API层提供基础服务，该层主要负责对上层不同接口提供基础服务，也是Flink分布式计算框架的核心实现层，支持分布式Stream作业的执行、JobGraph到ExecutionGraph的映射转换、任务调度等。将DataSteam和DataSet转成统一的可执行的Task Operator，达到在流式引擎下同时处理批量计算和流式计算的目的。
> 
>> `API&Libraries层`：Flink 首先支持了 Scala 和 Java 的 API，Python 也正在测试中。DataStream、DataSet、Table、SQL API，作为分布式数据处理框架，Flink同时提供了支撑计算和批计算的接口，两者都提供给用户丰富的数据处理高级API，例如Map、FlatMap操作等，也提供比较低级的Process Function API，用户可以直接操作状态和时间等底层数据。
> 
>> `扩展库`：Flink 还包括用于复杂事件处理的CEP，机器学习库FlinkML，图处理库Gelly等。Table 是一种接口化的 SQL 支持，也就是 API 支持(DSL)，而不是文本化的SQL 解析和执行。

### 1.6 Flink基石

> Flink之所以能这么流行，离不开它最重要的四个基石：`Checkpoint`、`State`、`Time`、`Window`。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/8.png)

> `Checkpoint`
>> 这是Flink最重要的一个特性。<br/>
Flink基于Chandy-Lamport算法实现了一个分布式的一致性的快照，从而提供了一致性的语义。<br/>
Chandy-Lamport算法实际上在1985年的时候已经被提出来，但并没有被很广泛的应用，而Flink则把这个算法发扬光大了。<br/>
Spark最近在实现Continue streaming，Continue streaming的目的是为了降低处理的延时，其也需要提供这种一致性的语义，最终也采用了Chandy-Lamport这个算法，说明Chandy-Lamport算法在业界得到了一定的肯定。<br/>
https://zhuanlan.zhihu.com/p/53482103

> `State`
>> 提供了一致性的语义之后，Flink为了让用户在编程时能够更轻松、更容易地去管理状态，还提供了一套非常简单明了的State API，包括里面的有ValueState、ListState、MapState，近期添加了BroadcastState，使用State API能够自动享受到这种一致性的语义。

> `Time`
>> 除此之外，Flink还实现了Watermark的机制，能够支持基于事件的时间的处理，能够容忍迟到/乱序的数据。

> `Window`
>> 另外流计算中一般在对流数据进行操作之前都会先进行开窗，即基于一个什么样的窗口上做这个计算。Flink提供了开箱即用的各种窗口，比如滑动窗口、滚动窗口、会话窗口以及非常灵活的自定义的窗口。

### 1.7 Flink用武之地

[http://www.liaojiayi.com/flink-IoT/](http://www.liaojiayi.com/flink-IoT/)

[https://flink.apache.org/zh/usecases.html](https://flink.apache.org/zh/usecases.html)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/9.png)

> 从很多公司的应用案例发现，其实Flink主要用在如下三大场景：

#### 1.7.1 Event-driven Applications【事件驱动】

> 事件驱动型应用是一类具有状态的应用，它从一个或多个事件流提取数据，并根据到来的事件触发计算、状态更新或其他外部动作。

> 事件驱动型应用是在计算存储分离的传统应用基础上进化而来。

> 在传统架构中，应用需要读写远程事务型数据库。

> 相反，事件驱动型应用是基于状态化流处理来完成。在该设计中，数据和计算不会分离，应用只需访问本地(内存或磁盘)即可获取数据。

> 系统容错性的实现依赖于定期向远程持久化存储写入 checkpoint。下图描述了传统应用和事件驱动型应用架构的区别。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/10.png)

> 从某种程度上来说，所有的实时的数据处理或者是流式数据处理都应该是属于Data Driven，流计算本质上是Data Driven 计算。应用较多的如风控系统，当风控系统需要处理各种各样复杂的规则时，Data Driven 就会把处理的规则和逻辑写入到Datastream 的API 或者是ProcessFunction 的API 中，然后将逻辑抽象到整个Flink 引擎，当外面的数据流或者是事件进入就会触发相应的规则，这就是Data Driven 的原理。在触发某些规则后，Data Driven 会进行处理或者是进行预警，这些预警会发到下游产生业务通知，这是Data Driven 的应用场景，Data Driven 在应用上更多应用于复杂事件的处理。

> 典型实例：
>> 欺诈检测(Fraud detection)
> 
>> 异常检测(Anomaly detection)
> 
>> 基于规则的告警(Rule-based alerting)
> 
>> 业务流程监控(Business process monitoring)
> 
>> Web应用程序(社交网络)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/11.png)

#### 1.7.2 Data Analytics Applications【数据分析】

> 数据分析任务需要从原始数据中提取有价值的信息和指标。

> 如下图所示，Apache Flink 同时支持流式及批量分析应用。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/12.png)

> `Data Analytics Applications`包含Batch analytics(批处理分析)和Streaming analytics(流处理分析)

> `Batch analytics`可以理解为周期性查询：Batch Analytics 就是传统意义上使用类似于Map Reduce、Hive、Spark Batch 等，对作业进行分析、处理、生成离线报表。比如Flink应用凌晨从Recorded Events中读取昨天的数据，然后做周期查询运算，最后将数据写入Database或者HDFS，或者直接将数据生成报表供公司上层领导决策使用。

> `Streaming analytics`可以理解为连续性查询：比如实时展示双十一天猫销售GMV(Gross Merchandise Volume成交总额)，用户下单数据需要实时写入消息队列，Flink 应用源源不断读取数据做实时计算，然后不断的将数据更新至Database或者K-VStore，最后做大屏实时展示。

> 典型实例
>> 电信网络质量监控
> 
>> 移动应用中的产品更新及实验评估分析
> 
>> 消费者技术中的实时数据即席分析
> 
>> 大规模图分析

#### 1.7.3 Data Pipeline Applications【数据管道】

> 什么是数据管道？

> 提取-转换-加载(ETL)是一种在存储系统之间进行数据转换和迁移的常用方法。

> ETL 作业通常会周期性地触发，将数据从事务型数据库拷贝到分析型数据库或数据仓库。

> 数据管道和 ETL 作业的用途相似，都可以转换、丰富数据，并将其从某个存储系统移动到另一个。

> 但数据管道是以持续流模式运行，而非周期性触发。

> 因此数据管道支持从一个不断生成数据的源头读取记录，并将它们以低延迟移动到终点。

> 例如：数据管道可以用来监控文件系统目录中的新文件，并将其数据写入事件日志；另一个应用可能会将事件流物化到数据库或增量构建和优化查询索引。

> 和周期性 ETL 作业相比，持续数据管道可以明显降低将数据移动到目的端的延迟。

> 此外，由于它能够持续消费和发送数据，因此用途更广，支持用例更多。

> 下图描述了周期性ETL作业和持续数据管道的差异。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/13.png)

> `Periodic ETL`：比如每天凌晨周期性的启动一个Flink ETL Job，读取传统数据库中的数据，然后做ETL，最后写入数据库和文件系统。

> `Data Pipeline`：比如启动一个Flink 实时应用，数据源(比如数据库、Kafka)中的数据不断的通过Flink Data Pipeline流入或者追加到数据仓库(数据库或者文件系统)，或者Kafka消息队列。

> `Data Pipeline` 的核心场景类似于数据搬运并在搬运的过程中进行部分数据清洗或者处理，而整个业务架构图的左边是Periodic ETL，它提供了流式ETL 或者实时ETL，能够订阅消息队列的消息并进行处理，清洗完成后实时写入到下游的Database或File system 中。

> 典型实例
>> 电子商务中的持续 ETL(实时数仓)
>> 
>>> 当下游要构建实时数仓时，上游则可能需要实时的Stream ETL。这个过程会进行实时清洗或扩展数据，清洗完成后写入到下游的实时数仓的整个链路中，可保证数据查询的时效性，形成实时数据采集、实时数据处理以及下游的实时Query。
>> 
>> 电子商务中的实时查询索引构建(搜索引擎推荐)
>> 
>>> 搜索引擎这块以淘宝为例，当卖家上线新商品时，后台会实时产生消息流，该消息流经过Flink 系统时会进行数据的处理、扩展。然后将处理及扩展后的数据生成实时索引，写入到搜索引擎中。这样当淘宝卖家上线新商品时，能在秒级或者分钟级实现搜索引擎的搜索。

### 1.8 Flink发展现状

#### 1.8.1 Flink在全球

> Flink近年来逐步被人们所熟知,不仅是因为Flink提供同时支持高吞吐/低延迟和Exactly-Once语义的实时计算能力,同时Flink还提供了基于流式计算引擎处理批量数据的计算能力,真正意义上实现批流统一

> 同时随着阿里对Blink的开源,极大地增强了Flink对批计算领域的支持.众多优秀的特性,使得Flink成为开源大数据处理框架中的一颗新星,随着国内社区的不断推动,越来越多的公司开始选择使用Flink作为实时数据处理技术,在不久的将来,Flink也将会成为企业内部主流的数据处理框架,最终成为下一代大数据处理的标准.

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/14.png)

#### 1.8.2 Flink在中国

> Flink在很多公司的生产环境中得到了使用, 例如: ebay, 腾讯, 阿里, 亚马逊, 华为等

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/15.png)

#### 1.8.3 Flink在阿里

> 阿里自15年起开始调研开源流计算引擎，最终决定基于Flink打造新一代计算引擎，阿里贡献了数百个commiter，并对Flink进行高度定制，并取名为Blink，

> 阿里是Flink SQL的最大贡献者，一半以上的功能都是阿里的工程师开发的，基于Apache Flink在阿里巴巴搭建的平台于2016年正式上线，并从阿里巴巴的搜索和推荐这两大场景开始实现。

> 2019年Flink的母公司被阿里7亿元全资收购，阿里一直致力于Flink在国内的推广使用，目前阿里巴巴所有的业务，包括阿里巴巴所有子公司都采用了基于Flink搭建的实时计算平台。

> 同时Flink计算平台运行在开源的Hadoop集群之上，采用Hadoop的YARN做为资源管理调度，以 HDFS作为数据存储。因此，Flink可以和开源大数据软件Hadoop无缝对接。

> 目前，这套基于Flink搭建的实时计算平台不仅服务于阿里巴巴集团内部，而且通过阿里云的云产品API向整个开发者生态提供基于Flink的云产品支持。

> 主要包含四个模块：`实时监控`、`实时报表`、`流数据分析`和`实时仓库`。
>> `实时监控`：
>>> 用户行为预警、app crash 预警、服务器攻击预警
>>> 对用户行为或者相关事件进行实时监测和分析，基于风控规则进行预警、复杂事件处理
>>
>> `实时报表`：
>>> 双11、双12等活动直播大屏
>>> 对外数据产品：生意参谋等
>>> 数据化运营
>> 
>> `流数据分析`：
>>> 实时计算相关指标反馈及时调整决策
>>> 内容投放、无线智能推送、实时个性化推荐等
>>
>> `实时仓库/ETL`：
>>> 数据实时清洗、归并、结构化
>>> 数仓的补充和优化

> Flink在阿里巴巴的大规模应用表现如何？
>> `规模`：一个系统是否成熟，规模是重要指标，Flink最初上线阿里巴巴只有数百台服务器，目前规模已达上万台，此等规模在全球范围内也是屈指可数；
>> `状态数据`：基于Flink，内部积累起来的状态数据已经是PB级别规模；
>> `Events`：如今每天在Flink的计算平台上，处理的数据已经超过十万亿条;
>> `TPS`：在峰值期间可以承担每秒超过17亿次的访问，最典型的应用场景是阿里巴巴双11大屏；

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/16.png)

#### 1.8.4 Flink在腾讯

[https://blog.csdn.net/qianshangding0708/article/details/91469978](https://blog.csdn.net/qianshangding0708/article/details/91469978)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/17.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/18.png)

#### 1.8.5 Flink在美团

[http://ju.outofmemory.cn/entry/367345](http://ju.outofmemory.cn/entry/367345)

[https://tech.meituan.com/2018/10/18/meishi-data-flink.html](https://tech.meituan.com/2018/10/18/meishi-data-flink.html)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/19.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/20.png)

### 1.9 为什么选择Flink

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/21.png)

> `主要原因`
>> 1.Flink 具备统一的框架处理有界和无界两种数据流的能力
> 
>> 2.部署灵活，Flink 底层支持多种资源调度器，包括Yarn、Kubernetes 等。Flink 自身带的Standalone 的调度器，在部署上也十分灵活。
> 
>> 3.极高的可伸缩性，可伸缩性对于分布式系统十分重要，阿里巴巴双11大屏采用Flink 处理海量数据，使用过程中测得Flink 峰值可达17 亿条/秒。
> 
>> 4.极致的流式处理性能。Flink 相对于Storm 最大的特点是将状态语义完全抽象到框架中，支持本地状态读取，避免了大量网络IO，可以极大提升状态存取的性能。

`其他更多的原因:`
> `1.同时支持高吞吐、低延迟、高性能`
>> Flink 是目前开源社区中唯一一套集高吞吐、低延迟、高性能三者于一身的分布式流式数据处理框架。
>> Spark 只能兼顾高吞吐和高性能特性，无法做到低延迟保障,因为Spark是用批处理来做流处理
>> Storm 只能支持低延时和高性能特性，无法满足高吞吐的要求
>> 下图显示了 Apache Flink 与 Apache Storm 在完成流数据清洗的分布式任务的性能对比。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/22.png)

> `2.支持事件时间(Event Time)概念`
>> 在流式计算领域中，窗口计算的地位举足轻重，但目前大多数框架窗口计算采用的都是系统时间(Process Time)，也就是事件传输到计算框架处理时，系统主机的当前时间。
>> Flink 能够支持基于事件时间(Event Time)语义进行窗口计算
>> 这种基于事件驱动的机制使得事件即使乱序到达甚至延迟到达，流系统也能够计算出精确的结果，保持了事件原本产生时的时序性，尽可能避免网络传输或硬件系统的影响。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/23.png)

> `3.支持有状态计算`
>> Flink1.4开始支持有状态计算
>> 所谓状态就是在流式计算过程中将算子的中间结果保存在内存或者文件系统中，等下一个事件进入算子后可以从之前的状态中获取中间结果，计算当前的结果，从而无须每次都基于全部的原始数据来统计结果，极大的提升了系统性能，状态化意味着应用可以维护随着时间推移已经产生的数据聚合

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/24.png)

> `4.支持高度灵活的窗口(Window)操作`
>> Flink 将窗口划分为基于 Time 、Count 、Session、以及Data-Driven等类型的窗口操作，窗口可以用灵活的触发条件定制化来达到对复杂的流传输模式的支持，用户可以定义不同的窗口触发机制来满足不同的需求

> `5.基于轻量级分布式快照(Snapshot/Checkpoints)的容错机制`
>> Flink 能够分布运行在上千个节点上，通过基于分布式快照技术的Checkpoints，将执行过程中的状态信息进行持久化存储，一旦任务出现异常停止，Flink 能够从 Checkpoints 中进行任务的自动恢复，以确保数据处理过程中的一致性
>> Flink 的容错能力是轻量级的，允许系统保持高并发，同时在相同时间内提供强一致性保证。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/25.png)

> `6.基于 JVM 实现的独立的内存管理`
>> Flink 实现了自身管理内存的机制，通过使用散列，索引，缓存和排序有效地进行内存管理，通过序列化/反序列化机制将所有的数据对象转换成二进制在内存中存储，降低数据存储大小的同时，更加有效的利用空间。使其独立于 Java 的默认垃圾收集器，尽可能减少 JVM GC 对系统的影响。


> `7.SavePoints 保存点`
>> 对于 7 * 24 小时运行的流式应用，数据源源不断的流入，在一段时间内应用的终止有可能导致数据的丢失或者计算结果的不准确。
>> 比如集群版本的升级，停机运维操作等。
>> 值得一提的是，Flink 通过SavePoints 技术将任务执行的快照保存在存储介质上，当任务重启的时候，可以从事先保存的 SavePoints 恢复原有的计算状态，使得任务继续按照停机之前的状态运行。
>> Flink 保存点提供了一个状态化的版本机制，使得能以无丢失状态和最短停机时间的方式更新应用或者回退历史数据。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/26.png)

> `8.灵活的部署方式，支持大规模集群`
>> Flink 被设计成能用上千个点在大规模集群上运行
>> 除了支持独立集群部署外，Flink 还支持 YARN 和Mesos 方式部署。

> `9.Flink 的程序内在是并行和分布式的`
>> 数据流可以被分区成 stream partitions，
>> operators 被划分为operator subtasks;
>> 这些 subtasks 在不同的机器或容器中分不同的线程独立运行；
>> operator subtasks 的数量就是operator的并行计算数，不同的 operator 阶段可能有不同的并行数；
>> 如下图所示，source operator 的并行数为 2，但最后的 sink operator 为1；

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/27.png)

> `10.丰富的库`
>> Flink 拥有丰富的库来进行机器学习，图形处理，关系数据处理等。

### 1.10 大数据框架发展史

> 这几年大数据的飞速发展，出现了很多热门的开源社区，其中著名的有 `Hadoop`、`Storm`，以及后来的 `Spark`，他们都有着各自专注的应用场景。Spark 掀开了内存计算的先河，也以内存为赌注，赢得了内存计算的飞速发展。Spark 的火热或多或少的掩盖了其他分布式计算的系统身影。就像 `Flink`，也就在这个时候默默的发展着。

> 在国外一些社区，有很多人将大数据的计算引擎分成了 4 代，当然，也有很多人不会认同。我们先姑且这么认为和讨论。

> `第1代——Hadoop MapReduce`
>> 首先第一代的计算引擎，无疑就是 Hadoop 承载的 MapReduce。它将计算分为两个阶段，分别为 Map 和 Reduce。对于上层应用来说，就不得不想方设法去拆分算法，甚至于不得不在上层应用实现多个 Job 的串联，以完成一个完整的算法，例如迭代计算。
>>> 批处理
>>> Mapper、Reducer
> 
> `第2代——DAG框架（Tez） + MapReduce`
>> 由于这样的弊端，催生了支持 DAG 框架的产生。因此，支持 DAG 的框架被划分为第二代计算引擎。如 Tez 以及更上层的 Oozie。这里我们不去细究各种 DAG 实现之间的区别，不过对于当时的 Tez 和 Oozie 来说，大多还是批处理的任务。
>>> 批处理
>>> 1个Tez = MR(1) + MR(2) + ... + MR(n)
>>> 相比MR效率有所提升
>>>  ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/28.png)
> 
> `第3代——Spark`
>> 接下来就是以 Spark 为代表的第三代的计算引擎。第三代计算引擎的特点主要是 Job 内部的 DAG 支持（不跨越 Job），以及强调的实时计算。在这里，很多人也会认为第三代计算引擎也能够很好的运行批处理的 Job。
>>> 批处理、流处理、SQL高层API支持
>>> 自带DAG
>>> 内存迭代计算、性能较之前大幅提升
> 
> `第4代——Flink`
>> 随着第三代计算引擎的出现，促进了上层应用快速发展，例如各种迭代计算的性能以及对流计算和 SQL 等的支持。Flink 的诞生就被归在了第四代。这应该主要表现在 Flink 对流计算的支持，以及更一步的实时性上面。当然 Flink 也可以支持 Batch 的任务，以及 DAG 的运算。
>>> 批处理、流处理、SQL高层API支持
>>> 自带DAG
>>> 流式计算性能更高、可靠性更高

### 1.11 流处理 VS 批处理

> `数据的时效性`
>> 日常工作中，我们一般会先把数据存储在`表`，然后对表的数据进行`加工、分析`。既然先存储在表中，那就会涉及到时效性概念。
>> 如果我们处理以年，月为单位的级别的数据处理，进行`统计分析`，`个性化推荐`，那么数据的的最新日期离当前有`几个甚至上月`都没有问题。但是如果我们处理的是`以天为级别`，或者`一小时`甚至`更小粒度`的数据处理，那么就要求数据的时效性更高了。比如：
>>> 对网站的实时监控
>>> 对异常日志的监控
>> 这些场景需要工作人员`立即响应`，这样的场景下，传统的统一收集数据，再存到数据库中，再取出来进行分析就无法满足高时效性的需求了。

> `流式计算和批量计算`
>>  ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/29.png)
>> Batch Analytics，右边是 Streaming Analytics。批量计算: 统一收集数据->存储到DB->对数据进行批量处理，就是传统意义上使用类似于 Map Reduce、Hive、Spark Batch 等，对作业进行分析、处理、生成离线报表
>> Streaming Analytics 流式计算，顾名思义，就是对数据流进行处理，如使用流式分析引擎如 Storm，Flink 实时处理分析数据，应用较多的场景如实时大屏、实时报表。

> `它们的主要区别是：`
>> 与批量计算那样慢慢积累数据不同，流式计算立刻计算，数据持续流动，计算完之后就丢弃。
>> 批量计算是维护一张表，对表进行实施各种计算逻辑。流式计算相反，是必须先定义好计算逻辑，提交到流式计算系统，这个计算作业逻辑在整个运行期间是不可更改的。
>> 计算结果上，批量计算对全部数据进行计算后传输结果，流式计算是每次小批量计算后，结果可以立刻实时化展现。

### 1.12 统一流处理与批处理

> 在大数据处理领域，批处理任务与流处理任务一般被认为是两种不同的任务，一个大数据框架一般会被设计为只能处理其中一种任务：

> MapReduce只支持批处理任务；

> Storm只支持流处理任务；

> Spark Streaming采用micro-batch架构，本质上还是基于Spark批处理对流式数据进行处理

> Flink通过灵活的执行引擎，能够同时支持批处理任务与流处理任务

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/30.png)

> 在执行引擎这一层，流处理系统与批处理系统最大不同在于节点间的数据传输方式：
>> 1.对于一个流处理系统，其节点间数据传输的标准模型是：当一条数据被处理完成后，序列化到缓存中，然后立刻通过网络传输到下一个节点，由下一个节点继续处理
>> 2.对于一个批处理系统，其节点间数据传输的标准模型是：当一条数据被处理完成后，序列化到缓存中，并不会立刻通过网络传输到下一个节点，当缓存写满，就持久化到本地硬盘上，当所有数据都被处理完成后，才开始将处理后的数据通过网络传输到下一个节点
> 
> 这两种数据传输模式是两个极端，对应的是流处理系统对低延迟的要求和批处理系统对高吞吐量的要求

> Flink的执行引擎采用了一种十分灵活的方式，同时支持了这两种数据传输模型：

> Flink以固定的缓存块为单位进行网络数据传输，用户可以通过设置缓存块超时值指定缓存块的传输时机。

> 如果缓存块的超时值为0，则Flink的数据传输方式类似上文所提到流处理系统的标准模型，此时系统可以获得最低的处理延迟

> 如果缓存块的超时值为无限大/-1，则Flink的数据传输方式类似上文所提到批处理系统的标准模型，此时系统可以获得最高的吞吐量

> 同时缓存块的超时值也可以设置为0到无限大之间的任意值。缓存块的超时阈值越小，则Flink流处理执行引擎的数据处理延迟越低，但吞吐量也会降低，反之亦然。通过调整缓存块的超时阈值，用户可根据需求灵活地权衡系统延迟和吞吐量

> 默认情况下，流中的元素并不会一个一个的在网络中传输，而是缓存起来伺机一起发送(taskmanager.network.memory.max网络缓冲区最大内存大小),这样可以避免导致频繁的网络传输,提高吞吐量，但如果数据源输入不够快的话会导致后续的数据处理延迟，所以可以使用env.setBufferTimeout(默认100ms)，来为缓存填入设置一个最大等待时间。等待时间到了之后，即使缓存还未填满，缓存中的数据也会自动发送。

> 总结:
>> Flink以缓存块为单位进行网络数据传输,用户可以设置缓存块超时时间和缓存块大小来控制缓冲块传输时机,从而控制Flink的延迟性和吞吐量

## 2 Flink安装部署

### 2.1 Local本地模式

#### 2.1.1 原理

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/31.png)

> 1.Flink程序由JobClient进行提交

> 2.JobClient将作业提交给JobManager

> 3.JobManager负责协调资源分配和作业执行。资源分配完成后，任务将提交给相应的TaskManager

> 4.TaskManager启动一个线程以开始执行。TaskManager会向JobManager报告状态更改,如开始执行，正在进行或已完成。

> 5.作业执行完成后，结果将发送回客户端(JobClient)

#### 2.1.2 操作

> 1.下载安装包

```
https://archive.apache.org/dist/flink/flink-1.10.0/
```

> 2.上传flink-1.10.0-bin-scala_2.11.tgz到node1的指定目录

> 3.解压

```
tar -zxvf flink-1.10.0-bin-scala_2.11.tgz  -C  /export/servers/
```

> 4.改名

```
mv flink-1.10.0 flink
```

> 5.如果出现权限问题，需要修改权限

```
chown -R root:root flink
```

#### 2.1.3 测试

> 1.启动shell交互式窗口

```
/export/servers/flink/bin/start-scala-shell.sh local
```

> 2.准备文件/root/words.txt

```
vim /root/words.txt
hello me you her
hello me you
hello me
hello
```

> 3.执行如下命令

```
benv.readTextFile("/root/words.txt").flatMap(_.split(" ")).map((_,1)).groupBy(0).sum(1).print()
```

> 4.退出shell

```
:quit
```

> 5.启动Flink本地“集群”

```
/export/servers/flink/bin/start-cluster.sh
```

> 6.使用jps可以查看到下面两个进程
>> `TaskManagerRunner`
>> `StandaloneSessionClusterEntrypoint`

> 7.访问Flink的Web UI

```
http://node1:8081/#/overview
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/32.png)

> slot在Flink里面可以认为是资源组，Flink是通过将任务分成子任务并且将这些子任务分配到slot来并行执行程序。

> 8.执行官方示例

```
/export/servers/flink/bin/flink run /export/servers/flink/examples/batch/WordCount.jar --input /root/words.txt --output /root/out
```

> 9.停止Flink

```
/export/servers/flink/bin/stop-cluster.sh
```

### 2.2 Standalone独立集群模式

#### 2.2.1 原理

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/33.png)

> 1.client客户端提交任务给JobManager

> 2.JobManager负责申请任务运行所需要的资源并管理任务和资源，

> 3.JobManager分发任务给TaskManager执行

> 4.TaskManager定期向JobManager汇报状态

#### 2.2.2 操作

> 1.集群规划:
>> 服务器: node1(Master + Slave): JobManager + TaskManager
>> 服务器: node2(Slave): TaskManager
>> 服务器: node3(Slave): TaskManager

> 2.修改flink-conf.yaml

```
vim /export/servers/flink/conf/flink-conf.yaml
jobmanager.rpc.address: node1
web.submit.enable: true
```

> 2.修改masters

```
vim /export/servers/flink/conf/masters
node1:8081
```

> 3.修改slaves

```
vim /export/servers/flink/conf/slaves
node1
node2
node3
```

> 4.添加HADOOP_CONF_DIR环境变量

```
vim /etc/profile
export HADOOP_CONF_DIR=/export/servers/hadoop-2.7.5/etc/hadoop
```

> 5.分发
```
scp -r /export/servers/flink node2:/export/servers/flink
scp -r /export/servers/flink node3:/export/servers/flink
scp -r /etc/profile node2:/etc/profile
scp -r /etc/profile node3:/etc/profile
```

或

```
for i in {2..3}; do scp -r flink node$i:$PWD; done
```

> 6.source

```
source /etc/profile
```

#### 2.2.3 测试

> 1.启动集群，在node1上执行如下命令

```
/export/servers/flink/bin/start-cluster.sh
```

或者单独启动

```
bin/jobmanager.sh ((start|start-foreground) cluster)|stop|stop-all
bin/taskmanager.sh start|start-foreground|stop|stop-all
```

> 2.访问Flink UI界面或使用jps查看

```
http://node1:8081/#/overview
```

> TaskManager界面：可以查看到当前Flink集群中有多少个TaskManager，每个TaskManager的slots、内存、CPU Core是多少


> 3.执行官方测试案例

```
/export/servers/flink/bin/flink run  /export/servers/flink/examples/batch/WordCount.jar --input hdfs://node1:8020/wordcount/input/words.txt --output hdfs://node1:8020/wordcount/output/result.txt  --parallelism 2
```

### 2.3 Standalone-HA高可用集群模式

#### 2.3.1 原理

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/34.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/35.png)

> 从之前的架构中我们可以很明显的发现 JobManager 有明显的单点问题(SPOF，single point of failure)。JobManager 肩负着任务调度以及资源分配，一旦 JobManager 出现意外，其后果可想而知。

> 在 Zookeeper 的帮助下，一个 Standalone的Flink集群会同时有多个活着的 JobManager，其中只有一个处于工作状态，其他处于 Standby 状态。当工作中的 JobManager 失去连接后(如宕机或 Crash)，Zookeeper 会从 Standby 中选一个新的 JobManager 来接管 Flink 集群。

#### 2.3.2 操作

> 1.集群规划
>> 服务器: node1(Master + Slave): JobManager + TaskManager
>> 服务器: node2(Master + Slave): JobManager + TaskManager
>> 服务器: node3(Slave): TaskManager

> 2.启动ZooKeeper

> 3.启动HDFS

> 4停止集群

```
/export/servers/flink/bin/stop-cluster.sh
```

> 5.修改flink-conf.yaml

```
vim /export/servers/flink/conf/flink-conf.yaml
```

增加如下内容

```
state.backend: filesystem
state.backend.fs.checkpointdir: hdfs://node1:8020/flink-checkpoints
high-availability: zookeeper
high-availability.storageDir: hdfs://node1:8020/flink/ha/
high-availability.zookeeper.quorum: node1:2181,node2:2181,node3:2181
```

> 配置解释

```
#开启HA，使用文件系统作为快照存储
state.backend: filesystem

#启用检查点，可以将快照保存到HDFS
state.backend.fs.checkpointdir: hdfs://node1:8020/flink-checkpoints

#使用zookeeper搭建高可用
high-availability: zookeeper

# 存储JobManager的元数据到HDFS
high-availability.storageDir: hdfs://node1:8020/flink/ha/

# 配置ZK集群地址
high-availability.zookeeper.quorum: node1:2181,node2:2181,node3:2181
```

> 6.修改masters

```
vim /export/servers/flink/conf/masters
node1:8081
node2:8081
```

> 7.同步

```
scp -r /export/servers/flink/conf/flink-conf.yaml node2:/export/servers/flink/conf/
scp -r /export/servers/flink/conf/flink-conf.yaml node3:/export/servers/flink/conf/
scp -r /export/servers/flink/conf/masters node2:/export/servers/flink/conf/
scp -r /export/servers/flink/conf/masters node3:/export/servers/flink/conf/
```

> 8.修改node2上的flink-conf.yaml

```
vim /export/servers/flink/conf/flink-conf.yaml
jobmanager.rpc.address: node2
```

> 9.重新启动Flink集群,node1上执行

```
/export/servers/flink/bin/stop-cluster.sh
/export/servers/flink/bin/start-cluster.sh
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/36.png)

> 10.使用jps命令查看
>> 发现没有Flink相关进程被启动

> 11.查看日志

```
cat /export/servers/flink/log/flink-root-standalonesession-0-node1.log
```

> 发现如下错误

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/37.png)

> 因为在Flink1.8版本后,Flink官方提供的安装包里没有整合HDFS的jar

> 12.下载jar包并在Flink的lib目录下放入该jar包并分发使Flink能够支持对Hadoop的操作

```
https://flink.apache.org/downloads.html
for i in {2..3}; do scp -r flink-shaded-hadoop-2-uber-2.7.5-10.0.jar node$i:$PWD; done
```

> 13.重新启动Flink集群,node1上执行

```
/export/servers/flink/bin/start-cluster.sh
```

> 14.使用jps命令查看,发现三台机器已经ok

```
jps
```

#### 2.3.3 测试

> 1.访问WebUI

```
http://node1:8081/#/job-manager/config
http://node2:8081/#/job-manager/config
```

> 2.执行wc

```
/export/servers/flink/bin/flink run  /export/servers/flink/examples/batch/WordCount.jar
```

> 3.kill掉其中一个master

> 4.重新执行wc,还是可以正常执行

```
/export/servers/flink/bin/flink run  /export/servers/flink/examples/batch/WordCount.jar
```

> 5.停止集群

```
/export/servers/flink/bin/stop-cluster.sh
```

### 2.4 Flink On Yarn 模式

#### 2.4.1 原理

##### 2.4.1.1 为什么使用Flink On Yarn?

> 在实际开发中，使用Flink时，更多的使用方式是Flink On Yarn模式，原因如下：
>> 1.Yarn的资源可以按需使用，提高集群的资源利用率,`资源按需使用分配`
>> 2.Yarn的任务有优先级，根据优先级运行作业，`支持优先级和多种调度策略`
>> 3.基于Yarn调度系统，能够自动化地处理各个角色的 Failover(容错)，`自带容错/高可用`
>>> JobManager 进程和 TaskManager 进程都由 Yarn NodeManager 监控
>>> 如果 JobManager 进程异常退出，则 Yarn ResourceManager 会重新调度 JobManager 到其他机器
>>> 如果 TaskManager 进程异常退出，JobManager 会收到消息并重新向 Yarn ResourceManager 申请资源，重新启动 TaskManager

##### 2.4.1.2 Flink如何和Yarn进行交互?

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/38.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/39.png)

> 1.Client上传jar包和配置文件到HDFS集群上

> 2.Client向Yarn ResourceManager提交任务并申请资源

> 3.ResourceManager分配Container资源并启动ApplicationMaster,然后AppMaster加载Flink的Jar包和配置构建环境,启动JobManager
>> JobManager和ApplicationMaster运行在同一个container上。
>> 一旦他们被成功启动，AppMaster就知道JobManager的地址(AM它自己所在的机器)。
>> 它就会为TaskManager生成一个新的Flink配置文件(他们就可以连接到JobManager)。
>> 这个配置文件也被上传到HDFS上。
>> 此外，AppMaster容器也提供了Flink的web服务接口。
>> YARN所分配的所有端口都是临时端口，这允许用户并行执行多个Flink

> 4.ApplicationMaster向ResourceManager申请工作资源,NodeManager加载Flink的Jar包和配置构建环境并启动TaskManager

> 5.TaskManager启动后向JobManager发送心跳包，并等待JobManager向其分配任务

##### 2.4.1.3 两种方式

###### 2.4.1.3.1 Session会话模式

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/40.png)

> `特点`：需要事先申请资源，启动JobManager和TaskManger

> `优点`：不需要每次递交作业申请资源，而是使用已经申请好的资源，从而提高执行效率

> `缺点`：作业执行完成以后，资源不会被释放，因此一直会占用系统资源

> `应用场景`：适合作业递交比较频繁的场景，小作业比较多的场景

###### 2.4.1.3.2 Job分离模式

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/41.png)

> `特点`：每次递交作业都需要申请一次资源

> `优点`：作业运行完成，资源会立刻被释放，不会一直占用系统资源

> `缺点`：每次递交作业都需要申请资源，会影响执行效率，因为申请资源需要消耗时间

> `应用场景`：适合作业比较少的场景、大作业的场景

#### 2.4.2 操作

> 1.关闭yarn的内存检查

```
vim /export/servers/hadoop-2.7.5/etc/hadoop/yarn-site.xml
```

添加：

```
<!-- 关闭yarn内存检查 -->
<property>
<name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>
<property>
     <name>yarn.nodemanager.vmem-check-enabled</name>
     <value>false</value>
</property>
```

> 说明:
>> 是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true。
>> 在这里面我们需要关闭，因为对于flink使用yarn模式下，很容易内存超标，这个时候yarn会自动杀掉job

> 2.同步

```
scp -r /export/servers/hadoop-2.7.5/etc/hadoop/yarn-site.xml node2:/export/servers/hadoop-2.7.5/etc/hadoop/yarn-site.xml
scp -r /export/servers/hadoop-2.7.5/etc/hadoop/yarn-site.xml node3:/export/servers/hadoop-2.7.5/etc/hadoop/yarn-site.xml
```

> 3.重启yarn

```
/export/servers/hadoop-2.7.5/sbin/stop-yarn.sh
/export/servers/hadoop-2.7.5/sbin/start-yarn.sh
```

#### 2.4.3 测试

##### 2.4.3.1 Session会话模式

> yarn-session.sh(开辟资源) + flink run(提交任务)

> 1.在yarn上启动一个Flink会话，node1上执行以下命令

```
/export/servers/flink/bin/yarn-session.sh -n 2 -tm 800 -s 1 -d
```

> 说明:
>> 申请2个CPU、1600M内存
>>  -n 表示申请2个容器，这里指的就是多少个taskmanager
>>  -tm 表示每个TaskManager的内存大小
>>  -s 表示每个TaskManager的slots数量
>>  -d 表示以后台程序方式运行

> 注意:该警告不用管

```
WARN  org.apache.hadoop.hdfs.DFSClient  - Caught exception
java.lang.InterruptedException
```

> 2.查看UI界面

```
http://node1:8088/cluster
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/42.png)

> 3.使用flink run提交任务：

```
/export/servers/flink/bin/flink run  /export/servers/flink/examples/batch/WordCount.jar
```

> 运行完之后可以继续运行其他的小任务

```
/export/servers/flink/bin/flink run  /export/servers/flink/examples/batch/WordCount.jar
```

> 4.通过上方的ApplicationMaster可以进入Flink的管理界面

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/43.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/44.png)

> 5.关闭yarn-session：

```
yarn application -kill application_1599402747874_0001
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/45.png)

```
rm -rf /tmp/.yarn-properties-root
```

##### 2.4.3.2 Job分离模式

> 1.直接提交job

```
/export/servers/flink/bin/flink run -m yarn-cluster -yjm 1024 -ytm 1024 /export/servers/flink/examples/batch/WordCount.jar
# -m  jobmanager的地址
# -yjm 1024 指定jobmanager的内存信息
# -ytm 1024 指定taskmanager的内存信息
```

> 2.查看UI界面

```
http://node1:8088/cluster
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/46.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/47.png)

> 3.如果再次提交任务，会在Yarn上再启动一个Flink集群

> 4.注意：
>> 在之前版本中如果使用的是flink on yarn方式，想切换回standalone模式的话，如果报错需要删除：【/tmp/.yarn-properties-root】

```
rm -rf /tmp/.yarn-properties-root
```

#### 2.4.4 参数总结

```
[root@node1 bin]# flink --help
```

## 3 Flink入门案例

### 3.1 API和编程模型

### 3.2 准备工程

#### 3.2.1 pom文件

```xml
<!-- 指定仓库位置，依次为aliyun、apache和cloudera仓库 -->
<repositories>
    <repository>
        <id>aliyun</id>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    </repository>
    <repository>
        <id>apache</id>
        <url>https://repository.apache.org/content/repositories/snapshots/</url>
    </repository>
    <repository>
        <id>cloudera</id>
        <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
    </repository>
</repositories>

<properties>
    <encoding>UTF-8</encoding>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <java.version>1.8</java.version>
    <scala.version>2.11</scala.version>
    <scala.binary.version>2.11</scala.binary.version>
    <flink.version>1.10.0</flink.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-shaded-hadoop-2-uber</artifactId>
        <version>2.7.5-10.0</version>
    </dependency>
    <!-- Flink -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-java</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-java_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-table-common</artifactId>
        <version>${flink.version}</version>
    </dependency>

    <!-- 适用于使用Java编程语言的纯表程序的Table＆SQL API（处于开发初期，不建议使用！）。-->
    <!--<dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-table-api-java</artifactId>
        <version>${flink.version}</version>
    </dependency>-->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-table-api-java-bridge_${scala.version}</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <!-- flink执行计划,这是1.9版本之前的。仍然是推荐的-->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-table-planner_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <!-- blink执行计划-->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-table-planner-blink_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <!-- flink连接器-->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-connector-kafka_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-sql-connector-kafka_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.bahir</groupId>
        <artifactId>flink-connector-redis_2.11</artifactId>
        <version>1.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-connector-filesystem_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-jdbc_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>

    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-csv</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-json</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-parquet_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>

    <!-- flink-cep -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-cep_${scala.version}</artifactId>
        <version>${flink.version}</version>
    </dependency>

    <!-- Apache提供的Collections4组件提供的一些特殊数据结构-->
    <!-- 参考：https://blog.csdn.net/f641385712/article/details/84109098-->
    <!--<dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-collections4</artifactId>
        <version>4.4</version>
    </dependency>

    <dependency>
        <groupId>org.apache.avro</groupId>
        <artifactId>avro</artifactId>
        <version>1.9.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.parquet</groupId>
        <artifactId>parquet-avro</artifactId>
        <version>1.10.0</version>
    </dependency>-->

    <!-- 操作hive所需要的jar包-->
    <!--<dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-connector-hive_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hive</groupId>
        <artifactId>hive-metastore</artifactId>
        <version>2.1.1</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.thrift</groupId>
        <artifactId>libfb303</artifactId>
        <version>0.9.3</version>
        <type>pom</type>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.hive</groupId>
        <artifactId>hive-exec</artifactId>
        <version>2.1.1</version>
        <scope>provided</scope>
    </dependency>-->

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.44</version>
    </dependency>

    <!-- 日志 -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.7</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
        <scope>runtime</scope>
    </dependency>
    <!-- 指定mysql-connector的依赖 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.38</version>
    </dependency>

    <!-- 高性能异步组件：Vertx-->
    <dependency>
        <groupId>io.vertx</groupId>
        <artifactId>vertx-core</artifactId>
        <version>3.9.0</version>
    </dependency>
    <dependency>
        <groupId>io.vertx</groupId>
        <artifactId>vertx-jdbc-client</artifactId>
        <version>3.9.0</version>
    </dependency>
    <dependency>
        <groupId>io.vertx</groupId>
        <artifactId>vertx-redis-client</artifactId>
        <version>3.9.0</version>
    </dependency>
    <!-- 使用布隆过滤器需要导入jar包 -->
    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>28.2-jre</version>
    </dependency>
</dependencies>

<build>
    <sourceDirectory>src/main/java</sourceDirectory>
    <plugins>
        <!-- 编译插件 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.5.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <!--<encoding>${project.build.sourceEncoding}</encoding>-->
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.18.1</version>
            <configuration>
                <useFile>false</useFile>
                <disableXmlReport>true</disableXmlReport>
                <includes>
                    <include>**/*Test.*</include>
                    <include>**/*Suite.*</include>
                </includes>
            </configuration>
        </plugin>
        <!-- 打包插件(会包含所有依赖) -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>2.3</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <filters>
                            <filter>
                                <artifact>*:*</artifact>
                                <excludes>
                                    <!--
                                    zip -d learn_spark.jar META-INF/*.RSA META-INF/*.DSA META-INF/*.SF -->
                                    <exclude>META-INF/*.SF</exclude>
                                    <exclude>META-INF/*.DSA</exclude>
                                    <exclude>META-INF/*.RSA</exclude>
                                </excludes>
                            </filter>
                        </filters>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <!-- 设置jar包的入口类(可选) -->
                                <mainClass></mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

#### 3.2.2 log4j.properties

> log4j.properties

```
log4j.rootLogger=WARN, console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{HH:mm:ss,SSS} %-5p %-60c %x - %m%n
```

### 3.3 Flink初体验

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/48.png)

> 编码步骤
>> 1.准备环境-env
>> 2.准备数据-source
>> 3.处理数据-transformation
>> 4.输出结果-sink
>> 5.触发执行-execute

#### 3.3.1 匿名内部类版

```
package cn.xiaoma.hello;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.operators.Order;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.*;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

/**
 * Author xiaoma
 * Desc Flink批处理初体验-WordCount
 */
public class WordCount1 {
    public static void main(String[] args) throws Exception {
        //1.准备环境-env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.准备数据-source
        DataSource<String> lines = env.fromElements("hadoop spark flink", "hadoop spark", "hadoop");
        //3.处理数据-transformation
        //切割
        /*
            @FunctionalInterface
            public interface FlatMapFunction<T, O> extends Function, Serializable {
                void flatMap(T value, Collector<O> out) throws Exception;
            }
         */
        FlatMapOperator<String, String> words = lines.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                //value就是每一行单词
                String[] words = value.split(" ");
                for (String word : words) {
                    out.collect(word);
                }
            }
        });
        //每个单词记为1 (单词,1)
        /*
            @FunctionalInterface
            public interface MapFunction<T, O> extends Function, Serializable {
                O map(T value) throws Exception;
            }
         */
        MapOperator<String, Tuple2<String, Integer>> wordAndOne = words.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return Tuple2.of(value, 1);
            }
        });

        //分组
        UnsortedGrouping<Tuple2<String, Integer>> grouped = wordAndOne.groupBy(0);

        //聚合
        AggregateOperator<Tuple2<String, Integer>> agg = grouped.sum(1);

        //排序
        SortPartitionOperator<Tuple2<String, Integer>> result = agg.sortPartition(1, Order.DESCENDING).setParallelism(1);

        //4.输出结果-sink
        result.print();

        //5.触发执行-execute
        //env.execute();
    }
}
```

#### 3.3.2 Lambda版本

```
package cn.xiaoma.hello;

import org.apache.flink.api.common.operators.Order;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.*;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

import java.util.Arrays;

/**
 * Author xiaoma
 * Desc Flink批处理初体验-WordCount
 */
public class WordCount2 {
    public static void main(String[] args) throws Exception {
        //1.准备环境-env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.准备数据-source
        DataSource<String> lines = env.fromElements("hadoop spark flink", "hadoop spark", "hadoop");
        //3.处理数据-transformation
        //切割
        FlatMapOperator<String, String> words = lines.flatMap(
                (String line, Collector<String> out) -> Arrays.stream(line.split(" ")).forEach(out::collect)
        ).returns(Types.STRING);

        //每个单词记为1 (单词,1)
        MapOperator<String, Tuple2<String, Integer>> wordAndOne = words.map(
                (word) -> Tuple2.of(word, 1)
        ).returns(Types.TUPLE(Types.STRING, Types.INT));

        //分组
        UnsortedGrouping<Tuple2<String, Integer>> grouped = wordAndOne.groupBy(0);

        //聚合
        AggregateOperator<Tuple2<String, Integer>> agg = grouped.sum(1);

        //排序
        SortPartitionOperator<Tuple2<String, Integer>> result = agg.sortPartition(1, Order.DESCENDING).setParallelism(1);

        //4.输出结果-sink
        result.print();

        //5.触发执行-execute
        //env.execute();
    }
}
```

#### 3.3.3 在Yarn上运行

> 注意:写入HDFS如果存在权限问题:<br/>
> 进行如下设置:

```
hadoop fs -chmod -R 777  /
```

> 并在代码中添加:

```
System.setProperty("HADOOP_USER_NAME", "root")
```

> 代码

```
package cn.itcast.hello;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.operators.Order;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.*;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

/**
 * Author itcast
 * Desc Flink批处理初体验-WordCount
 */
public class WordCount {
    public static void main(String[] args) throws Exception {
        //1.准备环境-env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.准备数据-source
        DataSource<String> lines = env.fromElements("hadoop spark flink", "hadoop spark", "hadoop");
        //3.处理数据-transformation
        //切割
        FlatMapOperator<String, String> words = lines.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String value, Collector<String> out) throws Exception {
                //value就是每一行单词
                String[] words = value.split(" ");
                for (String word : words) {
                    out.collect(word);
                }
            }
        });
        //每个单词记为1 (单词,1)
        MapOperator<String, Tuple2<String, Integer>> wordAndOne = words.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                return Tuple2.of(value, 1);
            }
        });

        //分组
        UnsortedGrouping<Tuple2<String, Integer>> grouped = wordAndOne.groupBy(0);

        //聚合
        AggregateOperator<Tuple2<String, Integer>> agg = grouped.sum(1);

        //排序
        SortPartitionOperator<Tuple2<String, Integer>> result = agg.sortPartition(1, Order.DESCENDING).setParallelism(1);

        //4.输出结果-sink
        //result.print();
        System.setProperty("HADOOP_USER_NAME", "root");
        result.writeAsText("hdfs://node1:8020/wordcount/output");

        //5.触发执行-execute
        env.execute();
    }
}
```

> 打包，改名

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/49.png)

> 上传

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/50.png)

> 执行

```
/export/servers/flink/bin/flink run -m yarn-cluster -yjm 1024 -ytm 1024 -c cn.itcast.hello.WordCount /root/wc.jar
```

> 5.在Web页面可以观察到提交的程序：

```
http://node1:8088/cluster
http://node1:50070/explorer.html#/
或者在Standalone模式下使用web界面提交
```

## 4 Flink原理初探

### 4.1 Flink角色分工

> 在实际生产中，Flink 都是以集群在运行，在运行的过程中包含了两类进程。

> `JobManager`：
>> 它扮演的是集群管理者的角色，负责调度任务、协调 checkpoints、协调故障恢复、收集 Job 的状态信息，并管理 Flink 集群中的从节点 TaskManager。

> `TaskManager`：
>> 实际负责执行计算的 Worker，在其上执行 Flink Job 的一组 Task；TaskManager 还是所在节点的管理员，它负责把该节点上的服务器信息比如内存、磁盘、任务运行情况等向 JobManager 汇报。

> `Client`：
>> 用户在提交编写好的 Flink 工程时，会先创建一个客户端再进行提交，这个客户端就是 Client

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/51.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/52.png)

### 4.2 Flink执行流程

[https://blog.csdn.net/sxiaobei/article/details/80861070](https://blog.csdn.net/sxiaobei/article/details/80861070)

[https://blog.csdn.net/super_wj0820/article/details/90726768](https://blog.csdn.net/super_wj0820/article/details/90726768)

[https://ci.apache.org/projects/flink/flink-docs-release-1.11/ops/deployment/yarn_setup.html](https://ci.apache.org/projects/flink/flink-docs-release-1.11/ops/deployment/yarn_setup.html)

#### 4.2.1 Standalone版

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/53.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/54.png)

#### 4.2.2 On Yarn版

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/55.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/56.png)

> 1.Client向HDFS上传Flink的Jar包和配置

> 2.Client向Yarn ResourceManager提交任务并申请资源

> 3.ResourceManager分配Container资源并启动ApplicationMaster,然后AppMaster加载Flink的Jar包和配置构建环境,启动JobManager

> 4.ApplicationMaster向ResourceManager申请工作资源,NodeManager加载Flink的Jar包和配置构建环境并启动TaskManager

> 5.TaskManager启动后向JobManager发送心跳包，并等待JobManager向其分配任务

### 4.3 Flink Streaming Dataflow

`就是Flink代码的执行过程,会涉及到很多的名词`

官网关于Flink的词汇表

[https://ci.apache.org/projects/flink/flink-docs-release-1.11/concepts/glossary.html#glossary](https://ci.apache.org/projects/flink/flink-docs-release-1.11/concepts/glossary.html#glossary)

#### 4.3.1 Dataflow、Operator、Partition、SubTask、Parallelism

> `1.Dataflow:Flink程序在执行的时候会被映射成一个数据流模型`

> `2.Operator:数据流模型中的每一个操作被称作Operator,Operator分为:Source/Transform/Sink`

> `3.Partition:数据流模型是分布式的和并行的,执行中会形成1~n个分区`

> `4.Subtask:多个分区任务可以并行,每一个都是独立运行在一个线程中的,也就是一个Subtask子任务`

> `5.Parallelism:并行度,就是可以同时真正执行的子任务数/分区数`

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/57.png)

#### 4.3.2 Operator传递模式

> 数据在两个operator(算子)之间传递的时候有两种模式：

> `1.One to One模式：`
>> 两个operator用此模式传递的时候，会保持数据的分区数和数据的排序；如上图中的Source1到Map1，它就保留的Source的分区特性，以及分区元素处理的有序性。--类似于Spark中的窄依赖

> `2.Redistributing 模式：`
>> 这种模式会改变数据的分区数；每个一个operator subtask会根据选择transformation把数据发送到不同的目标subtasks,比如keyBy()会通过hashcode重新分区,broadcast()和rebalance()方法会随机重新分区。--类似于Spark中的宽依赖

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/58.png)

#### 4.3.3 Operator Chain

> 客户端在提交任务的时候会对Operator进行优化操作，能进行合并的Operator会被合并为一个OperatoChain，

> 合并后的Operator称为Operator chain，实际上就是一个执行链，每个执行链会在TaskManager上一个独立的线程中执行--就是SubTask。

> `注意：只有OneToOne才可以合并`

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/59.png)

#### 4.3.4 TaskSlot And Slot Sharing

> `任务槽(TaskSlot)`
>> `TaskSlot任务槽就是运行任务的线程槽, 有多少个TaskSlot就表示该TaskManager可以同时运行多少个线程!`
>> 每个TaskManager是一个JVM的进程, 为了控制一个TaskManager(worker)能接收多少个task，Flink通过Task Slot来进行控制。TaskSlot数量是用来限制一个TaskManager工作进程中可以同时运行多少个工作线程，TaskSlot 是一个 TaskManager 中的最小资源分配单位，一个 TaskManager 中有多少个 TaskSlot 就意味着能支持多少并发的Task处理。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/60.png)

> Flink将进程的内存进行了划分到多个slot中，内存被划分到不同的slot之后可以获得如下好处:
>> TaskManager最多能同时并发执行的子任务数是可以通过TaskSolt数量来控制的
>> TaskSolt有独占的内存空间，这样在一个TaskManager中可以运行多个不同的作业，作业之间不受影响。

> `槽共享(Slot Sharing)`

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/61.png)

> `Slot Sharing槽共享就表示线程执行完之前的任务之后不会被立即销毁/回收,而是可以重复利用/共享给其他的subtask使用!`<br/>
> Flink允许子任务共享插槽，即使它们是不同任务(阶段)的子任务(subTask)，只要它们来自同一个作业。<br/>
> 比如图左下角中的map和keyBy和sink 在一个 TaskSlot 里执行以达到资源共享的目的。

> 允许插槽共享有两个主要好处：
>> 资源分配更加公平，如果有比较空闲的slot可以将更多的任务分配给它。
>> 有了任务槽共享，可以提高资源的利用率。

> 注意:
>> slot是静态的概念，是指taskmanager具有的并发执行能力
>> parallelism是动态的概念，是指程序运行时实际使用的并发能力

### 4.4 Flink运行时组件

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/62.png)

> Flink运行时架构主要包括四个不同的组件，它们会在运行流处理应用程序时协同工作：
>> `作业管理器（JobManager）`：分配任务、调度checkpoint做快照
> 
>> `任务管理器（TaskManager）`：主要干活的
> 
>> `资源管理器（ResourceManager）`：管理分配资源
> 
>> `分发器（Dispatcher）`：方便递交任务的接口，WebUI

> 因为Flink是用Java和Scala实现的，所以所有组件都会运行在Java虚拟机上。每个组件的职责如下：

> `1.作业管理器（JobManager）`
>> 控制一个应用程序执行的主进程，也就是说，每个应用程序都会被一个不同的JobManager 所控制执行。
> 
>> JobManager 会先接收到要执行的应用程序，这个应用程序会包括：作业图（JobGraph）、逻辑数据流图（logical dataflow graph）和打包了所有的类、库和其它资源的JAR包。
> 
>> JobManager 会把JobGraph转换成一个物理层面的数据流图，这个图被叫做“执行图”（ExecutionGraph），包含了所有可以并发执行的任务。
> 
>> JobManager 会向资源管理器（ResourceManager）请求执行任务必要的资源，也就是任务管理器（TaskManager）上的插槽（slot）。一旦它获取到了足够的资源，就会将执行图分发到真正运行它们的TaskManager上。而在运行过程中，JobManager会负责所有需要中央协调的操作，比如说检查点（checkpoints）的协调。

> `2.任务管理器（TaskManager）`
>> Flink中的工作进程。通常在Flink中会有多个TaskManager运行，每一个TaskManager都包含了一定数量的插槽（slots）。插槽的数量限制了TaskManager能够执行的任务数量。
> 
>> 启动之后，TaskManager会向资源管理器注册它的插槽；收到资源管理器的指令后，TaskManager就会将一个或者多个插槽提供给JobManager调用。JobManager就可以向插槽分配任务（tasks）来执行了。
> 
>> 在执行过程中，一个TaskManager可以跟其它运行同一应用程序的TaskManager交换数据。

> `3.资源管理器（ResourceManager）`
>> 主要负责管理任务管理器（TaskManager）的插槽（slot），TaskManger 插槽是Flink中定义的处理资源单元。
> 
>> Flink为不同的环境和资源管理工具提供了不同资源管理器，比如YARN、Mesos、K8s，以及standalone部署。
> 
>> 当JobManager申请插槽资源时，ResourceManager会将有空闲插槽的TaskManager分配给JobManager。如果ResourceManager没有足够的插槽来满足JobManager的请求，它还可以向资源提供平台发起会话，以提供启动TaskManager进程的容器。

> `4.分发器（Dispatcher）`
>> 可以跨作业运行，它为应用提交提供了REST接口。
> 
>> 当一个应用被提交执行时，分发器就会启动并将应用移交给一个JobManager。
> 
>> Dispatcher也会启动一个Web UI，用来方便地展示和监控作业执行的信息。
> 
>> Dispatcher在架构中可能并不是必需的，这取决于应用提交运行的方式。

### 4.5 Flink执行图（ExecutionGraph）

> 由Flink程序直接映射成的数据流图是`StreamGraph`，也被称为逻辑流图，因为它们表示的是计算逻辑的高级视图。为了执行一个流处理程序，Flink需要将逻辑流图转换为物理数据流图（也叫执行图），详细说明程序的执行方式。

> Flink 中的执行图可以分成四层：StreamGraph -> JobGraph -> ExecutionGraph -> 物理执行图。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/63.png)

> 原理介绍
>> Flink执行executor会自动根据程序代码生成DAG数据流图
> 
>> Flink 中的执行图可以分成四层：`StreamGraph` -> `JobGraph` -> `ExecutionGraph` -> `物理执行图`。
>> 
>>> `StreamGraph`：是根据用户通过 Stream API 编写的代码生成的最初的图。表示程序的拓扑结构。
>>
>>> `JobGraph`：StreamGraph经过优化后生成了 JobGraph，提交给 JobManager 的数据结构。主要的优化为，将多个符合条件的节点 chain 在一起作为一个节点，这样可以减少数据在节点之间流动所需要的序列化/反序列化/传输消耗。
>> 
>>> `ExecutionGraph`：JobManager 根据 JobGraph 生成ExecutionGraph。ExecutionGraph是JobGraph的并行化版本，是调度层最核心的数据结构。
>> 
>>> `物理执行图`：JobManager 根据 ExecutionGraph 对 Job 进行调度后，在各个TaskManager 上部署 Task 后形成的“图”，并不是一个具体的数据结构。

> 简单理解：
>> `StreamGraph`：最初的程序执行逻辑流程，也就是算子之间的前后顺序（全部都是Subtask）
> 
>> `JobGraph`：将部分可以合并的Subtask合并成一个Task
> 
>> `ExecutionGraph`：为Task赋予并行度
> 
>> `物理执行图`：将Task赋予并行度后的执行流程，落实到具体的TaskManager上，将具体的Task落实到具体的Slot内进行运行。

## 5 Flink批处理

[https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/batch/](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/batch/)

### 5.1 API和编程模型

> API
>> Flink提供了多个层次的API供开发者使用，越往上抽象程度越高，使用起来越方便；越往下越底层，使用起来难度越大

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/64.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/65.png)

> 编程模型
>> Flink 应用程序结构主要包含三部分,`Source`/`Transformation`/`Sink`,如下图所示：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/66.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/67.png)

### 5.2 Source

#### 5.2.1 基于集合的Source

> 代码演示

```
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;

import java.util.Arrays;

/**
 * Author xiaoma
 * Date 2020/10/12 11:19
 * Desc
 * 把本地的普通的Java集合/Scala集合变为分布式的Flink的DataSet集合!
 * 一般用于学习测试时编造数据时使用
 * 1.env.fromElements(可变参数);
 * 2.env.fromColletion(各种集合);
 * 3.env.generateSequence(开始,结束);
 */
public class SourceDemo01 {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        System.out.println(env.getParallelism());//获取并行度
        //2.source
        // * 1.env.fromElements(可变参数);
        DataSource<String> ds1 = env.fromElements("hadoop", "hive", "flink");
        // * 2.env.fromColletion(各种集合);
        DataSource<String> ds2 = env.fromCollection(Arrays.asList("hadoop", "hive", "flink"));
        // * 3.env.generateSequence(开始,结束);
        DataSource<Long> ds3 = env.generateSequence(1, 10);
        //3.Transformation
        //4.sink
        ds1.print();
        ds2.print();
        ds3.print();
        //5.execute
        //env.execute();
    }
}
```

#### 5.2.2 基于文件的Source

> 代码演示

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.configuration.Configuration;

/**
 * Author xiaoma
 * Date 2020/10/12 11:19
 * Desc
 * 1.env.readTextFile(本地文件/HDFS文件);//压缩文件也可以
 * 2.env.readCsvFile("本地文件/HDFS文件")
 * Configuration parameters = new Configuration();
 * parameters.setBoolean("recursive.file.enumeration", true);//设置是否递归读取目录
 * 3.env.readTextFile("目录").withParameters(parameters);
 */
public class SourceDemo02 {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        // * 1.env.readTextFile(本地文件/HDFS文件);//压缩文件也可以
        DataSource<String> ds1 = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\words.txt");
        DataSource<String> ds2 = env.readTextFile("hdfs://node1:8020//wordcount/input/words.txt");
        DataSource<String> ds3 = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\wordcount.txt.gz");

        // * 2.env.readCsvFile("本地文件/HDFS文件")
        DataSource<Subject> ds4 = env.readCsvFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\subject.csv")
                .fieldDelimiter(",")
                .pojoType(Subject.class, "id", "name");

        // * Configuration parameters = new Configuration();
        // * parameters.setBoolean("recursive.file.enumeration", true);//设置是否递归读取目录
        // * 3.env.readTextFile("目录").withParameters(parameters);
        Configuration parameters = new Configuration();
        parameters.setBoolean("recursive.file.enumeration", true);
        DataSource<String> ds5 = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\dir").withParameters(parameters);

        //3.Transformation
        //4.sink
        ds1.print();
        ds2.print();
        ds3.print();
        ds4.print();
        ds5.print();

        //5.execute
        //env.execute();
    }
    @Data//会自动生成get/set/toString/hashCode....
    @AllArgsConstructor//全参构造
    @NoArgsConstructor//空参构造
    public static class Subject {
        public int id;
        public String name;
    }
}
```

### 5.3 Transformation

#### 5.3.1 官网api

[https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/batch/](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/batch/)

#### 5.3.2 中文api

Transformation | Description
---|---
Map | 在算子中得到一个元素并生成一个新元素data.map { x => x.toInt }
FlatMap | 在算子中获取一个元素, 并生成任意个数的元素data.flatMap { str => str.split(" ") }
MapPartition | 类似Map, 但是一次Map一整个并行分区data.mapPartition { in => in map { (_, 1) } }
Filter | 如果算子返回true则包含进数据集, 如果不是则被过滤掉data.filter { _ > 100 }
Reduce | 通过将两个元素合并为一个元素, 从而将一组元素合并为一个元素data.reduce { _ + _ }
ReduceGroup | 将一组元素合并为一个或者多个元素data.reduceGroup { elements => elements.sum }
Aggregate | 讲一组值聚合为一个值, 聚合函数可以看作是内置的Reduce函数data.aggregate(SUM, 0).aggregate(MIN, 2)data.sum(0).min(2)
Distinct | 去重
Join | 按照相同的Key合并两个数据集input1.join(input2).where(0).equalTo(1)同时也可以选择进行合并的时候的策略, 是分区还是广播, 是基于排序的算法还是基于哈希的算法input1.join(input2, JoinHint.BROADCAST_HASH_FIRST).where(0).equalTo(1)
OuterJoin | 外连接, 包括左外, 右外, 完全外连接等left.leftOuterJoin(right).where(0).equalTo(1) { (left, right) => ... }
CoGroup | 二维变量的Reduce运算, 对每个输入数据集中的字段进行分组, 然后join这些组input1.coGroup(input2).where(0).equalTo(1)
Cross | 笛卡尔积input1.cross(input2)
Union | 并集input1.union(input2)
Rebalance | 分区重新平衡, 以消除数据倾斜input.rebalance()
Hash-Partition | 按照Hash分区input.partitionByHash(0)
Range-Partition | 按照Range分区input.partitionByRange(0)
CustomParititioning | 自定义分区input.partitionCustom(partitioner: Partitioner[K], key)
First-n | 返回数据集中的前n个元素input.first(3)
partitionByHash | 按照指定的key进行hash分区
sortPartition | 指定字段对分区中的数据进行排序

#### 5.3.3 map

> map:将函数作用在集合中的每一个元素上,并返回作用后的结果

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/68.png)

> 需求:将click.log中的每一条日志转为javaBean对象

```
{"browserType":"360浏览器","categoryID":2,"channelID":3,"city":"昌平","country":"china","entryTime":1577890860000,"leaveTime":1577898060000,"network":"移动","produceID":11,"province":"北京","source":"必应跳转","userID":18}
{"browserType":"火狐","categoryID":2,"channelID":13,"city":"昌平","country":"china","entryTime":1577887260000,"leaveTime":1577898060000,"network":"移动","produceID":6,"province":"北京","source":"必应跳转","userID":5}
{"browserType":"360浏览器","categoryID":9,"channelID":4,"city":"昌平","country":"china","entryTime":1577869260000,"leaveTime":1577898060000,"network":"移动","produceID":20,"province":"北京","source":"360搜索跳转","userID":1}
{"browserType":"qq浏览器","categoryID":1,"channelID":16,"city":"海淀","country":"china","entryTime":1577890860000,"leaveTime":1577898060000,"network":"联通","produceID":11,"province":"北京","source":"直接输入","userID":11}
......
```

> 代码实现

```
import com.alibaba.fastjson.JSON;
import lombok.Data;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.MapOperator;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * map:将函数作用在集合中的每一个元素上,并返回作用后的结果,一般用来做转换操作!
 * 将click.log中的每一条日志转为javaBean对象
 * json格式的字符串通过map转为javaBean对象
 */
public class TransformationDemo01_map {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSet<String> clickLogStrDS = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\click.log");
        //3.Transformation
        /*
        public interface MapFunction<T, O> extends Function, Serializable {
         O map(T value) throws Exception;
        }
         */
        MapOperator<String, ClickLog> clickLogDS = clickLogStrDS.map((MapFunction<String, ClickLog>) value -> {
            //value就是clickLogStrDS中的每一行数据,也就是每一行json字符串
            //将jsonStr转为JavaBean对象
            ClickLog clickLog = JSON.parseObject(value, ClickLog.class);
            return clickLog;
        });
        //4.sink
        clickLogDS.print();
        //5.execute
    }
    @Data
    public static class ClickLog {
        //频道ID
        private long channelID;
        //产品的类别ID
        private long categoryID;
        //产品ID
        private long produceID;
        //用户的ID
        private long userID;
        //国家
        private String country;
        //省份
        private String province;
        //城市
        private String city;
        //网络方式
        private String network;
        //来源方式
        private String source;
        //浏览器类型
        private String browserType;
        //进入网站时间
        private Long entryTime;
        //离开网站时间
        private Long leaveTime;
    }
}

```

#### 5.3.4 flatmap

> flatMap:将集合中的每个元素变成一个或多个元素,并返回扁平化之后的结果

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/69.png)

> 需求:将每一条clickLog转换为如下三个纬度
>> (年-月-日-时,1)
>> (年-月-日,1)
>> (年-月,1)

> 代码实现

```
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import lombok.Data;
import org.apache.commons.lang3.time.DateFormatUtils;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.FlatMapOperator;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * flatMap:将集合中的每个元素变成一个或多个元素,并返回扁平化之后的结果
 * 需求:
 * 将每一条clickLog转换为如下三个纬度
 * (年-月-日-时,1)
 * (年-月-日,1)
 * (年-月,1)
 * 也就是将集合中的每1条日志转为3条!
 */
public class TransformationDemo02_flatMap {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSet<String> clickLogStrDS = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\click.log");
        //3.Transformation
        /*
        public interface FlatMapFunction<T, O> extends Function, Serializable {
            void flatMap(T value, Collector<O> out) throws Exception;
        }
         */
        FlatMapOperator<String, Tuple2<String, Integer>> resultDS = clickLogStrDS.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                //value就是集合中的每一行jsonStr
                /*ClickLog clickLog = JSON.parseObject(value, ClickLog.class);
                Long entryTime = clickLog.entryime;*/
                JSONObject jsonObject = JSON.parseObject(value);
                Long entryTim = jsonObject.getLong("entryTime");
                // * 将每一条clickLog转换为如下三个纬度
                // * (年-月-日-时,1)
                // * (年-月-日,1)
                // * (年-月,1)
                String time1 = DateFormatUtils.format(entryTim, "yyyy-MM-dd-HH");
                String time2 = DateFormatUtils.format(entryTim, "yyyy-MM-dd");
                String time3 = DateFormatUtils.format(entryTim, "yyyy-MM");
                out.collect(Tuple2.of(time1, 1));
                out.collect(Tuple2.of(time2, 1));
                out.collect(Tuple2.of(time3, 1));
            }
        });

        //4.sink
        long count = resultDS.count();//求集合中有多少元素
        System.out.println(count);
        resultDS.print();
        //5.execute
    }
    @Data
    public static class ClickLog {
        //频道ID
        private long channelID;
        //产品的类别ID
        private long categoryID;
        //产品ID
        private long produceID;
        //用户的ID
        private long userID;
        //国家
        private String country;
        //省份
        private String province;
        //城市
        private String city;
        //网络方式
        private String network;
        //来源方式
        private String source;
        //浏览器类型
        private String browserType;
        //进入网站时间
        private Long entryTime;
        //离开网站时间
        private Long leaveTime;
    }
}

```

#### 5.3.5 filter

> filter:按照指定的条件对集合中的元素进行过滤,过滤出返回true/符合条件的元素

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/70.png)

> 需求:过滤出clickLog中使用谷歌浏览器访问的日志

> 代码实现

```
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import lombok.Data;
import org.apache.commons.lang3.time.DateFormatUtils;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.FlatMapOperator;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * flatMap:将集合中的每个元素变成一个或多个元素,并返回扁平化之后的结果
 * 需求:
 * 将每一条clickLog转换为如下三个纬度
 * (年-月-日-时,1)
 * (年-月-日,1)
 * (年-月,1)
 * 也就是将集合中的每1条日志转为3条!
 */
public class TransformationDemo02_flatMap {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSet<String> clickLogStrDS = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\click.log");
        //3.Transformation
        /*
        public interface FlatMapFunction<T, O> extends Function, Serializable {
            void flatMap(T value, Collector<O> out) throws Exception;
        }
         */
        FlatMapOperator<String, Tuple2<String, Integer>> resultDS = clickLogStrDS.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                //value就是集合中的每一行jsonStr
                /*ClickLog clickLog = JSON.parseObject(value, ClickLog.class);
                Long entryTime = clickLog.entryime;*/
                JSONObject jsonObject = JSON.parseObject(value);
                Long entryTim = jsonObject.getLong("entryTime");
                // * 将每一条clickLog转换为如下三个纬度
                // * (年-月-日-时,1)
                // * (年-月-日,1)
                // * (年-月,1)
                String time1 = DateFormatUtils.format(entryTim, "yyyy-MM-dd-HH");
                String time2 = DateFormatUtils.format(entryTim, "yyyy-MM-dd");
                String time3 = DateFormatUtils.format(entryTim, "yyyy-MM");
                out.collect(Tuple2.of(time1, 1));
                out.collect(Tuple2.of(time2, 1));
                out.collect(Tuple2.of(time3, 1));
            }
        });

        //4.sink
        long count = resultDS.count();//求集合中有多少元素
        System.out.println(count);
        resultDS.print();
        //5.execute
    }
    @Data
    public static class ClickLog {
        //频道ID
        private long channelID;
        //产品的类别ID
        private long categoryID;
        //产品ID
        private long produceID;
        //用户的ID
        private long userID;
        //国家
        private String country;
        //省份
        private String province;
        //城市
        private String city;
        //网络方式
        private String network;
        //来源方式
        private String source;
        //浏览器类型
        private String browserType;
        //进入网站时间
        private Long entryTime;
        //离开网站时间
        private Long leaveTime;
    }
}

```

#### 5.3.6 groupBy

> groupBy:对集合中的元素按照指定的key进行分组

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/71.png)

> 需求:对ClickLog按照浏览器类型记为1并分组

> 代码实现

```
import com.alibaba.fastjson.JSON;
import lombok.Data;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.AggregateOperator;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.api.java.operators.UnsortedGrouping;
import org.apache.flink.api.java.tuple.Tuple2;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * groupBy:对集合中的元素按照指定的key进行分组
 * 需求:
 * 对ClickLog按照浏览器类型记为1并分组
 */
public class TransformationDemo04_groupBy_sum {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSet<String> clickLogStrDS = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\click.log");
        //3.Transformation
        //取出浏览器类型并记为1
        MapOperator<String, Tuple2<String, Integer>> browserTypeAndOneDS = clickLogStrDS.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                ClickLog clickLog = JSON.parseObject(value, ClickLog.class);
                String browserType = clickLog.browserType;
                return Tuple2.of(browserType, 1);
            }
        });
        //分组
        UnsortedGrouping<Tuple2<String, Integer>> groupedDS = browserTypeAndOneDS.groupBy(0);

        //聚合
        AggregateOperator<Tuple2<String, Integer>> aggResult = groupedDS.sum(1);

        //4.sink
        aggResult.print();

        //5.execute
    }
    @Data
    public static class ClickLog {
        //频道ID
        private long channelID;
        //产品的类别ID
        private long categoryID;
        //产品ID
        private long produceID;
        //用户的ID
        private long userID;
        //国家
        private String country;
        //省份
        private String province;
        //城市
        private String city;
        //网络方式
        private String network;
        //来源方式
        private String source;
        //浏览器类型
        private String browserType;
        //进入网站时间
        private Long entryTime;
        //离开网站时间
        private Long leaveTime;
    }
}

```

#### 5.3.7 sum

> sum:按照指定的字段对集合中的元素进行求和

> 需求:统计各个浏览器类型的访问量

> 代码实现

```
在groupBy代码里包含了
```

#### 5.3.8 min和minBy/max和maxBy

> min只会求出最小的那个字段,其他的字段不管<br/>
> minBy会求出最小的那个字段和对应的其他的字段<br/>
> max和maxBy同理

> 需求:求最少的访问量以及对应的浏览器类型

> 代码实现

```
import com.alibaba.fastjson.JSON;
import lombok.Data;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.AggregateOperator;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.api.java.operators.ReduceOperator;
import org.apache.flink.api.java.operators.UnsortedGrouping;
import org.apache.flink.api.java.tuple.Tuple2;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * min只会求出最小的那个字段,其他的字段不管
 * minBy会求出最小的那个字段和对应的其他的字段
 * max和maxBy同理
 * 需求:
 * 求最少的访问量以及对应的浏览器类型
 */
public class TransformationDemo05_min_minBy {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSet<String> clickLogStrDS = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\click.log");
        //3.Transformation
        //取出浏览器类型并记为1
        MapOperator<String, Tuple2<String, Integer>> browserTypeAndOneDS = clickLogStrDS.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                ClickLog clickLog = JSON.parseObject(value, ClickLog.class);
                String browserType = clickLog.browserType;
                return Tuple2.of(browserType, 1);
            }
        });
        //分组
        UnsortedGrouping<Tuple2<String, Integer>> groupedDS = browserTypeAndOneDS.groupBy(0);

        //聚合
        AggregateOperator<Tuple2<String, Integer>> aggResult = groupedDS.sum(1);

        //求最少的访问量以及对应的浏览器类型
        AggregateOperator<Tuple2<String, Integer>> minResult = aggResult.min(1);
        ReduceOperator<Tuple2<String, Integer>> minByResult = aggResult.minBy(1);

        //4.sink
        aggResult.print();
        //(qq浏览器,29)
        //(火狐,24)
        //(360浏览器,23)
        //(谷歌浏览器,24)
        // * min只会求出最小的那个字段,其他的字段不管,所以输出可能为(qq浏览器,23),(火狐,23),(360浏览器,23),(谷歌浏览器,23)
        // * minBy会求出最小的那个字段和对应的其他的字段
        minResult.print();//(qq浏览器,23),只求出了最小值,其他的字段不管!
        minByResult.print();//(360浏览器,23),全部正确,

        //5.execute
    }
    @Data
    public static class ClickLog {
        //频道ID
        private long channelID;
        //产品的类别ID
        private long categoryID;
        //产品ID
        private long produceID;
        //用户的ID
        private long userID;
        //国家
        private String country;
        //省份
        private String province;
        //城市
        private String city;
        //网络方式
        private String network;
        //来源方式
        private String source;
        //浏览器类型
        private String browserType;
        //进入网站时间
        private Long entryTime;
        //离开网站时间
        private Long leaveTime;
    }
}

```

#### 5.3.9 aggregate

> aggregate:按照指定的聚合函数和字段对集合中的元素进行聚合,如SUM,MIN,MAX

> 需求:使用aggregate完成求sum和min

> 代码实现

```import com.alibaba.fastjson.JSON;
import lombok.Data;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.aggregation.Aggregations;
import org.apache.flink.api.java.operators.AggregateOperator;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.api.java.operators.UnsortedGrouping;
import org.apache.flink.api.java.tuple.Tuple2;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * aggregate:按照指定的聚合函数和字段对集合中的元素进行聚合,如SUM,MIN,MAX
 * 需求:
 * 使用aggregate完成求sum和min
 */
public class TransformationDemo06_aggregate {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSet<String> clickLogStrDS = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\click.log");
        //3.Transformation
        //取出浏览器类型并记为1
        MapOperator<String, Tuple2<String, Integer>> browserTypeAndOneDS = clickLogStrDS.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                ClickLog clickLog = JSON.parseObject(value, ClickLog.class);
                String browserType = clickLog.browserType;
                return Tuple2.of(browserType, 1);
            }
        });

        //分组
        UnsortedGrouping<Tuple2<String, Integer>> groupedDS = browserTypeAndOneDS.groupBy(0);

        //使用aggregate完成求sum/min
        //聚合
        AggregateOperator<Tuple2<String, Integer>> aggResult = groupedDS.aggregate(Aggregations.SUM, 1);

        //求最少的访问量以及对应的浏览器类型
        AggregateOperator<Tuple2<String, Integer>> minResult = aggResult.aggregate(Aggregations.MIN, 1);

        //4.sink
        aggResult.print();
        //(qq浏览器,29)
        //(火狐,24)
        //(360浏览器,23)
        //(谷歌浏览器,24)
        minResult.print();//(qq浏览器,23),只求出了最小值,其他的字段不管!
        //5.execute
    }
    @Data
    public static class ClickLog {
        //频道ID
        private long channelID;
        //产品的类别ID
        private long categoryID;
        //产品ID
        private long produceID;
        //用户的ID
        private long userID;
        //国家
        private String country;
        //省份
        private String province;
        //城市
        private String city;
        //网络方式
        private String network;
        //来源方式
        private String source;
        //浏览器类型
        private String browserType;
        //进入网站时间
        private Long entryTime;
        //离开网站时间
        private Long leaveTime;
    }
}
```

#### 5.3.10 reduce和reduceGroup

> reduce:对集合中的元素进行聚合<br/>
> reduceGroup:对集合中的元素先进行预聚合再合并结果

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/72.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/73.png)

> 需求:使用reduce和reduceGroup完成求sum

> 代码实现

```
import com.alibaba.fastjson.JSON;
import lombok.Data;
import org.apache.flink.api.common.functions.GroupReduceFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.GroupReduceOperator;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.api.java.operators.ReduceOperator;
import org.apache.flink.api.java.operators.UnsortedGrouping;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.util.Collector;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * reduce:对集合中的元素进行聚合
 * reduceGroup:对集合中的元素先进行预聚合再合并结果
 * 需求:
 * 使用reduce和reduceGroup完成求sum
 */
public class TransformationDemo07_reduce_reduceGroup {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSet<String> clickLogStrDS = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\click.log");
        //3.Transformation
        //取出浏览器类型并记为1
        MapOperator<String, Tuple2<String, Integer>> browserTypeAndOneDS = clickLogStrDS.map(new MapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> map(String value) throws Exception {
                ClickLog clickLog = JSON.parseObject(value, ClickLog.class);
                String browserType = clickLog.browserType;
                return Tuple2.of(browserType, 1);
            }
        });
        //分组
        //分好组的<浏览器类型,1>,<浏览器类型,1>,<浏览器类型,1>,<浏览器类型,1>
        UnsortedGrouping<Tuple2<String, Integer>> groupedDS = browserTypeAndOneDS.groupBy(0);

        // * reduce:对集合中的元素进行聚合
        // * reduceGroup:对集合中的元素先进行预聚合再合并结果
        // * 需求:
        // * 使用reduce和reduceGroup完成求sum
        /*
        public interface ReduceFunction<T> extends Function, Serializable {
         T reduce(T value1, T value2) throws Exception;
        }
         */
        ReduceOperator<Tuple2<String, Integer>> aggResult1 = groupedDS.reduce(new ReduceFunction<Tuple2<String, Integer>>() {
            @Override
            public Tuple2<String, Integer> reduce(Tuple2<String, Integer> value1, Tuple2<String, Integer> value2) throws Exception {
                //value1是历史值
                //value2是当前进来的数据
                int tempCount = value1.f1 + value2.f1;
                return Tuple2.of(value1.f0, tempCount);
            }
        });

        /*
        public interface GroupReduceFunction<T, O> extends Function, Serializable {
            void reduce(Iterable<T> values, Collector<O> out) throws Exception;
        }
         */
        GroupReduceOperator<Tuple2<String, Integer>, Tuple2<String, Integer>> aggResult2 = groupedDS.reduceGroup(new GroupReduceFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>() {
            @Override
            public void reduce(Iterable<Tuple2<String, Integer>> values, Collector<Tuple2<String, Integer>> out) throws Exception {
                int count = 0;
                String key = null;
                //values就是分好组的一组组数据
                for (Tuple2<String, Integer> value : values) {
                    count += value.f1;
                    key = value.f0;
                }
                out.collect(Tuple2.of(key, count));
            }
        });


        //4.sink
        aggResult1.print();
        aggResult2.print();
        //(qq浏览器,29)
        //(火狐,24)
        //(360浏览器,23)
        //(谷歌浏览器,24)


        //5.execute
    }
    @Data
    public static class ClickLog {
        //频道ID
        private long channelID;
        //产品的类别ID
        private long categoryID;
        //产品ID
        private long produceID;
        //用户的ID
        private long userID;
        //国家
        private String country;
        //省份
        private String province;
        //城市
        private String city;
        //网络方式
        private String network;
        //来源方式
        private String source;
        //浏览器类型
        private String browserType;
        //进入网站时间
        private Long entryTime;
        //离开网站时间
        private Long leaveTime;
    }
}

```

#### 5.3.11 union

> union:将两个集合进行合并但不会去重

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/74.png)

> 需求:读取click.log和click2.log并使用union合并结果

> 代码实现

```
import com.alibaba.fastjson.JSON;
import lombok.Data;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DistinctOperator;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.api.java.operators.UnionOperator;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * union:将两个集合进行合并但不会去重
 * 需求:
 * 读取click.log和click2.log并使用union合并结果
 *
 * distinct:对集合中的元素进行去重
 * 需求:
 * 对合并后的结果进行去重
 */
public class TransformationDemo08_union_distinct {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSet<String> clickLogStrDS = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\click.log");
        DataSet<String> clickLogStrDS2 = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\click2.log");
        //3.Transformation
        //合并
        UnionOperator<String> result = clickLogStrDS.union(clickLogStrDS2);

        //去重
        DistinctOperator<String> result2 = result.distinct();

        MapOperator<String, ClickLog> temp = result.map(new MapFunction<String, ClickLog>() {
            @Override
            public ClickLog map(String value) throws Exception {
                return JSON.parseObject(value, ClickLog.class);
            }
        });
        DistinctOperator<ClickLog> result3 = temp.distinct("browserType");

        //4.sink
        long count = result.count();
        System.out.println(count);//200
        long count2 = result2.count();
        System.out.println(count2);//100
        long count3 = result3.count();
        System.out.println(count3);//4
        //5.execute
    }
    @Data
    public static class ClickLog {
        //频道ID
        private long channelID;
        //产品的类别ID
        private long categoryID;
        //产品ID
        private long produceID;
        //用户的ID
        private long userID;
        //国家
        private String country;
        //省份
        private String province;
        //城市
        private String city;
        //网络方式
        private String network;
        //来源方式
        private String source;
        //浏览器类型
        private String browserType;
        //进入网站时间
        private Long entryTime;
        //离开网站时间
        private Long leaveTime;
    }
}

```

#### 5.3.12 distinct

> distinct:对集合中的元素进行去重

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/75.png)

> 需求:对合并后的结果进行去重

> 代码实现

```
在union中包括了
```

#### 5.3.13 join 

> join:将两个集合按照指定的条件进行连接

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/76.png)

> 需求:将学生成绩数据score.csv和学科数据subject.csv进行关联,得到学生对应学科的成绩

> 代码实现

```
import lombok.Data;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.operators.JoinOperator;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.tuple.Tuple4;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * join:将两个集合按照指定的条件进行连接
 * 需求:
 * 将学生成绩数据score.csv和学科数据subject.csv进行关联,得到学生对应学科的成绩
 */
public class TransformationDemo09_join {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        //学生id,学生姓名,学科id,成绩
        DataSource<Score> scoreDS = env.readCsvFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\score.csv")
                .pojoType(Score.class, "id", "stuName", "subId", "score");
        //学科id,学科名称
        DataSource<Subject> subjectDS = env.readCsvFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\subject.csv")
                .pojoType(Subject.class, "id", "name");

        //3.Transformation
        //注意:如果是tuple可以索引
        //scoreDS和subjectDS进行关联,条件是scoreDS的索引为2的字段 = subjectDS索引为0的字段,也就是要根据学科id进行关联
        //注意:如果是JavaBean需要写字段名
        JoinOperator.DefaultJoin<Score, Subject> joinResult = scoreDS.join(subjectDS).where("subId").equalTo("id");

        //取出我们关注的信息:4元组:Tuple4<学生id,学生姓名,学科名称,成绩>
        MapOperator<Tuple2<Score, Subject>, Tuple4<Integer, String, String, Double>> simpleResult = joinResult.map(new MapFunction<Tuple2<Score, Subject>, Tuple4<Integer, String, String, Double>>() {
            @Override
            public Tuple4<Integer, String, String, Double> map(Tuple2<Score, Subject> value) throws Exception {
                return Tuple4.of(value.f0.getId(), value.f0.getStuName(), value.f1.getName(), value.f0.getScore());
            }
        });

        //4.sink
        joinResult.print();
        //(Score(id=21, stuName=赵六, subId=3, score=65.0),Subject(id=3, name=英语))
        simpleResult.print();

        //5.execute
    }
    @Data
    public static class Score{
        private Integer id;
        private String stuName;
        private Integer subId;
        private Double score;

    }
    @Data
    public static class Subject {
        private Integer id;
        private String name;
    }
}

```

#### 5.3.14 leftOuterJoin

> leftOuterJoin: 左外连接, 左边集合的元素全部留下,右边的满足条件的元素留下

> 需求

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/77.png)

> 代码实现

```
import org.apache.flink.api.common.functions.JoinFunction;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.operators.JoinOperator;
import org.apache.flink.api.java.operators.join.JoinFunctionAssigner;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.tuple.Tuple3;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * leftOuterJoin: 左外连接, 左边集合的元素全部留下,右边的满足条件的元素留下
 *
 */
public class TransformationDemo10_leftOuterJoin {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSource<Tuple2<Integer, String>> userDS = env.fromElements(Tuple2.of(1, "tom"), Tuple2.of(2, "jack"), Tuple2.of(3, "rose"));
        DataSource<Tuple2<Integer, String>> cityDS = env.fromElements(Tuple2.of(1, "北京"), Tuple2.of(2, "上海"), Tuple2.of(4, "广州"));

        //3.Transformation
        JoinFunctionAssigner<Tuple2<Integer, String>, Tuple2<Integer, String>> joinAssigner = userDS.leftOuterJoin(cityDS).where(0).equalTo(0);
        JoinOperator<Tuple2<Integer, String>, Tuple2<Integer, String>, Tuple3<Integer, String, String>> result = joinAssigner.with(new JoinFunction<Tuple2<Integer, String>, Tuple2<Integer, String>, Tuple3<Integer, String, String>>() {
            @Override
            public Tuple3<Integer, String, String> join(Tuple2<Integer, String> first, Tuple2<Integer, String> second) throws Exception {
                //first就是左边的userDS中的Tuple2
                //second就是右边的cityDS中的Tuple2
                if (second != null) {
                    return Tuple3.of(first.f0, first.f1, second.f1);
                } else {//second == null
                    return Tuple3.of(first.f0, first.f1, "未知");
                }
            }
        });

        //4.sink
        result.print();

        //5.execute
    }

}

```

#### 5.3.15 rightOuterJoin

> rightOuterJoin: 右外连接, 右边集合的元素全部留下,左边的满足条件的元素留下

> 需求

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/78.png)

> 代码实现

```
import org.apache.flink.api.common.functions.JoinFunction;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.operators.JoinOperator;
import org.apache.flink.api.java.operators.join.JoinFunctionAssigner;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.tuple.Tuple3;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * rightOuterJoin: 右外连接, 右边集合的元素全部留下,左边的满足条件的元素留下
 *
 */
public class TransformationDemo11_rightOuterJoin {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSource<Tuple2<Integer, String>> userDS = env.fromElements(Tuple2.of(1, "tom"), Tuple2.of(2, "jack"), Tuple2.of(3, "rose"));
        DataSource<Tuple2<Integer, String>> cityDS = env.fromElements(Tuple2.of(1, "北京"), Tuple2.of(2, "上海"), Tuple2.of(4, "广州"));

        //3.Transformation
        JoinFunctionAssigner<Tuple2<Integer, String>, Tuple2<Integer, String>> joinAssigner = userDS.rightOuterJoin(cityDS).where(0).equalTo(0);
        JoinOperator<Tuple2<Integer, String>, Tuple2<Integer, String>, Tuple3<Integer, String, String>> result = joinAssigner.with(new JoinFunction<Tuple2<Integer, String>, Tuple2<Integer, String>, Tuple3<Integer, String, String>>() {
            @Override
            public Tuple3<Integer, String, String> join(Tuple2<Integer, String> first, Tuple2<Integer, String> second) throws Exception {
                if (first != null) {
                    return Tuple3.of(second.f0, first.f1, second.f1);
                } else {//first == null
                    return Tuple3.of(second.f0, "未知", second.f1);
                }
            }
        });
        //4.sink
        result.print();

        //5.execute
    }

}

```

#### 5.3.16 fullOuterJoin

> fullOuterJoin: 全外连接, 左右集合中的元素全部留下

> 需求

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/79.png)

> 代码实现

```
import org.apache.flink.api.common.functions.JoinFunction;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.operators.JoinOperator;
import org.apache.flink.api.java.operators.join.JoinFunctionAssigner;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.tuple.Tuple3;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * fullOuterJoin: 全外连接, 左右集合中的元素全部留下
 *
 */
public class TransformationDemo12_fullOuterJoin {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSource<Tuple2<Integer, String>> userDS = env.fromElements(Tuple2.of(1, "tom"), Tuple2.of(2, "jack"), Tuple2.of(3, "rose"));
        DataSource<Tuple2<Integer, String>> cityDS = env.fromElements(Tuple2.of(1, "北京"), Tuple2.of(2, "上海"), Tuple2.of(4, "广州"));

        //3.Transformation
        JoinFunctionAssigner<Tuple2<Integer, String>, Tuple2<Integer, String>> joinAssigner = userDS.fullOuterJoin(cityDS).where(0).equalTo(0);
        JoinOperator<Tuple2<Integer, String>, Tuple2<Integer, String>, Tuple3<Integer, String, String>> result = joinAssigner.with(new JoinFunction<Tuple2<Integer, String>, Tuple2<Integer, String>, Tuple3<Integer, String, String>>() {
            @Override
            public Tuple3<Integer, String, String> join(Tuple2<Integer, String> first, Tuple2<Integer, String> second) throws Exception {
                if (first == null) {
                    return Tuple3.of(second.f0, "未知", second.f1);
                }
                if (second == null) {
                    return Tuple3.of(first.f0, first.f1, "未知");
                }
                return Tuple3.of(second.f0, first.f1, second.f1);
            }
        });
        //4.sink
        result.print();

        //5.execute
    }

}
```

#### 5.3.17 cross

> cross:求两个集合的笛卡尔积

> 需求

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/80.png)

> 代码实现

```
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.CrossOperator;
import org.apache.flink.api.java.operators.DataSource;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * corss:求两个集合的笛卡尔积
 *
 */
public class TransformationDemo13_corss {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSource<String> studentDS = env.fromElements("tom", "jack", "rose");
        DataSource<String> subjectDS = env.fromElements("bigdata", "java", "python");

        //3.Transformation
        CrossOperator.DefaultCross<String, String> result = studentDS.cross(subjectDS);

        //4.sink
        result.print();

        //5.execute
    }

}

```

#### 5.3.18 reBalance重平衡分区

> 类似于Spark中的repartition,但是功能更强大,可以直接解决数据倾斜

> Flink也有数据倾斜的时候，比如当前有数据量大概10亿条数据需要处理，在处理过程中可能会发生如图所示的状况，出现了数据倾斜，其他3台机器执行完毕也要等待机器1执行完毕后才算整体将任务完成；

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/81.png)

> 所以在实际的工作中，出现这种情况比较好的解决方案就是rebalance(内部使用round robin方法将数据均匀打散)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/82.png)

> 代码实现

```
import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.operators.FilterOperator;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.api.java.tuple.Tuple2;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * rebalance:重平衡
 */
public class TransformationDemo14_rebalance {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.source
        DataSource<Long> longDS = env.generateSequence(0, 100);

        //3.Transformation
        //下面的操作相当于将数据随机分配一下,有可能出现数据倾斜
        FilterOperator<Long> filterDS = longDS.filter(new FilterFunction<Long>() {
            @Override
            public boolean filter(Long num) throws Exception {
                return num > 10;
            }
        });

        //接下来使用map操作,将数据转为(分区编号/子任务编号, 数据)
        //Rich表示多功能的,比MapFunction要多一些API可以供我们使用
        MapOperator<Long, Tuple2<Integer, Long>> result1 = filterDS.map(new RichMapFunction<Long, Tuple2<Integer, Long>>() {
            @Override
            public Tuple2<Integer, Long> map(Long value) throws Exception {
                //获取分区编号/子任务编号
                int id = getRuntimeContext().getIndexOfThisSubtask();
                return Tuple2.of(id, value);
            }
        });

        MapOperator<Long, Tuple2<Integer, Long>> result2 = filterDS.rebalance()
                .map(new RichMapFunction<Long, Tuple2<Integer, Long>>() {
                    @Override
                    public Tuple2<Integer, Long> map(Long value) throws Exception {
                        //获取分区编号/子任务编号
                        int id = getRuntimeContext().getIndexOfThisSubtask();
                        return Tuple2.of(id, value);
                    }
                });

        //4.sink
        result1.print();//有可能出现数据倾斜
        result2.print();//在输出前进行了rebalance重分区平衡,解决了数据倾斜

        //5.execute
    }

}

```

#### 5.3.19 其他分区

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/83.png)

> 代码实现

```
import org.apache.flink.api.common.functions.Partitioner;
import org.apache.flink.api.common.operators.Order;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.operators.PartitionOperator;
import org.apache.flink.api.java.operators.SortPartitionOperator;
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.core.fs.FileSystem;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

/**
 * Author xiaoma
 * Date 2020/10/12 14:39
 * Desc
 * otherPartition其他分区API
 */
public class TransformationDemo15_otherPartition {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(2);//设置并行度为2,表示后面的subTask数量为2/分区数为2,便于观察

        //2.Source
        List<Tuple3<Integer, Long, String>> list = new ArrayList<>();
        list.add(Tuple3.of(1, 1L, "Hello"));
        list.add(Tuple3.of(2, 2L, "Hello"));
        list.add(Tuple3.of(3, 2L, "Hello"));
        list.add(Tuple3.of(4, 3L, "Hello"));
        list.add(Tuple3.of(5, 3L, "Hello"));
        list.add(Tuple3.of(6, 3L, "hehe"));
        list.add(Tuple3.of(7, 4L, "hehe"));
        list.add(Tuple3.of(8, 4L, "hehe"));
        list.add(Tuple3.of(9, 4L, "hehe"));
        list.add(Tuple3.of(10, 4L, "hehe"));
        list.add(Tuple3.of(11, 5L, "hehe"));
        list.add(Tuple3.of(12, 5L, "hehe"));
        list.add(Tuple3.of(13, 5L, "hehe"));
        list.add(Tuple3.of(14, 5L, "hehe"));
        list.add(Tuple3.of(15, 5L, "hehe"));
        list.add(Tuple3.of(16, 6L, "hehe"));
        list.add(Tuple3.of(17, 6L, "hehe"));
        list.add(Tuple3.of(18, 6L, "hehe"));
        list.add(Tuple3.of(19, 6L, "hehe"));
        list.add(Tuple3.of(20, 6L, "hehe"));
        list.add(Tuple3.of(21, 6L, "hehe"));
        Collections.shuffle(list);//把数据打乱

        //3.Transformation
        DataSource<Tuple3<Integer, Long, String>> tupleDS = env.fromCollection(list);
        //按照String进行hash分区
        PartitionOperator<Tuple3<Integer, Long, String>> partitionByHash = tupleDS.partitionByHash(2);
        //按照Integer的范围进行分区
        PartitionOperator<Tuple3<Integer, Long, String>> partitionByRange = tupleDS.partitionByRange(0);
        //按照Integer自定义分区,奇数放一起,偶数放一起
        PartitionOperator<Tuple3<Integer, Long, String>> partitionCustom = tupleDS.partitionCustom(new Partitioner<Integer>() {
            @Override
            public int partition(Integer k, int i) {
                return k % 2;
            }
        }, 0);
        //按照Integer进行分区降序排列
        SortPartitionOperator<Tuple3<Integer, Long, String>> partitionBysort = tupleDS.sortPartition(0, Order.DESCENDING);

        //4.Sink
        partitionByHash.writeAsText("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\output\\partitionByHash", FileSystem.WriteMode.OVERWRITE);
        partitionByRange.writeAsText("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\output\\partitionByRange", FileSystem.WriteMode.OVERWRITE);
        partitionCustom.writeAsText("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\output\\partitionCustom", FileSystem.WriteMode.OVERWRITE);
        partitionBysort.writeAsText("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\output\\partitionBysort", FileSystem.WriteMode.OVERWRITE);
        //5.execute
        env.execute();
    }
}

```

### 5.4 Sink

> 1.ds.print 直接输出到控制台<br/>
> 2.ds.printToErr() 直接输出到控制台,用红色<br/>
> 3.ds.collect 将分布式数据收集为本地集合<br/>
> 4.ds.setParallelism(1).writeAsText("本地/HDFS的path",WriteMode.OVERWRITE)

> 注意:
>> 在输出到path的时候,可以在前面设置并行度,如果
>> 并行度>1,则path为目录
>> 并行度=1,则path为文件名

> 代码实现

```
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.core.fs.FileSystem;

import java.util.List;

/**
 * Author xiaoma
 * Date 2020/10/14 9:32
 * Desc
 * 1.ds.print 直接输出到控制台
 * 2.ds.printToErr() 直接输出到控制台,用红色
 * 3.ds.collect 将分布式数据收集为本地集合
 * 4.ds.setParallelism(1).writeAsText("本地/HDFS的path",WriteMode.OVERWRITE)
 */
public class SinkDemo {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        //2.source
        //DataSet<String> ds = env.fromElements("hadoop", "flink");//读可变参时并行度为1
        DataSource<String> ds = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\words.txt");//读文件时并行和机器配置有关

        //3.transformation
        //4.sink
        ds.print();
        ds.printToErr();
        List<String> list = ds.collect();//将分布式的ds收集为本机集合
        System.out.println(list);
        ds.setParallelism(1).writeAsText("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\output\\test", FileSystem.WriteMode.OVERWRITE);
        //注意:
        //1.读可变参时并行度为1
        //2.读文件时并行和机器配置有关
        //3.写的时候,Parallelism=1为文件
        //4.写的时候,Parallelism>1为文件夹


        //5.execute
        env.execute();
    }
}

```

### 5.5 累加器

> Flink累加器：
>> Flink中的累加器，与Mapreduce counter的应用场景类似，可以很好地观察task在运行期间的数据变化，如在Flink job任务中的算子函数中操作累加器，在任务执行结束之后才能获得累加器的最终结果。
>> Flink有以下内置累加器，每个累加器都实现了Accumulator接口。
>>> IntCounter
>>> LongCounter
>>> DoubleCounter

> 编码步骤:
>> 1.创建累加器
>>> private IntCounter numLines = new IntCounter();
> 
>> 2.注册累加器
>>> getRuntimeContext().addAccumulator("num-lines", this.numLines);
>
>> 3.使用累加器
>>> this.numLines.add(1);
>
>> 4.获取累加器的结果
>>> myJobExecutionResult.getAccumulatorResult("num-lines")

> 代码实现

```
import org.apache.flink.api.common.JobExecutionResult;
import org.apache.flink.api.common.accumulators.IntCounter;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.core.fs.FileSystem;

/**
 * Author xiaoma
 * Date 2020/10/14 9:46
 * Desc
 * 需求:
 * 不使用累加器和使用累加器来统计Flink程序处理了多少条数据!
 * API
 * 1.创建累加器
 * private IntCounter numLines = new IntCounter();
 * 2.注册累加器
 * getRuntimeContext().addAccumulator("num-lines", this.numLines);
 * 3.使用累加器
 * this.numLines.add(1);
 * 4.获取累加器的结果
 * myJobExecutionResult.getAccumulatorResult("num-lines")
 *
 */
public class OtherAPI_Accumulator {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataSource<String> dataDS = env.fromElements("aaa", "bbb", "ccc", "ddd");
        //3.Transformation
        //不使用累加器和使用累加器来统计Flink程序处理了多少条数据!
        MapOperator<String, String> result = dataDS.map(new RichMapFunction<String, String>() {
            private int count = 0;
            //-1.创建累加器
            private IntCounter myCounter = new IntCounter();
            //open方法执行执行一次
            @Override
            public void open(Configuration parameters) throws Exception {
                //-2.注册累加器
                getRuntimeContext().addAccumulator("myCounter", myCounter);
            }

            @Override
            public String map(String value) throws Exception {
                count++;
                System.out.println("不使用累加器统计的处理条数:" + count);
                //-3.使用累加器
                myCounter.add(1);
                return value;
            }
        }).setParallelism(2);

        //4.sink
        result.writeAsText("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\output\\result", FileSystem.WriteMode.OVERWRITE);
        //5.execute
        JobExecutionResult jobExecutionResult = env.execute();
        //-4.获取累加器结果
        Object myCounterResult = jobExecutionResult.getAccumulatorResult("myCounter");
        System.out.println("使用累加器统计的处理条数:" + myCounterResult);
    }
}


不使用累加器统计的处理条数:1
不使用累加器统计的处理条数:1
不使用累加器统计的处理条数:2
不使用累加器统计的处理条数:2

使用累加器统计的处理条数:4
```

### 5.6 广播变量

> Flink支持广播。可以将数据广播到TaskManager上就可以供TaskManager中的SubTask/task去使用，数据存储到内存中。这样可以减少大量的shuffle操作，而不需要多次传递给集群节点；

> 比如在数据join阶段，不可避免的就是大量的shuffle操作，我们可以把其中一个dataSet广播出去，一直加载到taskManager的内存中，可以直接在内存中拿数据，避免了大量的shuffle，导致集群性能下降；

> 图解：
>> 可以理解广播就是一个公共的共享变量
>> 将一个数据集广播后，不同的Task都可以在节点上获取到
>> 每个节点只存一份
>> 如果不使用广播，每一个Task都会拷贝一份数据集，造成内存资源浪费

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/84.png)

> 注意：
>> 广播变量是要把dataset广播到内存中，所以广播的数据量不能太大，否则会出现OOM
>> 广播变量的值不可修改，这样才能确保每个节点获取到的值都是一致的

> 编码步骤:
>> 1：广播数据
>>>.withBroadcastSet(DataSet, "name");
>
>> 2：获取广播的数据
>>> Collection<> broadcastSet = 	getRuntimeContext().getBroadcastVariable("name");
>
>> 3:使用广播数据

> 需求:
>> 将studentDS(学号,姓名)集合广播出去(广播到各个TaskManager内存中)
>> 然后使用scoreDS(学号,学科,成绩)和广播数据(学号,姓名)进行关联,得到这样格式的数据:(姓名,学科,成绩)

> 代码实现

```
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.common.operators.base.JoinOperatorBase;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.operators.JoinOperator;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.configuration.Configuration;

import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Author xiaoma
 * Date 2020/10/14 9:46
 * Desc
 * 需求:
 * 不使用广播和使用广播完成如下需求:
 * 将studentDS(学号,姓名)集合广播出去(广播到各个TaskManager内存中)
 * 然后使用scoreDS(学号,学科,成绩)和广播数据(学号,姓名)进行关联,得到这样格式的数据:(姓名,学科,成绩)
 * 编码步骤:
 * 1：广播数据
 * .withBroadcastSet(DataSet, "name");
 * 2：获取广播的数据
 * Collection<> broadcastSet = 	getRuntimeContext().getBroadcastVariable("name");
 * 3：使用广播数据
 */
public class OtherAPI_BroadcastVariable {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        //学生数据集(学号,姓名)
        DataSource<Tuple2<Integer, String>> studentDS = env.fromCollection(
                Arrays.asList(Tuple2.of(1, "张三"), Tuple2.of(2, "李四"), Tuple2.of(3, "王五"))
        );

        //成绩数据集(学号,学科,成绩)
        DataSource<Tuple3<Integer, String, Integer>> scoreDS = env.fromCollection(
                Arrays.asList(Tuple3.of(1, "语文", 50), Tuple3.of(2, "数学", 70), Tuple3.of(3, "英文", 86))
        );

        //3.Transformation
        //需求:
        // * 不使用广播和使用广播完成如下需求:
        // * 将studentDS(学号,姓名)集合广播出去(广播到各个TaskManager内存中)
        // * 然后使用scoreDS(学号,学科,成绩)和广播数据(学号,姓名)进行关联,得到这样格式的数据:(姓名,学科,成绩)
        //不使用广播
/*
 OPTIMIZER_CHOOSES：将选择权交予Flink优化器；
 BROADCAST_HASH_FIRST：广播第一个输入端，同时基于它构建一个哈希表，而第二个输入端作为探索端，选择这种策略的场景是第一个输入端规模很小；
 BROADCAST_HASH_SECOND：广播第二个输入端并基于它构建哈希表，第一个输入端作为探索端，选择这种策略的场景是第二个输入端的规模很小；
 REPARTITION_HASH_FIRST：该策略会导致两个输入端都会被重分区，但会基于第一个输入端构建哈希表。该策略适用于第一个输入端数据量小于第二个输入端的数据量，但这两个输入端的规模仍然很大，优化器也是当没有办法估算大小，没有已存在的分区以及排序顺序可被使用时系统默认采用的策略；
 REPARTITION_HASH_SECOND：该策略会导致两个输入端都会被重分区，但会基于第二个输入端构建哈希表。该策略适用于两个输入端的规模都很大，但第二个输入端的数据量小于第一个输入端的情况；
 REPARTITION_SORT_MERGE：输入端被以流的形式进行连接并合并成排过序的输入。该策略适用于一个或两个输入端都已排过序的情况；
*/
        JoinOperator.DefaultJoin<Tuple2<Integer, String>, Tuple3<Integer, String, Integer>> joinResult = studentDS.join(scoreDS, JoinOperatorBase.JoinHint.OPTIMIZER_CHOOSES).where(0).equalTo(0);
        //使用广播
        //RichMapFunction<Tuple3<学号,学科,成绩>, Tuple3<姓名,学科,成绩>>
        MapOperator<Tuple3<Integer, String, Integer>, Tuple3<String, String, Integer>> broadcastResult = scoreDS.map(new RichMapFunction<Tuple3<Integer, String, Integer>, Tuple3<String, String, Integer>>() {
            //定义一个集合用来接收广播中的数据
            //Map<id, 姓名>
            private Map<Integer, String> studentMap = new HashMap<>();

            //-2.获取广播中的数据
            @Override
            public void open(Configuration parameters) throws Exception {
                List<Tuple2<Integer, String>> studentList = getRuntimeContext().getBroadcastVariable("studentDS");
                for (Tuple2<Integer, String> t : studentList) {
                    Integer id = t.f0;
                    String name = t.f1;
                    studentMap.put(id, name);
                }
                //studentMap = studentList.stream().collect(Collectors.toMap(t -> t.f0, t -> t.f1));
            }

            @Override
            public Tuple3<String, String, Integer> map(Tuple3<Integer, String, Integer> value) throws Exception {
                //-3.使用广播中的数据
                Integer id = value.f0;
                String name = studentMap.getOrDefault(id, "未知");
                //String name1 = studentMap.get(id);
                //if(name1 ==null) name = "未知";
                //studentMap.get(id);
                return Tuple3.of(name, value.f1, value.f2);
            }
            //-1.广播数据
        }).withBroadcastSet(studentDS, "studentDS");
        //4.sink
        joinResult.print();
        broadcastResult.print();
        //5.execute
    }
}

```

### 5.7 分布式缓存 

> Flink提供了一个类似于Hadoop的分布式缓存，让并行运行实例的函数可以在本地访问。
> 这个功能可以被使用来分享外部静态的数据，例如：机器学习的逻辑回归模型等

> 注意
>> 广播变量是将变量分发到各个TaskManager节点的内存上，分布式缓存是将文件缓存到各个TaskManager节点上；

> 编码步骤:
>> 1：注册一个分布式缓存文件
>>> env.registerCachedFile("hdfs:///path/file", "cachefilename")  
> 
>> 2：访问分布式缓存文件中的数据
>>> File myFile = getRuntimeContext().getDistributedCache().getFile("cachefilename");
> 
>> 3：使用

> 需求
>> 将scoreDS(学号, 学科, 成绩)中的数据和分布式缓存中的数据(学号,姓名)关联,得到这样格式的数据: (学生姓名,学科,成绩)

> 代码实现

```
import org.apache.commons.io.FileUtils;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.configuration.Configuration;

import java.io.File;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Author xiaoma
 * Date 2020/10/14 9:46
 * Desc
 * 需求
 * 将scoreDS(学号, 学科, 成绩)中的数据和分布式缓存中的数据(学号,姓名)关联,得到这样格式的数据: (学生姓名,学科,成绩)
 * 编码步骤:
 * 1：注册一个分布式缓存文件
 *  env.registerCachedFile("hdfs:///path/file", "cachefilename")
 * 2：访问分布式缓存文件中的数据
 *  File myFile = getRuntimeContext().getDistributedCache().getFile("cachefilename");
 * 3：使用
 */
public class OtherAPI_DistributCahce {
    public static void main(String[] args) throws Exception {
        //1.env
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        //成绩数据集(学号,学科,成绩)
        DataSource<Tuple3<Integer, String, Integer>> scoreDS = env.fromCollection(
                Arrays.asList(Tuple3.of(1, "语文", 50), Tuple3.of(2, "数学", 70), Tuple3.of(3, "英文", 86))
        );
        //-1.注册一个分布式缓存文件
        env.registerCachedFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\distribute_cache_student", "studentFile");

        //3.Transformation
        MapOperator<Tuple3<Integer, String, Integer>, Tuple3<String, String, Integer>> result = scoreDS.map(new RichMapFunction<Tuple3<Integer, String, Integer>, Tuple3<String, String, Integer>>() {
            //定义一个集合用来接收广播中的数据
            //Map<id, 姓名>
            private Map<Integer, String> studentMap = new HashMap<>();

            @Override
            public void open(Configuration parameters) throws Exception {
                //-2.访问分布式缓存文件中的数据
                File file = getRuntimeContext().getDistributedCache().getFile("studentFile");
                List<String> lines = FileUtils.readLines(file);
                for (String line : lines) {
                    String[] arr = line.split(",");
                    Integer id = Integer.valueOf(arr[0]);
                    String name = arr[1];
                    studentMap.put(id, name);
                }
            }

            @Override
            public Tuple3<String, String, Integer> map(Tuple3<Integer, String, Integer> value) throws Exception {
                //3：使用
                Integer id = value.f0;
                String name = studentMap.getOrDefault(id, "未知");
                //studentMap.get(id);
                return Tuple3.of(name, value.f1, value.f2);
            }
        });

        //4.sink
        result.print();

        //5.execute
    }
}

```
## 6 Flink流处理

### 6.1 流处理相关概念

> 数据的时效性
>> 日常工作中，我们一般会先把数据存储在`表`，然后对表的数据进行`加工`、`分析`。既然先存储在表中，那就会涉及到时效性概念。
如果我们处理以年，月为单位的级别的数据处理，进行`统计分析`，`个性化推荐`，那么数据的的最新日期离当前有`几个甚至上月`都没有问题。但是如果我们处理的是`以天为级别`，或者`一小时`甚至`更小粒度`的数据处理，那么就要求数据的时效性更高了。比如：
>>> 对网站的实时监控
>>> 对异常日志的监控
> 
>> 这些场景需要工作人员`立即响应`，这样的场景下，传统的统一收集数据，再存到数据库中，再取出来进行分析就无法满足高时效性的需求了。

> `流式计算`和`批量计算`
>  ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/85.png)
>> Batch Analytics，右边是 Streaming Analytics。批量计算: 统一收集数据->存储到DB->对数据进行批量处理，就是传统意义上使用类似于 Map Reduce、Hive、Spark Batch 等，对作业进行分析、处理、生成离线报表
>> Streaming Analytics 流式计算，顾名思义，就是对数据流进行处理，如使用流式分析引擎如 Storm，Flink 实时处理分析数据，应用较多的场景如实时大屏、实时报表。
> 它们的主要区别是：
>> 与批量计算那样慢慢积累数据不同，流式计算立刻计算，数据持续流动，计算完之后就丢弃。
>> 批量计算是维护一张表，对表进行实施各种计算逻辑。流式计算相反，是必须先定义好计算逻辑，提交到流式计算系统，这个计算作业逻辑在整个运行期间是不可更改的。
>> 计算结果上，批量计算对全部数据进行计算后传输结果，流式计算是每次小批量计算后，结果可以立刻实时化展现。

> `批处理`:就是对`历史数据/有界数据`进行处理,如前一天/前7天/前2周/前1/3/6月,前1年.....,批处理的任务特点是:跑完一次,就停止,就结束,等待下一次周期调度!而且批处理对于任务完成时间要求不高!一般几个小时内跑完都能够接受!

> `流处理`:就是实时的源源不断到来的`流式数据/无界数据`进行处理!如:实时统计最近5s/1min/1h的数据,特点是程序启动之后会==一直运行==,等待数据到来!除非遇到异常或手动停止!

> `问题:流处理简单还是批处理简单?`
>> 批处理简单!
>> 流处理复杂,对时间要求高!对容错要求也高!对吞吐量要求也高!....

### 6.2 API和编程模型

> 和批处理类似，Flink的流处理也支持多个层次的API并包含三个主要部分:`Source`/`Transformation`/`Sink`

>  ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/86.png)

### 6.3 Source

#### 6.3.1 基于Socket的Source-入门案例

> 般用于学习测试

> 需求:
>> 1.在node1上使用nc -lk 9999 向指定端口发送数据
>> nc是netcat的简称，原本是用来设置路由器,我们可以利用它向某个端口发送数据

> 如果没有该命令可以下安装

```
yum install -y nc
```
> 2.使用Flink编写流处理应用程序实时统计单词数量

```
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/**
 * Author xiaoma
 * Date 2020/10/14 11:41
 * Desc
 * 使用Flink流处理程序获取node1:9999发送的实时数据并做WordCount!
 */
public class SourceDemo01_Socket_WordCount {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //2.source
        DataStream<String> linesDS = env.socketTextStream("node1", 9999);

        //3.Transformation
        //3.1切割出每个单词并直接记为1
        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOneDS = linesDS.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                //value就是每一行
                String[] words = value.split(" ");
                for (String word : words) {
                    out.collect(Tuple2.of(word, 1));
                }
            }
        });
        //3.2分组
        //注意:批处理的分组是groupBy,流处理的分组是keyBy
        KeyedStream<Tuple2<String, Integer>, Tuple> groupedDS = wordAndOneDS.keyBy(0);
        //3.3聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> result = groupedDS.sum(1);

        //4.sink
        result.print();

        //5.execute
        env.execute();
    }
}
```

#### 6.3.2 基于集合的Source

> 和批处理类似

```
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.Arrays;

/**
 * Author xiaoma
 * Date 2020/10/14 11:41
 * Desc
 * 基于文件和集合的流处理Source-一般用于学习测试和批处理的API类似
 */
public class SourceDemo02 {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataStreamSource<String> ds1 = env.fromElements("hadoop", "spark");
        DataStreamSource<String> ds2 = env.fromCollection(Arrays.asList("hadoop", "spark"));//.setParallelism(2);
        DataStreamSource<Long> ds3 = env.generateSequence(1, 10);
        DataStreamSource<String> ds4 = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\words.txt");

        //3.Transformation

        //4.Sink
        ds1.print();
        ds2.print();
        ds3.print();
        ds4.print();

        //5.execute
        env.execute();
    }
}
```

#### 6.3.3 基于文件的Source

> 和批处理类似

#### 6.3.4 自定义Source-随机生成数据

> API
>> 一般用于学习测试,模拟生成一些数据
>> Flink还提供了数据源接口,我们实现该接口就可以实现自定义数据源，不同的接口有不同的功能，分类如下：
>>> `SourceFunction`:非并行数据源(并行度只能=1)
>>> `RichSourceFunction`:多功能非并行数据源(并行度只能=1)
>>> `ParallelSourceFunction`:并行数据源(并行度能够>=1)
>>> `RichParallelSourceFunction`:多功能并行数据源(并行度能够>=1)--后续学习的Kafka数据源使用的就是该接口

> 需求
>> 每隔1秒随机生成一条订单信息(订单ID、用户ID、订单金额、时间戳)
> 要求:
>> 随机生成订单ID(UUID)
>> 随机生成用户ID(0-2)
>> 随机生成订单金额(0-100)
>> 时间戳为当前系统时间

> 代码实现

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.RichParallelSourceFunction;

import java.util.Random;
import java.util.UUID;

/**
 * Author xiaoma
 * Date 2020/10/14 11:41
 * Desc
 *需求
 * 每隔1秒随机生成一条订单信息(订单ID、用户ID、订单金额、时间戳)
 * 要求:
 * - 随机生成订单ID(UUID)
 * - 随机生成用户ID(0-2)
 * - 随机生成订单金额(0-100)
 * - 时间戳为当前系统时间
 *
 * API
 * 一般用于学习测试,模拟生成一些数据
 * Flink还提供了数据源接口,我们实现该接口就可以实现自定义数据源，不同的接口有不同的功能，分类如下：
 * SourceFunction:非并行数据源(并行度只能=1)
 * RichSourceFunction:多功能非并行数据源(并行度只能=1)
 * ParallelSourceFunction:并行数据源(并行度能够>=1)
 * RichParallelSourceFunction:多功能并行数据源(并行度能够>=1)--后续学习的Kafka数据源使用的就是该接口
 */
public class SourceDemo03_Customer {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataStream<Order> orderDS = env
                .addSource(new MyOrderSource())
                .setParallelism(1);

        //3.Transformation

        //4.Sink
        orderDS.print();
        //5.execute
        env.execute();
    }
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class Order {
        private String id;
        private Integer userId;
        private Integer money;
        private Long createTime;
    }
    public static class MyOrderSource extends RichParallelSourceFunction<Order> {
        private Boolean flag = true;
        @Override
        public void run(SourceContext<Order> ctx) throws Exception {
            Random random = new Random();
            while (flag){
                Thread.sleep(1000);
                String id = UUID.randomUUID().toString();
                int userId = random.nextInt(3);
                int money = random.nextInt(101);
                long createTime = System.currentTimeMillis();
                ctx.collect(new Order(id,userId,money,createTime));
            }
        }
        //取消任务/执行cancle命令的时候执行
        @Override
        public void cancel() {
            flag = false;
        }
    }
}

```

#### 6.3.5 自定义Source-MySQL

> 需求:
>> 实际开发中,经常会实时接收一些数据,要和MySQL中存储的一些规则进行匹配,那么这时候就可以使用Flink自定义数据源从MySQL中读取数据

> 那么现在先完成一个简单的需求:
>> 从MySQL中实时加载数据
>> 要求MySQL中的数据有变化,也能被实时加载出来

> 准备数据

```
CREATE TABLE `t_student` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(255) DEFAULT NULL,
`age` int(11) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;

INSERT INTO `t_student` VALUES ('1', 'jack', '18');
INSERT INTO `t_student` VALUES ('2', 'tom', '19');
INSERT INTO `t_student` VALUES ('3', 'rose', '20');
INSERT INTO `t_student` VALUES ('4', 'tom', '19');
INSERT INTO `t_student` VALUES ('5', 'jack', '18');
INSERT INTO `t_student` VALUES ('6', 'rose', '20');
```

> 代码实现:

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.RichParallelSourceFunction;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.concurrent.TimeUnit;

/**
 * Author xiaoma
 * Date 2020/10/14 11:41
 * Desc
 * 需求:
 * 实际开发中,经常会实时接收一些数据,要和MySQL中存储的一些规则进行匹配,那么这时候就可以使用Flink自定义数据源从MySQL中读取数据
 * 那么现在先完成一个简单的需求:
 * 从MySQL中实时加载数据
 * 要求MySQL中的数据有变化,也能被实时加载出来
 */
public class SourceDemo04_Customer_MySQL {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataStream<Student> studentDS = env.addSource(new MySQLSource()).setParallelism(1);

        //3.Transformation

        //4.Sink
        studentDS.print();

        //5.execute
        env.execute();
    }

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class Student {
        private Integer id;
        private String name;
        private Integer age;
    }

    public static class MySQLSource extends RichParallelSourceFunction<Student> {
        private Connection conn = null;
        private PreparedStatement ps = null;

        @Override
        public void open(Configuration parameters) throws Exception {
            //加载驱动,开启连接
            //Class.forName("com.mysql.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://192.168.88.161:3306/bigdata", "root", "123456");
            String sql = "select id,name,age from t_student";
            ps = conn.prepareStatement(sql);
        }

        private boolean flag = true;

        @Override
        public void run(SourceContext<Student> ctx) throws Exception {
            while (flag) {
                ResultSet rs = ps.executeQuery();
                while (rs.next()) {
                    int id = rs.getInt("id");
                    String name = rs.getString("name");
                    int age = rs.getInt("age");
                    ctx.collect(new Student(id, name, age));
                }
                TimeUnit.SECONDS.sleep(5);
            }
        }
        @Override
        public void cancel() {
            flag = false;
        }
        @Override
        public void close() throws Exception {
            if (conn != null) conn.close();
            if (ps != null) ps.close();
        }
    }
}

```

#### 6.3.6 自定义Source-Kafka-官方提供

##### 6.3.6.1 API及其版本

> Flink 里已经提供了一些绑定的 Connector，例如 kafka source 和 sink，Es sink 等。读写 kafka、es、rabbitMQ 时可以直接使用相应 connector 的 api 即可，虽然该部分是 Flink 项目源代码里的一部分，但是真正意义上不算作 Flink 引擎相关逻辑，并且该部分没有打包在二进制的发布包里面。所以在提交 Job 时候需要注意， job 代码 jar 包中一定要将相应的 connetor 相关类打包进去，否则在提交作业时就会失败，提示找不到相应的类，或初始化某些类异常。

> https://ci.apache.org/projects/flink/flink-docs-stable/dev/connectors/kafka.html

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/87.png)

##### 6.3.6.2 参数设置

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/88.png)

> 以下参数都必须/建议设置上
>> 1.订阅的主题
>> 2.反序列化规则
>> 3.消费者属性-集群地址
>> 4.消费者属性-消费者组id(如果不设置,会有默认的,但是默认的不方便管理)
>> 5.消费者属性-offset重置规则,如earliest/latest...
>> 6.动态分区检测

##### 6.3.6.3 参数说明

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/89.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/90.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/91.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/92.png)

> 实际的生产环境中可能有这样一些需求，比如：
>> 场景一：有一个 Flink 作业需要将五份数据聚合到一起，五份数据对应五个 kafka topic，随着业务增长，新增一类数据，同时新增了一个 kafka topic，如何在不重启作业的情况下作业自动感知新的 topic。
>> 场景二：作业从一个固定的 kafka topic 读数据，开始该 topic 有 10 个 partition，但随着业务的增长数据量变大，需要对 kafka partition 个数进行扩容，由 10 个扩容到 20。该情况下如何在不重启作业情况下动态感知新扩容的 partition？

> 针对上面的两种场景，首先需要在构建 FlinkKafkaConsumer 时的 properties 中设置 flink.partition-discovery.interval-millis 参数为非负值，表示开启动态发现的开关，以及设置的时间间隔。此时 FlinkKafkaConsumer 内部会启动一个单独的线程定期去 kafka 获取最新的 meta 信息。
>> 针对场景一，还需在构建 FlinkKafkaConsumer 时，topic 的描述可以传一个正则表达式描述的 pattern。每次获取最新 kafka meta 时获取正则匹配的最新 topic 列表。
>> 针对场景二，设置前面的动态发现参数，在定期获取 kafka 最新 meta 信息时会匹配新的 partition。为了保证数据的正确性，新发现的 partition 从最早的位置开始读取。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/93.png)

> `注意`:
>> 开启 checkpoint 时 offset 是 Flink 通过状态 state 管理和恢复的，并不是从 kafka 的 offset 位置恢复。在 checkpoint 机制下，作业从最近一次checkpoint 恢复，本身是会回放部分历史数据，导致部分数据重复消费，Flink 引擎仅保证计算状态的精准一次，要想做到端到端精准一次需要依赖一些幂等的存储系统或者事务操作。

##### 6.3.6.4 Kafka命令

> 查看当前服务器中的所有topic

```
/export/servers/kafka/bin/kafka-topics.sh --list --zookeeper  node1:2181
```

> 创建topic

```
/export/servers/kafka/bin/kafka-topics.sh --create --zookeeper node1:2181 --replication-factor 2 --partitions 3 --topic flink_kafka
```

> 查看某个Topic的详情

```
/export/servers/kafka/bin/kafka-topics.sh --topic flink_kafka --describe --zookeeper node1:2181
```

> 删除topic

```
/export/servers/kafka/bin/kafka-topics.sh --delete --zookeeper node1:2181 --topic flink_kafka
```

> 通过shell命令发送消息

```
/export/servers/kafka/bin/kafka-console-producer.sh --broker-list node1:9092 --topic flink_kafka
```

> 通过shell消费消息

```
/export/servers/kafka/bin/kafka-console-consumer.sh --bootstrap-server node1:9092 --topic flink_kafka --from-beginning
```

> 修改分区

```
/export/servers/kafka/bin/kafka-topics.sh --alter --partitions 4 --topic flink_kafka --zookeeper node1:2181
```

##### 6.3.6.5 代码实现

```
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.util.Collector;

import java.util.Properties;

/**
 * Author xiaoma
 * Date 2020/10/14 11:41
 * Desc
 * 需求:使用flink-connector-kafka_2.11中的FlinkKafkaConsumer消费Kafka中的数据做WordCount
 * 需要设置如下参数:
 * 1.订阅的主题
 * 2.反序列化规则
 * 3.消费者属性-集群地址
 * 4.消费者属性-消费者组id(如果不设置,会有默认的,但是默认的不方便管理)
 * 5.消费者属性-offset重置规则,如earliest/latest...
 * 6.动态分区检测(当kafka的分区数变化/增加时,Flink能够检测到!)
 * 7.如果没有设置Checkpoint,那么可以设置自动提交offset,后续学习了Checkpoint会把offset随着做Checkpoint的时候提交到Checkpoint和默认主题中
 */
public class SourceDemo05_Customer_Kafka {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        Properties props  = new Properties();
        props.setProperty("bootstrap.servers", "node1:9092");
        props.setProperty("group.id", "flink");
        props.setProperty("auto.offset.reset","latest");
        props.setProperty("flink.partition-discovery.interval-millis","5000");//会开启一个后台线程每隔5s检测一下Kafka的分区情况
        props.setProperty("enable.auto.commit", "true");
        props.setProperty("auto.commit.interval.ms", "2000");
        //kafkaSource就是KafkaConsumer
        FlinkKafkaConsumer<String> kafkaSource = new FlinkKafkaConsumer<>("flink_kafka", new SimpleStringSchema(), props);
        kafkaSource.setStartFromGroupOffsets();//设置从记录的offset开始消费,如果没有记录从auto.offset.reset配置开始消费
        //kafkaSource.setStartFromEarliest();//设置直接从Earliest消费,和auto.offset.reset配置无关
        DataStreamSource<String> kafkaDS = env.addSource(kafkaSource);

        //3.Transformation
        //3.1切割出每个单词并直接记为1
        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOneDS = kafkaDS.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                //value就是每一行
                String[] words = value.split(" ");
                for (String word : words) {
                    out.collect(Tuple2.of(word, 1));
                }
            }
        });
        //3.2分组
        //注意:批处理的分组是groupBy,流处理的分组是keyBy
        KeyedStream<Tuple2<String, Integer>, Tuple> groupedDS = wordAndOneDS.keyBy(0);
        //3.3聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> result = groupedDS.sum(1);

        //4.Sink
        result.print();

        //5.execute
        env.execute();
    }
}

```

### 6.4 Transformation

> 官网API

[https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/stream/operators/](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/stream/operators/)

#### 6.4.1 说明

> 流处理的很多API和批处理类似，也包括一系列的Transformation操作，如map、flatMap、filter、sum、reduce……等等，所以这些类似的就不再一一讲解了。主要讲解，和批处理不一样的一些操作。

> 整体来说，流式数据上的操作可以分为四类。
>> 第一类是对于单条记录的操作，比如筛除掉不符合要求的记录（Filter 操作），或者将每条记录都做一个转换（Map 操作） 
>> 第二类是对多条记录的操作。比如说统计一个小时内的订单总成交量，就需要将一个小时内的所有订单记录的成交量加到一起。为了支持这种类型的操作，就得通过 Window 将需要的记录关联到一起进行处理
>> 第三类是对多个流进行操作并转换为单个流。例如，多个流可以通过 Union、Join 或 Connect 等操作合到一起。这些操作合并的逻辑不同，但是它们最终都会产生了一个新的统一的流，从而可以进行一些跨流的操作。
>> 最后， DataStream 还支持与合并对称的拆分操作，即把一个流按一定规则拆分为多个流（Split 操作），每个流是之前流的一个子集，这样我们就可以对不同的流作不同的处理。

#### 6.4.2 API

> 官网

[https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/stream/operators/](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/stream/operators/)

> 中文

Transformation | Description
---|---
`Map`<br/>DataStream→DataStream | Takes one element and produces one element. A map function that doubles the values of the input stream:<br/>dataStream.map { x => x * 2 }
`FlatMap`<br/>DataStream→DataStream | 采用一个数据元并生成零个，一个或多个数据元。将句子分割为单词的flatmap函数：<br/>dataStream.flatMap { str => str.split(" ") }
`Filter`<br/>DataStream→DataStream | 计算每个数据元的布尔函数，并保存函数返回true的数据元。过滤掉零值的过滤器：<br/>dataStream.filter { _ != 0 }
`KeyBy`<br/>DataStream→KeyedStream | 逻辑上将流分区为不相交的分区。具有相同Keys的所有记录都分配给同一分区。在内部，keyBy()是使用散列分区实现的。指定键有不同的方法。<br/>此转换返回KeyedStream，其中包括使用被Keys化状态所需的KeyedStream。<br/>dataStream.keyBy("someKey") // Key by field "someKey"<br/>dataStream.keyBy(0) // Key by the first element of a Tuple
`Reduce`<br/>KeyedStream→DataStream | 被Keys化数据流上的“滚动”Reduce。将当前数据元与最后一个Reduce的值组合并发出新值。<br/>reduce函数，用于创建部分和的流：<br/>keyedStream.reduce { _ + _ }  
`Fold`<br/>KeyedStream→DataStream | 具有初始值的被Keys化数据流上的“滚动”折叠。将当前数据元与最后折叠的值组合并发出新值。<br/>折叠函数，当应用于序列(1,2,3,4,5)时，发出序列“start-1”，“start-1-2”，“start-1-2-3”,. ..<br/>val result: DataStream[String] =  keyedStream.fold("start")((str, i) => { str + "-" + i })   
`Aggregations`<br/>KeyedStream→DataStream | 在被Keys化数据流上滚动聚合。min和minBy之间的差异是min返回最小值，而minBy返回该字段中具有最小值的数据元(max和maxBy相同)。<br/>keyedStream.sum(0);<br/>keyedStream.sum("key");<br/>keyedStream.min(0);<br/>keyedStream.min("key");<br/>keyedStream.max(0);<br/>keyedStream.max("key");<br/>keyedStream.minBy(0);<br/>keyedStream.minBy("key");<br/>keyedStream.maxBy(0);<br/>keyedStream.maxBy("key");    
`Window`<br/>KeyedStream→WindowedStream | 可以在已经分区的KeyedStream上定义Windows。Windows根据某些特征(例如，在最后5秒内到达的数据)对每个Keys中的数据进行分组。有关窗口的完整说明，请参见windows。<br/>dataStream.keyBy(0).window(TumblingEventTimeWindows.of(Time.seconds(5))); // Last 5 seconds of data   
`WindowAll`<br/>DataStream→AllWindowedStream | Windows可以在常规DataStream上定义。Windows根据某些特征(例如，在最后5秒内到达的数据)对所有流事件进行分组。有关窗口的完整说明，请参见windows。<br/>警告：在许多情况下，这是非并行转换。所有记录将收集在windowAll 算子的一个任务中。<br/>dataStream.windowAll(TumblingEventTimeWindows.of(Time.seconds(5))); // Last 5 seconds of data  
`Window Apply`<br/>WindowedStream→DataStream<br/>AllWindowedStream→DataStream | 将一般函数应用于整个窗口。下面是一个手动求和窗口数据元的函数。<br/>注意：如果您正在使用windowAll转换，则需要使用AllWindowFunction。<br/>windowedStream.apply { WindowFunction }// applying an AllWindowFunction on non-keyed window stream<br/>allWindowedStream.apply { AllWindowFunction }
`Window Reduce`<br/>WindowedStream→DataStream | 将函数缩减函数应用于窗口并返回缩小的值。<br/>windowedStream.reduce { _ + _ }
`Window Fold`<br/>WindowedStream→DataStream | 将函数折叠函数应用于窗口并返回折叠值。示例函数应用于序列(1,2,3,4,5)时，将序列折叠为字符串“start-1-2-3-4-5”：<br/>val result: DataStream[String] = windowedStream.fold("start", (str, i) => { str + "-" + i })
`Windows上的聚合`<br/>WindowedStream→DataStream | 聚合窗口的内容。min和minBy之间的差异是min返回最小值，而minBy返回该字段中具有最小值的数据元(max和maxBy相同)。<br/>windowedStream.sum(0);windowedStream.sum("key");windowedStream.min(0);windowedStream.min("key");windowedStream.max(0);windowedStream.max("key");windowedStream.minBy(0);windowedStream.minBy("key");windowedStream.maxBy(0);windowedStream.maxBy("key");    
`Union`<br/>DataStream *→DataStream | 两个或多个数据流的联合，创建包含来自所有流的所有数据元的新流。注意：如果将数据流与自身联合，则会在结果流中获取两次数据元。<br/>dataStream.union(otherStream1, otherStream2, ...);    
`Window Join`<br/>DataStream，DataStream→DataStream | 在给定Keys和公共窗口上连接两个数据流。<br/>dataStream.join(otherStream)<br/>.where(<key selector>).equalTo(<key selector>)<br/>.window(TumblingEventTimeWindows.of(Time.seconds(3)))<br/>.apply (new JoinFunction () {...});    
`Interval Join`<br/>KeyedStream，KeyedStream→DataStream | 在给定的时间间隔内使用公共Keys关联两个被Key化的数据流的两个数据元e1和e2，以便e1.timestamp + lowerBound <= e2.timestamp <= e1.timestamp + upperBound<br/>// this will join the two streams so that// key1 == key2 && leftTs - 2 < rightTs < leftTs + 2keyedStream.intervalJoin(otherKeyedStream)<br/>.between(Time.milliseconds(-2), Time.milliseconds(2)) // lower and upper bound<br/>.upperBoundExclusive(true) // optional<br/>.lowerBoundExclusive(true) // optional<br/>.process(new IntervalJoinFunction() {...});    
`Window CoGroup`<br/>DataStream，DataStream→DataStream | 在给定Keys和公共窗口上对两个数据流进行Cogroup。<br/>dataStream.coGroup(otherStream)<br/>.where(0).equalTo(1)<br/>.window(TumblingEventTimeWindows.of(Time.seconds(3)))<br/>.apply (new CoGroupFunction () {...});    
`Connect`<br/>DataStream，DataStream→ConnectedStreams | “连接”两个保存其类型的数据流。连接允许两个流之间的共享状态。<br/>DataStream<Integer> someStream = //...DataStream<String> otherStream = //...ConnectedStreams<Integer, String> connectedStreams = someStream.connect(otherStream);<br/>CoMap，CoFlatMap<br/>ConnectedStreams→DataStream	类似于连接数据流上的map和flatMap<br/>connectedStreams.map(<br/>(_ : Int) => true,<br/>(_ : String) => false)connectedStreams.flatMap(<br/>(_ : Int) => true,<br/>(_ : String) => false)
`Split`<br/>DataStream→SplitStream | 根据某些标准将流拆分为两个或更多个流。<br/>val split = someDataStream.split(<br/>(num: Int) =><br/>(num % 2) match {<br/>case 0 => List("even")<br/>case 1 => List("odd")<br/>})               
`Select`<br/>SplitStream→DataStream | 从拆分流中选择一个或多个流。<br/>SplitStream<Integer> split;DataStream<Integer> even = split.select("even");DataStream<Integer> odd = split.select("odd");DataStream<Integer> all = split.select("even","odd");               
`Iterate`<br/>DataStream→IterativeStream→DataStream | 通过将一个 算子的输出重定向到某个先前的 算子，在流中创建“反馈”循环。这对于定义不断更新模型的算法特别有用。以下代码以流开头并连续应用迭代体。大于0的数据元将被发送回反馈通道，其余数据元将向下游转发。有关完整说明，请参阅迭代。<br/>initialStream.iterate {<br/>iteration => {<br/>val iterationBody = iteration.map {/*do something*/}<br/>(iterationBody.filter(_ > 0), iterationBody.filter(_ <= 0))<br/>}}           
`Extract Timestamps`<br/>DataStream→DataStream | 从记录中提取时间戳，以便使用使用事件时间语义的窗口。查看活动时间。<br/>stream.assignTimestamps (new TimeStampExtractor() {...});

#### 6.4.3 keyBy

> 按照指定的key来对流中的数据进行分组，前面入门案例中已经演示过

> 注意:
>> 流处理中没有groupBy,而是keyBy

#### 6.4.4 union和connect

> `API`

>> `union`：
>> union算子可以合并多个同类型的数据流，并生成同类型的数据流，即可以将多个DataStream[T]合并为一个新的DataStream[T]。数据将按照先进先出（First In First Out）的模式合并，且不去重。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/94.png)

> `connect`：
>> connect提供了和union类似的功能，用来连接两个数据流，它与union的区别在于：
>> connect只能连接两个数据流，union可以连接多个数据流。
>> connect所连接的两个数据流的数据类型可以不一致，union所连接的两个数据流的数据类型必须一致。
> 
> 两个DataStream经过connect之后被转化为ConnectedStreams，ConnectedStreams会对两个流的数据应用不同的处理方法，且双流之间可以共享状态。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/95.png)

> `需求`
>> 将两个String类型的流进行union
>> 将一个String类型和一个Long类型的流进行connect

> 代码实现:

```
import org.apache.flink.streaming.api.datastream.ConnectedStreams;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.CoMapFunction;

/**
 * Author xiaoma
 * Date 2020/10/14 16:19
 * Desc
 * 流处理中不光有union合并还有connect连接,区别如下:
 * union合并同类型的多个流数据
 * connect可以连接不同类型的两个流
 * 需求
 * 将两个String类型的流进行合并用union
 * 将一个String类型和一个Long类型的流进行连接用connect
 */
public class TransformationDemo01_union_connect {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataStreamSource<String> ds1 = env.fromElements("hadoop", "spark", "flink");
        DataStreamSource<String> ds2 = env.fromElements("hadoop", "spark", "hive");
        DataStreamSource<Long> ds3 = env.fromElements(1L, 2L, 3L);

        //3.Transformation
        DataStream<String> result1 = ds1.union(ds2);//合并但不去重
        ConnectedStreams<String, Long> tempResult = ds1.connect(ds3);
        //interface CoMapFunction<IN1, IN2, OUT>
        SingleOutputStreamOperator<String> result2 = tempResult.map(new CoMapFunction<String, Long, String>() {
            @Override
            public String map1(String value) throws Exception {
                return "String->String:" + value;
            }

            @Override
            public String map2(Long value) throws Exception {
                return "Long->String:" + value.toString();
            }
        });

        //4.Sink
        result1.print();
        result2.print();

        //5.execute
        env.execute();
    }
}

```

#### 6.4.5 split和select 

> API
>> `Split`就是将一个流分成多个流
>> `Select`就是获取分流后对应的数据

> `需求`:
>> 对流中的数据按照奇数和偶数进行分流，并获取分流后的数据

> 代码实现:

```
import org.apache.flink.streaming.api.collector.selector.OutputSelector;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SplitStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.ArrayList;

/**
 * Author xiaoma
 * Date 2020/10/14 16:19
 * Desc
 *  API
 * Split就是将一个流分成多个流
 * Select就是获取分流后对应的数据
 *  需求:
 * 对流中的数据按照奇数和偶数进行分流/切割，并获取分流/切割后的数据
 */
public class TransformationDemo02_split_select {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataStreamSource<Integer> ds = env.fromElements(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        //3.Transformation
        SplitStream<Integer> splitResult = ds.split(new OutputSelector<Integer>() {
            @Override
            public Iterable<String> select(Integer value) {
                //value是进来的数字
                if (value % 2 == 0) {
                    //偶数
                    ArrayList<String> list = new ArrayList<>();
                    list.add("偶数");
                    return list;
                } else {
                    //奇数
                    ArrayList<String> list = new ArrayList<>();
                    list.add("奇数");
                    return list;
                }
            }
        });
        DataStream<Integer> evenResult = splitResult.select("偶数");
        DataStream<Integer> oddResult = splitResult.select("奇数");

        //4.Sink
        evenResult.print("偶数");
        oddResult.print("奇数");

        //5.execute
        env.execute();
    }
}
```

```
import org.apache.flink.api.common.typeinfo.TypeInformation;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.util.Collector;
import org.apache.flink.util.OutputTag;

/**
 * Author xiaoma
 * Date 2020/10/14 16:19
 * Desc
 * OutputSelector
 * 是源码中推荐的用来取代split和selet的相关API,有:
 * OutputTag、process、getSideOutput
 * 需求:
 * 对流中的数据按照奇数和偶数进行分流/切割，并获取分流/切割后的数据
 */
public class TransformationDemo03_OutputSelector {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataStreamSource<Integer> ds = env.fromElements(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        //3.Transformation
        /*SplitStream<Integer> splitResult = ds.split(new OutputSelector<Integer>() {
            @Override
            public Iterable<String> select(Integer value) {
                //value是进来的数字
                if (value % 2 == 0) {
                    //偶数
                    ArrayList<String> list = new ArrayList<>();
                    list.add("偶数");
                    return list;
                } else {
                    //奇数
                    ArrayList<String> list = new ArrayList<>();
                    list.add("奇数");
                    return list;
                }
            }
        });
        DataStream<Integer> evenResult = splitResult.select("偶数");
        DataStream<Integer> oddResult = splitResult.select("奇数");
        evenResult.print("偶数");
        oddResult.print("奇数");*/

        //定义两个输出标签
        OutputTag<Integer> tag_even = new OutputTag<Integer>("偶数", TypeInformation.of(Integer.class));
        OutputTag<Integer> tag_odd = new OutputTag<Integer>("奇数"){};
        //对ds中的数据进行处理
        SingleOutputStreamOperator<Integer> tagResult = ds.process(new ProcessFunction<Integer, Integer>() {
            @Override
            public void processElement(Integer value, Context ctx, Collector<Integer> out) throws Exception {
                if (value % 2 == 0) {
                    //偶数
                    ctx.output(tag_even, value);
                } else {
                    //奇数
                    ctx.output(tag_odd, value);
                }
            }
        });

        //取出标记好的数据
        DataStream<Integer> evenResult = tagResult.getSideOutput(tag_even);
        DataStream<Integer> oddResult = tagResult.getSideOutput(tag_odd);

        //4.Sink
        evenResult.print("偶数");
        oddResult.print("奇数");

       /* SingleOutputStreamOperator<Integer> evenResult2 = ds.filter(new FilterFunction<Integer>() {
            @Override
            public boolean filter(Integer value) throws Exception {
                return value % 2 == 0;
            }
        });
        SingleOutputStreamOperator<Integer> oddResult2 = ds.filter(new FilterFunction<Integer>() {
            @Override
            public boolean filter(Integer value) throws Exception {
                return value % 2 != 0;
            }
        });

        evenResult2.print("偶数");
        oddResult2.print("奇数");*/

        //5.execute
        env.execute();
    }
}
```

#### 6.4.6 分区

> API

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/96.png)

> 说明:
>> recale分区。基于上下游Operator的并行度，将记录以循环的方式输出到下游Operator的每个实例。

> 举例:
>> 上游并行度是2，下游是4，则上游一个并行度以循环的方式将记录输出到下游的两个并行度上;上游另一个并行度以循环的方式将记录输出到下游另两个并行度上。若上游并行度是4，下游并行度是2，则上游两个并行度将记录输出到下游一个并行度上；上游另两个并行度将记录输出到下游另一个并行度上。

> 需求:
>> 对流中的元素使用各种分区,并输出

> 代码实现

```
import org.apache.flink.api.common.functions.Partitioner;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

/**
 * Author xiaoma
 * Date 2020/10/14 16:19
 * Desc
 * global();全部发往第一个
 * broadcast();广播
 * forward();前后并行度一样的时候一对一转发
 * shuffle();随机打乱
 * rebalance();重平衡
 * rescale();本地轮流分区,如前2后4,前1->,如前4后2,前2->1
 * partitionCustom:自定义分区
 */
public class TransformationDemo04_partition {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataStreamSource<Tuple2<Integer, String>> ds = env.fromElements(Tuple2.of(1, "1"), Tuple2.of(2, "2"), Tuple2.of(3, "3"), Tuple2.of(4, "4"));
        int parallelism = ds.getParallelism();
        System.out.println(parallelism);
        //3.Transformation
        DataStream<Tuple2<Integer, String>> result1 = ds.global();
        DataStream<Tuple2<Integer, String>> result2 = ds.broadcast();
        DataStream<Tuple2<Integer, String>> result3 = ds.forward();
        DataStream<Tuple2<Integer, String>> result4 = ds.shuffle();
        DataStream<Tuple2<Integer, String>> result5 = ds.rebalance();
        DataStream<Tuple2<Integer, String>> result6 = ds.rescale();
        DataStream<Tuple2<Integer, String>> result7 = ds.partitionCustom(new Partitioner<Integer>() {
            @Override
            public int partition(Integer key, int numPartitions) {
                return key % 2 == 0 ? 0 : 1;
            }
        }, 0);

        //4.sink
        result1.print();
        result2.print();
        result3.print().setParallelism(1);
        result4.print();
        result5.print();
        result6.print();
        result7.print();

        //5.execute
        env.execute();
    }
}

```

### 6.5 Sink

#### 6.5.1 基于控制台和文件的Sink

> 直接参考批处理的API即可--学习测试会使用,开发中更多的是数据实时处理统计分析完之后存入MySQL/Kafka/Redis/HBase......

#### 6.5.2 自定义Sink-MySQL

> 将Flink集合中的数据通过自定义Sink保存到MySQL

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.RichSinkFunction;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;

/**
 * Author xiaoma
 * Date 2020/10/15 9:33
 * Desc
 * 使用自定义sink将数据保存到MySQL
 */
public class SinkDemo01_MySQL {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataStreamSource<Student> studentDS = env.fromElements(new Student(null, "tonyma", 18));
        //3.Transformation

        //4.Sink
        studentDS.addSink(new MySQLSink());

        //5.execute
        env.execute();
    }
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class Student {
        private Integer id;
        private String name;
        private Integer age;
    }

    public static class MySQLSink extends RichSinkFunction<Student> {
        private Connection conn = null;
        private PreparedStatement ps = null;

        @Override
        public void open(Configuration parameters) throws Exception {
            //加载驱动,开启连接
            //Class.forName("com.mysql.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://192.168.88.161:3306/bigdata", "root", "123456");
            String sql = "INSERT INTO `t_student` (`id`, `name`, `age`) VALUES (null, ?, ?)";
            ps = conn.prepareStatement(sql);
        }

        @Override
        public void invoke(Student value, Context context) throws Exception {
            String name = value.getName();
            Integer age = value.getAge();
            //给ps中的?设置具体值
            ps.setString(1,name);
            ps.setInt(2,age);
            //执行sql
            ps.executeUpdate();
        }

        @Override
        public void close() throws Exception {
            if (conn != null) conn.close();
            if (ps != null) ps.close();
        }
    }
}

```

#### 6.5.3 自定义Sink-Kafka-官方提供

> 将Flink集合中的数据通过自定义Sink保存到Kafka

```
import com.alibaba.fastjson.JSON;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;

import java.util.Properties;

/**
 * Author xiaoma
 * Date 2020/10/15 9:33
 * Desc
 * 使用自定义sink-官方提供的flink-connector-kafka_2.11-将数据保存到Kafka
 */
public class SinkDemo02_Kafka {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataStreamSource<Student> studentDS = env.fromElements(new Student(null, "tonyma", 18));
        //3.Transformation
        //注意:目前来说我们使用Kafka使用的序列化和反序列化都是直接使用最简单的字符串,所以先将Student转为字符串
        //可以直接调用Student的toString,也可以转为JSON
        SingleOutputStreamOperator<String> studentJsonstrDS = studentDS.map(new MapFunction<Student, String>() {
            @Override
            public String map(Student value) throws Exception {
                //String str = value.toString();
                String jsonStr = JSON.toJSONString(value);
                return jsonStr;
            }
        });


        //4.Sink
        studentJsonstrDS.print();
        //根据参数创建KafkaProducer/KafkaSink
        Properties props = new Properties();
        props.setProperty("bootstrap.servers", "node1:9092");
        FlinkKafkaProducer<String> kafkaSink = new FlinkKafkaProducer<>("flink_kafka",  new SimpleStringSchema(),  props);
        studentJsonstrDS.addSink(kafkaSink);

        //5.execute
        env.execute();

        // /export/server/kafka/bin/kafka-console-consumer.sh --bootstrap-server node1:9092 --topic flink_kafka
    }
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class Student {
        private Integer id;
        private String name;
        private Integer age;
    }
}
```

#### 6.5.4 自定义Sink-Redis-官方提供

> 通过flink 操作redis 其实我们可以通过传统的redis 连接池Jpoools 进行redis 的相关操作，但是flink 提供了专门操作redis 的RedisSink，使用起来更方便，而且不用我们考虑性能的问题，接下来将主要介绍RedisSink 如何使用。
>> https://bahir.apache.org/docs/flink/current/flink-streaming-redis/

> RedisSink 核心类是RedisMapper 是一个接口，使用时我们要编写自己的redis 操作类实现这个接口中的三个方法，如下所示
>> 1.getCommandDescription() ：
>>> 设置使用的redis 数据结构类型，和key 的名称，通过RedisCommand 设置数据结构类型
> 
>> 2.String getKeyFromData(T data)：
>>> 设置value 中的键值对key的值
> 
>> 3.String getValueFromData(T data);
>>>设置value 中的键值对value的值

> 使用RedisCommand设置数据结构类型时和redis结构对应关系

Data Type | Redis Command [Sink]
---|---
HASH | HSET
LIST | RPUSH, LPUSH
SET | SADD
PUBSUB | PUBLISH
STRING | SET
HYPER_LOG_LOG | PFADD
SORTED_SET | ZADD
SORTED_SET | ZREM

> 需求
>> 将Flink集合中的数据通过自定义Sink保存到Redis

> 代码实现

```
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.streaming.connectors.redis.RedisSink;
import org.apache.flink.streaming.connectors.redis.common.config.FlinkJedisPoolConfig;
import org.apache.flink.streaming.connectors.redis.common.mapper.RedisCommand;
import org.apache.flink.streaming.connectors.redis.common.mapper.RedisCommandDescription;
import org.apache.flink.streaming.connectors.redis.common.mapper.RedisMapper;
import org.apache.flink.util.Collector;

import java.util.Properties;

/**
 * Author xiaoma
 * Date 2020/10/14 11:41
 * Desc
 * 需求:
 * 从Kafka接收消息并做WordCount,
 * 最后将结果保存到Redis
 * 注意:存储到Redis的数据结构:要使用hash也就是map
 * key            value
 * WordCount 	 (单词,数量)
 */
public class SinkDemo03_Redis {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        Properties props  = new Properties();
        props.setProperty("bootstrap.servers", "node1:9092");
        props.setProperty("group.id", "flink");
        props.setProperty("auto.offset.reset","latest");
        props.setProperty("flink.partition-discovery.interval-millis","5000");//会开启一个后台线程每隔5s检测一下Kafka的分区情况
        props.setProperty("enable.auto.commit", "true");
        props.setProperty("auto.commit.interval.ms", "2000");
        //kafkaSource就是KafkaConsumer
        FlinkKafkaConsumer<String> kafkaSource = new FlinkKafkaConsumer<>("flink_kafka", new SimpleStringSchema(), props);
        kafkaSource.setStartFromGroupOffsets();//设置从记录的offset开始消费,如果没有记录从auto.offset.reset配置开始消费
        //kafkaSource.setStartFromEarliest();//设置直接从Earliest消费,和auto.offset.reset配置无关
        DataStreamSource<String> kafkaDS = env.addSource(kafkaSource);

        //3.Transformation
        //3.1切割出每个单词并直接记为1
        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOneDS = kafkaDS.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                //value就是每一行
                String[] words = value.split(" ");
                for (String word : words) {
                    out.collect(Tuple2.of(word, 1));
                }
            }
        });
        //3.2分组
        //注意:批处理的分组是groupBy,流处理的分组是keyBy
        KeyedStream<Tuple2<String, Integer>, Tuple> groupedDS = wordAndOneDS.keyBy(0);
        //3.3聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> result = groupedDS.sum(1);

        //4.Sink
        result.print();
        //从Kafka接收消息并做WordCount,
        // * 最后将结果保存到Redis
        // * 注意:存储到Redis的数据结构:要使用hash也就是map
        // * key            value
        // * WordCount 	 (单词,数量)

        //-1.创建RedisSink之前需要创建RedisConfig
        //连接单机版Redis
        FlinkJedisPoolConfig conf = new FlinkJedisPoolConfig.Builder().setHost("127.0.0.1").build();
        //连接集群版Redis
        //HashSet<InetSocketAddress> nodes = new HashSet<>(Arrays.asList(new InetSocketAddress(InetAddress.getByName("node1"), 6379),new InetSocketAddress(InetAddress.getByName("node2"), 6379),new InetSocketAddress(InetAddress.getByName("node3"), 6379)));
        //FlinkJedisClusterConfig conf2 = new FlinkJedisClusterConfig.Builder().setNodes(nodes).build();
        //连接哨兵版Redis
        //Set<String> sentinels = new HashSet<>(Arrays.asList("node1:26379", "node2:26379", "node3:26379"));
        //FlinkJedisSentinelConfig conf3 = new FlinkJedisSentinelConfig.Builder().setMasterName("mymaster").setSentinels(sentinels).build();

        //-3.创建并使用RedisSink
        result.addSink(new RedisSink<Tuple2<String, Integer>>(conf, new RedisWordCountMapper()));

        //5.execute
        env.execute();
        // /export/server/kafka/bin/kafka-console-producer.sh --broker-list node1:9092 --topic flink_kafka
    }

    /**
     * -2.定义一个Mapper用来指定存储到Redis中的数据结构
     */
    public static class RedisWordCountMapper implements RedisMapper<Tuple2<String, Integer>> {
        @Override
        public RedisCommandDescription getCommandDescription() {
            return new RedisCommandDescription(RedisCommand.HSET, "WordCount");
        }
        @Override
        public String getKeyFromData(Tuple2<String, Integer> data) {
            return data.f0;
        }
        @Override
        public String getValueFromData(Tuple2<String, Integer> data) {
            return data.f1.toString();
        }
    }
}
```

## 7 Flink高级特性

### 7.1 Flink四大基石

> Flink之所以能这么流行，离不开它最重要的四个基石：`Checkpoint`、`State`、`Time`、`Window`。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/97.png)

> `Checkpoint`
>> 这是Flink最重要的一个特性。
>> Flink基于Chandy-Lamport算法实现了一个分布式的一致性的快照，从而提供了一致性的语义。
>> Chandy-Lamport算法实际上在1985年的时候已经被提出来，但并没有被很广泛的应用，而Flink则把这个算法发扬光大了。
>> Spark最近在实现Continue streaming，Continue streaming的目的是为了降低处理的延时，其也需要提供这种一致性的语义，最终也采用了Chandy-Lamport这个算法，说明Chandy-Lamport算法在业界得到了一定的肯定。
>> https://zhuanlan.zhihu.com/p/53482103

> `State`
>> 提供了一致性的语义之后，Flink为了让用户在编程时能够更轻松、更容易地去管理状态，还提供了一套非常简单明了的State API，包括ValueState、ListState、MapState，BroadcastState。

> `Time`
>> 除此之外，Flink还实现了Watermark的机制，能够支持基于事件的时间的处理，能够容忍迟到/乱序的数据。

> `Window`
>> 另外流计算中一般在对流数据进行操作之前都会先进行开窗，即基于一个什么样的窗口上做这个计算。Flink提供了开箱即用的各种窗口，比如滑动窗口、滚动窗口、会话窗口以及非常灵活的自定义的窗口。

### 7.2 Flink-Window操作

#### 7.2.1 为什么需要Window

> 在流处理应用中，数据是连续不断的，有时我们需要做一些聚合类的处理，例如：在过去的1分钟内有多少用户点击了我们的网页。

> 在这种情况下，我们必须定义一个窗口(window)，用来收集最近1分钟内的数据，并对这个窗口内的数据进行计算。

#### 7.2.2 Window的分类

##### 7.2.2.1 按照time和count分类

> `time-window`:时间窗口:根据时间划分窗口,如:每xx分钟统计最近xx分钟的数据

> `count-window`:数量窗口:根据数量划分窗口,如:每xx个数据统计最近xx个数据

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/98.png)

##### 7.2.2.2 按照slide和size分类

> 窗口有两个重要的属性: `窗口大小size`和`滑动间隔slide`,根据它们的大小关系可分为:

> `tumbling-window:`滚动窗口:size=slide,如:每隔10s统计最近10s的数据

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/99.png)

> `sliding-window`:滑动窗口:size>slide,如:每隔5s统计最近10s的数据

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/100.png)

> `注意`:当size<slide的时候,如每隔15s统计最近10s的数据,那么中间5s的数据会丢失,所有开发中不用

##### 7.2.2.3 总结

> 按照上面窗口的分类方式进行组合,可以得出如下的窗口:
>> 1.基于时间的滚动窗口`tumbling-time-window`--`用的较多`
>> 2.基于时间的滑动窗口`sliding-time-window`--`用的较多`
>> 3.基于数量的滚动窗口`tumbling-count-window`--`用的较少`
>> 4.基于数量的滑动窗口`sliding-count-window`--`用的较少`
> 
>注意:Flink还支持一个特殊的窗口:Session会话窗口,需要设置一个会话超时时间,如30s,则表示30s内没有数据到来,则触发上个窗口的计算

#### 7.2.3 Window的API

##### 7.2.3.1 Window和WindowAll

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/101.png)

> `使用keyby`的流,应该使用`window`方法

> `未使用keyby`的流,应该调用`windowAll`方法

##### 7.2.3.2 WindowAssigner

> window/windowAll 方法接收的输入是一个 WindowAssigner， WindowAssigner 负责将每条输入的数据分发到正确的 window 中，

> Flink提供了很多各种场景用的WindowAssigner：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/102.png)

> 如果需要自己定制数据分发策略，则可以实现一个 class，继承自 WindowAssigner。

##### 7.2.3.3 evictor(了解)

> evictor 主要用于做一些数据的自定义操作，可以在执行用户代码之前，也可以在执行

> 用户代码之后，更详细的描述可以参考org.apache.flink.streaming.api.windowing.evictors.Evictor 的 evicBefore 和 evicAfter两个方法。

> Flink 提供了如下三种通用的 evictor：
>> `CountEvictor` 保留指定数量的元素
>> `TimeEvictor` 设定一个阈值 interval，删除所有不再 max_ts - interval 范围内的元素，其中 max_ts 是窗口内时间戳的最大值。
>> `DeltaEvictor` 通过执行用户给定的 DeltaFunction 以及预设的 theshold，判断是否删除一个元素。

##### 7.2.3.4 trigger(了解)

> trigger 用来判断一个窗口是否需要被触发，每个 WindowAssigner 都自带一个默认的trigger， 如果默认的 trigger 不能满足你的需求，则可以自定义一个类，继承自Trigger 即可，我们详细描述下 Trigger 的接口以及含义：
>> `onElement()` 每次往 window 增加一个元素的时候都会触发
>> `onEventTime()` 当 event-time timer 被触发的时候会调用
>> `onProcessingTime()` 当 processing-time timer 被触发的时候会调用
>> `onMerge()` 对两个 `rigger 的 state 进行 merge 操作
>> `clear()` window 销毁的时候被调用

> 上面的接口中前三个会返回一个 TriggerResult， TriggerResult 有如下几种可能的选择：
>> `CONTINUE` 不做任何事情
>> `FIRE` 触发 window
>> `PURGE` 清空整个 window 的元素并销毁窗口
>> `FIRE_AND_PURGE` 触发窗口，然后销毁窗口

##### 7.2.3.5 API调用示例

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/103.png)

```
source.keyBy(0).window(TumblingProcessingTimeWindows.of(Time.seconds(5)));
或
source.keyBy(0)..timeWindow(Time.seconds(5))
```

#### 7.2.4 案例一：基于时间的滚动和滑动窗口

##### 7.2.4.1 需求

```
nc -lk 9999
```
> 有如下数据表示:信号灯编号和通过该信号灯的车的数量
```
9,3
9,2
9,7
4,9
2,6
1,5
2,3
5,7
5,4
```
> 需求1:每5秒钟统计一次，最近5秒钟内，各个路口通过红绿灯汽车的数量--基于时间的滚动窗口

> 需求2:每5秒钟统计一次，最近10秒钟内，各个路口通过红绿灯汽车的数量--基于时间的滑动窗口

##### 7.2.4.2 代码实现

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;

/**
 * Author xiaoma
 * Date 2020/10/15 11:22
 * Desc
 * nc -lk 9999
 * 有如下数据表示:
 * 信号灯编号和通过该信号灯的车的数量
9,3
9,2
9,7
4,9
2,6
1,5
2,3
5,7
5,4
 * 需求1:每5秒钟统计一次，最近5秒钟内，各个路口通过红绿灯汽车的数量--基于时间的滚动窗口
 * 需求2:每5秒钟统计一次，最近10秒钟内，各个路口通过红绿灯汽车的数量--基于时间的滑动窗口
 */
public class WindowDemo12_TimeWindow {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataStreamSource<String> socketDS = env.socketTextStream("node1", 9999);

        //3.Transformation
        //将9,3转为CartInfo(9,3)
        SingleOutputStreamOperator<CartInfo> cartInfoDS = socketDS.map(new MapFunction<String, CartInfo>() {
            @Override
            public CartInfo map(String value) throws Exception {
                String[] arr = value.split(",");
                return new CartInfo(arr[0], Integer.parseInt(arr[1]));
            }
        });

        //分组
        //KeyedStream<CartInfo, Tuple> keyedDS = cartInfoDS.keyBy("sensorId");

        // * 需求1:每5秒钟统计一次，最近5秒钟内，各个路口/信号灯通过红绿灯汽车的数量--基于时间的滚动窗口
        //timeWindow(Time size窗口大小, Time slide滑动间隔)
        SingleOutputStreamOperator<CartInfo> result1 = cartInfoDS
                .keyBy("sensorId")
                .timeWindow(Time.seconds(5))//当size==slide,可以只写一个
                //.timeWindow(Time.seconds(5), Time.seconds(5))
                .sum("count");

        // * 需求2:每5秒钟统计一次，最近10秒钟内，各个路口/信号灯通过红绿灯汽车的数量--基于时间的滑动窗口
        SingleOutputStreamOperator<CartInfo> result2 = cartInfoDS
                .keyBy("sensorId")
                .timeWindow(Time.seconds(10), Time.seconds(5))
                .sum("count");

        //4.Sink
        result1.print();
        /*
1,5
2,5
3,5
4,5
         */
//        result2.print();
        /*
1,5
2,5
3,5
4,5
         */

        //5.execute
        env.execute();
    }
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class CartInfo {
        private String sensorId;//信号灯id
        private Integer count;//通过该信号灯的车的数量
    }
}

```

#### 7.2.5 案例二：基于数量的滚动和滑动窗口

##### 7.2.5.1 需求

> 需求1:统计在最近5条消息中,各自路口通过的汽车数量,相同的key每出现5次进行统计--基于数量的滚动窗口

> 需求2:统计在最近5条消息中,各自路口通过的汽车数量,相同的key每出现3次进行统计--基于数量的滑动窗口

##### 7.2.5.2 代码实现

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

/**
 * Author xiaoma
 * Date 2020/10/15 11:22
 * Desc
 * nc -lk 9999
 * 有如下数据表示:
 * 信号灯编号和通过该信号灯的车的数量
9,3
9,2
9,7
4,9
2,6
1,5
2,3
5,7
5,4
 * 需求1:统计在最近5条消息中,各自路口通过的汽车数量,相同的key每出现5次进行统计--基于数量的滚动窗口
 * 需求2:统计在最近5条消息中,各自路口通过的汽车数量,相同的key每出现3次进行统计--基于数量的滑动窗口
 */
public class WindowDemo34_CountWindow {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataStreamSource<String> socketDS = env.socketTextStream("node1", 9999);

        //3.Transformation
        //将9,3转为CartInfo(9,3)
        SingleOutputStreamOperator<CartInfo> cartInfoDS = socketDS.map(new MapFunction<String, CartInfo>() {
            @Override
            public CartInfo map(String value) throws Exception {
                String[] arr = value.split(",");
                return new CartInfo(arr[0], Integer.parseInt(arr[1]));
            }
        });

        //分组
        //KeyedStream<CartInfo, Tuple> keyedDS = cartInfoDS.keyBy("sensorId");

        // * 需求1:统计在最近5条消息中,各自路口通过的汽车数量,相同的key每出现5次进行统计--基于数量的滚动窗口
        //countWindow(long size, long slide)
        SingleOutputStreamOperator<CartInfo> result1 = cartInfoDS
                .keyBy("sensorId")
                //.countWindow(5L, 5L)
                .countWindow( 5L)
                .sum("count");

        // * 需求2:统计在最近5条消息中,各自路口通过的汽车数量,相同的key每出现3次进行统计--基于数量的滑动窗口
        //countWindow(long size, long slide)
        SingleOutputStreamOperator<CartInfo> result2 = cartInfoDS
                .keyBy("sensorId")
                .countWindow(5L, 3L)
                .sum("count");


        //4.Sink
//        result1.print();
        /*
1,1
1,1
1,1
1,1
2,1
1,1
         */
        result2.print();
        /*
1,1
1,1
2,1
1,1
2,1
3,1
4,1
2,1
         */

        //5.execute
        env.execute();
    }
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class CartInfo {
        private String sensorId;//信号灯id
        private Integer count;//通过该信号灯的车的数量
    }
}
```

#### 7.2.6 案例三：会话窗口

##### 7.2.6.1 需求

> 设置会话超时时间为10s,10s内没有数据到来,则触发上个窗口的计算

##### 7.2.6.2 代码实现

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.ProcessingTimeSessionWindows;
import org.apache.flink.streaming.api.windowing.time.Time;

/**
 * Author xiaoma
 * Date 2020/10/15 11:22
 * Desc
 * nc -lk 9999
 * 有如下数据表示:
 * 信号灯编号和通过该信号灯的车的数量
9,3
9,2
9,7
4,9
2,6
1,5
2,3
5,7
5,4
 * 需求:设置会话超时时间为10s,10s内没有数据到来,则触发上个窗口的计算(前提是上一个窗口得有数据!)
 */
public class WindowDemo55_SessionWindow {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        DataStreamSource<String> socketDS = env.socketTextStream("node1", 9999);

        //3.Transformation
        //将9,3转为CartInfo(9,3)
        SingleOutputStreamOperator<CartInfo> cartInfoDS = socketDS.map(new MapFunction<String, CartInfo>() {
            @Override
            public CartInfo map(String value) throws Exception {
                String[] arr = value.split(",");
                return new CartInfo(arr[0], Integer.parseInt(arr[1]));
            }
        });

        //需求:设置会话超时时间为10s,10s内没有数据到来,则触发上个窗口的计算(前提是上一个窗口得有数据!)
        SingleOutputStreamOperator<CartInfo> result = cartInfoDS.keyBy("sensorId")
                .window(ProcessingTimeSessionWindows.withGap(Time.seconds(10)))
                .sum("count");


        //4.Sink
        result.print();

        //5.execute
        env.execute();
    }
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class CartInfo {
        private String sensorId;//信号灯id
        private Integer count;//通过该信号灯的车的数量
    }
}
```

### 7.3 Flink-Time与WaterMaker

#### 7.3.1 Time分类

> 在Flink的流式处理中，会涉及到时间的不同概念，如下图所示：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/104.png)

> 事件时间`EventTime`:	事件真真正正发生产生的时间<br/>
> 摄入时间`IngestionTime`:	事件到达Flink的时间<br/
> 处理时间`ProcessingTime`:	事件真正被处理/计算的时间

> 问题: 上面的三个时间,我们更关注哪一个?
>> 答案: 更关注事件时间 !
>> 因为: 事件时间更能反映事件的本质! 只要事件时间一产生就不会变化

#### 7.3.2 EventTime的重要性

##### 7.3.2.1 示例1

> 假设，你正在去往地下停车场的路上，并且打算用手机点一份外卖。选好了外卖后，你就用在线支付功能付款了，这个时候是11点59分。恰好这时，你走进了地下停车库，而这里并没有手机信号。因此外卖的在线支付并没有立刻成功，而支付系统一直在Retry重试“支付”这个操作。<br/>
> 当你找到自己的车并且开出地下停车场的时候，已经是12点01分了。这个时候手机重新有了信号，手机上的支付数据成功发到了外卖在线支付系统，支付完成。

> 在上面这个场景中你可以看到，支付数据的事件时间是11点59分，而支付数据的处理时间是12点01分

> 问题:
>> 如果要统计12之前的订单金额,那么这笔交易是否应被统计?
> 
> 答案:
>> 应该被统计,因为该数据的真真正正的产生时间为11点59分,即该数据的事件时间为11点59分,
> 
>事件时间能够真正反映/代表事件的本质! 所以一般`在实际开发中会以事件时间作为计算标准`

##### 7.3.2.2 示例2

> 一条错误日志的内容为：

```
2020-11:11 22:59:00 error NullPointExcep --事件时间
```

> 进入Flink的时间为2020-11:11 23:00:00    --摄入时间

> 到达Window的时间为2020-11:11 23:00:10 --处理时间

> 问题:
>> 对于业务来说，要统计1h内的故障日志个数，哪个时间是最有意义的？
> 
> 答案:
>> EventTime事件时间，因为bug真真正正产生的时间就是事件时间,只有事件时间才能真正反映/代表事件的本质!

##### 7.3.2.3 示例3

> 某 App 会记录用户的所有点击行为，并回传日志（在网络不好的情况下，先保存在本地，延后回传）。<br/>
> A用户在 11:01:00 对 App 进行操作，B用户在 11:02:00 操作了 App， 但是A用户的网络不太稳定，回传日志延迟了，导致我们在服务端先接受到B用户的消息，然后再接受到A用户的消息，消息乱序了。

> 问题:
>> 如果这个是一个根据用户操作先后顺序,进行抢购的业务,那么是A用户成功还是B用户成功?
> 
> 答案:
>> 应该算A成功,因为A确实比B操作的早,但是实际中考虑到实现难度,可能直接按B成功算,也就是说，实际开发中希望基于事件时间来处理数据，但因为数据可能因为网络延迟等原因，出现了乱序，按照事件时间处理起来有难度！

##### 7.3.2.4 示例4

> 在实际环境中，经常会出现，因为网络原因，数据有可能会延迟一会才到达Flink实时处理系统。我们先来设想一下下面这个场景:

> 原本应该被该窗口计算的数据因为网络延迟等原因晚到了,就有可能丢失了

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/105.png)

##### 7.3.2.5 总结

> 实际开发中我们希望基于事件时间来处理数据，但因为数据可能因为网络延迟等原因，出现了乱序或延迟到达，那么可能处理的结果不是我们想要的甚至出现数据丢失的情况，所以需要一种机制来解决一定程度上的数据乱序或延迟到底的问题！也就是我们接下来要学习的Watermaker水印机制/水位线机制

#### 7.3.3 WaterMaker水印机制/水位线机制

##### 7.3.3.1 什么是WaterMaker？

> Watermaker就是给数据再额外的加的一个时间列

> 也就是Watermaker是个时间戳!

##### 7.3.3.2 如何计算WaterMaker？

```
Watermaker = 数据的事件时间  -  最大允许的延迟时间或乱序时间
```
> 注意:后面通过源码会发现,准确来说:

```
Watermaker = 当前窗口的最大的事件时间  -  最大允许的延迟时间或乱序时
```

> 这样可以保证Watermaker水位线会一直上升(变大),不会下降

##### 7.3.3.3 WaterMaker有什么用？

> 之前的窗口都是按照系统时间来触发计算的,如: [10:00:00 ~ 10:00:10) 的窗口， 一但系统时间到了10:00:10就会触发计算,那么可能会导致延迟到达的数据丢失!

> 那么现在有了Watermaker,窗口就可以按照Watermaker来触发计算!<br/>
> 也就是说Watermaker是用来触发窗口计算的！

##### 7.3.3.4 WaterMaker如何触发窗口计算的？

> 窗口计算的触发条件为:
>> 1.窗口中有数据
>> 2.Watermaker >= 窗口的结束时间

> 因为前面说到

```
Watermaker = 当前窗口的最大的事件时间  -  最大允许的延迟时间或乱序时间
```

> 也就是说只要不断有数据来,就可以保证Watermaker水位线是会一直上升/变大的,不会下降/减小的,所以最终一定是会触发窗口计算的

> 注意:
>> 上面的触发公式进行如下变形:

```
Watermaker >= 窗口的结束时间
Watermaker = 当前窗口的最大的事件时间  -  最大允许的延迟时间或乱序时间
当前窗口的最大的事件时间  -  最大允许的延迟时间或乱序时间  >= 窗口的结束时间
当前窗口的最大的事件时间  >= 窗口的结束时间 +  最大允许的延迟时间或乱序时间
```

##### 7.3.3.5 图解WaterMaker

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/106.png)

#### 7.3.4 WaterMaker案例演示 

##### 7.3.4.1 需求

> 有订单数据,格式为: (订单ID，用户ID，时间戳/事件时间，订单金额)

> 要求每隔5s,计算5秒内，每个用户的订单总金额

> 并添加Watermaker来解决一定程度上的数据延迟和数据乱序问题。

##### 7.3.4.2 API

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/107.png)

> 注意:一般我们都是直接使用Flink提供好的`BoundedOutOfOrdernessTimestampExtractor`来给数据添加WaterMaker（是基于时间添加）

##### 7.3.4.3 代码实现1-开发版-掌握

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.api.windowing.time.Time;

import java.util.Random;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

/**
 * Author xiaoma
 * Date 2020/10/15 15:44
 * Desc
 * 模拟实时订单数据,格式为: (订单ID，用户ID，订单金额，时间戳/事件时间)
 * 要求每隔5s,计算5秒内(基于时间的滚动窗口)，每个用户的订单总金额
 * 并添加Watermaker来解决一定程度上的数据延迟和数据乱序问题。
 */
public class WatermakerDemo01_Develop {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        //模拟实时订单数据(数据有延迟和乱序)
        DataStreamSource<Order> orderDS = env.addSource(new SourceFunction<Order>() {
            private boolean flag = true;
            @Override
            public void run(SourceContext<Order> ctx) throws Exception {
                Random random = new Random();
                while (flag) {
                    String orderId = UUID.randomUUID().toString();
                    int userId = random.nextInt(3);
                    int money = random.nextInt(100);
                    //模拟数据延迟和乱序!
                    long eventTime = System.currentTimeMillis() - random.nextInt(5) * 1000;
                    ctx.collect(new Order(orderId, userId, money, eventTime));

                    TimeUnit.SECONDS.sleep(1);
                }
            }
            @Override
            public void cancel() {
                flag = false;
            }
        });

        //3.Transformation

        //-1.告诉Flink要基于事件时间来计算!
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        //-2.告诉Flink每隔多久给数据添加Watermaker
        env.getConfig().setAutoWatermarkInterval(200);//每隔200ms给数据添加Watermaker
        //-3.告诉Flnk数据中的哪一列是事件时间,因为Watermaker = 当前最大的事件时间 - 最大允许的延迟时间或乱序时间
        SingleOutputStreamOperator<Order> watermakerDS = orderDS.assignTimestampsAndWatermarks(
                new BoundedOutOfOrdernessTimestampExtractor<Order>(Time.seconds(5)) {//最大允许的延迟时间或乱序时间
                    @Override
                    public long extractTimestamp(Order element) {
                        return element.eventTime;
                        //指定事件时间是哪一列,Flink底层会自动计算:
                        //Watermaker = 当前最大的事件时间 - 最大允许的延迟时间或乱序时间
                    }
        });
        //代码走到这里,就已经被添加上Watermaker了!接下来就可以进行窗口计算了
        //要求每隔5s,计算5秒内(基于时间的滚动窗口)，每个用户的订单总金额
        SingleOutputStreamOperator<Order> result = watermakerDS
                .keyBy("userId")
                .timeWindow(Time.seconds(5), Time.seconds(5))
                .sum("money");


        //4.Sink
        result.print();

        //5.execute
        env.execute();
    }
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class Order {
        private String orderId;
        private Integer userId;
        private Integer money;
        private Long eventTime;
    }
}
```

##### 7.3.4.4 代码实现2-验证版-了解

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.commons.lang3.time.FastDateFormat;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.AssignerWithPeriodicWatermarks;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.streaming.api.functions.windowing.WindowFunction;
import org.apache.flink.streaming.api.watermark.Watermark;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

import javax.annotation.Nullable;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

/**
 * Author xiaoma
 * Date 2020/10/15 15:44
 * Desc
 * 模拟实时订单数据,格式为: (订单ID，用户ID，订单金额，时间戳/事件时间)
 * 要求每隔5s,计算5秒内(基于时间的滚动窗口)，每个用户的订单总金额
 * 并添加Watermaker来解决一定程度上的数据延迟和数据乱序问题。
 */
public class WatermakerDemo02_Check {
    public static void main(String[] args) throws Exception {
        FastDateFormat df = FastDateFormat.getInstance("HH:mm:ss");

        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        //模拟实时订单数据(数据有延迟和乱序)
        DataStreamSource<Order> orderDS = env.addSource(new SourceFunction<Order>() {
            private boolean flag = true;
            @Override
            public void run(SourceContext<Order> ctx) throws Exception {
                Random random = new Random();
                while (flag) {
                    String orderId = UUID.randomUUID().toString();
                    int userId = random.nextInt(3);
                    int money = random.nextInt(100);
                    //模拟数据延迟和乱序!
                    long eventTime = System.currentTimeMillis() - random.nextInt(5) * 1000;
                    System.out.println(userId+" : "+df.format(eventTime));
                    ctx.collect(new Order(orderId, userId, money, eventTime));
                    TimeUnit.SECONDS.sleep(1);
                }
            }
            @Override
            public void cancel() {
                flag = false;
            }
        });

        //3.Transformation

        //-1.告诉Flink要基于事件时间来计算!
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        //-2.告诉Flink每隔多久给数据添加Watermaker
        env.getConfig().setAutoWatermarkInterval(1000);//每隔200ms给数据添加Watermaker,默认是200
        //-3.告诉Flnk数据中的哪一列是事件时间,因为Watermaker = 当前最大的事件时间 - 最大允许的延迟时间或乱序时间
       /* SingleOutputStreamOperator<Order> watermakerDS = orderDS.assignTimestampsAndWatermarks(
                new BoundedOutOfOrdernessTimestampExtractor<Order>(Time.seconds(5)) {//最大允许的延迟时间或乱序时间
                    @Override
                    public long extractTimestamp(Order element) {
                        return element.eventTime;
                        //指定事件时间是哪一列,Flink底层会自动计算:
                        //Watermaker = 当前最大的事件时间 - 最大允许的延迟时间或乱序时间
                    }
        });*/
        //开发中直接使用上面的BoundedOutOfOrdernessTimestampExtractor
        //学习测试时可以自己实现BoundedOutOfOrdernessTimestampExtractor的底层也就是AssignerWithPeriodicWatermarks接口
        SingleOutputStreamOperator<Order> watermakerDS = orderDS.assignTimestampsAndWatermarks(
                new AssignerWithPeriodicWatermarks<Order>() {
                    private long currentMaxTimestamp = 0L;//用来记录当前最大事件时间!
                    private final long maxOutOfOrderness = 2000L;//最大允许的延迟时间或乱序时间
                    private long lastEmittedWatermark = Long.MIN_VALUE;
                    private long eventTime = 0L;
                    private int userId = 0;

                    @Override
                    public long extractTimestamp(Order element, long previousElementTimestamp) {
                        userId = element.userId;
                        eventTime = element.eventTime;
                        long timestamp = element.eventTime;
                        if (timestamp > currentMaxTimestamp) {
                            currentMaxTimestamp = timestamp;
                        }
                        return timestamp;
                    }

                    @Nullable
                    @Override
                    public Watermark getCurrentWatermark() {
                        //potentialWM = 当前最大事件时间 - 最大允许的延迟时间或乱序时间
                        long potentialWM = currentMaxTimestamp - maxOutOfOrderness;
                        //如果potentialWM >= 上一次的Watermaker就把potentialWM作为Watermaker
                        //保证Watermaker一直上升!
                        if (potentialWM >= lastEmittedWatermark) {
                            lastEmittedWatermark = potentialWM;
                        }
                        Watermark watermark = new Watermark(lastEmittedWatermark);
                        System.out.println("key:" + userId + ",系统时间:" + df.format(System.currentTimeMillis()) + ",事件时间:" + df.format(eventTime) + ",水印时间:" + df.format(watermark.getTimestamp()));
                        return watermark;
                    }
                });

        //代码走到这里,就已经被添加上Watermaker了!接下来就可以进行窗口计算了
        //要求每隔5s,计算5秒内(基于时间的滚动窗口)，每个用户的订单总金额
       /* SingleOutputStreamOperator<Order> result = watermakerDS
                .keyBy("userId")
                .timeWindow(Time.seconds(5), Time.seconds(5))
                .sum("money");*/

       //开发中使用上面的代码进行业务计算即可
       //学习测试时可以使用下面的代码对数据进行更详细的输出,如输出窗口触发时各个窗口中的数据的事件时间,Watermaker时间
        SingleOutputStreamOperator<String> result = watermakerDS
                .keyBy("userId")
                .timeWindow(Time.seconds(5), Time.seconds(5))
                //把apply中的函数应用在窗口中的数据上
                //WindowFunction<IN, OUT, KEY, W extends Window>
                .apply(new WindowFunction<Order, String, Tuple, TimeWindow>() {
                    @Override
                    public void apply(Tuple tuple, TimeWindow window, Iterable<Order> input, Collector<String> out) throws Exception {
                        //准备一个集合用来存放属于该窗口的数据的事件时间
                        List<String> eventTimeList = new ArrayList<>();
                        for (Order order : input) {
                            Long eventTime = order.eventTime;
                            eventTimeList.add(df.format(eventTime));
                        }
                        String outStr = String.format("key:%s,窗口开始结束:[%s~%s),属于该窗口的事件时间:%s",
                                tuple.toString(), df.format(window.getStart()), df.format(window.getEnd()), eventTimeList);
                        out.collect(outStr);
                    }
                });
        //4.Sink
        result.print();

        //5.execute
        env.execute();
    }
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class Order {
        private String orderId;
        private Integer userId;
        private Integer money;
        private Long eventTime;
    }
}

```

#### 7.3.5 Allowed Lateness案例演示

> 在上面的案例基础之上,允许接收延迟/迟到/乱序严重的数据!

> 如,设置的最大允许延迟时间为5s,而实际延迟了10s的数据就会丢失,这时候可以使用该`Allowed Lateness + OutputTag `,也就是允许延迟机制 + 侧道输出流`彻底解决数据丢失问题`,也就是不管数据延迟多久都会被单独收集到侧道输出流!


##### 7.4.5.1 需求

> 有订单数据,格式为: (订单ID，用户ID，时间戳/事件时间，订单金额)

> 要求每隔5s,计算5秒内，每个用户的订单总金额

> 并添加Watermaker来解决一定程度上的数据延迟和数据乱序问题。

> 并使用OutputTag+allowedLateness解决数据丢失问题

##### 7.4.5.2 API

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/108.png)

##### 7.4.5.3 代码实现

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.api.common.typeinfo.TypeInformation;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.OutputTag;

import java.util.Random;
import java.util.UUID;

/**
 * Author xiaoma
 * Date 2020/10/15 15:44
 * Desc
 * 模拟实时订单数据,格式为: (订单ID，用户ID，订单金额，时间戳/事件时间)
 * 要求每隔5s,计算5秒内(基于时间的滚动窗口)，每个用户的订单总金额
 * 并添加Watermaker来解决一定程度上的数据延迟和数据乱序问题。
 */
public class WatermakerDemo03_AllowedLateness {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        //模拟实时订单数据(数据有延迟和乱序)
        DataStreamSource<Order> orderDS = env.addSource(new SourceFunction<Order>() {
            private boolean flag = true;
            @Override
            public void run(SourceContext<Order> ctx) throws Exception {
                Random random = new Random();
                while (flag) {
                    String orderId = UUID.randomUUID().toString();
                    int userId = random.nextInt(3);
                    int money = random.nextInt(100);
                    //模拟数据延迟和乱序!
                    long eventTime = System.currentTimeMillis() - random.nextInt(20) * 1000;
                    ctx.collect(new Order(orderId, userId, money, eventTime));

                    //TimeUnit.SECONDS.sleep(1);
                }
            }
            @Override
            public void cancel() {
                flag = false;
            }
        });

        //3.Transformation

        //-1.告诉Flink要基于事件时间来计算!
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        //-2.告诉Flink每隔多久给数据添加Watermaker
        env.getConfig().setAutoWatermarkInterval(200);//每隔200ms给数据添加Watermaker
        //-3.告诉Flnk数据中的哪一列是事件时间,因为Watermaker = 当前最大的事件时间 - 最大允许的延迟时间或乱序时间
        SingleOutputStreamOperator<Order> watermakerDS = orderDS.assignTimestampsAndWatermarks(
                new BoundedOutOfOrdernessTimestampExtractor<Order>(Time.seconds(5)) {//最大允许的延迟时间或乱序时间
                    @Override
                    public long extractTimestamp(Order element) {
                        return element.eventTime;
                        //指定事件时间是哪一列,Flink底层会自动计算:
                        //Watermaker = 当前最大的事件时间 - 最大允许的延迟时间或乱序时间
                    }
        });
        //代码走到这里,就已经被添加上Watermaker了!接下来就可以进行窗口计算了
        //要求每隔5s,计算5秒内(基于时间的滚动窗口)，每个用户的订单总金额
        OutputTag<Order> outputTag = new OutputTag<>("Seriouslylate", TypeInformation.of(Order.class));
        SingleOutputStreamOperator<Order> result = watermakerDS
                .keyBy("userId")
                .timeWindow(Time.seconds(5), Time.seconds(5))
                .allowedLateness(Time.seconds(5))
                .sideOutputLateData(outputTag)
                .sum("money");

        DataStream<Order> result2 = result.getSideOutput(outputTag);

        //4.Sink
        result.print("正常的数据和迟到不严重的数据");
        result2.print("迟到严重的数据");

        //5.execute
        env.execute();
    }
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class Order {
        private String orderId;
        private Integer userId;
        private Integer money;
        private Long eventTime;
    }
}

```

### 7.4 Flink-状态管理

#### 7.4.1 Flink的有状态计算

> 注意:

> Flink中已经对需要进行有状态计算的API,做了封装,底层已经维护好了状态!

> 例如,之前下面代码,直接使用即可,不需要像SparkStreaming那样还得自己写updateStateByKey

> 也就是说我们今天学习的State只需要掌握原理,实际开发中一般都是使用Flink底层维护好的状态或第三方维护好的状态(如Flink整合Kafka的offset维护底层就是使用的State,但是人家已经写好了的)

```
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/**
 * Author itcast
 * Date 2020/10/14 11:41
 * Desc
 * 使用Flink流处理程序获取node1:9999发送的实时数据并做WordCount!
 */
public class SourceDemo01_Socket_WordCount {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //2.source
        DataStream<String> linesDS = env.socketTextStream("node1", 9999);

        //3.Transformation
        //3.1切割出每个单词并直接记为1
        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOneDS = linesDS.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                //value就是每一行
                String[] words = value.split(" ");
                for (String word : words) {
                    out.collect(Tuple2.of(word, 1));
                }
            }
        });
        //3.2分组
        //注意:批处理的分组是groupBy,流处理的分组是keyBy
        KeyedStream<Tuple2<String, Integer>, Tuple> groupedDS = wordAndOneDS.keyBy(0);
        //3.3聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> result = groupedDS.sum(1);

        //4.sink
        result.print();

        //5.execute
        env.execute();
    }
}
```

#### 7.4.2 无状态计算和有状态计算

##### 7.4.2.1 无状态计算

> 不需要考虑历史数据

> `相同的输入得到相同的输出就是无状态计算, 如map/flatMap/filter....`

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/109.png)

> 首先举一个无状态计算的例子：消费延迟计算。<br/>
> 假设现在有一个消息队列，消息队列中有一个生产者持续往消费队列写入消息，多个消费者分别从消息队列中读取消息。<br/>
> 从图上可以看出，生产者已经写入 16 条消息，Offset 停留在 15 ；有 3 个消费者，有的消费快，而有的消费慢。消费快的已经消费了 13 条数据，消费者慢的才消费了 7、8 条数据。<br/>
> 如何实时统计每个消费者落后多少条数据，如图给出了输入输出的示例。可以了解到输入的时间点有一个时间戳，生产者将消息写到了某个时间点的位置，每个消费者同一时间点分别读到了什么位置。刚才也提到了生产者写入了 15 条，消费者分别读取了 10、7、12 条。那么问题来了，怎么将生产者、消费者的进度转换为右侧示意图信息呢？<br/>
> consumer 0 落后了 5 条，consumer 1 落后了 8 条，consumer 2 落后了 3 条，根据 Flink 的原理，此处需进行 Map 操作。Map 首先把消息读取进来，然后分别相减，即可知道每个 consumer 分别落后了几条。Map 一直往下发，则会得出最终结果。<br/>
> 大家会发现，在这种模式的计算中，无论这条输入进来多少次，输出的结果都是一样的，因为单条输入中已经包含了所需的所有信息。消费落后等于生产者减去消费者。生产者的消费在单条数据中可以得到，消费者的数据也可以在单条数据中得到，所以相同输入可以得到相同输出，这就是一个无状态的计算。

##### 7.4.2.2 有状态计算

> 需要考虑历史数据

> `相同的输入得到不同的输出/不一定得到相同的输出,就是有状态计算,如:sum/reduce`

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/110.png)

> 以访问日志统计量的例子进行说明，比如当前拿到一个 Nginx 访问日志，一条日志表示一个请求，记录该请求从哪里来，访问的哪个地址，需要实时统计每个地址总共被访问了多少次，也即每个 API 被调用了多少次。可以看到下面简化的输入和输出，输入第一条是在某个时间点请求 GET 了 /api/a；第二条日志记录了某个时间点 Post /api/b ;第三条是在某个时间点 GET了一个 /api/a，总共有 3 个 Nginx 日志。<br/>
> 从这 3 条 Nginx 日志可以看出，第一条进来输出 /api/a 被访问了一次，第二条进来输出 /api/b 被访问了一次，紧接着又进来一条访问 api/a，所以 api/a 被访问了 2 次。不同的是，两条 /api/a 的 Nginx 日志进来的数据是一样的，但输出的时候结果可能不同，第一次输出 count=1 ，第二次输出 count=2，说明相同输入可能得到不同输出。输出的结果取决于当前请求的 API 地址之前累计被访问过多少次。第一条过来累计是 0 次，count = 1，第二条过来 API 的访问已经有一次了，所以 /api/a 访问累计次数 count=2。单条数据其实仅包含当前这次访问的信息，而不包含所有的信息。要得到这个结果，还需要依赖 API 累计访问的量，即状态。<br/>
> 这个计算模式是将数据输入算子中，用来进行各种复杂的计算并输出数据。这个过程中算子会去访问之前存储在里面的状态。另外一方面，它还会把现在的数据对状态的影响实时更新，如果输入 200 条数据，最后输出就是 200 条结果。

#### 7.4.3 有状态计算的场景

> `无状态计算的场景:各种简单的转换/过滤等操作,如简单的map/flatMap/filter.....`

> `有状态计算的场景:如各种聚合统计,sum/reduce/max/min....`

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/111.png)

> 什么场景会用到状态呢？下面列举了常见的 4 种：
>> 1.去重：比如上游的系统数据可能会有重复，落到下游系统时希望把重复的数据都去掉。去重需要先了解哪些数据来过，哪些数据还没有来，也就是把所有的主键都记录下来，当一条数据到来后，能够看到在主键当中是否存在。
> 
>> 2.窗口计算：比如统计每分钟 Nginx 日志 API 被访问了多少次。窗口是一分钟计算一次，在窗口触发前，如 08:00 ~ 08:01 这个窗口，前59秒的数据来了需要先放入内存，即需要把这个窗口之内的数据先保留下来，等到 8:01 时一分钟后，再将整个窗口内触发的数据输出。未触发的窗口数据也是一种状态。
> 
>> 3.机器学习/深度学习：如训练的模型以及当前模型的参数也是一种状态，机器学习可能每次都用有一个数据集，需要在数据集上进行学习，对模型进行一个反馈。
> 
>> 4.访问历史数据：比如与昨天的数据进行对比，需要访问一些历史数据。如果每次从外部去读，对资源的消耗可能比较大，所以也希望把这些历史数据也放入状态中做对比。

#### 7.4.4 状态的分类

##### 7.4.4.1 Managed State & Raw State

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/112.png)

> 从Flink是否接管角度:可以分为
>> ManagedState(托管状态)
> 
>> RawState(原始状态)

> 两者的区别如下：
>> 1.从状态管理方式的方式来说，Managed State 由 Flink Runtime 管理，自动存储，自动恢复，在内存管理上有优化；而 Raw State 需要用户自己管理，需要自己序列化，Flink 不知道 State 中存入的数据是什么结构，只有用户自己知道，需要最终序列化为可存储的数据结构。
> 
>> 2.从状态数据结构来说，Managed State 支持已知的数据结构，如 Value、List、Map 等。而 Raw State只支持字节数组 ，所有状态都要转换为二进制字节数组才可以。
> 
>> 3.从推荐使用场景来说，Managed State 大多数情况下均可使用，而 Raw State 是当 Managed State 不够用时，比如需要自定义 Operator 时，才会使用 Raw State。
> 
> `在实际生产中，都只推荐使用ManagedState，后续将围绕该话题进行讨论。`

##### 7.4.4.2 Keyed State & Operator State

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/113.png)

> `Managed State` 分为两种，`Keyed State` 和 `Operator State`<br/>
> (`Raw State`都是`Operator State`)

> `Keyed State`

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/114.png)

> 在Flink Stream模型中，Datastream 经过 keyBy 的操作可以变为 KeyedStream。<br/>
> Keyed State是基于KeyedStream上的状态。这个状态是跟特定的key绑定的，对KeyedStream流上的每一个key，都对应一个state，如stream.keyBy(…)<br/>
> KeyBy之后的State,可以理解为分区过的State，每个并行keyed Operator的每个实例的每个key都有一个Keyed State，即<parallel-operator-instance,key>就是一个唯一的状态，由于每个key属于一个keyed Operator的并行实例，因此我们将其简单的理解为<operator,key>

> `Operator State`

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/115.png)

> 这里的fromElements会调用FromElementsFunction的类，其中就使用了类型为 list state 的 operator state<br/>
> Operator State又称为 non-keyed state，与Key无关的State，每一个 operator state 都仅与一个 operator 的实例绑定。<br/>
> Operator State 可以用于所有算子，但一般常用于 Source

#### 7.4.5 存储State的数据机构/API介绍

> 前面说过有状态计算其实就是需要考虑历史数据<br/>
> 而历史数据需要搞个地方存储起来<br/>
> Flink为了方便不同分类的State的存储和管理,提供了如下的API/数据结构来存储State!

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/116.png)

> Keyed State 通过 RuntimeContext 访问，这需要 Operator 是一个`RichFunction`。 保存Keyed state的数据结构:

> `ValueState<T>`:即类型为T的单值状态。这个状态与对应的key绑定，是最简单的状态了。它可以通过update方法更新状态值，通过value()方法获取状态值，如求按用户id统计用户交易总额

> `ListState<T>`:即key上的状态值为一个列表。可以通过add方法往列表中附加值；也可以通过get()方法返回一个Iterable<T>来遍历状态值，如统计按用户id统计用户经常登录的Ip

> `ReducingState<T>`:这种状态通过用户传入的reduceFunction，每次调用add方法添加值的时候，会调用reduceFunction，最后合并到一个单一的状态值

> `MapState<UK, UV>`:即状态值为一个map。用户通过put或putAll方法添加元素

> 需要注意的是，以上所述的State对象，仅仅用于与状态进行交互(更新、删除、清空等)，而真正的状态值，有可能是存在内存、磁盘、或者其他分布式存储系统中。相当于我们只是持有了这个状态的句柄
> 
> Operator State 需要自己实现 `CheckpointedFunction` 或 `ListCheckpointed` 接口。
> 
> 保存Operator state的数据结构:
>
>> ListState<T>
>> BroadcastState<K,V>

> 举例来说，Flink中的FlinkKafkaConsumer，就使用了operator state。它会在每个connector实例中，保存该实例中消费topic的所有(partition, offset)映射

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/117.png)

#### 7.4.6 State代码示例

##### 7.4.6.1 Keyed State

> 下图就 word count 的 sum 所使用的StreamGroupedReduce类为例讲解了如何在代码中使用 keyed state：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/118.png)

> 官网代码示例

[https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/state/state.html#using-managed-keyed-state](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/state/state.html#using-managed-keyed-state)
 
> 需求
>> 使用KeyState中的ValueState获取流数据中的最大值(实际中直接使用maxBy即可)

> 编码步骤

```
//-1.定义一个状态用来存放最大值
private transient ValueState<Long> maxValueState;
//-2.创建一个状态描述符对象
ValueStateDescriptor descriptor = new ValueStateDescriptor("maxValueState", Long.class);
//-3.根据状态描述符获取State
maxValueState = getRuntimeContext().getState(maxValueStateDescriptor);
 //-4.使用State
Long historyValue = maxValueState.value();
//判断当前值和历史值谁大
if (historyValue == null || currentValue > historyValue) 
//-5.更新状态
maxValueState.update(currentValue);     
```

> 代码示例

```
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

/**
 * Author xiaoma
 * Date 2020/10/17 10:18
 * Desc
 * 使用KeyState中的ValueState获取流数据中的最大值(实际中直接使用maxBy即可)
 */
public class StateDemo01_KeyedState {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);//方便观察

        //2.Source
        DataStreamSource<Tuple2<String, Long>> tupleDS = env.fromElements(
                Tuple2.of("北京", 1L),
                Tuple2.of("上海", 2L),
                Tuple2.of("北京", 6L),
                Tuple2.of("上海", 8L),
                Tuple2.of("北京", 3L),
                Tuple2.of("上海", 4L)
        );

        //3.Transformation
        //使用KeyState中的ValueState获取流数据中的最大值(实际中直接使用maxBy即可)
        //实现方式1:直接使用maxBy--开发中使用该方式即可
        SingleOutputStreamOperator<Tuple2<String, Long>> result = tupleDS.keyBy(0).maxBy(1);

        //实现方式2:使用KeyState中的ValueState---学习测试时使用,或者后续项目中/实际开发中遇到复杂的Flink没有实现的逻辑,才用该方式!
        SingleOutputStreamOperator<Tuple3<String, Long, Long>> result2 = tupleDS.keyBy(0).map(new RichMapFunction<Tuple2<String, Long>, Tuple3<String, Long, Long>>() {
            //-1.定义状态用来存储最大值
            private ValueState<Long> maxValueState = null;

            @Override
            public void open(Configuration parameters) throws Exception {
                //-2.定义状态描述符:描述状态的名称和里面的数据类型
                ValueStateDescriptor descriptor = new ValueStateDescriptor("maxValueState", Long.class);
                //-3.根据状态描述符初始化状态
                maxValueState = getRuntimeContext().getState(descriptor);
            }

            @Override
            public Tuple3<String, Long, Long> map(Tuple2<String, Long> value) throws Exception {
                //-4.使用State,取出State中的最大值/历史最大值
                Long historyMaxValue = maxValueState.value();
                Long currentValue = value.f1;
                if (historyMaxValue == null || currentValue > historyMaxValue) {
                    //5-更新状态,把当前的作为新的最大值存到状态中
                    maxValueState.update(currentValue);
                    return Tuple3.of(value.f0, currentValue, currentValue);
                } else {
                    return Tuple3.of(value.f0, currentValue, historyMaxValue);
                }
            }
        });


        //4.Sink
        //result.print();
        result2.print();

        //5.execute
        env.execute();
    }
}
```

##### 7.4.6.2 Operator State

> 下图对 word count 示例中的FromElementsFunction类进行详解并分享如何在代码中使用 operator state：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/119.png)

> 官网代码示例

[https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/state/state.html#using-managed-operator-state](https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/state/state.html#using-managed-operator-state)

> 需求:
>> 使用ListState存储offset模拟Kafka的offset维护

> 编码步骤:

```
//-1.声明一个OperatorState来记录offset
private ListState<Long> offsetState = null;
private Long offset = 0L;
//-2.创建状态描述器
ListStateDescriptor<Long> descriptor = new ListStateDescriptor<Long>("offsetState", Long.class);
//-3.根据状态描述器初始化State
offsetState = context.getOperatorStateStore().getListState(descriptor);

//-4.获取State中的值
Iterator<Long> iterator = offsetState.get().iterator();
if (iterator.hasNext()) {//迭代器中有值
    offset = iterator.next();//取出的值就是offset
}
offset += 1L;
ctx.collect("subTaskId:" + getRuntimeContext().getIndexOfThisSubtask() + ",当前的offset为:" + offset);
if (offset % 5 == 0) {//每隔5条消息,模拟一个异常
//-5.保存State到Checkpoint中
offsetState.clear();//清理内存中存储的offset到Checkpoint中
//-6.将offset存入State中
offsetState.add(offset);
```
> 示例代码

```
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.state.ListState;
import org.apache.flink.api.common.state.ListStateDescriptor;
import org.apache.flink.runtime.state.FunctionInitializationContext;
import org.apache.flink.runtime.state.FunctionSnapshotContext;
import org.apache.flink.runtime.state.filesystem.FsStateBackend;
import org.apache.flink.streaming.api.CheckpointingMode;
import org.apache.flink.streaming.api.checkpoint.CheckpointedFunction;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.CheckpointConfig;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.RichParallelSourceFunction;

import java.util.Iterator;
import java.util.concurrent.TimeUnit;

/**
 * Author xiaoma
 * Date 2020/10/17 10:58
 * Desc
 * 需求:
 * 使用OperatorState支持的数据结构ListState存储offset信息, 模拟Kafka的offset维护,
 * 其实就是FlinkKafkaConsumer底层对应offset的维护!
 */
public class StateDemo02_OperatorState {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        //先直接使用下面的代码设置Checkpoint时间间隔和磁盘路径以及代码遇到异常后的重启策略,下午会学
        env.enableCheckpointing(1000);//每隔1s执行一次Checkpoint
        env.setStateBackend(new FsStateBackend("file:///D:/data/ckp"));
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        //固定延迟重启策略: 程序出现异常的时候，重启2次，每次延迟3秒钟重启，超过2次，程序退出
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(2, 3000));

        //2.Source
        DataStreamSource<String> sourceData = env.addSource(new MyKafkaSource());

        //3.Transformation
        //4.Sink
        sourceData.print();

        //5.execute
        env.execute();
    }

    /**
     * MyKafkaSource就是模拟的FlinkKafkaConsumer并维护offset
     */
    public static class MyKafkaSource extends RichParallelSourceFunction<String> implements CheckpointedFunction {
        //-1.声明一个OperatorState来记录offset
        private ListState<Long> offsetState = null;
        private Long offset = 0L;
        private boolean flag = true;

        @Override
        public void initializeState(FunctionInitializationContext context) throws Exception {
            //-2.创建状态描述器
            ListStateDescriptor descriptor = new ListStateDescriptor("offsetState", Long.class);
            //-3.根据状态描述器初始化状态
            offsetState = context.getOperatorStateStore().getListState(descriptor);
        }

        @Override
        public void run(SourceContext<String> ctx) throws Exception {
            //-4.获取并使用State中的值
            Iterator<Long> iterator = offsetState.get().iterator();
            if (iterator.hasNext()){
                offset = iterator.next();
            }
            while (flag){
                offset += 1;
                int id = getRuntimeContext().getIndexOfThisSubtask();
                ctx.collect("分区:"+id+"消费到的offset位置为:" + offset);//1 2 3 4 5 6
                //Thread.sleep(1000);
                TimeUnit.SECONDS.sleep(2);
                if(offset % 5 == 0){
                    System.out.println("程序遇到异常了.....");
                    throw new Exception("程序遇到异常了.....");
                }
            }
        }

        @Override
        public void cancel() {
            flag = false;
        }

        /**
         * 下面的snapshotState方法会按照固定的时间间隔将State信息存储到Checkpoint/磁盘中,也就是在磁盘做快照!
         * @param context
         * @throws Exception
         */
        @Override
        public void snapshotState(FunctionSnapshotContext context) throws Exception {
            //-5.保存State到Checkpoint中
            offsetState.clear();//清理内存中存储的offset到Checkpoint中
            //-6.将offset存入State中
            offsetState.add(offset);
        }
    }
}
```

### 7.5 Flink-容错机制

#### 7.5.1 CheckPoint

##### 7.5.1.1 State VS CheckPoint

> `State`:
>> 维护/存储的是某一个Operator的运行的状态/历史值,是维护在内存中!
>> 一般指一个具体的Operator的状态(operator的状态表示一些算子在运行的过程中会产生的一些历史结果,如前面的maxBy底层会维护当前的最大值,也就是会维护一个keyedOperator,这个State里面存放就是maxBy这个Operator中的最大值)
>> State数据默认保存在Java的堆内存中/TaskManage节点的内存中
>> State可以被记录，在失败的情况下数据还可以恢复

> `Checkpoint`:
>> 某一时刻,Flink中所有的Operator的当前State的全局快照,一般存在磁盘上
>> 表示了一个Flink Job在一个特定时刻的一份全局状态快照，即包含了所有Operator的状态
>> 可以理解为Checkpoint是把State数据定时持久化存储了
>> 比如FlinkKafkaConsumer算子中维护的Offset状态,当任务重新恢复的时候可以从Checkpoint中获取


> `注意`:
>> Flink中的Checkpoint底层使用了Chandy-Lamport algorithm分布式快照算法可以保证数据的在分布式环境下的一致性!
>> https://zhuanlan.zhihu.com/p/53482103
>> Chandy-Lamport algorithm算法的作者也是ZK中Paxos 一致性算法的作者
>> https://www.cnblogs.com/shenguanpu/p/4048660.html
>> Flink中使用Chandy-Lamport algorithm分布式快照算法取得了成功,后续Spark的StructuredStreaming也借鉴了该算法

> `CheckPoint就是State的全局的分布式快照`

##### 7.5.1.2 CheckPoint执行流程

###### 7.5.1.2.1 简单流程

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/120.png)

> 0.JobManager创建CheckpointCoordinator,并根据设置的Checkpoint时间间隔,向SourceOperator发送Barrier栅栏/其实就是告诉SourceOperator要将State进行快照/进行Checkpoint的命令!

> 1.SourceOperator接收到Barrier会暂停当前的工作,并异步调用API将当前的State状态数据保存到指定位置,一般为HDFS,并和CheckpointCoordinator确认已经完成Checkpoint操作,同时将Barrier发送给下游的TransformationOperator,同时恢复自己的工作!

> 2.下游的TransformationOperator接收到Barrier,同样也暂停当前的工作,并异步调用API将当前的State状态数据保存到指定位置,一般为HDFS,并和CheckpointCoordinator确认已经完成Checkpoint操作,同时将Barrier发送给下游,同时恢复自己的工作!

> 3.直到Barrier被发送给SinkOperator,SinkOperator同样也暂停当前的工作,并异步调用API将当前的State状态数据保存到指定位置,一般为HDFS,并和CheckpointCoordinator确认已经完成Checkpoint操作,同时恢复自己的工作!

> 4.CheckpointCoordinator接收到所有的Operator的确认消息,那么本次Checkpoint结束!如果有没有没有收到的,超时之后可以认为失败,Checkpoint失败可以让任务失败也可以不管,直接进行下一次Checkpoint!这些都可以通过配置参数来实现!

> 注意:
>> 1.在往介质(如HDFS)中写入快照数据的时候是异步的(为了提高效率)
>> 2.分布式快照执行时的数据一致性由Chandy-Lamport algorithm分布式快照算法保证!

###### 7.5.1.2.2 复杂流程-自行阅读

> 下图左侧是 Checkpoint Coordinator，是整个 Checkpoint 的发起者，中间是由两个 source，一个 sink 组成的 Flink 作业，最右侧的是持久化存储，在大部分用户场景中对应 HDFS。

> 1.Checkpoint Coordinator 向所有 source 节点 trigger Checkpoint。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/121.png)

> 2.source 节点向下游广播 barrier，这个 barrier 就是实现 Chandy-Lamport 分布式快照算法的核心，下游的 task 只有收到所有 input 的 barrier 才会执行相应的 Checkpoint。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/122.png)

> 3.当 task 完成 state 备份后，会将备份数据的地址（state handle）通知给 Checkpoint coordinator。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/123.png)

> 4.下游的 sink 节点收集齐上游两个 input 的 barrier 之后，会执行本地快照，(栅栏对齐),这里还展示了 RocksDB incremental Checkpoint (增量Checkpoint)的流程，首先 RocksDB 会全量刷数据到磁盘上（红色大三角表示），然后 Flink 框架会从中选择没有上传的文件进行持久化备份（紫色小三角）。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/124.png)

> 5.同样的，sink 节点在完成自己的 Checkpoint 之后，会将 state handle 返回通知 Coordinator。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/125.png)

> 6.最后，当 Checkpoint coordinator 收集齐所有 task 的 state handle，就认为这一次的 Checkpoint 全局完成了，向持久化存储中再备份一个 Checkpoint meta 文件。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/126.png)

##### 7.5.1.3 State状态后端/State存储介质

> 注意:
>> 前面学习了Checkpoint其实就是Flink中某一时刻,所有的Operator的全局快照,
>> 那么快照应该要有一个地方进行存储,而这个存储的地方叫做状态后端
>> Flink中的State状态后端有很多种:

###### 7.5.1.3.1 MenStateBackend（了解）

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/127.png)

> 第一种是内存存储，即 MemoryStateBackend，构造方法是设置最大的StateSize，选择是否做异步快照，<br/>
> 对于State状态存储在 TaskManager 节点也就是执行节点内存中的，因为内存有容量限制，所以单个 State maxStateSize 默认 5 M，且需要注意 maxStateSize <= akka.framesize 默认 10 M。<br/>
> 对于Checkpoint 存储在 JobManager 内存中，因此总大小不超过 JobManager 的内存。<br/>
> 推荐使用的场景为：本地测试、几乎无状态的作业，比如 ETL、JobManager 不容易挂，或挂掉影响不大的情况。<br/>
> `不推荐在生产场景使用。`

###### 7.5.1.3.2 FsStateBackend

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/128.png)

> 另一种就是在文件系统上的 FsStateBackend 构建方法是需要传一个文件路径和是否异步快照。<br/>
> State 依然在 TaskManager 内存中，但不会像 MemoryStateBackend 是 5 M 的设置上限<br/>
> Checkpoint 存储在外部文件系统（本地或 HDFS），打破了总大小 Jobmanager 内存的限制。<br/>
> `推荐使用的场景为：常规使用状态的作业、例如分钟级窗口聚合或 join、需要开启HA的作业。`

> 如果使用HDFS，则初始化FsStateBackend时，需要传入以 “hdfs://”开头的路径(即: new FsStateBackend("hdfs:///hacluster/checkpoint"))，<br/>
> 如果使用本地文件，则需要传入以“file://”开头的路径(即:new FsStateBackend("file:///Data"))。<br/>
> 在分布式情况下，不推荐使用本地文件。因为如果某个算子在节点A上失败，在节点B上恢复，使用本地文件时，在B上无法读取节点 A上的数据，导致状态恢复失败

###### 7.5.1.3.3 RocksDBStateBackend

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/129.png)

> 还有一种存储为 RocksDBStateBackend ，<br/>
> RocksDB 是一个 key/value 的内存存储系统，和其他的 key/value 一样，先将状态放到内存中，如果内存快满时，则写入到磁盘中，<br/>
> 但需要注意 RocksDB 不支持同步的 Checkpoint，构造方法中没有同步快照这个选项。<br/>
> 不过 RocksDB 支持增量的 Checkpoint，意味着并不需要把所有 sst 文件上传到 Checkpoint 目录，仅需要上传新生成的 sst 文件即可。它的 Checkpoint 存储在外部文件系统（本地或HDFS），<br/>
> 其容量限制只要单个 TaskManager 上 State 总量不超过它的内存+磁盘，单 Key最大 2G，总大小不超过配置的文件系统容量即可。<br/>
> `推荐使用的场景为：超大状态的作业，例如天级窗口聚合、需要开启 HA 的作业、最好是对状态读写性能要求不高的作业。`

##### 7.5.1.4 CheckPoint配置方式

###### 7.5.1.4.1 全局配置 

> 修改flink-conf.yaml

```
#这里可以配置
#jobmanager(即MemoryStateBackend), 
#filesystem(即FsStateBackend), 
#rocksdb(即RocksDBStateBackend)
state.backend: filesystem 
state.checkpoints.dir: hdfs://namenode:8020/flink/checkpoints
```

###### 7.5.1.4.2 在代码中配置

```
//1.MemoryStateBackend--开发中不用
    env.setStateBackend(new MemoryStateBackend)
//2.FsStateBackend--开发中可以使用--适合一般状态--秒级/分钟级窗口...
    env.setStateBackend(new FsStateBackend("hdfs路径或测试时的本地路径"))
//3.RocksDBStateBackend--开发中可以使用--适合超大状态--天级窗口...
env.setStateBackend(new RocksDBStateBackend(filebackend, true))

注意:RocksDBStateBackend还需要引入依赖
    <dependency>
       <groupId>org.apache.flink</groupId>
       <artifactId>flink-statebackend-rocksdb_2.11</artifactId>
       <version>1.7.2</version>
    </dependency>
```

> 总结:
>> `后面的学习测试和开发都使用`:
>> env.setStateBackend(new FsStateBackend("hdfs路径或测试时的本地路径"))
> 
> `特殊情况下的超大状态用`:
>> env.setStateBackend(new RocksDBStateBackend(filebackend, true))

##### 7.5.1.5 代码演示

```
import org.apache.commons.lang3.SystemUtils;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.runtime.state.filesystem.FsStateBackend;
import org.apache.flink.streaming.api.CheckpointingMode;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.CheckpointConfig;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;
import org.apache.flink.util.Collector;

import java.util.Properties;

/**
 * Author xiaoma
 * Date 2020/10/17 15:35
 * Desc 演示Checkpoint参数设置(也就是Checkpoint执行流程中的步骤0相关的参数设置)
 */
public class CheckpointDemo01 {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //===========Checkpoint参数设置====
        //===========类型1:必须参数=============
        //设置Checkpoint的时间间隔为1000ms做一次Checkpoint/其实就是每隔1000ms发一次Barrier!
        env.enableCheckpointing(1000);
        //设置State状态存储介质
        /*if(args.length > 0){
            env.setStateBackend(new FsStateBackend(args[0]));
        }else {
            env.setStateBackend(new FsStateBackend("file:///D:\\data\\ckp"));
        }*/
        if(SystemUtils.IS_OS_WINDOWS){
            env.setStateBackend(new FsStateBackend("file:///D:\\data\\ckp"));
        }else{
            env.setStateBackend(new FsStateBackend("hdfs://node1:8020/flink-checkpoint/checkpoint"));
        }
        //===========类型2:建议参数===========
        //设置两个Checkpoint 之间最少等待时间,如设置Checkpoint之间最少是要等 500ms(为了避免每隔1000ms做一次Checkpoint的时候,前一次太慢和后一次重叠到一起去了)
        //如:高速公路上,每隔1s关口放行一辆车,但是规定了两车之前的最小车距为500m
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);//默认是0
        //设置如果在做Checkpoint过程中出现错误，是否让整体任务失败：true是  false不是
        env.getCheckpointConfig().setFailOnCheckpointingErrors(false);//默认是true
        //设置是否清理检查点,表示 Cancel 时是否需要保留当前的 Checkpoint，默认 Checkpoint会在作业被Cancel时被删除
        //ExternalizedCheckpointCleanup.DELETE_ON_CANCELLATION：true,当作业被取消时，删除外部的checkpoint(默认值)
        //ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION：false,当作业被取消时，保留外部的checkpoint
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);

        //===========类型3:直接使用默认的即可===============
        //设置checkpoint的执行模式为EXACTLY_ONCE(默认)
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        //设置checkpoint的超时时间,如果 Checkpoint在 60s内尚未完成说明该次Checkpoint失败,则丢弃。
        env.getCheckpointConfig().setCheckpointTimeout(60000);//默认10分钟
        //设置同一时间有多少个checkpoint可以同时执行
        env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);//默认为1


        DataStream<String> linesDS = env.socketTextStream("node1", 9999);

        //3.Transformation
        //3.1切割出每个单词并直接记为1
        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOneDS = linesDS.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                //value就是每一行
                String[] words = value.split(" ");
                for (String word : words) {
                    out.collect(Tuple2.of(word, 1));
                }
            }
        });
        //3.2分组
        //注意:批处理的分组是groupBy,流处理的分组是keyBy
        KeyedStream<Tuple2<String, Integer>, Tuple> groupedDS = wordAndOneDS.keyBy(0);
        //3.3聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> aggResult = groupedDS.sum(1);

        SingleOutputStreamOperator<String> result = (SingleOutputStreamOperator<String>) aggResult.map(new RichMapFunction<Tuple2<String, Integer>, String>() {
            @Override
            public String map(Tuple2<String, Integer> value) throws Exception {
                return value.f0 + ":::" + value.f1;
            }
        });

        //4.sink
        //result.print();
        Properties props = new Properties();
        props.setProperty("bootstrap.servers", "node1:9092");
        FlinkKafkaProducer<String> kafkaSink = new FlinkKafkaProducer<>("flink_kafka",  new SimpleStringSchema(),  props);
        result.addSink(kafkaSink);
        //5.execute
        env.execute();

        // /export/server/kafka/bin/kafka-console-consumer.sh --bootstrap-server node1:9092 --topic flink_kafka
    }
}
```

#### 7.5.2 状态恢复和重启策略

##### 7.5.2.1 自动重启策略和恢复

###### 7.5.2.1.1 重启策略配置方式

> `配置文件中`
>> 在flink-conf.yml中可以进行配置,示例如下:

```
restart-strategy: fixed-delay
restart-strategy.fixed-delay.attempts: 3
restart-strategy.fixed-delay.delay: 10 s
```

> `代码中`
>> 还可以在代码中针对该任务进行配置,示例如下:

```
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
    > 3, // 重启次数
    Time.of(10, TimeUnit.SECONDS) // 延迟时间间隔
))
```

###### 7.5.2.1.2 重启策略分类

> 1 `默认重启策略`
>> 如果配置了Checkpoint,而没有配置重启策略,那么代码中出现了非致命错误时,程序会无限重启

> 2 `无重启策略`

```
 Job直接失败，不会尝试进行重启
 设置方式1:
 restart-strategy: none
  
 设置方式2:
 无重启策略也可以在程序中设置
 val env = ExecutionEnvironment.getExecutionEnvironment()
 env.setRestartStrategy(RestartStrategies.noRestart())
```

> 3 `固定延迟重启策略--开发中使用`

```
设置方式1:
 重启策略可以配置flink-conf.yaml的下面配置参数来启用，作为默认的重启策略:
 例子:
 restart-strategy: fixed-delay
 restart-strategy.fixed-delay.attempts: 3
 restart-strategy.fixed-delay.delay: 10 s
  
 设置方式2:
 也可以在程序中设置:
 val env = ExecutionEnvironment.getExecutionEnvironment()
 env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
   3, // 最多重启3次数
   Time.of(10, TimeUnit.SECONDS) // 重启时间间隔
 ))
 上面的设置表示:如果job失败,重启3次, 每次间隔10
```

> 4 `失败率重启策略--开发偶尔使用`

```
 设置方式1:
 失败率重启策略可以在flink-conf.yaml中设置下面的配置参数来启用:
 例子:
 restart-strategy:failure-rate
 restart-strategy.failure-rate.max-failures-per-interval: 3
 restart-strategy.failure-rate.failure-rate-interval: 5 min
 restart-strategy.failure-rate.delay: 10 s
  
 设置方式2:
 失败率重启策略也可以在程序中设置:
 val env = ExecutionEnvironment.getExecutionEnvironment()
 env.setRestartStrategy(RestartStrategies.failureRateRestart(
   3, // 每个测量时间间隔最大失败次数
   Time.of(5, TimeUnit.MINUTES), //失败率测量的时间间隔
   Time.of(10, TimeUnit.SECONDS) // 两次连续重启的时间间隔
 ))
 上面的设置表示:如果5分钟内job失败不超过三次,自动重启, 每次间隔10s (如果5分钟内程序失败超过3次,则程序退出)
```
###### 7.5.2.1.3 代码演示

```
import org.apache.commons.lang3.SystemUtils;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.runtime.state.filesystem.FsStateBackend;
import org.apache.flink.streaming.api.CheckpointingMode;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.CheckpointConfig;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

import java.util.concurrent.TimeUnit;

/**
 * Author xiaoma
 * Date 2020/10/17 15:35
 * Desc 演示Checkpoint+重启策略
 */
public class CheckpointDemo02_RestartStrategy {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //===========Checkpoint参数设置====
        //===========类型1:必须参数=============
        //设置Checkpoint的时间间隔为1000ms做一次Checkpoint/其实就是每隔1000ms发一次Barrier!
        env.enableCheckpointing(1000);
        //设置State状态存储介质
        /*if(args.length > 0){
            env.setStateBackend(new FsStateBackend(args[0]));
        }else {
            env.setStateBackend(new FsStateBackend("file:///D:\\data\\ckp"));
        }*/
        if(SystemUtils.IS_OS_WINDOWS){
            env.setStateBackend(new FsStateBackend("file:///D:\\data\\ckp"));
        }else{
            env.setStateBackend(new FsStateBackend("hdfs://node1:8020/flink-checkpoint/checkpoint"));
        }
        //===========类型2:建议参数===========
        //设置两个Checkpoint 之间最少等待时间,如设置Checkpoint之间最少是要等 500ms(为了避免每隔1000ms做一次Checkpoint的时候,前一次太慢和后一次重叠到一起去了)
        //如:高速公路上,每隔1s关口放行一辆车,但是规定了两车之前的最小车距为500m
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);//默认是0
        //设置如果在做Checkpoint过程中出现错误，是否让整体任务失败：true是  false不是
        env.getCheckpointConfig().setFailOnCheckpointingErrors(false);//默认是true
        //设置是否清理检查点,表示 Cancel 时是否需要保留当前的 Checkpoint，默认 Checkpoint会在作业被Cancel时被删除
        //ExternalizedCheckpointCleanup.DELETE_ON_CANCELLATION：true,当作业被取消时，删除外部的checkpoint(默认值)
        //ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION：false,当作业被取消时，保留外部的checkpoint
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);

        //===========类型3:直接使用默认的即可===============
        //设置checkpoint的执行模式为EXACTLY_ONCE(默认)
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        //设置checkpoint的超时时间,如果 Checkpoint在 60s内尚未完成说明该次Checkpoint失败,则丢弃。
        env.getCheckpointConfig().setCheckpointTimeout(60000);//默认10分钟
        //设置同一时间有多少个checkpoint可以同时执行
        env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);//默认为1

        //=============重启策略===========
        //-1.默认策略:配置了Checkpoint而没有配置重启策略默认使用无限重启
        //-2.配置无重启策略
        //env.setRestartStrategy(RestartStrategies.noRestart());
        //-3.固定延迟重启策略--开发中使用!
        //重启3次,每次间隔10s
        /*env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
                3, //尝试重启3次
                Time.of(10, TimeUnit.SECONDS))//每次重启间隔10s
        );*/
        //-4.失败率重启--偶尔使用
        //5分钟内重启3次(第3次不包括,也就是最多重启2次),每次间隔10s
        /*env.setRestartStrategy(RestartStrategies.failureRateRestart(
                3, // 每个测量时间间隔最大失败次数
                Time.of(5, TimeUnit.MINUTES), //失败率测量的时间间隔
                Time.of(10, TimeUnit.SECONDS) // 每次重启的时间间隔
        ));*/

        //上面的能看懂就行,开发中使用下面的代码即可
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, Time.of(10, TimeUnit.SECONDS)));

        DataStream<String> linesDS = env.socketTextStream("node1", 9999);

        //3.Transformation
        //3.1切割出每个单词并直接记为1
        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOneDS = linesDS.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                //value就是每一行
                String[] words = value.split(" ");
                for (String word : words) {
                    if(word.equals("bug")){
                        System.out.println("手动模拟的bug...");
                        throw new RuntimeException("手动模拟的bug...");
                    }
                    out.collect(Tuple2.of(word, 1));
                }
            }
        });
        //3.2分组
        //注意:批处理的分组是groupBy,流处理的分组是keyBy
        KeyedStream<Tuple2<String, Integer>, Tuple> groupedDS = wordAndOneDS.keyBy(0);
        //3.3聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> result = groupedDS.sum(1);

        //4.sink
        result.print();

        //5.execute
        env.execute();
    }
}

```

#### 7.5.3 SavePoint 状态恢复

##### 7.5.3.1 SavePoint介绍

> Savepoint:保存点,类似于以前玩游戏的时候,遇到难关了/遇到boss了,赶紧手动存个档,然后接着玩,如果失败了,赶紧从上次的存档中恢复,然后接着玩

> 在实际开发中,可能会遇到这样的情况:如要对集群进行停机维护/扩容...<br/>
> 那么这时候需要执行一次Savepoint也就是执行一次手动的Checkpoint/也就是手动的发一个barrier栅栏,那么这样的话,程序的所有状态都会被执行快照并保存,当维护/扩容完毕之后,可以从上一次Savepoint的目录中进行恢复!

##### 7.5.3.2 SavePoint VS CheckPoint

> `Checkpoint`:`自动的`! 是程序按照配置参数定期自动生成的快照,是程序用来容错的,程序自动重启恢复的时候可以使用!
>> 也可以按照上面的方式使用Savepoint的恢复方式对Checkpoint进行恢复!

> `SavePoint` :`手动的`! 是手动的执行命令让让程序进行一次全局的Checkpoint!是程序用来手动恢复的!
>> 一般是运维对公司的Flink任务做定期备份


 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/130.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/131.png)

##### 7.5.3.3 SavePoint演示

###### 7.5.3.3.1 CheckPoint按照SavePoint方式恢复

> 前面的重启策略是Checkpoint的自动恢复,接下来演示Checkpoint按照Savepoint的方式进行手动恢复<br/>
> 了解一下`Checkpoint的手动恢复`<br/>
> (也就是使用Savepoint恢复的方式来恢复Checkpoint,帮助后续理解Savepoint的恢复)

> 1.打包

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/132.png)

> 2.启动Flink集群(本地单机版,集群版都可以)

```
/export/server/flink/bin/start-cluster.sh
```

> 3.访问webUI

http://node1:8081/#/overview
http://node2:8081/#/overview

> 4.上传jar

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/133.png)

> 5.开启

```
nc -lk 9999
/export/server/kafka/bin/kafka-console-consumer.sh --bootstrap-server node1:9092 --topic flink_kafka
```

> 6.提交任务

```
cn.itcast.checkpoint.CheckpointDemo01
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/134.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/135.png)

> 7.查看kafak消费者输出和HDFS目录

> 8.取消任务/停止任务

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/136.png)

> 9.再次启动任务

```
cn.itcast.checkpoint.CheckpointDemo01
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/137.png)

> 需要指定从哪个Checkpoint目录恢复

```
hdfs://node1:8020/flink-checkpoint/checkpoint/4654a268cb1839b0c0491bb317599c6a/chk-68
```


> 10.再次发送数据并观察输出

> 11.停止所有

###### 7.5.3.3.2 SavePoint恢复

```
#准备
nc -lk 9999
/export/server/kafka/bin/kafka-console-consumer.sh --bootstrap-server node1:9092 --topic flink_kafka
上传jar包到root

# 启动yarn session
/export/server/flink/bin/yarn-session.sh -n 2 -tm 800 -s 1 -d


# 运行job-会自动执行Checkpoint
/export/server/flink/bin/flink run --class cn.itcast.checkpoint.CheckpointDemo01 /root/ckp.jar

# 手动创建savepoint--相当于手动做了一次Checkpoint
/export/server/flink/bin/flink savepoint 6d6142bf677315430a4f2857edc507eb hdfs://node1:8020/flink-checkpoint/savepoint/

# 停止job
/export/server/flink/bin/flink cancel 6d6142bf677315430a4f2857edc507eb 

# 重新启动job,手动加载savepoint数据
/export/server/flink/bin/flink run -s hdfs://node1:8020/flink-checkpoint/savepoint/savepoint-6d6142-65ab2a28b08f --class cn.itcast.checkpoint.CheckpointDemo01  /root/ckp.jar 

# 停止yarn session
yarn application -kill application_1602930321327_0002	
```

### 7.6 扩展：End-to-End Exactly-Once

#### 7.6.1 流处理的数据处理语义

> 对于批处理，fault-tolerant（容错性）很容易做，失败只需要replay，就可以完美做到容错。

> 对于流处理，数据流本身是动态，没有所谓的开始或结束，虽然可以replay buffer的部分数据，但fault-tolerant做起来会复杂的多

> 流处理（有时称为事件处理）可以简单地描述为是对无界数据或事件的连续处理。流或事件处理应用程序可以或多或少地被描述为有向图，并且通常被描述为有向无环图（DAG）。在这样的图中，每个边表示数据或事件流，每个顶点表示运算符，会使用程序中定义的逻辑处理来自相邻边的数据或事件。有两种特殊类型的顶点，通常称为 sources 和 sinks。sources读取外部数据/事件到应用程序中，而 sinks 通常会收集应用程序生成的结果。下图是流式应用程序的示例。有如下特点：

> 分布式情况下是由多个Source(读取数据)节点、多个Operator(数据处理)节点、多个Sink(输出)节点构成

> 每个节点的并行数可以有差异，且每个节点都有可能发生故障

> 对于数据正确性最重要的一点，就是当发生故障时，是怎样容错与恢复的。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/138.png)

> 流处理引擎通常为应用程序提供了三种数据处理语义：最多一次、至少一次和精确一次。

> 如下是对这些不同处理语义的宽松定义(一致性由弱到强)：
>> `At most noce < At least once < Exactly once < End to End Exactly once`

##### 7.6.1.1 At-most-once-最多一次

> 这本质上是简单的恢复方式，也就是直接从失败处的下个数据开始恢复程序，之前的失败数据处理就不管了。可以保证数据或事件最多由应用程序中的所有算子处理一次。 这意味着如果数据在被流应用程序完全处理之前发生丢失，则不会进行其他重试或者重新发送。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/139.png)

##### 7.6.1.2 At-least-once-至少一次

> 应用程序中的所有算子都保证数据或事件至少被处理一次。这通常意味着如果事件在流应用程序完全处理之前丢失，则将从源头重放或重新传输事件。然而，由于事件是可以被重传的，因此一个事件有时会被处理多次(至少一次)，至于有没有重复数据，不会关心，所以这种场景需要人工干预自己处理重复数据

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/140.png)

##### 7.6.1.3 Exactly-once-精确一次

> Exactly-Once 是 Flink、Spark 等流处理系统的核心特性之一，这种语义会保证每一条消息只被流处理系统处理一次。即使是在各种故障的情况下，流应用程序中的所有算子都保证事件只会被『精确一次』的处理。（也有文章将 Exactly-once 翻译为：完全一次，恰好一次）

> Flink实现『精确一次』的分布式快照/状态检查点方法受到 Chandy-Lamport 分布式快照算法的启发。通过这种机制，流应用程序中每个算子的所有状态都会定期做 checkpoint。如果是在系统中的任何地方发生失败，每个算子的所有状态都回滚到最新的全局一致 checkpoint 点。在回滚期间，将暂停所有处理。源也会重置为与最近 checkpoint 相对应的正确偏移量。整个流应用程序基本上是回到最近一次的一致状态，然后程序可以从该状态重新启动。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/141.png)

##### 7.6.1.4 End-to-End Exactly-Once-端到端的精确一次

> Flink 在1.4.0 版本引入『exactly-once』并号称支持『End-to-End Exactly-Once』“端到端的精确一次”语义。
> 它指的是 Flink 应用从 Source 端开始到 Sink 端结束，数据必须经过的起始点和结束点。

> 注意：
>> `『exactly-once』`和`『End-to-End Exactly-Once』`的区别:
>>> `Exactly-Once:只要求中间的Transformation做到数据只被正确处理一次!`
>>>  ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/142.png)
>>> `End-to-End Exactly-Once:要求数据从Source到Transformation到Sink都只被正确处理一次`
>>>  ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/143.png)
>>>  ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/144.png)

##### 7.6.1.5 注意：精确一次? `有效一次!`

> 有些人可能认为『Exactly-Once』描述了事件处理的保证，其中流中的每个事件只被处理一次。实际上，没有引擎能够保证正好只处理一次。在面对任意故障时，不可能保证每个算子中的用户定义逻辑在每个事件中只执行一次，因为用户代码被部分执行的可能性是永远存在的。

> 那么，当引擎声明『Exactly-Once』处理语义时，它们能保证什么呢？如果不能保证用户逻辑只执行一次，那么什么逻辑只执行一次？当引擎声明『精确一次』处理语义时，`它们实际上是在说，它们可以保证引擎管理的状态更新只提交一次到持久的后端存储浊者说数据只被正确处理一次`。

> 事件的处理可以发生多次，但是该处理的效果只在持久后端状态存储中反映一次。因此，我们认为有效地描述这些处理语义最好的术语是『`有效一次`』（effectively once）

##### 7.6.1.6 补充：流计算系统如何支持一致性语义

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/145.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/146.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/147.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/148.png)

#### 7.6.2 End-to-End Exactly-Once的实现

> 通过前面的学习，我们了解到，Flink内部借助分布式快照Checkpoint已经实现了内部的Exactly-Once，但是Flink 自身是无法保证外部其他系统“精确一次”语义的，所以 Flink 若要实现所谓“端到端（End to End）的精确一次”的要求，那么外部系统必须支持“精确一次”语义；然后借助一些其他手段才能实现。如下：

##### 7.6.2.1 Source

> 发生故障时需要支持重设数据的读取位置，如Kafka可以通过offset来实现（其他的没有offset系统，我们可以自己实现累加器计数）

##### 7.6.2.2 Transformation

> 也就是Flink内部，已经通过Checkpoint保证了，如果发生故障或出错时，Flink应用重启后会从最新成功完成的checkpoint中恢复——重置应用状态并回滚状态到checkpoint中输入流的正确位置，之后再开始执行数据处理，就好像该故障或崩溃从未发生过一般。

> 分布式快照机制
>> 我们在之前的课程中讲解过 Flink 的容错机制，Flink 提供了失败恢复的容错机制，而这个容错机制的核心就是持续创建分布式数据流的快照来实现。
>>  ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/149.png)
>> 同 Spark 相比，Spark 仅仅是针对 Driver 的故障恢复 Checkpoint。而 Flink 的快照可以到算子级别，并且对全局数据也可以做快照。Flink 的分布式快照受到  Chandy-Lamport 分布式快照算法启发，同时进行了量身定做。

> Barrier
>> Flink 分布式快照的核心元素之一是 Barrier（数据栅栏），我们也可以把 Barrier 简单地理解成一个标记，该标记是严格有序的，并且随着数据流往下流动。每个 Barrier 都带有自己的 ID，Barrier 极其轻量，并不会干扰正常的数据处理。
> 
>>  ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/150.png)
> 
>> 如上图所示，假如我们有一个从左向右流动的数据流，Flink 会依次生成 snapshot 1、 snapshot 2、snapshot 3……Flink 中有一个专门的“协调者”负责收集每个 snapshot 的位置信息，这个“协调者”也是高可用的。
>
>>Barrier 会随着正常数据继续往下流动，每当遇到一个算子，算子会插入一个标识，这个标识的插入时间是上游所有的输入流都接收到 snapshot n。与此同时，当我们的 sink 算子接收到所有上游流发送的 Barrier 时，那么就表明这一批数据处理完毕，Flink 会向“协调者”发送确认消息，表明当前的 snapshot n 完成了。当所有的 sink 算子都确认这批数据成功处理后，那么本次的 snapshot 被标识为完成。
> 
>> 这里就会有一个问题，因为 Flink 运行在分布式环境中，一个 operator 的上游会有很多流，每个流的 barrier n 到达的时间不一致怎么办？这里 Flink 采取的措施是：快流等慢流。
> 
>>  ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/151.png)
>
> 拿上图的 barrier n 来说，其中一个流到的早，其他的流到的比较晚。当第一个 barrier n到来后，当前的 operator 会继续等待其他流的 barrier n。直到所有的barrier n 到来后，operator 才会把所有的数据向下发送。

> 异步和增量
>> 按照上面我们介绍的机制，每次在把快照存储到我们的状态后端时，如果是同步进行就会阻塞正常任务，从而引入延迟。因此 Flink 在做快照存储时，可采用异步方式。
> 
>> 此外，由于 checkpoint 是一个全局状态，用户保存的状态可能非常大，多数达 G 或者 T 级别。在这种情况下，checkpoint 的创建会非常慢，而且执行时占用的资源也比较多，因此 Flink 提出了增量快照的概念。也就是说，每次都是进行的全量 checkpoint，是基于上次进行更新的。

##### 7.6.2.3 Sink

> 需要支持幂等写入或事务写入(Flink的两阶段提交需要事务支持)

###### 7.6.2.3.1 幂等写入（Idempotent Writes）

> 幂等写操作是指：任意多次向一个系统写入数据，只对目标系统产生一次结果影响。

> 例如，重复向一个HashMap里插入同一个Key-Value二元对，第一次插入时这个HashMap发生变化，后续的插入操作不会改变HashMap的结果，这就是一个幂等写操作。

> HBase、Redis和Cassandra这样的KV数据库一般经常用来作为Sink，用以实现端到端的Exactly-Once。

> 需要注意的是，并不是说一个KV数据库就百分百支持幂等写。幂等写对KV对有要求，那就是Key-Value必须是可确定性（Deterministic）计算的。假如我们设计的Key是：name + curTimestamp，每次执行数据重发时，生成的Key都不相同，会产生多次结果，整个操作不是幂等的。因此，为了追求端到端的Exactly-Once，我们设计业务逻辑时要尽量使用确定性的计算逻辑和数据模型。

> 如:往下面的Sink写入数据
>> HBase:写入同样的数据会覆盖!
>> Redis:写入同样的数据会覆盖!

###### 7.6.2.3.2 事务写入（Transactional Writes）

> Flink借鉴了数据库中的事务处理技术，同时结合自身的Checkpoint机制来保证Sink只对外部输出产生一次影响。大致的流程如下:

> Flink先将待输出的数据保存下来暂时不向外部系统提交，等到Checkpoint结束时，Flink上下游所有算子的数据都是一致的时候，Flink将之前保存的数据全部提交（Commit）到外部系统。换句话说，只有经过Checkpoint确认的数据才向外部系统写入。

> 如下图所示，如果使用事务写，那只把时间戳3之前的输出提交到外部系统，时间戳3以后的数据（例如时间戳5和8生成的数据）暂时保存下来，等待下次Checkpoint时一起写入到外部系统。这就避免了时间戳5这个数据产生多次结果，多次写入到外部系统。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/152.png)

> 在事务写的具体实现上，Flink目前提供了两种方式：
>> 1.预写日志（Write-Ahead-Log，WAL）
>> 2.两阶段提交（Two-Phase-Commit，2PC）

> 这两种方式区别主要在于：
>> 1.WAL方式通用性更强，适合几乎所有外部系统，但也不能提供百分百端到端的Exactly-Once，因为WAL预习日志会先写内存，而内存是易失介质。
>> 2.如果外部系统自身就支持事务（比如MySQL、Kafka），可以使用2PC方式，可以提供百分百端到端的Exactly-Once。

> 事务写的方式能提供端到端的Exactly-Once一致性，它的代价也是非常明显的，就是牺牲了延迟。输出数据不再是实时写入到外部系统，而是分批次地提交。目前来说，没有完美的故障恢复和Exactly-Once保障机制，对于开发者来说，需要在不同需求之间权衡。

#### 7.6.3 Flink+Kafka的End-to-End Exactly-Once

> 在上一小节我们了解到Flink的 End-to-End Exactly-Once需要Checkpoint+事务的提交/回滚操作，在分布式系统中协调提交和回滚的一个常见方法就是使用两阶段提交协议。接下来我们了解下Flink的TwoPhaseCommitSinkFunction是如何支持End-to-End Exactly-Once的

##### 7.6.3.1 版本说明

> Flink 1.4版本之前，支持Exactly Once语义，仅限于应用内部。

> Flink 1.4版本之后，通过两阶段提交(TwoPhaseCommitSinkFunction)支持End-To-End Exactly Once，而且要求Kafka 0.11+。

> 利用TwoPhaseCommitSinkFunction是通用的管理方案，只要实现对应的接口，而且Sink的存储支持变乱提交，即可实现端到端的划一性语义。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/153.png)

##### 7.6.3.2 两阶段提交-API

> 在 Flink 中的Two-Phase-Commit-2PC两阶段提交的实现方法被封装到了 TwoPhaseCommitSinkFunction 这个抽象类中，只需要实现其中的beginTransaction、preCommit、commit、abort 四个方法就可以实现“精确一次”的处理语义，如FlinkKafkaProducer就实现了该类并实现了这些方法

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/154.png)

> 1.beginTransaction，在开启事务之前，我们在目标文件系统的临时目录中创建一个临时文件，后面在处理数据时将数据写入此文件；

> 2.preCommit，在预提交阶段，刷写（flush）文件，然后关闭文件，之后就不能写入到文件了，我们还将为属于下一个检查点的任何后续写入启动新事务；

> 3.commit，在提交阶段，我们将预提交的文件原子性移动到真正的目标目录中，请注意，这会增加输出数据可见性的延迟；

> 4.abort，在中止阶段，我们删除临时文件。

##### 7.6.3.3 两阶段提交-简单流程

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/155.png)

> 整个过程可以总结为下面四个阶段：
>> 1.一旦 Flink 开始做 checkpoint 操作，那么就会进入 pre-commit “预提交”阶段，同时JobManager的Coordinator 会将 Barrier 注入数据流中 ；
>> 2.当所有的 barrier 在算子中成功进行一遍传递，并完成快照后，则“预提交”阶段完成；
>> 3.等所有的算子完成“预提交”，就会发起一个commit“提交”动作，但是任何一个“预提交”失败都会导致 Flink 回滚到最近的 checkpoint；

##### 7.6.3.4 两阶段提交-详细流程

> 需求
>> 接下来将介绍两阶段提交协议，以及它如何在一个读写Kafka的Flink程序中实现端到端的Exactly-Once语义。Kafka经常与Flink一起使用，且Kafka在最近的0.11版本中添加了对事务的支持。这意味着现在通过Flink读写Kafaka，并提供端到端的Exactly-Once语义有了必要的支持。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/156.png)

> 在上图中，我们有：
>> – 从Kafka读取的数据源（Flink内置的KafkaConsumer）
>> – 窗口聚合
>> – 将数据写回Kafka的数据输出端（Flink内置的KafkaProducer）

> 要使数据输出端提供Exactly-Once保证，它必须将所有数据通过一个事务提交给Kafka。提交捆绑了两个checkpoint之间的所有要写入的数据。这可确保在发生故障时能回滚写入的数据。

> 但是在分布式系统中，通常会有多个并发运行的写入任务的，简单的提交或回滚是不够的，因为所有组件必须在提交或回滚时“一致”才能确保一致的结果。

> Flink使用两阶段提交协议及预提交阶段来解决这个问题。

> `预提交-内部状态`
>> 在checkpoint开始的时候，即两阶段提交协议的“预提交”阶段。当checkpoint开始时，Flink的JobManager会将checkpoint barrier（将数据流中的记录分为进入当前checkpoint与进入下一个checkpoint）注入数据流。
>>  brarrier在operator之间传递。对于每一个operator，它触发operator的状态快照写入到state backend。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/157.png)

> 数据源保存了消费Kafka的偏移量(offset)，之后将checkpoint barrier传递给下一个operator。<br/>
> 这种方式仅适用于operator具有『内部』状态。所谓内部状态，是指Flink state backend保存和管理的 -例如，第二个operator中window聚合算出来的sum值。当一个进程有它的内部状态的时候，除了在checkpoint之前需要将数据变更写入到state backend，不需要在预提交阶段执行任何其他操作。Flink负责在checkpoint成功的情况下正确提交这些写入，或者在出现故障时中止这些写入。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/158.png)

> `预提交-外部状态`
>> 但是，当进程具有『外部』状态时，需要作些额外的处理。外部状态通常以写入外部系统（如Kafka）的形式出现。在这种情况下，为了提供Exactly-Once保证，外部系统必须支持事务，这样才能和两阶段提交协议集成。
>>在该示例中的数据需要写入Kafka，因此数据输出端（Data Sink）有外部状态。在这种情况下，在预提交阶段，除了将其状态写入state backend之外，数据输出端还必须预先提交其外部事务。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/159.png)

> 当checkpoint barrier在所有operator都传递了一遍，并且触发的checkpoint回调成功完成时，预提交阶段就结束了。所有触发的状态快照都被视为该checkpoint的一部分。checkpoint是整个应用程序状态的快照，包括预先提交的外部状态。如果发生故障，我们可以回滚到上次成功完成快照的时间点。

> `提交阶段`
>> 下一步是通知所有operator，checkpoint已经成功了。这是两阶段提交协议的提交阶段，JobManager为应用程序中的每个operator发出checkpoint已完成的回调。
>> 数据源和widnow operator没有外部状态，因此在提交阶段，这些operator不必执行任何操作。但是，数据输出端（Data Sink）拥有外部状态，此时应该提交外部事务。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/160.png)

> 总结
>> 我们对上述知识点总结下：
> 
>> 1.一旦所有operator完成预提交，就提交一个commit。
> 
>> 2.如果只要有一个预提交失败，则所有其他提交都将中止，我们将回滚到上一个成功完成的checkpoint。
> 
>> 3.在预提交成功之后，提交的commit需要保证最终成功 – operator和外部系统都需要保障这点。如果commit失败（例如，由于间歇性网络问题），整个Flink应用程序将失败，应用程序将根据用户的重启策略重新启动，还会尝试再提交。这个过程至关重要，因为如果commit最终没有成功，将会导致数据丢失。
> 
>> 4.完整的实现两阶段提交协议可能有点复杂，这就是为什么Flink将它的通用逻辑提取到抽象类TwoPhaseCommitSinkFunction中的原因。

#### 7.6.4 代码示例

##### 7.6.4.1 Flink+Kafka实现End-to-End Exactly-Once

```
import org.apache.commons.lang3.SystemUtils;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.common.restartstrategy.RestartStrategies;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.runtime.state.filesystem.FsStateBackend;
import org.apache.flink.streaming.api.CheckpointingMode;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.CheckpointConfig;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;
import org.apache.flink.streaming.connectors.kafka.internals.KeyedSerializationSchemaWrapper;
import org.apache.flink.util.Collector;

import java.util.Properties;
import java.util.Random;
import java.util.concurrent.TimeUnit;

/**
 * Author xiaoma
 * Date 2020/10/18 11:42
 * Desc
 * Kafka --> Flink-->Kafka  的End-To-End-Exactly-once
 * 直接使用
 * FlinkKafkaConsumer  +  Flink的Checkpoint  +  FlinkKafkaProducer
 */
public class Kafka_Flink_Kafka_EndToEnd_ExactlyOnce_Yes {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //===========Checkpoint参数设置====
        //===========类型1:必须参数=============
        //设置Checkpoint的时间间隔为1000ms做一次Checkpoint/其实就是每隔1000ms发一次Barrier!
        env.enableCheckpointing(1000);
        //设置State状态存储介质
        if (SystemUtils.IS_OS_WINDOWS) {
            env.setStateBackend(new FsStateBackend("file:///D:\\data\\ckp"));
        } else {
            env.setStateBackend(new FsStateBackend("hdfs://node1:8020/flink-checkpoint/checkpoint"));
        }
        //===========类型2:建议参数===========
        //设置两个Checkpoint 之间最少等待时间,如设置Checkpoint之间最少是要等 500ms(为了避免每隔1000ms做一次Checkpoint的时候,前一次太慢和后一次重叠到一起去了)
        //如:高速公路上,每隔1s关口放行一辆车,但是规定了两车之前的最小车距为500m
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);//默认是0
        //设置如果在做Checkpoint过程中出现错误，是否让整体任务失败：true是  false不是
        env.getCheckpointConfig().setFailOnCheckpointingErrors(false);//默认是true
        //设置是否清理检查点,表示 Cancel 时是否需要保留当前的 Checkpoint，默认 Checkpoint会在作业被Cancel时被删除
        //ExternalizedCheckpointCleanup.DELETE_ON_CANCELLATION：true,当作业被取消时，删除外部的checkpoint(默认值)
        //ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION：false,当作业被取消时，保留外部的checkpoint
        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);

        //===========类型3:直接使用默认的即可===============
        //设置checkpoint的执行模式为EXACTLY_ONCE(默认)
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        //设置checkpoint的超时时间,如果 Checkpoint在 60s内尚未完成说明该次Checkpoint失败,则丢弃。
        env.getCheckpointConfig().setCheckpointTimeout(60000);//默认10分钟
        //设置同一时间有多少个checkpoint可以同时执行
        env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);//默认为1

        //=============重启策略===========
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, Time.of(10, TimeUnit.SECONDS)));

        //2.Source
        Properties props_source = new Properties();
        props_source.setProperty("bootstrap.servers", "node1:9092");
        props_source.setProperty("group.id", "flink");
        props_source.setProperty("auto.offset.reset", "latest");
        props_source.setProperty("flink.partition-discovery.interval-millis", "5000");//会开启一个后台线程每隔5s检测一下Kafka的分区情况
        //props_source.setProperty("enable.auto.commit", "true");//没有Checkpoint的时候使用自动提交偏移量到默认主题:__consumer_offsets中
        //props_source.setProperty("auto.commit.interval.ms", "2000");
        //kafkaSource就是KafkaConsumer
        FlinkKafkaConsumer<String> kafkaSource = new FlinkKafkaConsumer<>("flink_kafka", new SimpleStringSchema(), props_source);
        kafkaSource.setStartFromLatest();
        //kafkaSource.setStartFromGroupOffsets();//设置从记录的offset开始消费,如果没有记录从auto.offset.reset配置开始消费
        //kafkaSource.setStartFromEarliest();//设置直接从Earliest消费,和auto.offset.reset配置无关
        kafkaSource.setCommitOffsetsOnCheckpoints(true);//执行Checkpoint的时候提交offset到Checkpoint(Flink用),并且提交一份到默认主题:__consumer_offsets(外部其他系统想用的话也可以获取到)
        DataStreamSource<String> kafkaDS = env.addSource(kafkaSource);

        //3.Transformation
        //3.1切割出每个单词并直接记为1
        SingleOutputStreamOperator<Tuple2<String, Integer>> wordAndOneDS = kafkaDS.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                //value就是每一行
                String[] words = value.split(" ");
                for (String word : words) {
                    Random random = new Random();
                    int i = random.nextInt(5);
                    if (i > 3) {
                        System.out.println("出bug了...");
                        throw new RuntimeException("出bug了...");
                    }
                    out.collect(Tuple2.of(word, 1));
                }
            }
        });
        //3.2分组
        //注意:批处理的分组是groupBy,流处理的分组是keyBy
        KeyedStream<Tuple2<String, Integer>, Tuple> groupedDS = wordAndOneDS.keyBy(0);
        //3.3聚合
        SingleOutputStreamOperator<Tuple2<String, Integer>> aggResult = groupedDS.sum(1);


        //3.4将聚合结果转为自定义的字符串格式
        SingleOutputStreamOperator<String> result = (SingleOutputStreamOperator<String>) aggResult.map(new RichMapFunction<Tuple2<String, Integer>, String>() {
            @Override
            public String map(Tuple2<String, Integer> value) throws Exception {
                return value.f0 + ":::" + value.f1;
            }
        });

        //4.sink
        //result.print();
        Properties props_sink = new Properties();
        props_sink.setProperty("bootstrap.servers", "node1:9092");
        props_sink.setProperty("transaction.timeout.ms", 1000 * 5 + "");//设置事务超时时间，也可在kafka配置中设置
        /*FlinkKafkaProducer<String> kafkaSink0 = new FlinkKafkaProducer<>(
                "flink_kafka",
                new SimpleStringSchema(),
                props_sink);*/
        FlinkKafkaProducer<String> kafkaSink = new FlinkKafkaProducer<>(
                "flink_kafka2",
                new KeyedSerializationSchemaWrapper<String>(new SimpleStringSchema()),
                props_sink,
                FlinkKafkaProducer.Semantic.EXACTLY_ONCE
        );

        result.addSink(kafkaSink);

        //5.execute
        env.execute();
        //测试:
        //1.创建主题 /export/server/kafka/bin/kafka-topics.sh --create --zookeeper node1:2181 --replication-factor 2 --partitions 3 --topic flink_kafka2
        //2.开启控制台生产者 /export/server/kafka/bin/kafka-console-producer.sh --broker-list node1:9092 --topic flink_kafka
        //3.开启控制台消费者 /export/server/kafka/bin/kafka-console-consumer.sh --bootstrap-server node1:9092 --topic flink_kafka2
    }
}
```

##### 7.6.4.2 Flink+MySQL实现End-to-End Exactly-Once

[https://www.jianshu.com/p/5bdd9a0d7d02](https://www.jianshu.com/p/5bdd9a0d7d02)

> 需求
>> 1.checkpoint每10s进行一次，此时用FlinkKafkaConsumer实时消费kafka中的消息
> 
>> 2.消费并处理完消息后，进行一次预提交数据库的操作
> 
>> 3.如果预提交没有问题，10s后进行真正的插入数据库操作，如果插入成功，进行一次checkpoint，flink会自动记录消费的offset，可以将checkpoint保存的数据放到hdfs中
> 
>> 4.如果预提交出错，比如在5s的时候出错了，此时Flink程序就会进入不断的重启中，重启的策略可以在配置中设置，checkpoint记录的还是上一次成功消费的offset，因为本次消费的数据在checkpoint期间，消费成功，但是预提交过程中失败了
> 
>> 5.注意此时数据并没有真正的执行插入操作，因为预提交（preCommit）失败，提交（commit）过程也不会发生。等将异常数据处理完成之后，再重新启动这个Flink程序，它会自动从上一次成功的checkpoint中继续消费数据，以此来达到Kafka到Mysql的Exactly-Once。

> 代码1

```
import org.apache.flink.api.common.ExecutionConfig;
import org.apache.flink.api.common.typeutils.base.VoidSerializer;
import org.apache.flink.api.java.typeutils.runtime.kryo.KryoSerializer;
import org.apache.flink.runtime.state.filesystem.FsStateBackend;
import org.apache.flink.shaded.jackson2.com.fasterxml.jackson.databind.node.ObjectNode;
import org.apache.flink.streaming.api.CheckpointingMode;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.TwoPhaseCommitSinkFunction;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.streaming.util.serialization.JSONKeyValueDeserializationSchema;
import org.apache.kafka.clients.CommonClientConfigs;

import java.sql.*;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Properties;


public class ExactlyOnceDemo_FlinkMysql {

    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);//方便测试
        env.enableCheckpointing(10000);
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(1000);
        //env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
        env.setStateBackend(new FsStateBackend("file:///D:/data/ckp"));

        //2.Source
        String topic = "flink_kafka";
        Properties props = new Properties();
        props.setProperty(CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG,"node1:9092");
        props.setProperty("group.id","flink");
        props.setProperty("auto.offset.reset","latest");//如果有记录偏移量从记录的位置开始消费,如果没有从最新的数据开始消费
        props.setProperty("flink.partition-discovery.interval-millis","5000");//开一个后台线程每隔5s检查Kafka的分区状态
        FlinkKafkaConsumer<ObjectNode> kafkaSource = new FlinkKafkaConsumer<>("topic_in", new JSONKeyValueDeserializationSchema(true), props);

        kafkaSource.setStartFromGroupOffsets();//从group offset记录的位置位置开始消费,如果kafka broker 端没有该group信息，会根据"auto.offset.reset"的设置来决定从哪开始消费
        kafkaSource.setCommitOffsetsOnCheckpoints(true);//Flink执行Checkpoint的时候提交偏移量(一份在Checkpoint中,一份在Kafka的默认主题中__comsumer_offsets(方便外部监控工具去看))

        DataStreamSource<ObjectNode> kafkaDS = env.addSource(kafkaSource);

        //3.transformation

        //4.Sink
        kafkaDS.addSink(new MySqlTwoPhaseCommitSink()).name("MySqlTwoPhaseCommitSink");

        //5.execute
        env.execute(ExactlyOnceDemo_FlinkMysql.class.getName());
    }
}

/**
 自定义kafka to mysql，继承TwoPhaseCommitSinkFunction,实现两阶段提交。
 功能：保证kafak to mysql 的Exactly-Once
 CREATE TABLE `t_test` (
   `id` bigint(20) NOT NULL AUTO_INCREMENT,
   `value` varchar(255) DEFAULT NULL,
   `insert_time` datetime DEFAULT NULL,
   PRIMARY KEY (`id`)
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
 */
class MySqlTwoPhaseCommitSink extends TwoPhaseCommitSinkFunction<ObjectNode, Connection, Void> {

    public MySqlTwoPhaseCommitSink() {
        super(new KryoSerializer<>(Connection.class, new ExecutionConfig()), VoidSerializer.INSTANCE);
    }

    /**
     * 执行数据入库操作
     */
    @Override
    protected void invoke(Connection connection, ObjectNode objectNode, Context context) throws Exception {
        System.err.println("start invoke.......");
        String date = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
        System.err.println("===>date:" + date + " " + objectNode);
        String value = objectNode.get("value").toString();
        String sql = "insert into `t_test` (`value`,`insert_time`) values (?,?)";
        PreparedStatement ps = connection.prepareStatement(sql);
        ps.setString(1, value);
        ps.setTimestamp(2, new Timestamp(System.currentTimeMillis()));
        //执行insert语句
        ps.execute();
        //手动制造异常
        if(Integer.parseInt(value) == 15) System.out.println(1/0);
    }

    /**
     * 获取连接，开启手动提交事物（getConnection方法中）
     */
    @Override
    protected Connection beginTransaction() throws Exception {
        String url = "jdbc:mysql://localhost:3306/bigdata?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&useSSL=false&autoReconnect=true";
        Connection connection = DBConnectUtil.getConnection(url, "root", "root");
        System.err.println("start beginTransaction......."+connection);
        return connection;
    }

    /**
     * 预提交，这里预提交的逻辑在invoke方法中
     */
    @Override
    protected void preCommit(Connection connection) throws Exception {
        System.err.println("start preCommit......."+connection);

    }

    /**
     * 如果invoke执行正常则提交事物
     */
    @Override
    protected void commit(Connection connection) {
        System.err.println("start commit......."+connection);
        DBConnectUtil.commit(connection);

    }

    @Override
    protected void recoverAndCommit(Connection connection) {
        System.err.println("start recoverAndCommit......."+connection);

    }

    @Override
    protected void recoverAndAbort(Connection connection) {
        System.err.println("start abort recoverAndAbort......."+connection);
    }

    /**
     * 如果invoke执行异常则回滚事物，下一次的checkpoint操作也不会执行
     */
    @Override
    protected void abort(Connection connection) {
        System.err.println("start abort rollback......."+connection);
        DBConnectUtil.rollback(connection);
    }
}




class DBConnectUtil {
    /**
     * 获取连接
     * @throws SQLException
     */
    public static Connection getConnection(String url, String user, String password) throws SQLException {
        Connection conn = null;
        conn = DriverManager.getConnection(url, user, password);
        //设置手动提交
        conn.setAutoCommit(false);
        return conn;
    }

    /**
     * 提交事物
     */
    public static void commit(Connection conn) {
        if (conn != null) {
            try {
                conn.commit();
            } catch (SQLException e) {
                e.printStackTrace();
            } finally {
                close(conn);
            }
        }
    }

    /**
     * 事物回滚
     */
    public static void rollback(Connection conn) {
        if (conn != null) {
            try {
                conn.rollback();
            } catch (SQLException e) {
                e.printStackTrace();
            } finally {
                close(conn);
            }
        }
    }

    /**
     * 关闭连接
     */
    public static void close(Connection conn) {
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

> 代码2

```
import com.alibaba.fastjson.JSON;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

public class DataProducer {
    public static void main(String[] args) throws InterruptedException {
        Properties props = new Properties();
        props.put("bootstrap.servers", "node1:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        Producer<String, String> producer = new org.apache.kafka.clients.producer.KafkaProducer<>(props);

        try {
            for (int i = 1; i <= 20; i++) {
                DataBean data = new DataBean(String.valueOf(i));
                ProducerRecord record = new ProducerRecord<String, String>("flink_kafka", null, null, JSON.toJSONString(data));
                producer.send(record);
                System.out.println("发送数据: " + JSON.toJSONString(data));
                Thread.sleep(1000);
            }
        }catch (Exception e){
            System.out.println(e);
        }
        producer.flush();
    }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class DataBean {
    private String value;
}
```

### 7.7 扩展：异步IO

#### 7.7.1 介绍

##### 7.7.1.1 异步IO操作的需求

https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/stream/operators/asyncio.html

> 流计算系统中经常需要与外部系统进行交互，我们通常的做法如向数据库发送用户a的查询请求，然后等待结果返回，在这之前，我们的程序无法发送用户b的查询请求。这是一种同步访问方式，如下图所示

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/161.png)

> 左图所示：通常实现方式是向数据库发送用户a的查询请求（例如在`MapFunction`中），然后等待结果返回，在这之前，我们无法发送用户b的查询请求，这是一种同步访问的模式，图中棕色的长条标识等待时间，可以发现网络等待时间极大的阻碍了吞吐和延迟

> 右图所示：为了解决同步访问的问题，`异步模式可以并发的处理多个请求和回复`，可以连续的向数据库发送用户a、b、c、d等的请求，与此同时，哪个请求的回复先返回了就处理哪个回复，从而连续的请求之间不需要阻塞等待，这也正是Async I/O的实现原理。

##### 7.7.1.2 使用Async I/O的前提条件 

> 数据库(或key/value存储系统)提供支持异步请求的client。（如java的vertx）

> 没有异步请求客户端的话也可以将同步客户端丢到线程池中执行作为异步客户端

##### 7.7.1.3 Async I/O API

> Async I/O API允许用户在数据流中使用异步客户端访问外部存储，该API处理与数据流的集成，以及消息顺序性（Order），事件时间（EventTime），一致性（容错）等脏活累活，用户只专注于业务

> 如果目标数据库中有异步客户端，则三步即可实现异步流式转换操作（针对该数据库的异步）：
> 
>> 实现用来分发请求的AsyncFunction，用来向数据库发送异步请求并设置回调
> 
>> 获取操作结果的callback，并将它提交给ResultFuture
> 
>> 将异步I/O操作应用于DataStream

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/162.png)

#### 7.7.2 案例演示

https://blog.csdn.net/weixin_41608066/article/details/105957940

> 需求：
>> 使用异步IO实现从MySQL中读取数据

> 准备数据

```
DROP TABLE IF EXISTS `t_category`;
CREATE TABLE `t_category` (
  `id` int(11) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of t_category
-- ----------------------------
INSERT INTO `t_category` VALUES ('1', '手机');
INSERT INTO `t_category` VALUES ('2', '电脑');
INSERT INTO `t_category` VALUES ('3', '服装');
INSERT INTO `t_category` VALUES ('4', '化妆品');
INSERT INTO `t_category` VALUES ('5', '食品');
```

> 代码演示

```
import io.vertx.core.AsyncResult;
import io.vertx.core.Handler;
import io.vertx.core.Vertx;
import io.vertx.core.VertxOptions;
import io.vertx.core.json.JsonObject;
import io.vertx.ext.jdbc.JDBCClient;
import io.vertx.ext.sql.SQLClient;
import io.vertx.ext.sql.SQLConnection;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.AsyncDataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.async.ResultFuture;
import org.apache.flink.streaming.api.functions.async.RichAsyncFunction;
import org.apache.flink.streaming.api.functions.source.RichSourceFunction;

import java.sql.*;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * 使用异步io的先决条件
 * 1.数据库(或key/value存储)提供支持异步请求的client。
 * 2.没有异步请求客户端的话也可以将同步客户端丢到线程池中执行作为异步客户端。
 */
public class ASyncIODemo {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.Source
        //DataStreamSource[1,2,3,4,5]
        DataStreamSource<CategoryInfo> categoryDS = env.addSource(new RichSourceFunction<CategoryInfo>() {
            private Boolean flag = true;
            @Override
            public void run(SourceContext<CategoryInfo> ctx) throws Exception {
                Integer[] ids = {1, 2, 3, 4, 5};
                for (Integer id : ids) {
                    ctx.collect(new CategoryInfo(id, null));
                }
            }
            @Override
            public void cancel() {
                this.flag = false;
            }
        });
        //3.Transformation


        //方式一：Java-vertx中提供的异步client实现异步IO
        //unorderedWait无序等待
        SingleOutputStreamOperator<CategoryInfo> result1 = AsyncDataStream
                .unorderedWait(categoryDS, new ASyncIOFunction1(), 1000, TimeUnit.SECONDS, 10);

        //方式二：MySQL中同步client+线程池模拟异步IO
        //unorderedWait无序等待
        SingleOutputStreamOperator<CategoryInfo> result2 = AsyncDataStream
                .unorderedWait(categoryDS, new ASyncIOFunction2(), 1000, TimeUnit.SECONDS, 10);

        //4.Sink
        result1.print("方式一：Java-vertx中提供的异步client实现异步IO \n");
        result2.print("方式二：MySQL中同步client+线程池模拟异步IO \n");

        //5.execute
        env.execute();
    }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class CategoryInfo {
    private Integer id;
    private String name;
}

class MysqlSyncClient {
    private static transient Connection connection;
    private static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
    private static final String URL = "jdbc:mysql://localhost:3306/bigdata";
    private static final String USER = "root";
    private static final String PASSWORD = "root";

    static {
        init();
    }

    private static void init() {
        try {
            Class.forName(JDBC_DRIVER);
        } catch (ClassNotFoundException e) {
            System.out.println("Driver not found!" + e.getMessage());
        }
        try {
            connection = DriverManager.getConnection(URL, USER, PASSWORD);
        } catch (SQLException e) {
            System.out.println("init connection failed!" + e.getMessage());
        }
    }

    public void close() {
        try {
            if (connection != null) {
                connection.close();
            }
        } catch (SQLException e) {
            System.out.println("close connection failed!" + e.getMessage());
        }
    }

    public CategoryInfo query(CategoryInfo category) {
        try {
            String sql = "select id,name from t_category where id = "+ category.getId();
            Statement statement = connection.createStatement();
            ResultSet rs = statement.executeQuery(sql);
            if (rs != null && rs.next()) {
                category.setName(rs.getString("name"));
            }
        } catch (SQLException e) {
            System.out.println("query failed!" + e.getMessage());
        }
        return category;
    }
}

/**
 * 方式一：Java-vertx中提供的异步client实现异步IO
 */
class ASyncIOFunction1 extends RichAsyncFunction<CategoryInfo, CategoryInfo> {
    private transient SQLClient mySQLClient;

    @Override
    public void open(Configuration parameters) throws Exception {
        JsonObject mySQLClientConfig = new JsonObject();
        mySQLClientConfig
                .put("driver_class", "com.mysql.jdbc.Driver")
                .put("url", "jdbc:mysql://localhost:3306/bigdata")
                .put("user", "root")
                .put("password", "root")
                .put("max_pool_size", 20);

        VertxOptions options = new VertxOptions();
        options.setEventLoopPoolSize(10);
        options.setWorkerPoolSize(20);
        Vertx vertx = Vertx.vertx(options);
        //根据上面的配置参数获取异步请求客户端
        mySQLClient = JDBCClient.createNonShared(vertx, mySQLClientConfig);
    }

    //使用异步客户端发送异步请求
    @Override
    public void asyncInvoke(CategoryInfo input, ResultFuture<CategoryInfo> resultFuture) throws Exception {
        mySQLClient.getConnection(new Handler<AsyncResult<SQLConnection>>() {
            @Override
            public void handle(AsyncResult<SQLConnection> sqlConnectionAsyncResult) {
                if (sqlConnectionAsyncResult.failed()) {
                    return;
                }
                SQLConnection connection = sqlConnectionAsyncResult.result();
                connection.query("select id,name from t_category where id = " +input.getId(), new Handler<AsyncResult<io.vertx.ext.sql.ResultSet>>() {
                    @Override
                    public void handle(AsyncResult<io.vertx.ext.sql.ResultSet> resultSetAsyncResult) {
                        if (resultSetAsyncResult.succeeded()) {
                            List<JsonObject> rows = resultSetAsyncResult.result().getRows();
                            for (JsonObject jsonObject : rows) {
                                CategoryInfo categoryInfo = new CategoryInfo(jsonObject.getInteger("id"), jsonObject.getString("name"));
                                resultFuture.complete(Collections.singletonList(categoryInfo));
                            }
                        }
                    }
                });
            }
        });
    }
    @Override
    public void close() throws Exception {
        mySQLClient.close();
    }

    @Override
    public void timeout(CategoryInfo input, ResultFuture<CategoryInfo> resultFuture) throws Exception {
        System.out.println("async call time out!");
        input.setName("未知");
        resultFuture.complete(Collections.singleton(input));
    }
}

/**
 * 方式二：同步调用+线程池模拟异步IO
 */
class ASyncIOFunction2 extends RichAsyncFunction<CategoryInfo, CategoryInfo> {
    private transient MysqlSyncClient client;
    private ExecutorService executorService;//线程池

    @Override
    public void open(Configuration parameters) throws Exception {
        super.open(parameters);
        client = new MysqlSyncClient();
        executorService = new ThreadPoolExecutor(10, 10, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
    }

    //异步发送请求
    @Override
    public void asyncInvoke(CategoryInfo input, ResultFuture<CategoryInfo> resultFuture) throws Exception {
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                resultFuture.complete(Collections.singletonList((CategoryInfo) client.query(input)));
            }
        });
    }


    @Override
    public void close() throws Exception {
    }

    @Override
    public void timeout(CategoryInfo input, ResultFuture<CategoryInfo> resultFuture) throws Exception {
        System.out.println("async call time out!");
        input.setName("未知");
        resultFuture.complete(Collections.singleton(input));
    }
}
```

#### 7.7.3 扩展阅读：异步IO原理

##### 7.7.3.1 AsyncDataStream

> AsyncDataStream是一个工具类，用于将AsyncFunction应用于DataStream，AsyncFunction发出的并发请求都是无序的，该顺序基于哪个请求先完成，为了控制结果记录的发出顺序，flink提供了两种模式，分别对应AsyncDataStream的两个静态方法，`OrderedWait`和`unorderedWait`
> 
>> orderedWait（有序）：消息的发送顺序与接收到的顺序相同（包括 watermark ），也就是先进先出。
> 
>> unorderWait（无序）：
>> 
>>> 在ProcessingTime中，完全无序，即哪个请求先返回结果就先发送（最低延迟和最低消耗）。
>>  
>>> 在EventTime中，以watermark为边界，介于两个watermark之间的消息可以乱序，但是watermark和消息之间不能乱序，这样既认为在无序中又引入了有序，这样就有了与有序一样的开销。

> AsyncDataStream.(un)orderedWait 的主要工作就是创建了一个 AsyncWaitOperator。AsyncWaitOperator 是支持异步 IO 访问的算子实现，该算子会运行 AsyncFunction 并处理异步返回的结果，其内部原理如下图所示。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/163.png)

> 如图所示，AsyncWaitOperator 主要由两部分组成： 
>> StreamElementQueue
> 
>> Emitter

> `StreamElementQueue` 是一个 `Promise` 队列，所谓 Promise 是一种异步抽象表示将来会有一个值（海底捞排队给你的小票），这个队列是未完成的 Promise 队列，也就是进行中的请求队列。Emitter 是一个单独的线程，负责发送消息（收到的异步回复）给下游。

> 图中E5表示进入该算子的第五个元素（”Element-5”）
> 
>> 在执行过程中首先会将其包装成一个 “Promise” P5，然后将P5放入队列
> 
>> 最后调用 AsyncFunction 的 ayncInvoke 方法，该方法会向外部服务发起一个异步的请求，并注册回调
> 
>> 该回调会在异步请求成功返回时调用 AsyncCollector.collect 方法将返回的结果交给框架处理。
> 
>> 实际上 AsyncCollector 是一个 Promise ，也就是 P5，在调用 collect 的时候会标记 Promise 为完成状态，并通知 Emitter 线程有完成的消息可以发送了。
> 
>> Emitter 就会从队列中拉取完成的 Promise ，并从 Promise 中取出消息发送给下游。

##### 7.7.3.2 消息的顺序性

> 上文提到 Async I/O 提供了两种输出模式。其实细分有三种模式:
> 
>> 有序
> 
>> ProcessingTime 无序
> 
>> EventTime 无序
> 
>> Flink 使用队列来实现不同的输出模式，并抽象出一个队列的接口（StreamElementQueue），这种分层设计使得AsyncWaitOperator和Emitter不用关心消息的顺序问题。StreamElementQueue有两种具体实现，分别是 `OrderedStreamElementQueue` 和`UnorderedStreamElementQueue`。UnorderedStreamElementQueue 比较有意思，它使用了一套逻辑巧妙地实现完全无序和 EventTime 无序。

> 有序
>> 有序比较简单，使用一个队列就能实现。所有新进入该算子的元素（包括 watermark），都会包装成 Promise 并按到达顺序放入该队列。如下图所示，尽管P4的结果先返回，但并不会发送，只有 P1 （队首）的结果返回了才会触发 Emitter 拉取队首元素进行发送。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/164.png)

> ProcessingTime 无序
>> ProcessingTime 无序也比较简单，因为没有 watermark，不需要协调 watermark 与消息的顺序性，所以使用两个队列就能实现，一个 `uncompletedQueue` 一个 `completedQueue`。所有新进入该算子的元素，同样的包装成 `Promise` 并放入 `uncompletedQueue` 队列，当`uncompletedQueue`队列中任意的Promise返回了数据，则将该 `Promise` 移到 `completedQueue` 队列中，并通知 `Emitter` 消费。如下图所示：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/165.png)

> EventTime 无序
>> `EventTime 无序类似于有序与 ProcessingTime 无序的结合体`。因为有 `watermark`，需要协调 watermark与消息之间的顺序性，所以`uncompletedQueue`中存放的元素从原先的` Promise` 变成了 `Promise 集合`。
>>> 如果进入算子的是消息元素，则会包装成 Promise 放入队尾的集合中
>>> 如果进入算子的是 watermark，也会包装成 Promise 并放到一个独立的集合中，再将该集合加入到 uncompletedQueue 队尾，最后再创建一个空集合加到 uncompletedQueue 队尾
>>> 这样，watermark 就成了消息顺序的边界。
>>> 只有处在队首的集合中的 Promise 返回了数据，才能将该 Promise 移到completedQueue
>>> 队列中，由 Emitter 消费发往下游。
>>> 只有队首集合空了，才能处理第二个集合。

> 这样就保证了当且仅当某个 watermark 之前所有的消息都已经被发送了，该 watermark 才能被发送。过程如下图所示：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/166.png)

### 7.8 扩展：Streaming File Sink

#### 7.8.1 介绍

https://blog.csdn.net/u013220482/article/details/100901471

##### 7.8.1.1 场景描述

> StreamingFileSink是Flink1.7中推出的新特性，是为了解决如下的问题：

> 大数据业务场景中，经常有一种场景：外部数据发送到kafka中，flink作为中间件消费kafka数据并进行业务处理；处理完成之后的数据可能还需要写入到数据库或者文件系统中，比如写入hdfs中。

> StreamingFileSink就可以用来将分区文件写入到支持 Flink FileSystem 接口的文件系统中，支持Exactly-Once语义。

> 这种sink实现的Exactly-Once都是基于Flink checkpoint来实现的两阶段提交模式来保证的，主要应用在实时数仓、topic拆分、基于小时分析处理等场景下。

##### 7.8.1.2 Bucket和SubTast、PartFile

> `Bucket`
>> StreamingFileSink可向由Flink FileSystem抽象支持的文件系统写入分区文件（因为是流式写入，数据被视为无界）。该分区行为可配，默认按时间，具体来说每小时写入一个Bucket，该Bucket包括若干文件，内容是这一小时间隔内流中收到的所有record。

> `PartFile`
>> 每个Bukcket内部分为多个PartFile来存储输出数据，该Bucket生命周期内接收到数据的sink的每个子任务至少有一个PartFile。
>> 而额外文件滚动由可配的滚动策略决定，默认策略是根据文件大小和打开超时（文件可以被打开的最大持续时间）以及文件最大不活动超时等决定是否滚动。
>> Bucket和SubTask、PartFile关系如图所示

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/167.png)

#### 7.8.2 案例演示

> 需求
>> 编写Flink程序，接收socket的字符串数据，然后将接收到的数据流式方式存储到hdfs

> 开发步骤
>> 1．初始化流计算运行环境
>> 2．设置Checkpoint（10s）周期性启动
>> 3．指定并行度为1
>> 4．接入socket数据源，获取数据
>> 5．指定文件编码格式为行编码格式
>> 6．设置桶分配策略
>> 7．设置文件滚动策略
>> 8．指定文件输出配置
>> 9．将streamingfilesink对象添加到环境
>> 10．执行任务

> 实现代码

```
import org.apache.flink.api.common.serialization.SimpleStringEncoder;
import org.apache.flink.core.fs.Path;
import org.apache.flink.runtime.state.filesystem.FsStateBackend;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.filesystem.OutputFileConfig;
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink;
import org.apache.flink.streaming.api.functions.sink.filesystem.bucketassigners.DateTimeBucketAssigner;
import org.apache.flink.streaming.api.functions.sink.filesystem.rollingpolicies.DefaultRollingPolicy;

import java.util.concurrent.TimeUnit;

public class StreamFileSinkDemo {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //设置ckp
        env.enableCheckpointing(TimeUnit.SECONDS.toMillis(10));
        env.setStateBackend(new FsStateBackend("file:///D:\\data\\ckp"));

        //2.source
        DataStreamSource<String> lines = env.socketTextStream("node1", 9999);

        //3.sink
        //设置sink的前缀和后缀
        //文件的头和文件扩展名
        //prefix-xxx-.txt
        OutputFileConfig config = OutputFileConfig
                .builder()
                .withPartPrefix("prefix")
                .withPartSuffix(".txt")
                .build();

        //设置sink的路径
        String outputPath = "hdfs://node1:8020/FlinkStreamFileSink/parquet";

        //创建StreamingFileSink
        final StreamingFileSink<String> sink = StreamingFileSink
                .forRowFormat(
                        new Path(outputPath),
                        new SimpleStringEncoder<String>("UTF-8"))
                /**
                 * 设置桶分配政策
                 * DateTimeBucketAssigner --默认的桶分配政策，默认基于时间的分配器，每小时产生一个桶，格式如下yyyy-MM-dd--HH
                 * BasePathBucketAssigner ：将所有部分文件（part file）存储在基本路径中的分配器（单个全局桶）
                 */
                .withBucketAssigner(new DateTimeBucketAssigner<>())
                /**
                 * 有三种滚动政策
                 *  CheckpointRollingPolicy
                 *  DefaultRollingPolicy
                 *  OnCheckpointRollingPolicy
                 */
                .withRollingPolicy(
                        /**
                         * 滚动策略决定了写出文件的状态变化过程
                         * 1. In-progress ：当前文件正在写入中
                         * 2. Pending ：当处于 In-progress 状态的文件关闭（closed）了，就变为 Pending 状态
                         * 3. Finished ：在成功的 Checkpoint 后，Pending 状态将变为 Finished 状态
                         *
                         * 观察到的现象
                         * 1.会根据本地时间和时区，先创建桶目录
                         * 2.文件名称规则：part-<subtaskIndex>-<partFileIndex>
                         * 3.在macos中默认不显示隐藏文件，需要显示隐藏文件才能看到处于In-progress和Pending状态的文件，因为文件是按照.开头命名的
                         *
                         */
                        DefaultRollingPolicy.builder()
                                .withRolloverInterval(TimeUnit.SECONDS.toMillis(2)) //设置滚动间隔
                                .withInactivityInterval(TimeUnit.SECONDS.toMillis(1)) //设置不活动时间间隔
                                .withMaxPartSize(1024 * 1024 * 1024) // 最大尺寸
                                .build())
                .withOutputFileConfig(config)
                .build();

        lines.addSink(sink).setParallelism(1);

        env.execute();
    }
}
```

#### 7.8.3 扩展阅读：其他配置

##### 7.8.3.1 PartFile

> 前面提到过，每个Bukcket内部分为多个部分文件，该Bucket内接收到数据的sink的每个子任务至少有一个PartFile。而额外文件滚动由可配的滚动策略决定。

> 关于顺序性
>> 对于任何给定的Flink子任务，PartFile索引都严格增加（按创建顺序），但是，这些索引并不总是顺序的。当`作业重新启动时，所有子任务的下一个PartFile索引将是max PartFile索引+ 1，其中max是指在所有子任务中对所有计算的索引最大值`。

```
return new Path(bucketPath, outputFileConfig.getPartPrefix() + '-' + subtaskIndex + '-' + partCounter + outputFileConfig.getPartSuffix());
```

###### 7.8.3.1.1 PartFile生命周期

> 输出文件的命名规则和生命周期。由上图可知，部分文件（part file）可以处于以下三种状态之一：

> `In-progress` ：
>> 当前文件正在写入中

> `Pending` ：
>> 当处于 In-progress 状态的文件关闭（closed）了，就变为 `Pending` 状态

> `Finished` ：
>> 在成功的 `Checkpoint` 后，`Pending` 状态将变为 `Finished` 状态,处于 Finished 状态的文件不会再被修改，可以被下游系统安全地读取。

> `注意`：
>> 使用 StreamingFileSink 时需要`启用 Checkpoint` ，每次做 Checkpoint 时写入完成。如果 Checkpoint 被禁用，部分文件（part file）将永远处于 '`in-progress`' 或 '`pending`' 状态，下游系统无法安全地读取。

###### 7.8.3.1.2 PartFile的生成规则

> 在每个活跃的Bucket期间，每个Writer的子任务在任何时候都只会有一个单独的`In-progress PartFile`，但可有多个Peding和Finished状态文件。

> 一个Sink的两个Subtask的PartFile分布情况实例如下:

> 初始状态，两个inprogress文件正在被两个subtask分别写入

```
└── 2020-03-25--12
├── part-0-0.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
└── part-1-0.inprogress.ea65a428-a1d0-4a0b-bbc5-7a436a75e575
```

> 当part-1-0因文件大小超过阈值等原因发生滚动时，变为`Pending`状态等待完成，但此时不会被重命名。注意此时Sink会创建一个新的PartFile即part-1-1：

```
└── 2020-03-25--12
├── part-0-0.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
├── part-1-0.inprogress.ea65a428-a1d0-4a0b-bbc5-7a436a75e575
└── part-1-1.inprogress.bc279efe-b16f-47d8-b828-00ef6e2fbd11
```

> 待下次checkpoint成功后，part-1-0完成变为Finished状态，被重命名：

```
└── 2020-03-25--12
├── part-0-0.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
├── part-1-0
└── part-1-1.inprogress.bc279efe-b16f-47d8-b828-00ef6e2fbd11
```

> 下一个Bucket周期到了，创建新的Bucket目录，不影响之前Bucket内的的in-progress文件，依然要等待文件RollingPolicy以及checkpoint来改变状态：

```
└── 2020-03-25--12
├── part-0-0.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
├── part-1-0
└── part-1-1.inprogress.bc279efe-b16f-47d8-b828-00ef6e2fbd11
└── 2020-03-25--13
└── part-0-2.inprogress.2b475fec-1482-4dea-9946-eb4353b475f1
```
###### 7.8.3.1.3 PartFile命名设置

> 默认，PartFile命名规则如下：

> In-progress / Pending
>> part--.inprogress.uid

> Finished
>> part--

> 比如part-1-20表示`1号子任务已完成的20号文件`。
> 可以使用OutputFileConfig来改变前缀和后缀，代码示例如下：

```
OutputFileConfig config = OutputFileConfig
.builder()
.withPartPrefix("prefix")
.withPartSuffix(".ext")
.build()

StreamingFileSink sink = StreamingFileSink
.forRowFormat(new Path(outputPath), new SimpleStringEncoder<String>("UTF-8"))
.withBucketAssigner(new KeyBucketAssigner())
.withRollingPolicy(OnCheckpointRollingPolicy.build())
.withOutputFileConfig(config)
.build()
```

> 得到的PartFile示例如下

```
└── 2019-08-25--12
├── prefix-0-0.ext
├── prefix-0-1.ext.inprogress.bd053eb0-5ecf-4c85-8433-9eff486ac334
├── prefix-1-0.ext
└── prefix-1-1.ext.inprogress.bc279efe-b16f-47d8-b828-00ef6e2fbd11
```

##### 7.8.3.2 PartFile序列化编码

> StreamingFileSink 支持`行编码格式和批量编码格式`，比如 Apache Parquet 。这两种变体可以使用以下静态方法创建：

> Row-encoded sink:
>> StreamingFileSink.forRowFormat(basePath, rowEncoder)

```
//行
StreamingFileSink.forRowFormat(new Path(path), new SimpleStringEncoder<T>())
.withBucketAssigner(new PaulAssigner<>()) //分桶策略
.withRollingPolicy(new PaulRollingPolicy<>()) //滚动策略
.withBucketCheckInterval(CHECK_INTERVAL) //检查周期
.build();
```

> Bulk-encoded sink:
>> StreamingFileSink.forBulkFormat(basePath, bulkWriterFactory)

```
//列 parquet
StreamingFileSink.forBulkFormat(new Path(path), ParquetAvroWriters.forReflectRecord(clazz))
.withBucketAssigner(new PaulBucketAssigner<>())
.withBucketCheckInterval(CHECK_INTERVAL)
.build();
```
> 创建行或批量编码的 Sink 时，我们需要指定`存储桶的基本路径和数据的编码`

> 这两种写入格式除了文件格式的不同，另外一个很重要的区别就是`回滚策略`的不同：
>> forRowFormat行写可基于文件大小、滚动时间、不活跃时间进行滚动，
> 
>> forBulkFormat列写方式只能基于checkpoint机制进行文件滚动，即在执行snapshotState方法时滚动文件，如果基于大小或者时间滚动文件，那么在任务失败恢复时就必须对处于in-processing状态的文件按照指定的offset进行truncate，由于列式存储是无法针对文件offset进行truncate的，因此就必须在每次checkpoint使文件滚动，其使用的滚动策略实现是OnCheckpointRollingPolicy。

> `重要`:`forBulkFormat`只能和 `OnCheckpointRollingPolicy` 结合使用，每次做 checkpoint 时滚动文件。

###### 7.8.3.2.1 Row Encoding

> 此时，StreamingFileSink会以每条记录为单位进行编码和序列化。

> 必须配置项：

> `输出数据的BasePath`<br/>
> 序列化每行数据写入PartFile的Encoder
>> 使用RowFormatBuilder可选配置项：

> `自定义RollingPolicy`
>>  默认使用DefaultRollingPolicy来滚动文件，可自定义

> `bucketCheckInterval`
>> 默认1分钟。该值单位为毫秒，指定按时间滚动文件间隔时间

> 例子如下：

```
import org.apache.flink.api.common.serialization.SimpleStringEncoder
import org.apache.flink.core.fs.Path
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink

// 1. 构建DataStream
DataStream input  = ...
// 2. 构建StreamingFileSink，指定BasePath、Encoder、RollingPolicy
StreamingFileSink sink  = StreamingFileSink
.forRowFormat(new Path(outputPath), new SimpleStringEncoder[String]("UTF-8"))
.withRollingPolicy(
DefaultRollingPolicy.builder()
.withRolloverInterval(TimeUnit.MINUTES.toMillis(15))
.withInactivityInterval(TimeUnit.MINUTES.toMillis(5))
.withMaxPartSize(1024 * 1024 * 1024)
.build())
.build()
// 3. 添加Sink到InputDataSteam即可
input.addSink(sink)
```

> 以上例子构建了一个简单的拥有默认Bucket构建行为（继承自BucketAssigner的DateTimeBucketAssigner）的StreamingFileSink，每小时构建一个Bucket，内部使用继承自RollingPolicy的DefaultRollingPolicy，以下三种情况任一发生会滚动PartFile：
>> PartFile包含至少15分钟的数据
> 
>> 在过去5分钟内没有接收到新数据
> 
>> 在最后一条记录写入后，文件大小已经达到1GB

> 除了使用DefaultRollingPolicy，也可以自己实现RollingPolicy接口来实现自定义滚动策略。

###### 7.8.3.2.2 Bulk Encoding

> 要使用批量编码，请将StreamingFileSink.forRowFormat()替换为StreamingFileSink.forBulkFormat()，注意此时必须指定一个BulkWriter.Factory而不是行模式的Encoder。BulkWriter在逻辑上定义了如何添加、fllush新记录以及如何最终确定记录的bulk以用于进一步编码。

> 需要注意的是，使用Bulk Encoding时，Filnk1.9版本的文件滚动就只能使用OnCheckpointRollingPolicy的策略，该策略在每次checkpoint时滚动part-file。

> Flink有三个内嵌的BulkWriter：
> `ParquetAvroWriters`
>> 有一些静态方法来创建ParquetWriterFactory。
> 
> `SequenceFileWriterFactory`
> 
> `CompressWriterFactory`

> Flink有内置方法可用于为Avro数据创建Parquet writer factory。

> 要使用ParquetBulkEncoder，需要添加以下Maven依赖：

```
<!-- streaming File Sink所需要的jar包-->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-parquet_2.11</artifactId>
    <version>1.10.0</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.avro/avro -->
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro</artifactId>
    <version>1.10.0</version>
</dependency>

<dependency>
    <groupId>org.apache.parquet</groupId>
    <artifactId>parquet-avro</artifactId>
    <version>1.10.0</version>
</dependency>
```

##### 7.8.3.3 桶分配策略

> 桶分配策略定义了`将数据结构化后写入基本输出目录中的子目录`，`行格式和批量格式`都需要使用。

> 具体来说，StreamingFileSink使用`BucketAssigner`来确定每条输入的`数据应该被放入哪个Bucket`，

> 默认情况下，DateTimeBucketAssigner 基于系统默认时区每小时创建一个桶：

> 格式如下：`yyyy-MM-dd--HH`。日期格式（即桶的大小）和时区都可以手动配置。

> 我们可以在格式构建器上调用 .withBucketAssigner(assigner) 来自定义 BucketAssigner。

> Flink 有两个内置的 BucketAssigners ：
>> DateTimeBucketAssigner：默认基于时间的分配器
> 
>> BasePathBucketAssigner：将所有部分文件（part file）存储在基本路径中的分配器（单个全局桶）

###### 7.8.3.3.1 DateTimeBucketAssigner

> Row格式和Bulk格式编码都使用DateTimeBucketAssigner作为默认BucketAssigner。 默认情况下，DateTimeBucketAssigner 基于系统默认时区每小时以格式yyyy-MM-dd--HH来创建一个Bucket，Bucket路径为/{basePath}/{dateTimePath}/。
>> basePath是指StreamingFileSink.forRowFormat(new Path(outputPath)时的路径
> 
>> dateTimePath中的日期格式和时区都可在初始化DateTimeBucketAssigner时配置

```
public class DateTimeBucketAssigner<IN> implements BucketAssigner<IN, String> {
private static final long serialVersionUID = 1L;

	// 默认的时间格式字符串
	private static final String DEFAULT_FORMAT_STRING = "yyyy-MM-dd--HH";

	// 时间格式字符串
	private final String formatString;

	// 时区
	private final ZoneId zoneId;
	
	// DateTimeFormatter被用来通过当前系统时间和DateTimeFormat来生成时间字符串
	private transient DateTimeFormatter dateTimeFormatter;

	/**
	 * 使用默认的`yyyy-MM-dd--HH`和系统时区构建DateTimeBucketAssigner
	 */
	public DateTimeBucketAssigner() {
		this(DEFAULT_FORMAT_STRING);
	}

	/**
	 * 通过能被SimpleDateFormat解析的时间字符串和系统时区
	 * 来构建DateTimeBucketAssigner
	 */
	public DateTimeBucketAssigner(String formatString) {
		this(formatString, ZoneId.systemDefault());
	}

	/**
	 * 通过默认的`yyyy-MM-dd--HH`和指定的时区
	 * 来构建DateTimeBucketAssigner
	 */
	public DateTimeBucketAssigner(ZoneId zoneId) {
		this(DEFAULT_FORMAT_STRING, zoneId);
	}

	/**
	 * 通过能被SimpleDateFormat解析的时间字符串和指定的时区
	 * 来构建DateTimeBucketAssigner
	 */
	public DateTimeBucketAssigner(String formatString, ZoneId zoneId) {
		this.formatString = Preconditions.checkNotNull(formatString);
		this.zoneId = Preconditions.checkNotNull(zoneId);
	}

	/**
	 * 使用指定的时间格式和时区来格式化当前ProcessingTime，以获取BucketId
	 */
	@Override
	public String getBucketId(IN element, BucketAssigner.Context context) {
		if (dateTimeFormatter == null) {
			dateTimeFormatter = DateTimeFormatter.ofPattern(formatString).withZone(zoneId);
		}
		return dateTimeFormatter.format(Instant.ofEpochMilli(context.currentProcessingTime()));
	}

	@Override
	public SimpleVersionedSerializer<String> getSerializer() {
		return SimpleVersionedStringSerializer.INSTANCE;
	}

	@Override
	public String toString() {
		return "DateTimeBucketAssigner{" +
			"formatString='" + formatString + '\'' +
			", zoneId=" + zoneId +
			'}';
	}
}
```

###### 7.8.3.3.2 BasePathBucketAssigner

> 将所有PartFile存储在BasePath中（此时只有单个全局Bucket）。

> 先看看BasePathBucketAssigner的源码，方便继续学习DateTimeBucketAssigner：

```
@PublicEvolving
public class BasePathBucketAssigner<T> implements BucketAssigner<T, String> {

    private static final long serialVersionUID = -6033643155550226022L;
    /**
    * BucketId永远为""，即Bucket全路径为用户指定的BasePath
    */
    
    @Override
    public String getBucketId(T element, BucketAssigner.Context context) {
        return "";
    }
    /**
    * 用SimpleVersionedStringSerializer来序列化BucketId
    */
    
    @Override
    public SimpleVersionedSerializer<String> getSerializer() {
        // in the future this could be optimized as it is the empty string.
        return SimpleVersionedStringSerializer.INSTANCE;
    }

	@Override
	public String toString() {
		return "BasePathBucketAssigner";
	}
}
```
##### 7.8.3.4 滚动策略

> 滚动策略 `RollingPolicy` 定义了`指定的文件在何时关闭（closed）并将其变为 Pending 状态，随后变为 Finished 状态`。处于 Pending 状态的文件会在下一次 Checkpoint 时变为 Finished 状态，通过设置 Checkpoint 间隔时间，可以控制部分文件（part file）对下游读取者可用的速度、大小和数量。

> Flink 有两个内置的滚动策略：
>> `DefaultRollingPolicy`
> 
>> `OnCheckpointRollingPolicy`

> 需要注意的是，使用`Bulk Encoding`时，文件滚动就只能使用`OnCheckpointRollingPolicy`的策略，该策略在每次checkpoint时滚动part-file。

### 7.9 扩展：关于并行度

> 一个Flink程序由多个Operator组成(source、transformation和 sink)。

> 一个Operator由多个并行的Task(线程)来执行， 一个Operator的并行Task(线程)数目就被称为该Operator(任务)的并行度(Parallel)

并行度可以有如下几种指定方式
> `1.Operator Level`（算子级别）(可以使用)
>> 一个算子、数据源和sink的并行度可以通过调用 setParallelism()方法来指定
>>  ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/168.png)
> 
> `2.Execution Environment Level`（Env级别）(可以使用)
>> 执行环境(任务)的默认并行度可以通过调用setParallelism()方法指定。为了以并行度3来执行所有的算子、数据源和data sink， 可以通过如下的方式设置执行环境的并行度：
>> 执行环境的并行度可以通过显式设置算子的并行度而被重写
>>  ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/169.png)
> 
> `3.Client Level`(客户端级别,推荐使用)(可以使用)
>> 并行度可以在客户端将job提交到Flink时设定。
>> 对于CLI客户端，可以通过-p参数指定并行度
```
./bin/flink run -p 10 WordCount-java.jar
```
> 
> `4.System Level`（系统默认级别,尽量不使用）
>> 在系统级可以通过设置flink-conf.yaml文件中的parallelism.default属性来指定所有执行环境的默认并行度

> 示例

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/170.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/171.png)

> `Example1`
>> 在fink-conf.yaml中 taskmanager.numberOfTaskSlots 默认值为1，即每个Task Manager上只有一个Slot ，此处是3
>> Example1中，WordCount程序设置了并行度为1，意味着程序 Source、Reduce、Sink在一个Slot中，占用一个Slot
> 
> `Example2`
>> 通过设置并行度为2后，将占用2个Slot
> 
> `Example3`
>> 通过设置并行度为9，将占用9个Slot
> 
> `Example4`
>> 通过设置并行度为9，并且设置sink的并行度为1，则Source、Reduce将占用9个Slot，但是Sink只占用1个Slot


> `注意`
>> 1.并行度的优先级：算子级别 > env级别 > Client级别 > 系统默认级别  (越靠前具体的代码并行度的优先级越高)
> 
>> 2.如果source不可以被并行执行，即使指定了并行度为多个，也不会生效
> 
>> 3.在实际生产中，我们推荐在算子级别显示指定各自的并行度，方便进行显示和精确的资源控制。
> 
>> 4.slot是静态的概念，是指taskmanager具有的并发执行能力; parallelism是动态的概念，是指程序运行时实际使用的并发能力



## 8 Flink Table与SQL

### 8.1 Table API & SQL介绍

#### 8.1.1 为什么使用Table API & SQL

> 当我们在使用flink做流式和批式任务计算的时候，往往会想到几个问题：
>> 1. 需要熟悉两套API : DataStream/DataSet API，API有一定难度，开发人员无法集中精力到具体业务的开发
>> 2. 需要有Java或Scala的开发经验
>> 3. Flink 同时支持批任务与流任务，如何做到API层的统一 

> Flink SQL 是 Flink 实时计算为简化计算模型，降低用户使用实时计算门槛而设计的一套符合标准 SQL 语义的开发语言。

> Flink 已经拥有了强大的DataStream/DataSetAPI，满足流计算和批计算中的各种场景需求，但是关于以上几个问题，我们还需要提供一种关系型的API来实现Flink API层的流与批的统一，那么这就是Flink的Table API & SQL。

#### 8.1.2 Table API & SQL的特点

> Flink之所以选择将 Table API & SQL 作为未来的核心 API，是因为其具有一些非常重要的特点：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/172.png)

> 1. 声明式:属于设定式语言，用户只要表达清楚需求即可，不需要了解底层执行；<br/>
> 2. 高性能:可优化，内置多种查询优化器，这些查询优化器可为 SQL 翻译出最优执行计划；<br/>
> 3. 简单易学:易于理解，不同行业和领域的人都懂，学习成本较低；<br/>
> 4. 标准稳定:语义遵循SQL标准，非常稳定，在数据库 30 多年的历史中，SQL 本身变化较少；<br/>
> 5. 流批统一:可以做到API层面上流与批的统一，相同的SQL逻辑，既可流模式运行，也可批模式运行，Flink底层Runtime本身就是一个流与批统一的引擎

> Table API& SQL 是一种关系型API，用户可以像操作MySQL数据库表一样的操作数据，而不需要写Java代码完成Flink function，更不需要手工的优化Java代码调优。另外，SQL作为一个非程序员可操作的语言，学习成本很低，如果一个系统提供SQL支持，将很容易被用户接受。
   
> 当然除了SQL的特性，因为Table API是在Flink中专门设计的，所以Table API还具有自身的特点：
>> 1. 表达方式的扩展性 - 在Flink中可以为Table API开发很多便捷性功能，如：Row.flatten(), map/flatMap 等
>> 2. 功能的扩展性 - 在Flink中可以为Table API扩展更多的功能，如：Iteration，flatAggregate 等新功能
>> 3. 编译检查 - Table API支持Java和Scala语言开发，支持IDE中进行编译检查。

#### 8.1.3 Table API & SQL发展历程

> 自 2015 年开始，阿里巴巴开始调研开源流计算引擎，最终决定基于 Flink 打造新一代计算引擎，针对 Flink 存在的不足进行优化和改进，并且在 2019 年初将最终代码开源，也就是Blink。Blink 在原来的 Flink 基础上最显著的一个贡献就是 Flink SQL 的实现。随着版本的不断更新，API 也出现了很多不兼容的地方。

> 在 Flink 1.9 中，Table 模块迎来了核心架构的升级，引入了阿里巴巴Blink团队贡献的诸多功能，Flink 的 Table 模块 包括 Table API 和 SQL：

> Table API 是一种类SQL的API，通过Table API，用户可以像操作表一样操作数据，非常直观和方便

> SQL作为一种声明式语言，有着标准的语法和规范，用户可以不用关心底层实现即可进行数据的处理，非常易于上手

> Flink Table API 和 SQL 的实现上有80%左右的代码是公用的。作为一个流批统一的计算引擎，Flink 的 Runtime 层是统一的。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/173.png)

> 在Flink 1.9 之前，Flink API 层 一直分为DataStream API 和 DataSet API，Table API & SQL 位于 DataStream API 和 DataSet API 之上。可以看处流处理和批处理有各自独立的api (流处理DataStream，批处理DataSet)。而且有不同的执行计划解析过程，codegen过程也完全不一样，完全没有流批一体的概念，面向用户不太友好。

> 在Flink1.9之后新的架构中，有两个查询处理器：Flink Query Processor，也称作Old Planner和Blink Query Processor，也称作Blink Planner。为了兼容老版本Table及SQL模块，插件化实现了Planner，Flink原有的Flink Planner不变，后期版本会被移除。新增加了Blink Planner，新的代码及特性会在Blink planner模块上实现。批或者流都是通过解析为Stream Transformation来实现的，不像Flink Planner，批是基于Dataset，流是基于DataStream。后期的架构会进一步实现流批统一

> 查询处理器是 Planner 的具体实现，通过parser、optimizer、codegen(代码生成技术)等流程将 Table API & SQL作业转换成 Flink Runtime 可识别的 Transformation DAG，最终由 Flink Runtime 进行作业的调度和执行。

> Flink Query Processor查询处理器针对流计算和批处理作业有不同的分支处理，流计算作业底层的 API 是 DataStream API， 批处理作业底层的 API 是 DataSet API

> Blink Query Processor查询处理器则实现流批作业接口的统一，底层的 API 都是Transformation，这就意味着我们和Dataset完全没有关系了


> Blink planner和Flink Planner具体区别如下：
>> https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/common.html

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/174.png)

#### 8.1.4 注意

> https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/common.html

> API稳定性

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/175.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/176.png)

> 性能对比
>> 注意：目前FlinkSQL性能不如SparkSQL，未来FlinkSQL可能会越来越好
>> 下图是Hive、Spark、Flink的SQL执行速度对比：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/177.png)

### 8.2 案例准备

#### 8.2.1 依赖

> https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/table/

```xml
<!-- Either... -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-api-java-bridge_2.11</artifactId>
  <version>1.11.2</version>
  <scope>provided</scope>
</dependency>
<!-- or... -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-api-scala-bridge_2.11</artifactId>
  <version>1.11.2</version>
  <scope>provided</scope>
</dependency>

<!-- Either... (for the old planner that was available before Flink 1.9) -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-planner_2.11</artifactId>
  <version>1.11.2</version>
  <scope>provided</scope>
</dependency>
<!-- or.. (for the new Blink planner) -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-planner-blink_2.11</artifactId>
  <version>1.11.2</version>
  <scope>provided</scope>
</dependency>

<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-streaming-scala_2.11</artifactId>
  <version>1.11.2</version>
  <scope>provided</scope>
</dependency>

<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-common</artifactId>
  <version>1.11.2</version>
  <scope>provided</scope>
</dependency>
```

> `flink-table-common`：这个包中主要是包含 Flink Planner 和 Blink Planner一些共用的代码。<br/>
> `flink-table-api-java`：这部分是用户编程使用的 API，包含了大部分的 API。<br/>
> `flink-table-api-scala`：这里只是非常薄的一层，仅和 Table API 的 Expression 和 DSL 相关。<br/>
> 两个 Planner：`flink-table-planner` 和 `flink-table-planner-blink`。<br/>
> 两个 Bridge：`flink-table-api-scala-bridge` 和 `flink-table-api-java-bridge`，

> Flink Planner 和 Blink Planner 都会依赖于具体的 JAVAAPI，也会依赖于具体的 Bridge，通过 Bridge 可以将 API 操作相应的转化为Scala 的 DataStream、DataSet，或者转化为 JAVA 的 DataStream 或者Data Set

#### 8.2.2 API

> 注意:
>> 目前新版本的Flink的Table和SQL的API还不够稳定,依然在不断完善中,所以课程中的案例还是以老版本文档的API来演示

##### 8.2.2.1 获取环境

> `old-API`
>> https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/178.png)

> `new-API`
>> https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/common.html

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/179.png)

##### 8.2.2.2 程序结构

> `old-API`
>> https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/180.png)

> `new-API`
>> https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/common.html

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/181.png)

### 8.3 批处理案例

#### 8.3.1 数据

> 有sql_students.txt 保存学生信息：

```
id，name，classname
1,张三,1班
2,李四,1班
3,王五,2班
4,赵六,2班
5,田七,2班
```

> 有sql_scores.txt 保存成绩：

```
id，chinese，math，english
1,100,90,80
2,97,87,74
3,70,50,43
4,100,99,99
5,80,81,82
```

#### 8.3.2 需求

> 1.初始化flink env<br/>
> 2.读取文件数据，读取student.txt、scores.txt两张表<br/>
> 3.数据预处理，通过id字段将两个表的数据join出dataset<br/>
> 4.将dataset映射成table，并执行sql求各班级每个学科的平均分、三科总分平均分<br/>
> 5.结果保存到文件中

#### 8.3.3 代码实现-方式1

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.operators.DataSource;
import org.apache.flink.api.java.operators.JoinOperator;
import org.apache.flink.api.java.operators.MapOperator;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.api.java.tuple.Tuple4;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.java.BatchTableEnvironment;

/**
 * Author xiaoma
 * Date 2020/10/18 16:48
 * Desc
 * 求各班级每个学科的平均分和三科总分平均分
 * 有sql_students.txt 保存学生信息：
 * id，name，classname
 * 1,张三,1班
 * .....
 * 有sql_scores.txt 保存成绩：
 * id，chinese，math，english
 * 1,100,90,80
 * ...
 * 1.创建环境
 * 2.加载数据
 * 3.关联数据会有结果
 * 4.注册为表
 * 5.执行sql
 * 6.输出结果
 */
public class FlinkTable_SQL_Demo1_1_Yes {
    public static void main(String[] args) throws Exception {
        //1.创建环境
        //1.10
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        BatchTableEnvironment bTableEnv = BatchTableEnvironment.create(env);
        //1.7
        /* ExecutionEnvironment bEnv = ExecutionEnvironment.getExecutionEnvironment();
        BatchTableEnvironment bTableEnv = TableEnvironment.getTableEnvironment(bEnv);*/
        //2.加载数据
        DataSource<String> studentStrDS = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\sql_students.txt");
        DataSource<String> scoreStrDS = env.readTextFile("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\sql_scores.txt");
        //解析数据.解析出里面的
        //sql_students.txt: id，name，classname
        MapOperator<String, Tuple3<Integer, String, String>> studentTupleDS = studentStrDS.map(new MapFunction<String, Tuple3<Integer, String, String>>() {
            @Override
            public Tuple3<Integer, String, String> map(String value) throws Exception {
                String[] arr = value.split(",");
                return Tuple3.of(Integer.parseInt(arr[0]), arr[1], arr[2]);
            }
        });
        //sql_scores.txt: id，chinese，math，english
        MapOperator<String, Tuple4<Integer, Integer, Integer, Integer>> scoreTupleDS = scoreStrDS.map(new MapFunction<String, Tuple4<Integer, Integer, Integer,Integer>>() {
            @Override
            public Tuple4<Integer, Integer, Integer, Integer> map(String value) throws Exception {
                String[] arr = value.split(",");
                return Tuple4.of(Integer.parseInt(arr[0]), Integer.parseInt(arr[1]), Integer.parseInt(arr[2]),Integer.parseInt(arr[3]));
            }
        });
        //3.关联数据会有结果(id，name，classname,chinese，math，english)
        JoinOperator.ProjectJoin<Tuple3<Integer, String, String>, Tuple4<Integer, Integer, Integer, Integer>, Tuple> student_score_DS = studentTupleDS.join(scoreTupleDS).where(0).equalTo(0)
                .projectFirst(0, 1, 2)
                .projectSecond(1, 2, 3);

        //student_score_DS.print();

        //4.注册/创建表
        //bTableEnv.registerDataSet("t_student_score", student_score_DS,"id, name, classname, chinese, math, english");
        bTableEnv.createTemporaryView("t_student_score",student_score_DS,"id, name, classname, chinese, math, english");

        //5.执行sql
        //求各班级每个学科的平均分和三科总分平均分
/*
select
 classname,
 avg(chinese) as chineseAvg,
 avg(math) as mathAvg,
 avg(english) as englishAvg,
 avg(chinese+math+english) as totalAvg
from
 t_student_score
group by
 classname
*/
        /*String sql = "select classname, avg(chinese) as chineseAvg, avg(math) as mathAvg, avg(english) as englishAvg, avg(chinese+math+english) as totalAvg\n" +
                "from t_student_score\n" +
                "group by classname";*/
        String sql = "select\n" +
                " classname,\n" +
                " avg(chinese) as chineseAvg,\n" +
                " avg(math) as mathAvg,\n" +
                " avg(english) as englishAvg,\n" +
                " avg(chinese+math+english) as totalAvg\n" +
                "from \n" +
                " t_student_score\n" +
                "group by \n" +
                " classname";
        Table resultTable = bTableEnv.sqlQuery(sql);

        //6.输出结果
        resultTable.printSchema();//打印约束也就是打印里面有哪些字段什么类型
        //可以转为DS再打印
        DataSet<ResultInfo> resultDS = bTableEnv.toDataSet(resultTable, ResultInfo.class);
        resultDS.print();
        //FlinkTable_SQL_Demo1_1.ResultInfo(classname=1班, chineseAvg=98, mathAvg=88, englishAvg=77, totalAvg=264)
        //FlinkTable_SQL_Demo1_1.ResultInfo(classname=2班, chineseAvg=83, mathAvg=76, englishAvg=74, totalAvg=234)
    }
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class ResultInfo {
        public String classname;
        public Integer chineseAvg;
        public Integer mathAvg;
        public Integer englishAvg;
        public Integer totalAvg;
    }
}
```

#### 8.3.4 代码实现-方式2

```

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.java.BatchTableEnvironment;
import org.apache.flink.table.sources.CsvTableSource;

/**
 * Author xiaoma
 * Date 2020/10/18 16:48
 * Desc
 * 求各班级每个学科的平均分和三科总分平均分
 * 有sql_students.txt 保存学生信息：
 * id，name，classname
 * 1,张三,1班
 * .....
 * 有sql_scores.txt 保存成绩：
 * id，chinese，math，english
 * 1,100,90,80
 * ...
 * 1.创建环境
 * 2.加载数据-可以直接作为CSV加载
 * 3.注册为表
 * 4.再关联
 * 5.执行sql
 * 6.输出结果
 */
public class FlinkTable_SQL_Demo1_2_Yes {
    public static void main(String[] args) throws Exception {
        //1.创建环境
        //1.10
        ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
        BatchTableEnvironment bTableEnv = BatchTableEnvironment.create(env);
        //1.7
        /* ExecutionEnvironment bEnv = ExecutionEnvironment.getExecutionEnvironment();
        BatchTableEnvironment bTableEnv = TableEnvironment.getTableEnvironment(bEnv);*/
        //2.加载数据-可以直接作为CSV加载并解析完毕!
        CsvTableSource studentSource = CsvTableSource.builder()
                .path("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\sql_students.txt") //文件路径
                .field("id", Types.INT) //第一列数据
                .field("name", Types.STRING) //第二列数据
                .field("classname", Types.STRING) //第三列数据
                .fieldDelimiter(",") //列分隔符，默认是"，"
                .lineDelimiter("\n") //换行符
                //.ignoreFirstLine() //忽略第一行
                .ignoreParseErrors() //忽略解析错误
                .build();
        CsvTableSource scoreSource = CsvTableSource.builder()
                .path("E:\\code_bigData\\bigdata42_parent\\flink_study_42\\data\\input\\sql_scores.txt") //文件路径
                .field("id", Types.INT) //第一列数据
                .field("chinese", Types.INT) //第二列数据
                .field("math", Types.INT) //第三列数据
                .field("english", Types.INT) //第三列数据
                .build();
        //3.注册为表
        bTableEnv.registerTableSource("t_student",studentSource);
        bTableEnv.registerTableSource("t_score",scoreSource);

        //4.再关联并查询出结果并起个表名
        String sql ="select t1.id, name, classname, chinese, math, english from t_student t1 join t_score t2 on t1.id = t2.id";
        Table joinResultTable = bTableEnv.sqlQuery(sql);
        bTableEnv.registerTable("t_student_score",joinResultTable);


        //5.执行sql
        //求各班级每个学科的平均分和三科总分平均分
/*
select
 classname,
 avg(chinese) as chineseAvg,
 avg(math) as mathAvg,
 avg(english) as englishAvg,
 avg(chinese+math+english) as totalAvg
from
 t_student_score
group by
 classname
*/
        /*String sql2 = "select classname, avg(chinese) as chineseAvg, avg(math) as mathAvg, avg(english) as englishAvg, avg(chinese+math+english) as totalAvg\n" +
                "from t_student_score\n" +
                "group by classname";*/
        String sql2 = "select\n" +
                " classname,\n" +
                " avg(chinese) as chineseAvg,\n" +
                " avg(math) as mathAvg,\n" +
                " avg(english) as englishAvg,\n" +
                " avg(chinese+math+english) as totalAvg\n" +
                "from \n" +
                " t_student_score\n" +
                "group by \n" +
                " classname";
        Table resultTable = bTableEnv.sqlQuery(sql2);

        //6.输出结果
        resultTable.printSchema();//打印约束也就是打印里面有哪些字段什么类型
        //可以转为DS再打印
        DataSet<ResultInfo> resultDS = bTableEnv.toDataSet(resultTable, ResultInfo.class);
        resultDS.print();
        //FlinkTable_SQL_Demo1_1.ResultInfo(classname=1班, chineseAvg=98, mathAvg=88, englishAvg=77, totalAvg=264)
        //FlinkTable_SQL_Demo1_1.ResultInfo(classname=2班, chineseAvg=83, mathAvg=76, englishAvg=74, totalAvg=234)
    }
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class ResultInfo {
        public String classname;
        public Integer chineseAvg;
        public Integer mathAvg;
        public Integer englishAvg;
        public Integer totalAvg;
    }
}
```

### 8.4 流处理案例

#### 8.4.1 需求

> 使用Flink SQL来统计5秒内 每个用户的 订单总数、订单的最大金额、订单的最小金额

> 每隔5秒统计最近5秒的每个用户的订单总数、订单的最大金额、订单的最小金额

#### 8.4.2 代码实现-方式1

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.java.StreamTableEnvironment;
import org.apache.flink.types.Row;

import java.util.Random;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

/**
 * Author xiaoma
 * Date 2020/10/18 17:49
 * Desc
 * 使用Flink SQL来统计5秒内 每个用户的 订单总数、订单的最大金额、订单的最小金额
 * 每隔5秒统计最近5秒的每个用户的订单总数、订单的最大金额、订单的最小金额
 * 上面的需求使用流处理的Window的基于时间的滚动窗口就可以搞定!
 * 那么接下来使用FlinkTable&SQL-API来实现
 * 1.创建环境
 * 2.使用自定义函数模拟实时流数据
 * 3.设置事件时间和Watermaker
 * 4.注册表
 * 5.执行sql-分为sql风格和table风格(了解)
 * 6.输出结果
 */
public class FlinkTable_SQL_Demo2_1_Yes {
    public static void main(String[] args) throws Exception {
        //1.创建环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        StreamTableEnvironment tenv = StreamTableEnvironment.create(env);

        //2.使用自定义函数模拟实时流数据
        DataStreamSource<Order> orderDS = env.addSource(new SourceFunction<Order>() {
            private Boolean flag = true;

            @Override
            public void run(SourceContext<Order> ctx) throws Exception {
                Random random = new Random();
                while (flag) {
                    Order order = new Order(
                            UUID.randomUUID().toString(),
                            random.nextInt(3),
                            random.nextInt(101),
                            System.currentTimeMillis());
                    TimeUnit.SECONDS.sleep(1);
                    ctx.collect(order);
                }
            }

            @Override
            public void cancel() {
                flag = false;
            }
        });
        //3.设置事件时间和Watermaker
        //告诉Flink使用事件时间
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        //告诉Flink每隔多久生成Watermaker
        env.getConfig().setAutoWatermarkInterval(200);
        //告诉Flink哪一列是事件时间且指定最大允许的延迟时间或乱序时间
        //Watermaker = 当前最大的事件时间 - 最大允许的延迟时间或乱序时间
        //Watermaker >= 窗口介绍时间 的时候触发计算
        SingleOutputStreamOperator<Order> watermakerDS = orderDS.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<Order>(Time.seconds(2)) {
            @Override
            public long extractTimestamp(Order element) {
                return element.getCreateTime();
            }
        });

        //4.注册表
        //.rowtime表示告诉FlinkSQL这一列是事件时间列
        tenv.createTemporaryView("t_order",watermakerDS, "orderId, userId, money, createTime.rowtime");

        //5.执行sql-分为sql风格和table风格(了解)
        //每隔5秒统计最近5秒的每个用户的订单总数、订单的最大金额、订单的最小金额
        /*
select
 userId,count(*) as orderCount ,max(money) as maxMoney,min(money) as minMoney
from
 t_order
group by
 userId,
 tumble(createTime, interval '5' second)
         */
        String sql = "select \n" +
                " userId,count(*) as orderCount ,max(money) as maxMoney,min(money) as minMoney\n" +
                "from \n" +
                " t_order\n" +
                "group by \n" +
                " userId,\n" +
                " tumble(createTime, interval '5' second)";
        //滚动tumble(createTime, interval '5' second)
        //滑动hop(createTime,interval '5' second,interval '10' second)//每隔5s计算最近10s的数据
        //HOP(timeCol, slide, size)

        Table resultTable = tenv.sqlQuery(sql);

        //6.输出结果
        resultTable.printSchema();
        //转为DS
        //Row.class表示结果表中的每一行就转为一个Row对象/或单独定义一个JavaBean来接收
        DataStream<Row> resultDS = tenv.toAppendStream(resultTable, Row.class);
        resultDS.print();

        //7.触发执行
        env.execute();
    }
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class Order {
        private String orderId;
        private Integer userId;
        private Integer money;
        private Long createTime;
    }
}
```

#### 8.4.3 代码实现-方式2

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.Tumble;
import org.apache.flink.table.api.java.StreamTableEnvironment;
import org.apache.flink.types.Row;

import java.util.Random;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

/**
 * Author xiaoma
 * Date 2020/10/18 17:49
 * Desc
 * 使用Flink SQL来统计5秒内 每个用户的 订单总数、订单的最大金额、订单的最小金额
 * 每隔5秒统计最近5秒的每个用户的订单总数、订单的最大金额、订单的最小金额
 * 上面的需求使用流处理的Window的基于时间的滚动窗口就可以搞定!
 * 那么接下来使用FlinkTable&SQL-API来实现
 * 1.创建环境
 * 2.使用自定义函数模拟实时流数据
 * 3.设置事件时间和Watermaker
 * 4.注册表
 * 5.执行sql-分为sql风格和table风格(了解)
 * 6.输出结果
 */
public class FlinkTable_SQL_Demo2_2 {
    public static void main(String[] args) throws Exception {
        //1.创建环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        StreamTableEnvironment tenv = StreamTableEnvironment.create(env);

        //2.使用自定义函数模拟实时流数据
        DataStreamSource<Order> orderDS = env.addSource(new SourceFunction<Order>() {
            private Boolean flag = true;
            @Override
            public void run(SourceContext<Order> ctx) throws Exception {
                Random random = new Random();
                while (flag) {
                    Order order = new Order(
                            UUID.randomUUID().toString(),
                            random.nextInt(3),
                            random.nextInt(101),
                            System.currentTimeMillis());
                    TimeUnit.SECONDS.sleep(1);
                    ctx.collect(order);
                }
            }

            @Override
            public void cancel() {
                flag = false;
            }
        });
        //3.设置事件时间和Watermaker
        //告诉Flink使用事件时间
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        //告诉Flink每隔多久生成Watermaker
        env.getConfig().setAutoWatermarkInterval(200);
        //告诉Flink哪一列是事件时间且指定最大允许的延迟时间或乱序时间
        //Watermaker = 当前最大的事件时间 - 最大允许的延迟时间或乱序时间
        //Watermaker >= 窗口介绍时间 的时候触发计算
        SingleOutputStreamOperator<Order> watermakerDS = orderDS.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<Order>(Time.seconds(2)) {
            @Override
            public long extractTimestamp(Order element) {
                return element.getCreateTime();
            }
        });

        //4.注册表
        //.rowtime表示告诉FlinkSQL这一列是事件时间列
        tenv.createTemporaryView("t_order",watermakerDS, "orderId, userId, money, createTime.rowtime");

        //5.执行sql-分为sql风格和table风格(了解)
        //每隔5秒统计最近5秒的每个用户的订单总数、订单的最大金额、订单的最小金额
        /*
select
 userId,count(*) as orderCount ,max(money) as maxMoney,min(money) as minMoney
from
 t_order
group by
 userId,
 tumble(createTime, interval '5' second)
         */
       /* String sql = "select \n" +
                " userId,count(*) as orderCount ,max(money) as maxMoney,min(money) as minMoney\n" +
                "from \n" +
                " t_order\n" +
                "group by \n" +
                " userId,\n" +
                " tumble(createTime, interval '5' second)";

        Table resultTable = tenv.sqlQuery(sql);*/

        //使用table风格-了解
        Table resultTable = tenv.from("t_order")
                .window(Tumble.over("5.second").on("createTime").as("secondsWindow"))
                .groupBy("secondsWindow, userId")
                .select("userId,count(1) as totalCount,max(money) as maxMoney,min(money) as minMoney");


        //6.输出结果
        resultTable.printSchema();
        //转为DS
        //Row.class表示结果表中的每一行就转为一个Row对象/或单独定义一个JavaBean来接收
        DataStream<Row> resultDS = tenv.toAppendStream(resultTable, Row.class);
        //下面的Boolean表示数据是新增/更新true还是删除false
        DataStream<Tuple2<Boolean, Row>> resultDS2 = tenv.toRetractStream(resultTable, Row.class);

        resultDS.print();

        //7.触发执行
        env.execute();
    }
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class Order {
        private String orderId;
        private Integer userId;
        private Integer money;
        private Long createTime;
    }
}
```

### 8.5 总结-Flink-SQL常用算子

#### 8.5.1 SELECT

> SELECT 用于从 DataSet/DataStream 中选择数据，用于筛选出某些列。

> 示例：

```
SELECT * FROM Table；// 取出表中的所有列
SELECT name，age FROM Table；// 取出表中 name 和 age 两列
```

> 与此同时 SELECT 语句中可以使用函数和别名，例如我们上面提到的 WordCount 中：

```
SELECT word, COUNT(word) FROM table GROUP BY word;
```

#### 8.5.2 WHERE

> WHERE 用于从数据集/流中过滤数据，与 SELECT 一起使用，用于根据某些条件对关系做水平分割，即选择符合条件的记录。

> 示例：

```
SELECT name，age FROM Table where name LIKE ‘% 小明 %’；
SELECT * FROM Table WHERE age = 20；
```

> WHERE 是从原数据中进行过滤，那么在 WHERE 条件中，Flink SQL 同样支持 =、<、>、<>、>=、<=，以及 AND、OR 等表达式的组合，最终满足过滤条件的数据会被选择出来。并且 WHERE 可以结合 IN、NOT IN 联合使用。举个例子：

```
SELECT name, age
FROM Table
WHERE name IN (SELECT name FROM Table2)
```

#### 8.5.3 DISTINCT

> DISTINCT 用于从数据集/流中去重根据 SELECT 的结果进行去重。

> 示例：

```
SELECT DISTINCT name FROM Table;
```

> 对于流式查询，计算查询结果所需的 State 可能会无限增长，用户需要自己控制查询的状态范围，以防止状态过大。

#### 8.5.4 GROUP BY

> GROUP BY 是对数据进行分组操作。例如我们需要计算成绩明细表中，每个学生的总分。

> 示例：

```
SELECT name, SUM(score) as TotalScore FROM Table GROUP BY name;
```

#### 8.5.5 UNION 和 UNION ALL

> UNION 用于将两个结果集合并起来，要求两个结果集字段完全一致，包括字段类型、字段顺序。

> 不同于 UNION ALL 的是，UNION 会对结果数据去重。

> 示例：

```
SELECT * FROM T1 UNION (ALL) SELECT * FROM T2；
```

#### 8.5.6 JOIN

> JOIN 用于把来自两个表的数据联合起来形成结果表，Flink 支持的 JOIN 类型包括：

```
JOIN - INNER JOIN
LEFT JOIN - LEFT OUTER JOIN
RIGHT JOIN - RIGHT OUTER JOIN
FULL JOIN - FULL OUTER JOIN
这里的 JOIN 的语义和我们在关系型数据库中使用的 JOIN 语义一致。
```

> 示例：

```
JOIN(将订单表数据和商品表进行关联)
SELECT * FROM Orders INNER JOIN Product ON Orders.productId = Product.id
```

> LEFT JOIN 与 JOIN 的区别是当右表没有与左边相 JOIN 的数据时候，右边对应的字段补 NULL 输出，RIGHT JOIN 相当于 LEFT JOIN 左右两个表交互一下位置。FULL JOIN 相当于 RIGHT JOIN 和 LEFT JOIN 之后进行 UNION ALL 操作。

> 示例：

```
SELECT * FROM Orders LEFT JOIN Product ON Orders.productId = Product.id
SELECT * FROM Orders RIGHT JOIN Product ON Orders.productId = Product.id
SELECT * FROM Orders FULL OUTER JOIN Product ON Orders.productId = Product.id
```

#### 8.5.7 Group Window

> 根据窗口数据划分的不同，目前 Apache Flink 有如下 3 种 Bounded Window：

> Tumble，滚动窗口，窗口数据有固定的大小，窗口数据无叠加；

> `Hop，滑动窗口，窗口数据有固定大小，并且有固定的窗口重建频率，窗口数据有叠加；`

> Session，会话窗口，窗口数据没有固定的大小，根据窗口数据活跃程度划分窗口，窗口数据无叠加。

##### 8.5.7.1 Tumble Window

> Tumble 滚动窗口有固定大小，窗口数据不重叠，具体语义如下：

> Tumble 滚动窗口对应的语法如下：

```
SELECT
[gk],
[TUMBLE_START(timeCol, size)],
[TUMBLE_END(timeCol, size)],
agg1(col1),
...
aggn(colN)
FROM Tab1
GROUP BY [gk], TUMBLE(timeCol, size)
```

> 其中：

```
[gk] 决定了是否需要按照字段进行聚合；
TUMBLE_START 代表窗口开始时间；
TUMBLE_END 代表窗口结束时间；
timeCol 是流表中表示时间字段；
size 表示窗口的大小，如 秒、分钟、小时、天。
```

> 举个例子，假如我们要计算每个人每天的订单量，按照 user 进行聚合分组：

```
SELECT user, TUMBLE_START(rowtime, INTERVAL ‘1’ DAY) as wStart, SUM(amount)
FROM Orders
GROUP BY TUMBLE(rowtime, INTERVAL ‘1’ DAY), user;
```

##### 8.5.7.2 Hop Window

> Hop 滑动窗口和滚动窗口类似，窗口有固定的 size，与滚动窗口不同的是滑动窗口可以通过 slide 参数控制滑动窗口的新建频率。因此当 slide 值小于窗口 size 的值的时候多个滑动窗口会重叠，具体语义如下：

> Hop 滑动窗口对应语法如下：

```
SELECT
[gk],
[HOP_START(timeCol, slide, size)] ,  
[HOP_END(timeCol, slide, size)],
agg1(col1),
...
aggN(colN)
FROM Tab1
GROUP BY [gk], HOP(timeCol, slide, size)
```

> 每次字段的意思和 Tumble 窗口类似：

```
[gk] 决定了是否需要按照字段进行聚合；
HOP_START 表示窗口开始时间；
HOP_END 表示窗口结束时间；
timeCol 表示流表中表示时间字段；
slide 表示每次窗口滑动的大小；
size 表示整个窗口的大小，如 秒、分钟、小时、天。
```

> 举例说明，我们要每过一小时计算一次过去 24 小时内每个商品的销量：

```
SELECT product, SUM(amount)
FROM Orders
GROUP BY product,HOP(rowtime, INTERVAL '1' HOUR, INTERVAL '1' DAY)
```

##### 8.5.7.3 Session Window

> 会话时间窗口没有固定的持续时间，但它们的界限由 interval 不活动时间定义，即如果在定义的间隙期间没有出现事件，则会话窗口关闭。

> Seeeion 会话窗口对应语法如下：

```
SELECT
[gk],
SESSION_START(timeCol, gap) AS winStart,  
SESSION_END(timeCol, gap) AS winEnd,
agg1(col1),
...
aggn(colN)
FROM Tab1
GROUP BY [gk], SESSION(timeCol, gap)
```

```
[gk] 决定了是否需要按照字段进行聚合；
SESSION_START 表示窗口开始时间；
SESSION_END 表示窗口结束时间；
timeCol 表示流表中表示时间字段；
gap 表示窗口数据非活跃周期的时长。
```

> 例如，我们需要计算每个用户访问时间 12 小时内的订单量：

```
SELECT user, SESSION_START(rowtime, INTERVAL ‘12’ HOUR) AS sStart, SESSION_ROWTIME(rowtime, INTERVAL ‘12’ HOUR) AS sEnd, SUM(amount)
FROM Orders
GROUP BY SESSION(rowtime, INTERVAL ‘12’ HOUR), user
```

## 9 Flink-Action综合案例

> 掌握使用Flink实现模拟双十一实时大屏统计

> 掌握使用Flink实现订单自动好评

> 掌握使用Flink-BroadcastState实现配置动态更新

### 9.1 Flink实现模拟双十一实时大屏统计

#### 9.1.1 需求

> 在大数据的实时处理中，实时的大屏展示已经成了一个很重要的展示项，比如最有名的双十一大屏实时销售总价展示。除了这个，还有一些其他场景的应用，比如我们在我们的后台系统实时的展示我们网站当前的pv、uv等等，其实做法都是类似的。

> 今天我们就做一个最简单的模拟电商统计大屏的小例子，

> 需求如下：
>> 1.实时计算出当天零点截止到当前时间的销售总额
>> 2.计算出各个分类的销售top3
>> 3.每秒钟更新一次统计结果

#### 9.1.2 数据

> 首先我们通过自定义source 模拟订单的生成，生成了一个Tuple2,第一个元素是分类，第二个元素表示这个分类下产生的订单金额，金额我们通过随机生成.

```
/**
* 模拟生成某一个分类下的订单
  */
  public static class MySource implements SourceFunction<Tuple2<String,Double>> {
      private volatile boolean isRunning = true;
      private Random random = new Random();
      String category[] = {
          "女装", "男装",
          "图书", "家电",
          "洗护", "美妆",
          "运动", "游戏",
          "户外", "家具",
          "乐器", "办公"
        };

      @Override
      public void run(SourceContext<Tuple2<String,Double>> ctx) throws Exception{
          while (isRunning){
              Thread.sleep(10);
              //随机生成一个分类
              String c = category[(int) (Math.random() * (category.length - 1))];
              //随机生成一个该分类下的随机金额的成交订单
              double price = random.nextDouble() * 100;
              ctx.collect(Tuple2.of(c, price));
          }
      }
      @Override
      public void cancel(){
        isRunning = false;
      }
  }
```

#### 9.1.3 代码实现

```
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.apache.commons.lang3.StringUtils;
import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple1;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.functions.windowing.WindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingProcessingTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.triggers.ContinuousProcessingTimeTrigger;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

import java.math.BigDecimal;
import java.text.SimpleDateFormat;
import java.util.*;
import java.util.stream.Collectors;

/**
 * Author xiaoma
 * Date 2020/10/20 10:12
 * Desc
 * 模拟双11商品实时交易大屏统计分析
 */
public class DoubleElevenBigScreem {
    public static void main(String[] args) throws Exception{
        //编码步骤:
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);//学习测试方便观察

        //2.source
        //模拟实时订单信息
        DataStreamSource<Tuple2<String, Double>> sourceDS = env.addSource(new MySource());

        /*
        注意:需求如下：
        -1.实时计算出11月11日00:00:00零点开始截止到当前时间的销售总额
        -2.计算出各个分类的销售额top3
        -3.每1秒钟更新一次统计结果
        如果使用之前学习的简单的timeWindow(Time size窗口大小, Time slide滑动间隔)来处理,
        如xxx.timeWindow(24小时,1s),计算的是需求中的吗?
        不是!如果使用之前的做法那么是完成不了需求的,因为:
        如11月11日00:00:01计算的是11月10号[00:00:00~23:59:59s]的数据
        而我们应该要计算的是:11月11日00:00:00~11月11日00:00:01
        所以不能使用之前的简单做法!*/

        //3.transformation
        //.keyBy(0)
        SingleOutputStreamOperator<CategoryPojo> tempAggResult = sourceDS.keyBy(0)
                //3.1定义大小为一天的窗口,第二个参数表示中国使用的UTC+08:00时区比UTC时间早
                /*
                of(Time 窗口大小, Time 带时间校准的从哪开始)源码中有解释:
                如果您居住在不使用UTC±00：00时间的地方，例如使用UTC + 08：00的中国，并且您需要一个大小为一天的时间窗口，
                并且窗口从当地时间的每00:00:00开始，您可以使用of(Time.days(1)，Time.hours(-8))
                注意:该代码如果在11月11日运行就会从11月11日00:00:00开始记录直到11月11日23:59:59的1天的数据
                注意:我们这里简化了没有把之前的Watermaker那些代码拿过来,所以直接ProcessingTime
                */
                .window(TumblingProcessingTimeWindows.of(Time.days(1), Time.hours(-8)))//仅仅只定义了一个窗口大小
                //3.2定义一个1s的触发器
                .trigger(ContinuousProcessingTimeTrigger.of(Time.seconds(1)))
                //上面的3.1和3.2相当于自定义窗口的长度和触发时机
                //3.3聚合结果.aggregate(new PriceAggregate(), new WindowResult());
                //.sum(1)//以前的写法用的默认的聚合和收集
                //现在可以自定义如何对price进行聚合,并自定义聚合结果用怎样的格式进行收集
                .aggregate(new PriceAggregate(), new WindowResult());
        //3.4看一下初步聚合的结果
        tempAggResult.print("初步聚合结果");
        //CategoryPojo(category=运动, totalPrice=118.69, dateTime=2020-10-20 08:04:12)
        //上面的结果表示:当前各个分类的销售总额

        /*
        注意:需求如下：
        -1.实时计算出11月11日00:00:00零点开始截止到当前时间的销售总额
        -2.计算出各个分类的销售额top3
        -3.每1秒钟更新一次统计结果
         */
        //4.使用上面初步聚合的结果,实现业务需求,并sink
        tempAggResult.keyBy("dateTime")//按照时间分组是因为需要每1s更新截至到当前时间的销售总额
        //每秒钟更新一次统计结果
                //Time size 为1s,表示计算最近1s的数据
        .window(TumblingProcessingTimeWindows.of(Time.seconds(1)))
        //在ProcessWindowFunction中实现该复杂业务逻辑,一次性将需求1和2搞定
        //-1.实时计算出11月11日00:00:00零点开始截止到当前时间的销售总额
        //-2.计算出各个分类的销售额top3
        //-3.每1秒钟更新一次统计结果
       .process(new WindowResultProcess());//window后的process方法可以处理复杂逻辑


        //5.execute
        env.execute();
    }

    /**
     * 自定义数据源实时产生订单数据Tuple2<分类, 金额>
     */
    public static class MySource implements SourceFunction<Tuple2<String, Double>>{
        private boolean flag = true;
        private String[] categorys = {"女装", "男装","图书", "家电","洗护", "美妆","运动", "游戏","户外", "家具","乐器", "办公"};
        private Random random = new Random();

        @Override
        public void run(SourceContext<Tuple2<String, Double>> ctx) throws Exception {
            while (flag){
                //随机生成分类和金额
                int index = random.nextInt(categorys.length);//[0~length) ==> [0~length-1]
                String category = categorys[index];//获取的随机分类
                double price = random.nextDouble() * 100;//注意nextDouble生成的是[0~1)之间的随机数,*100之后表示[0~100)
                ctx.collect(Tuple2.of(category,price));
                Thread.sleep(20);
            }
        }

        @Override
        public void cancel() {
            flag = false;
        }
    }

    /**
     * 自定义价格聚合函数,其实就是对price的简单sum操作
     * AggregateFunction<IN, ACC, OUT>
     * AggregateFunction<Tuple2<String, Double>, Double, Double>
     *
     */
    private static class PriceAggregate implements AggregateFunction<Tuple2<String, Double>, Double, Double> {
        //初始化累加器为0
        @Override
        public Double createAccumulator() {
            return 0D; //D表示Double,L表示long
        }

        //把price往累加器上累加
        @Override
        public Double add(Tuple2<String, Double> value, Double accumulator) {
            return value.f1 + accumulator;
        }

        //获取累加结果
        @Override
        public Double getResult(Double accumulator) {
            return accumulator;
        }

        //各个subTask的结果合并
        @Override
        public Double merge(Double a, Double b) {
            return a + b;
        }
    }

    /**
     * 用于存储聚合的结果
     */
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public static class CategoryPojo {
        private String category;//分类名称
        private double totalPrice;//该分类总销售额
        private String dateTime;// 截止到当前时间的时间,本来应该是EventTime,但是我们这里简化了代码没有用Watermaker那一套,所以这块直接用当前系统时间即可
    }

    /**
     * 自定义WindowFunction,实现如何收集窗口结果数据
     * interface WindowFunction<IN, OUT, KEY, W extends Window>
     * interface WindowFunction<Double, CategoryPojo, Tuple的真实类型就是String就是分类, W extends Window>
     */
    private static class WindowResult implements WindowFunction<Double, CategoryPojo, Tuple, TimeWindow> {
        //定义一个时间格式化工具用来将当前时间(双十一那天订单的时间)转为String格式
        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        @Override
        public void apply(Tuple tuple, TimeWindow window, Iterable<Double> input, Collector<CategoryPojo> out) throws Exception {
            String category = ((Tuple1<String>) tuple).f0;

            Double price = input.iterator().next();
            //为了后面项目铺垫,使用一下用Bigdecimal来表示精确的小数
            BigDecimal bigDecimal = new BigDecimal(price);
            //setScale设置精度保留2位小数,
            double roundPrice = bigDecimal.setScale(2, BigDecimal.ROUND_HALF_UP).doubleValue();//四舍五入

            long currentTimeMillis = System.currentTimeMillis();
            String dateTime = df.format(currentTimeMillis);

            CategoryPojo categoryPojo = new CategoryPojo(category, roundPrice, dateTime);
            out.collect(categoryPojo);
        }
    }

    /**
     * 实现ProcessWindowFunction
     * abstract class ProcessWindowFunction<IN, OUT, KEY, W extends Window>
     * abstract class ProcessWindowFunction<CategoryPojo, Object, Tuple就是String类型的dateTime, TimeWindow extends Window>
     *
     * 把各个分类的总价加起来，就是全站的总销量金额，
     * 然后我们同时使用优先级队列计算出分类销售的Top3，
     * 最后打印出结果，在实际中我们可以把这个结果数据存储到hbase或者redis中，以供前端的实时页面展示。
     */
    private static class WindowResultProcess extends ProcessWindowFunction<CategoryPojo, Object, Tuple, TimeWindow> {
        @Override
        public void process(Tuple tuple, Context context, Iterable<CategoryPojo> elements, Collector<Object> out) throws Exception {
            String dateTime = ((Tuple1<String>)tuple).f0;
            //Java中的大小顶堆可以使用优先级队列来实现
            //https://blog.csdn.net/hefenglian/article/details/81807527
            //注意:
            // 小顶堆用来计算:最大的topN
            // 大顶堆用来计算:最小的topN
            Queue<CategoryPojo> queue = new PriorityQueue<>(3,//初识容量
                    //正常的排序,就是小的在前,大的在后,也就是c1>c2的时候返回1,也就是小顶堆
                    (c1, c2) -> c1.getTotalPrice() >= c2.getTotalPrice() ? 1 : -1);

            //在这里我们要完成需求:
            // * -1.实时计算出11月11日00:00:00零点开始截止到当前时间的销售总额,其实就是把之前的初步聚合的price再累加!
            double totalPrice = 0D;
            double roundPrice = 0D;
            Iterator<CategoryPojo> iterator = elements.iterator();
            for (CategoryPojo element : elements) {
                double price = element.totalPrice;//某个分类的总销售额
                totalPrice += price;
                BigDecimal bigDecimal = new BigDecimal(totalPrice);
                roundPrice = bigDecimal.setScale(2, BigDecimal.ROUND_HALF_UP).doubleValue();//四舍五入
            // * -2.计算出各个分类的销售额top3,其实就是对各个分类的price进行排序取前3
                //注意:我们只需要top3,也就是只关注最大的前3个的顺序,剩下不管!所以不要使用全局排序,只需要做最大的前3的局部排序即可
                //那么可以使用小顶堆,把小的放顶上
                // c:80
                // b:90
                // a:100
                //那么来了一个数,和最顶上的比,如d,
                //if(d>顶上),把顶上的去掉,把d放上去,再和b,a比较并排序,保证顶上是最小的
                //if(d<=顶上),不用变
                if (queue.size() < 3) {//小顶堆size<3,说明数不够,直接放入
                    queue.add(element);
                }else{//小顶堆size=3,说明,小顶堆满了,进来一个需要比较
                    //"取出"顶上的(不是移除)
                    CategoryPojo top = queue.peek();
                    if(element.totalPrice > top.totalPrice){
                        //queue.remove(top);//移除指定的元素
                        queue.poll();//移除顶上的元素
                        queue.add(element);
                    }
                }
            }
            // * -3.每1秒钟更新一次统计结果,可以直接打印/sink,也可以收集完结果返回后再打印,
            //   但是我们这里一次性处理了需求1和2的两种结果,不好返回,所以直接输出!
            //对queue中的数据逆序
            //各个分类的销售额top3
            List<String> top3Result = queue.stream()
                    .sorted((c1, c2) -> c1.getTotalPrice() > c2.getTotalPrice() ? -1 : 1)//逆序
                    .map(c -> "(分类：" + c.getCategory() + " 销售总额：" + c.getTotalPrice() + ")")
                    .collect(Collectors.toList());
            System.out.println("时间 ： " + dateTime + "  总价 : " + roundPrice + " top3:\n" + StringUtils.join(top3Result, ",\n"));
            System.out.println("-------------");

        }
    }
}
```

#### 9.1.4 效果

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/182.png)

### 9.2 Flink实现订单自动好评

#### 9.2.1 需求

> 在电商领域会有这么一个场景，如果用户买了商品，在订单完成之后，一定时间之内没有做出评价，系统自动给与五星好评，我们今天主要使用Flink的定时器来简单实现这一功能。

#### 9.2.2 数据

> 自定义source模拟生成一些订单数据.

> 在这里，我们生了一个最简单的二元组Tuple2,包含订单id和订单完成时间两个字段.

```
/**
 * 自定义source模拟生成一些订单数据.
 * 在这里，我们生了一个最简单的二元组Tuple2,包含订单id和订单完成时间两个字段.
 */
public static class MySource implements SourceFunction<Tuple2<String, Long>> {
    private volatile boolean isRunning = true;

    @Override
    public void run(SourceContext<Tuple2<String, Long>> ctx) throws Exception {
        Random random = new Random();
        while (isRunning) {
            Thread.sleep(1000);
            //订单id
            String orderid = UUID.randomUUID().toString();
            //订单完成时间
            long orderFinishTime = System.currentTimeMillis();
            ctx.collect(Tuple2.of(orderid, orderFinishTime));
        }
    }

    @Override
    public void cancel() {
        isRunning = false;
    }
}
```

#### 9.2.3 代码实现

```
import org.apache.flink.api.common.state.MapState;
import org.apache.flink.api.common.state.MapStateDescriptor;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.util.Collector;

import java.util.Iterator;
import java.util.Map;
import java.util.UUID;

/**
 * Author xiaoma
 * Date 2020/10/20 14:34
 * Desc
 * 在电商领域会有这么一个场景，如果用户买了商品，在订单完成之后，一定时间之内没有做出评价，系统自动给与五星好评，
 * 我们今天主要使用Flink的定时器来简单实现这一功能。
 */
public class OrderAutomaticFavorableComments {
    public static void main(String[] args) throws Exception{
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        //2.source
        DataStreamSource<Tuple2<String, Long>> sourceDS = env.addSource(new MySource());
        //这里可以使用订单生成时间作为事件时间,代码和之前的一样
        //这里不作为重点,所以简化处理!

        //3.transformation
        //设置经过interval用户未对订单做出评价，自动给与好评.为了演示方便，设置5000ms的时间
        long interval = 5000L;
        //分组后使用自定义KeyedProcessFunction完成定时判断超时订单并自动好评
        sourceDS.keyBy(0) //实际中可以对用户id进行分组
                //KeyedProcessFunction:进到窗口的数据是分好组的
                //ProcessFunction:进到窗口的数据是不区分分组的
        .process(new TimerProcessFuntion(interval));
        //4.execute
        env.execute();
    }

    /**
     * 自定义source实时产生订单数据Tuple2<订单id, 订单生成时间>
     */
    public static class MySource implements SourceFunction<Tuple2<String, Long>>{
        private boolean flag = true;
        @Override
        public void run(SourceContext<Tuple2<String, Long>> ctx) throws Exception {
            while (flag){
                String orderId = UUID.randomUUID().toString();
                long currentTimeMillis = System.currentTimeMillis();
                ctx.collect(Tuple2.of(orderId,currentTimeMillis));
                Thread.sleep(500);
            }
        }

        @Override
        public void cancel() {
            flag = false;
        }
    }

    /**
     * 自定义处理函数用来给超时订单做自动好评!
     * 如一个订单进来:<订单id, 2020-10-10 12:00:00>
     * 那么该订单应该在12:00:00 + 5s 的时候超时!
     * 所以我们可以在订单进来的时候设置一个定时器,在订单时间 + interval的时候触发!
     * KeyedProcessFunction<K, I, O>
     * KeyedProcessFunction<Tuple就是String, Tuple2<订单id, 订单生成时间>, Object>
     */
    public static class TimerProcessFuntion extends KeyedProcessFunction<Tuple, Tuple2<String, Long>, Object> {
        private long interval;
        public TimerProcessFuntion(long interval){
            this.interval = interval;//传过来的是5000ms/5s
        }
        //3.1定义MapState类型的状态，key是订单号，value是订单完成时间
        //定义一个状态用来记录订单信息
        //MapState<订单id, 订单完成时间>
        private MapState<String, Long> mapState;

        //3.2初始化MapState
        @Override
        public void open(Configuration parameters) throws Exception {
            //创建状态描述器
            MapStateDescriptor<String, Long> mapStateDesc = new MapStateDescriptor<>("mapState", String.class, Long.class);
            //根据状态描述器初始化状态
            mapState = getRuntimeContext().getMapState(mapStateDesc);
        }


        //3.3注册定时器
        //处理每一个订单并设置定时器
        @Override
        public void processElement(Tuple2<String, Long> value, Context ctx, Collector<Object> out) throws Exception {
            mapState.put(value.f0,value.f1);
            //如一个订单进来:<订单id, 2020-10-10 12:00:00>
            //那么该订单应该在12:00:00 + 5s 的时候超时!
            //在订单进来的时候设置一个定时器,在订单时间 + interval的时候触发!!!
            ctx.timerService().registerProcessingTimeTimer(value.f1 + interval);
        }

        //3.4定时器被触发时执行并输出结果并sink
        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<Object> out) throws Exception {
            //能够执行到这里说明订单超时了!超时了得去看看订单是否评价了(实际中应该要调用外部接口/方法查订单系统!,我们这里没有,所以模拟一下)
            //没有评价才给默认好评!并直接输出提示!
            //已经评价了,直接输出提示!
            Iterator<Map.Entry<String, Long>> iterator = mapState.iterator();
            while (iterator.hasNext()){
                Map.Entry<String, Long> entry = iterator.next();
                String orderId = entry.getKey();
                //调用订单系统查询是否已经评价
                boolean result = isEvaluation(orderId);
                if (result){//已评价
                    System.out.println("订单(orderid: "+orderId+")在"+interval+"毫秒时间内已经评价，不做处理");
                }else{//未评价
                    System.out.println("订单(orderid: "+orderId+")在"+interval+"毫秒时间内未评价，系统自动给了默认好评!");
                    //实际中还需要调用订单系统将该订单orderId设置为5星好评!
                }
                //从状态中移除已经处理过的订单,避免重复处理
                mapState.remove(orderId);
            }
        }

        //在生产环境下，可以去查询相关的订单系统.
        private boolean isEvaluation(String key) {
            return key.hashCode() % 2 == 0;//随机返回订单是否已评价
        }
    }
}
```

#### 9.2.4 效果

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/183.png)

### 9.3 Flink-BroadcastState实现配置动态更新

#### 9.3.1 需求

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/184.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/185.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Flink/186.png)

> 实时过滤出配置中的用户，并在事件流中补全这批用户的基础信息。

> 事件流：表示用户在某个时刻浏览或点击了某个商品，格式如下。

```
{"userID": "user_3", "eventTime": "2019-08-17 12:19:47", "eventType": "browse", "productID": 1}
{"userID": "user_2", "eventTime": "2019-08-17 12:19:48", "eventType": "click", "productID": 1}
```

> 配置数据: 表示用户的详细信息，在Mysql中，如下。

```
DROP TABLE IF EXISTS `user_info`;
CREATE TABLE `user_info`  (
`userID` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
`userName` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
`userAge` int(11) NULL DEFAULT NULL,
PRIMARY KEY (`userID`) USING BTREE
) ENGINE = MyISAM CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
-- ----------------------------
-- Records of user_info
-- ----------------------------
INSERT INTO `user_info` VALUES ('user_1', '张三', 10);
INSERT INTO `user_info` VALUES ('user_2', '李四', 20);
INSERT INTO `user_info` VALUES ('user_3', '王五', 30);
INSERT INTO `user_info` VALUES ('user_4', '赵六', 40);
SET FOREIGN_KEY_CHECKS = 1;
```

> 输出结果:

```
(user_3,2019-08-17 12:19:47,browse,1,王五,33)
(user_2,2019-08-17 12:19:48,click,1,李四,20)
```

#### 9.3.2 BroadcastState介绍

> 在开发过程中，如果遇到需要下发/广播配置、规则等低吞吐事件流到下游所有 task 时，就可以使用 Broadcast State。Broadcast State 是 Flink 1.5 引入的新特性。

> 下游的 task 接收这些配置、规则并保存为 BroadcastState, 将这些配置应用到另一个数据流的计算中 。

> 场景举例
>> 1)动态更新计算规则: 如事件流需要根据最新的规则进行计算，则可将规则作为广播状态广播到下游Task中。
>> 2)实时增加额外字段: 如事件流需要实时增加用户的基础信息，则可将用户的基础信息作为广播状态广播到下游Task中。

#### 9.3.3 API介绍

> 首先创建一个Keyed 或Non-Keyed 的DataStream，<br/>
> 然后再创建一个BroadcastedStream，<br/>
> 最后通过DataStream来连接(调用connect 方法)到Broadcasted Stream 上，<br/>
> 这样实现将BroadcastState广播到Data Stream 下游的每个Task中。

> 1.如果DataStream是Keyed Stream ，则连接到Broadcasted Stream 后， 添加处理ProcessFunction 时需要使用KeyedBroadcastProcessFunction 来实现， 下面是KeyedBroadcastProcessFunction 的API，代码如下所示：

```
public abstract class KeyedBroadcastProcessFunction<KS, IN1, IN2, OUT> extends BaseBroadcastProcessFunction {
    public abstract void processElement(final IN1 value, final ReadOnlyContext ctx, final Collector<OUT> out) throws Exception;
    public abstract void processBroadcastElement(final IN2 value, final Context ctx, final Collector<OUT> out) throws Exception;
}
```

> 上面泛型中的各个参数的含义，说明如下：
>>  KS：表示Flink 程序从最上游的Source Operator 开始构建Stream，当调用keyBy 时所依赖的Key 的类型；
> 
>>  IN1：表示非Broadcast 的Data Stream 中的数据记录的类型；
> 
>>  IN2：表示Broadcast Stream 中的数据记录的类型；
> 
>>  OUT：表示经过KeyedBroadcastProcessFunction 的processElement()和processBroadcastElement()方法处理后输出结果数据记录的类型。

> 2.如果Data Stream 是Non-Keyed Stream，则连接到Broadcasted Stream 后，添加处理ProcessFunction 时需要使用BroadcastProcessFunction 来实现， 下面是BroadcastProcessFunction 的API，代码如下所示：

```
public abstract class BroadcastProcessFunction<IN1, IN2, OUT> extends BaseBroadcastProcessFunction {
    public abstract void processElement(final IN1 value, final ReadOnlyContext ctx, final Collector<OUT> out) throws Exception;
    public abstract void processBroadcastElement(final IN2 value, final Context ctx, final Collector<OUT> out) throws Exception;
}
```

> 上面泛型中的各个参数的含义，与前面KeyedBroadcastProcessFunction 的泛型类型中的后3 个含义相同，只是没有调用keyBy 操作对原始Stream 进行分区操作，就不需要KS 泛型参数。

> 具体如何使用上面的BroadcastProcessFunction，接下来我们会在通过实际编程，来以使用KeyedBroadcastProcessFunction 为例进行详细说明。

> 注意事项
>> 1) Broadcast State 是Map 类型，即K-V 类型。
> 
>> 2) Broadcast State 只有在广播的一侧, 即在BroadcastProcessFunction 或KeyedBroadcastProcessFunction 的processBroadcastElement 方法中可以修改。在非广播的一侧， 即在BroadcastProcessFunction 或KeyedBroadcastProcessFunction 的processElement 方法中只读。
> 
>> 3) Broadcast State 中元素的顺序，在各Task 中可能不同。基于顺序的处理，需要注意。
> 
>> 4) Broadcast State 在Checkpoint 时，每个Task 都会Checkpoint 广播状态。
> 
>> 5) Broadcast State 在运行时保存在内存中，目前还不能保存在RocksDB State Backend 中。

#### 9.3.4 代码实现

```
import org.apache.flink.api.common.state.BroadcastState;
import org.apache.flink.api.common.state.MapStateDescriptor;
import org.apache.flink.api.common.state.ReadOnlyBroadcastState;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.tuple.Tuple4;
import org.apache.flink.api.java.tuple.Tuple6;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.BroadcastConnectedStream;
import org.apache.flink.streaming.api.datastream.BroadcastStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.BroadcastProcessFunction;
import org.apache.flink.streaming.api.functions.source.RichSourceFunction;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.util.Collector;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;

/**
 * Author xiaoma
 * Date 2020/10/20 16:05
 * Desc
 * 需求:
 * 使用Flink的BroadcastState来完成
 * 事件流和配置流(需要广播为State)的关联,并实现配置的动态更新!
 */
public class BroadcastStateConfigUpdate {
    public static void main(String[] args) throws Exception{
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.source
        //-1.构建实时的自定义随机数据事件流-数据源源不断产生,量会很大
        //<userID, eventTime, eventType, productID>
        DataStreamSource<Tuple4<String, String, String, Integer>> eventDS = env.addSource(new MySource());

        //-2.构建配置流-从MySQL定期查询最新的,数据量较小
        //<用户id,<姓名,年龄>>
        DataStreamSource<Map<String, Tuple2<String, Integer>>> configDS = env.addSource(new MySQLSource());

        //3.transformation
        //-1.定义状态描述器-准备将配置流作为状态广播
        MapStateDescriptor<Void, Map<String, Tuple2<String, Integer>>> descriptor =
                new MapStateDescriptor<>("config", Types.VOID, Types.MAP(Types.STRING, Types.TUPLE(Types.STRING, Types.INT)));
        //-2.将配置流根据状态描述器广播出去,变成广播状态流
        BroadcastStream<Map<String, Tuple2<String, Integer>>> broadcastDS = configDS.broadcast(descriptor);

        //-3.将事件流和广播流进行连接
        BroadcastConnectedStream<Tuple4<String, String, String, Integer>, Map<String, Tuple2<String, Integer>>> connectDS =eventDS.connect(broadcastDS);
        //-4.处理连接后的流-根据配置流补全事件流中的用户的信息
        SingleOutputStreamOperator<Tuple6<String, String, String, Integer, String, Integer>> result = connectDS
                //BroadcastProcessFunction<IN1, IN2, OUT>
                .process(new BroadcastProcessFunction<
                //<userID, eventTime, eventType, productID> //事件流
                Tuple4<String, String, String, Integer>,
                //<用户id,<姓名,年龄>> //广播流
                Map<String, Tuple2<String, Integer>>,
                //<用户id，eventTime，eventType，productID，姓名，年龄> //需要收集的数据
                Tuple6<String, String, String, Integer, String, Integer>>() {

            //处理事件流中的元素
            @Override
            public void processElement(Tuple4<String, String, String, Integer> value, ReadOnlyContext ctx, Collector<Tuple6<String, String, String, Integer, String, Integer>> out) throws Exception {
                //取出事件流中的userId
                String userId = value.f0;
                //根据状态描述器获取广播状态
                ReadOnlyBroadcastState<Void, Map<String, Tuple2<String, Integer>>> broadcastState = ctx.getBroadcastState(descriptor);
                if (broadcastState != null) {
                    //取出广播状态中的map<用户id,<姓名,年龄>>
                    Map<String, Tuple2<String, Integer>> map = broadcastState.get(null);
                    if (map != null) {
                        //通过userId取map中的<姓名,年龄>
                        Tuple2<String, Integer> tuple2 = map.get(userId);
                        //取出tuple2中的姓名和年龄
                        String userName = tuple2.f0;
                        Integer userAge = tuple2.f1;
                        out.collect(Tuple6.of(userId, value.f1, value.f2, value.f3, userName, userAge));
                    }
                }
            }

            //处理广播流中的元素
            @Override
            public void processBroadcastElement(Map<String, Tuple2<String, Integer>> value, Context ctx, Collector<Tuple6<String, String, String, Integer, String, Integer>> out) throws Exception {
                //value就是MySQLSource中每隔一段时间获取到的最新的map数据
                //先根据状态描述器获取历史的广播状态
                BroadcastState<Void, Map<String, Tuple2<String, Integer>>> broadcastState = ctx.getBroadcastState(descriptor);
                //再清空历史状态数据
                broadcastState.clear();
                //最后将最新的广播流数据放到state中（更新状态数据）
                broadcastState.put(null, value);
            }
        });
        //4.sink
        result.print();
        //5.execute
        env.execute();
    }

    /**
     * <userID, eventTime, eventType, productID>
     */
    public static class MySource implements SourceFunction<Tuple4<String, String, String, Integer>>{
        private boolean isRunning = true;
        @Override
        public void run(SourceContext<Tuple4<String, String, String, Integer>> ctx) throws Exception {
            Random random = new Random();
            SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            while (isRunning){
                int id = random.nextInt(4) + 1;
                String user_id = "user_" + id;
                String eventTime = df.format(new Date());
                String eventType = "tpye_" + random.nextInt(3);
                int productId = random.nextInt(4);
                ctx.collect(Tuple4.of(user_id,eventTime,eventType,productId));
                Thread.sleep(500);
            }
        }

        @Override
        public void cancel() {
            isRunning = false;
        }
    }
    /**
     * <用户id,<姓名,年龄>>
     */
    public static class MySQLSource extends RichSourceFunction<Map<String, Tuple2<String, Integer>>> {
        private boolean flag = true;
        private Connection conn = null;
        private PreparedStatement ps = null;
        private ResultSet rs = null;

        @Override
        public void open(Configuration parameters) throws Exception {
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/bigdata", "root", "root");
            String sql = "select `userID`, `userName`, `userAge` from `user_info`";
            ps = conn.prepareStatement(sql);
        }
        @Override
        public void run(SourceContext<Map<String, Tuple2<String, Integer>>> ctx) throws Exception {
            while (flag){
                Map<String, Tuple2<String, Integer>> map = new HashMap<>();
                ResultSet rs = ps.executeQuery();
                while (rs.next()){
                    String userID = rs.getString("userID");
                    String userName = rs.getString("userName");
                    int userAge = rs.getInt("userAge");
                    //Map<String, Tuple2<String, Integer>>
                    map.put(userID,Tuple2.of(userName,userAge));
                }
                ctx.collect(map);
                Thread.sleep(5000);//每隔5s更新一下用户的配置信息!
            }
        }
        @Override
        public void cancel() {
            flag = false;
        }
        @Override
        public void close() throws Exception {
            if (conn != null) conn.close();
            if (ps != null) ps.close();
            if (rs != null) rs.close();
        }
    }
}
```

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

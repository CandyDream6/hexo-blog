---
title: Druid 高性能的实时分析数据库
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(59).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(59).jpg
tags:
  - 大数据
  - Druid
category:
  - - 编程
    - 大数据
 
date: 2020-04-04 00:34:27
---

云原生、流原生的分析型数据库。

可轻松与现有的数据管道进行集成。

较传统方案提升近百倍的效率。

解锁了一种新型的工作流程。

可部署在AWS/GCP/Azure,混合云,Kubernetes, 以及裸机上。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 Druid介绍

### 1.1 RDBMS的劣势

> 关系型数据库如mysql在实际开发中经常使用，那么关系型数据库有哪些优劣势，我们可以做以下分析：

> `关系型数据库的优点`：
>> 1.基于ACID，支持事务，适合于对安全性和一致性要求高的的数据访问
> 
>> 2.可以进行Join等复杂查询，处理复杂业务逻辑，比如：报表
> 
>> 3.使用方便，通用的SQL语言使得操作关系型数据库非常方便

> `关系型数据库的劣势`：
>> 1.不擅长大量数据的写入处理
> 
>> 2.每个字段都会占用一定的磁盘空间，不具有稀疏性
> 
>> 3.高并发下性能、吞吐量较低
> 
>> 4.扩展性不如非关系型数据库方便

> 根据上面的总结，随着每日增量数据的累加，短期来看mysql数据库是能够承载一定程度的数据量的，但是长期来看，mysql数据库的存储压力将不堪重负且查询性能也会急剧下降。因此，我们需要寻找mysql数据库的替代方案，这里我们选择了Apache Druid 实时数据库。

### 1.2 Hadoop的劣势

> 基于 Hadoop 的大数据平台，有如下一些问题：
>> `(1)无法保障查询性能`
>>> 对于Hadoop使用的MapReduce批处理框架，数据何时能够查询没有性能保证
>  
>> `(2)随机IO问题`
>>> HDFS以集群硬盘作为存储资源池的分布式文件系统;
>>> 在海量数据的处理过程中，会引起大量的读写操作，随机IO是高并发场景下的性能瓶颈
> 
>> `(3)数据查询效率问题`
>>> HDFS对于数据分析以及数据的即席查询，HDFS并不是最优的选择。

> 传统的Hadoop大数据处理架构更倾向于一种“后台批处理的数据仓库系统”，其作为海量历史数据保存、冷数据分析，确实是一个优秀的通用解决方案，但问题主要体现为：
>> 1.无法保证高并发环境下海量数据的查询分析性能
> 
>> 2.无法实现海量实时数据的查询分析与可视化

### 1.3 Apache Druid概述

> 官网：http://druid.apache.org/

> 中文文档：http://www.apache-druid.cn/

#### 1.3.1 Druid是什么

> `Apache Druid` 是一个`高性能实时分析数据库`。它是为大型数据集上实时探索查询的引擎，提供专为 OLAP 设计的开源分析数据存储系统

> `OLAP（Online Analytical Process）`，`联机分析处理`，以多维度的方式分析数据，一般带有主观的查询需求，多应用在数据仓库。

> `OLTP（Online Transaction Process）`，`联机事务处理`，侧重于数据库的增删查改等常用业务操作。

> Apache Druid是由一个名为 MetaMarket 的公司开发的；2011年，MetaMarket 开始研发自己的"轮子"Druid，将Druid定义为“`开源、分布式、面向列式存储的实时分析数据存储系统`”。

> 注意：阿里巴巴也曾创建过一个开源项目叫 Druid （简称阿里 Druid），它是一个数据库连接池项目。阿里 Druid 和 我们要讨论的 Druid 没有任何关系，它们解决完全不同的问题

#### 1.3.2 Druid要解决的“痛点”

> 1.在高并发环境下，保证海量数据查询分析性能

> 2.同时提供海量实时数据的查询、分析与可视化功能

#### 1.3.3 Druid主要特性

> Druid是面向海量数据的、用于实时查询与分析的OLAP存储系统，主要特性：
>> `1.为分析而设计`——Druid 是为 OLAP 工作流的探索性分析而构建。采用了列式存储、倒排索引、位图索引等关键技术，它支持各种 filter、aggregator 和多维分析等查询类型，并为添加新功能提供了一个框架。用户已经利用 Druid 的基础设施开发了高级K查询和直方图功能。
>
>> `2.交互式查询`——Druid 的低延迟数据摄取架构允许事件在它们创建后毫秒内查询，因为 Druid 的查询延时通过只读取和扫描优必要的元素被优化。Aggregate 和 filter 没有坐等结果。
>
>> `3.实时流数据分析`——传统分析型数据库采用的批量导入数据，进行分析的方式，Druid提供了实时流数据分析，以及高效实时写入，并能做到亚秒级内的可视化
>
>> `4.高可用性`——Druid 是用来支持需要一直在线的 SaaS 的实现。Druid工作节点功能单一，不相互依赖，数据在系统更新时依然可用、可查询。规模的扩大和缩小不会造成数据丢失。
>
>> `5.可伸缩`——现有的 Druid 部署每天处理数十亿事件和 TB 级数据。Druid 被设计成 PB 级别，Druid集群在管理、容错、灾备、扩容都很容易
>
>> `6.友好的界面和丰富的数据分析功能`

> Druid提供了友好的可视化界面并支持SQL查询语言、REST查询接口

#### 1.3.4 Druid的典型应用场景

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/1.png)

> Apache Druid是一个实时分析型数据库，旨在对大型数据集进行快速的查询分析（"OLAP"查询)。Druid最常被当做数据库来用以支持实时摄取、高性能查询和高稳定运行的应用场景，同时，Druid也通常被用来助力分析型应用的图形化界面，或者当做需要快速聚合的高并发后端API，Druid最适合应用于面向事件类型的数据。

> `Druid通常应用于以下场景`：
>> 点击流分析（Web端和移动端）
> 
>> 网络监测分析（网络性能监控）
> 
>> 服务指标存储
> 
>> 供应链分析（制造类指标）
> 
>> 应用性能指标分析
> 
>> 数字广告分析
> 
>> 商务智能 / OLAP

> `如果您的使用场景符合以下的几个特征，那么Druid是一个非常不错的选择`：
>> 数据插入频率比较高，但较少更新数据
> 
>> 大多数查询场景为聚合查询和分组查询（GroupBy），同时还有一定得检索与扫描查询
> 
>> 将数据查询延迟目标定位100毫秒到几秒钟之间
> 
>> 数据具有时间属性（Druid针对时间做了优化和设计）
> 
>> 在多表场景下，每次查询仅命中一个大的分布式表，查询又可能命中多个较小的lookup表
> 
>> 场景中包含高基维度数据列（例如URL，用户ID等），并且需要对其进行快速计数和排序
> 
>> 需要从Kafka、HDFS、对象存储（如Amazon S3）中加载数据

> `如果您的使用场景符合以下特征，那么使用Druid可能是一个不好的选择`：
>> 根据主键对现有数据进行低延迟更新操作。Druid支持流式插入，但不支持流式更新（更新操作是通过后台批处理作业完成）
> 
>> 延迟不重要的离线数据系统
> 
>> 场景中包括大连接（将一个大事实表连接到另一个大事实表），并且可以接受花费很长时间来完成这些查询

#### 1.3.5 Druid对比其他OLAP

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/2.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/3.png)

> `Druid vs. Elasticsearch`
>> Druid在导入过程会对原始数据进行Rollup，而ES会保存原始数据
> 
>> Druid专注于OLAP，针对数据导入以及快速聚合操作做了优化
> 
>> Druid不支持全文检索

> Druid vs. Key/Value Stores (HBase/Cassandra/OpenTSDB)
>> Druid采用列式存储，使用倒排和bitmap索引，可以做到快速扫描相应的列

> `Druid vs. Spark`
>> Spark SQL的响应还不做到亚秒
> 
>> Druid可以做到超低的响应时间，例如亚秒，而且高并发面向用户的应用。

> `Druid vs SQL-on-Hadoop` (Impala/Drill/Spark SQL/Presto)
>> Driud查询速度更快
> 
>> 数据导入，Druid支持实时导入，SQL-on-Hadoop一般将数据存储在Hdfs上，Hdfs的写入速度有可能成为瓶颈
> 
>> SQL支持，Druid也支持SQL，但Druid不支持Join操作

> `Druid vs. Kylin`
>> Kylin不支持实时查询，Druid支持
> 
>> Kylin支持表连接（Join），Druid不支持

#### 1.3.6 国内哪些公司在使用Druid

> `1.腾讯`
>> 腾讯企点采用Druid用于分析大量的用户行为，帮助提升客户价值

> `2.阿里巴巴`
>> 阿里搜索组使用Druid的实时分析功能用于获取用户交互行为

> `3.新浪微博`
>> 新浪广告团队使用Druid构建数据洞察系统的实时分析部分，每天处理数十亿的消息

> `4.小米`
>> Druid用于小米统计的后台数据收集和分析
> 
>> 也用于广告平台的数据分析

> `5.滴滴打车`
>> Druid是滴滴实时大数据处理的核心模块，用于滴滴实时监控系统，支持数百个关键业务指标
> 
>> 通过Druid，滴滴能够快速得到各种实时的数据洞察

>` 6.优酷土豆`
>> Druid用于其广告的数据处理和分析

## 2 Druid原理

### 2.1 Druid架构设计

> Druid是微服务架构，可以理解为一个拆解成多个服务的数据库。Druid的每一个核心服务都可以单独部署或联合部署在商业硬件上。独立失败而不影响其他服务的运行。

#### 2.1.1 架构图解

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/4.png)

> Druid有若干不同类型的进程，简单描述如下：
>> `Overlord进程`: 控制数据摄取负载的分配
>> `Coordinator进程`: 管理集群中数据的可用性
> 
>
>> `Broker进程`: 处理来自外部客户端的查询请求
>> `Router进程`: 是一个可选进程，可以将请求路由到Brokers、Coordinators和Overlords
> 
>
>> `Historical进程`: 存储可查询的数据
>> `MiddleManager进程`: 负责摄取数据

> Druid进程可以按照您喜欢的方式部署
>> 但是为了便于部署，我们建议将它们组织成三种服务器类型：Master、Query和Data。
> 
>> `Master`: 运行Coordinator和Overlord进程，管理数据可用性和摄取
> 
>> `Query`: 运行Broker和可选的Router进程，处理来自外部客户端的请求
> 
>> `Data`: 运行Historical和MiddleManager进程，执行摄取负载和存储所有可查询的数据

#### 2.1.2 名词解释

##### 2.1.2.1 Indexing Service

> Indexing Service索引服务是数据摄入创建和销毁Segment的重要方式，Druid提供一组支持索引服务(Indexing Service)的组件，即Overlord和MiddleManager节点。

> 索引服务采用的是主从架构，Overlord为主节点，MiddleManager是从节点

> 索引服务架构图如下图所示：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/5.png)

> 索引服务由三部分组件组成：
>> `Overlord组件` ：分配任务给MiddleManager
> 
>> `MiddleManager组件` ：用于管理Peon的
> 
>> `Peon组件` ：用于执行任务

> 部署：
>> 1.`MiddleManager和Overlord组件`可以部署在相同节点也可以跨节点部署
>> 2.`Peon和MiddleManager`是部署在同一个节点上的

> 索引服务架构和Yarn的架构很像：
>>` Overlaod => ResourceManager`，负责集群资源管理和任务分配
>> `MiddleManager => NodeManager`，负责接受任务和管理本节点的资源
>> `Peon => Container`，执行节点上具体的任务

##### 2.1.2.2 Overlord节点

> Overlord是索引服务的主节点，对外负责接受索引任务，对内负责将任务分解并下发给MiddleManager

> Overlord有两种运行模式：
>> `1.本地模式(Local Mode)`：默认模式
>> 
>>> 本地模式下的Overlord不仅负责任务协调工作，还会负责启动一些peon来完成具体的任务。
>
>> `2.远程模式(Remote Mode)`
>> 
>>> 该模式下，Overlord和MiddleManager运行在不同的节点上，它仅负责任务的协调工作，不负责完成具体的任务。
> 
>> `3.UI客户端`
>> 
>>> Overlord提供了一个UI客户端，可以用于查看任务、运行任务和终止任务等
>> 
>>> UI访问地址：http://node01:8090/console.html
>> 
>>> Overlord提供了RESETful的访问形式，所以客户端可以通过HTTP POST形式向请求节点提交任务：
>>>  
>>>>提交任务 ： http://node01:8090/druid/indexer/v1/task
>>> 
>>>>杀死任务 ： http://node01:8090/druid/indexer/v1/task/{task_id}/shutdown

##### 2.1.2.3 MiddleManager节点

> MiddleManager是执行任务的工作节点

> MiddleManager会将任务单独发给每个单独JVM运行的Peon

> 每个Peon一次只能运行一个任务

##### 2.1.2.4 Coordinator节点

> `Coordinator`是`Historical`的mater节点，主要负责管理和`分发Segment`

> 具体工作就是：
>> 1.告知Historical加载或删除Segment
>> 2.管理Segment副本以及负载Segment在Historical上的均衡

> `Coordinator`是定期运行的，通过Zookeeper获取当前集群状态，通过评估集群状态来进行均衡负载Segment。

> `Coordinator`连接数据库(MetaStore)，获取Segment信息和规则(Rule)，Coordinator根据数据库中表的数据来进行集群 segment 管理

> `Coordinator`提供了一UI界面，用于显示集群信息和规则配置：http://node1:8081/index.html#/

##### 2.1.2.5 Historical节点

> 1.Historical节点负责管理历史Segment

> 2.Historical节点通过Zookeeper监听指定的路径来发现是否有新的Segment需要加载

> 3.Historical节点收到有新的Segment时候，就会检测本地cache和磁盘，查看是否有该Segment信息。如果没有Historical节点会从Zookeeper中拉取该Segment相关的信息，然后进行下载

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/6.png)

##### 2.1.2.6 Broker节点

> Broker节点负责转发Client查询请求的

> Broker通过zookeeper能够知道哪个Segment在哪些节点上，将查询转发给相应节点

> 所有节点返回数据后，Broker会将所有节点的数据进行合并，然后返回给Client

### 2.2 Druid数据存储

> Druid提供对大数据集的实时摄入和高效复杂查询的性能，主要原因：基于Segment的数据存储结构

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/7.png)

#### 2.2.1 Chunk

> Druid中的数据存储在被称为DataSource中，DataSource类似RDMS中的table

> 每个DataSource按照时间划分，每个时间范围称为一个`Chunk`（(比如按天分区，则一个chunk为一天)）

> Druid处理的是事件数据，每条数据都会带有一个时间戳，可以使用时间进行分区，上图指定了分区粒度为天，那么每天的数据都会被单独存储和查询。

#### 2.2.2 Segment

> 1.在chunk中数据被分为一个或多个Segment,Segment是数据实际存储结构，Datasource、Chunk只是一个逻辑概念

> 2.每个Segment都是一个单独的文件，通常包含几百万行数据

> 3.Segment是数据存储、复制、均衡和计算的基本单元；

> 4.Segment是按照时间组织成的chunk，所以在按照时间查询数据时，效率非常高。

> 5.Segment具备不可变性，一个Segment一旦创建完成后(MiddleManager节点发布后)就无法被修改只能通过生成一个新的Segment来代替旧版本的Segment

> Segment内部结构-了解
>> Druid采用列式存储，每列数据都是在独立的结构中存储
> 
>> Segment中的`数据类型`主要分为三种：
>>> `1.时间戳`
>> 
>>> `2.维度列`
>>  
>>> `3.指标列`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/8.png)

> `1.时间戳列和指标列`
>> Druid采用LZ4压缩每列的整数或浮点数
> 
>> 收到查询请求后，会拉出所需的行数据(对于不需要的列不会拉出来)，并且对其进行解压缩。

> `2.维度列`
>> 维度列需要支持filter和group by
> 
>> Druid使用了字典编码(Dictionary Encoding)和位图索引(Bitmap Index)来存储每个维度列。
> 
>> `每个维度列需要三个数据结构：`
> 
>>> 1.需要一个字典数据结构，将维度值映射成一个整数ID
>>  
>>> 2.使用上面的字典编码，将该列所有维度值放在一个列表中
>> 
>>> 3.对于列中不同的值，使用bitmap数据结构标识哪些行包含这些值。
>> 
>> `Druid针对维度列之所以使用这三个数据结构，是因为：`
>> 
>>> 1.使用字典将字符串映射成整数ID，可以紧凑的表示维度数据
>> 
>>> 2.使用Bitmap位图索引可以执行快速过滤操作
>>  
>>> 找到符合条件的行号，以减少读取的数据量
>> 
>>> Bitmap可以快速执行AND和OR操作

#### 2.2.3 roll-up预聚合

> 1.Druid通过一个roll-up的处理，将原始数据在注入的时候就进行汇总处理；

> 2.roll-up可以压缩我们需要保存的数据量；

> 3.Druid会把选定的相同维度的数据进行聚合操作，可减少存储的大小；

> 4.Druid可以通过 queryGranularity 来控制注入数据的粒度。 最小的queryGranularity 是 millisecond(毫秒级)。

> `聚合案例1:`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/9.png)

> `聚合案例2:`
>> Roll-up聚合前：

time | app_key | area | value
---|---|---|---
2019-10-05 | 10:00:00 | area_key1 | Beijing | 1
2019-10-05 | 10:30:00 | area_key1 | Beijing | 1
2019-10-05 | 11:00:00 | area_key1 | Beijing | 1
2019-10-05 | 11:00:00 | area_key2 | Shanghai | 2

> Roll-up聚合后：

time | app_key | area | value
---|---|---|---
2019-10-05 | area_key1 | Beijing | 3
2019-10-05 | area_key2 | Shanghai | 2

#### 2.2.4 位图索引

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/10.png)

> 以下为一个DataSource（表）中存储的数据。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/11.png)

> 第一列为时间，`Appkey和area都是dimension维度列`，`value为metric指标列`；

> Druid会在导入阶段自动对数据进行Rollup，将维度相同组合的数据进行聚合处理；

> 按天聚合后的数据如下：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/12.png)

> Druid通过建立位图索引，来实现快速进行数据查找。

> 索引如下所示：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/13.png)

> 索引位图可以看作是`HashMap<String, Bitmap>`
>> `key`就是维度的取值；
> 
>> `value`就是聚合后的表中对应的行是否有该维度的值；

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/14.png)

> 以SQL查询为例：
>> `1）boolean条件查询`

```
select
sum(value)
from AD_areauser
where
time=’2018-10-11’ and
Appkey in (‘appkey1’,’appkey2’) and
area=’北京’
```

> `执行过程分析`：
>> 1.根据时间段定位到segment
> 
>> 2.Appkey in ('appkey1', 'appkey2') and area=’北京’查到各自的bitmap
>>   --> (appkey1 (1000)  or  appkey2 (0110)) and (北京 (1100)   
>>   --> (1000 or 0110 )and 1100
>>   --> 1110 and 1100
>>   --> 1100
   符合条件的列为第一行和第二行，这两行的 sum(value) 的和为26.


> `2）group by 查询`

```
select
area,
sum(value)
from AD_areauser
where
time=’2018-10-11’and
Appkey in (‘appkey1’,’appkey2’)
group by area
```

> 该查询与上面的查询不同之处在于将符合条件的列
>> 1.appkey1(1000) or appkey2(0110) = (1110)
>> 2.将第一行、第二行、第三行取出来
>> 3.在内存中做分组聚合。结果为：北京：26， 上海：13.

> 本次项目使用Druid来进行实时OLAP分析，通过Flink预处理Kafka的数据，再将预处理后的数据下沉到Kafka中。再基于Druid进行数据分析。

## 3 Druid启动

### 3.1 启动Druid-imply

```
cd /export/servers/imply-3.0.4/
nohup bin/supervise -c conf/supervise/quickstart.conf > quickstart.log &

或者( --daemonize 代表后台启动)
/export/servers/imply-3.0.4/bin/supervise -c /export/servers/imply-3.0.4/conf/supervise/quickstart.conf --daemonize
```

> 成功之后可以看到6个Main进程

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/15.png)

### 3.2 停止Druid-imply

```
/export/servers/imply-3.0.4/bin/service --down
```

### 3.3 访问WebUI

组件名 | URL
---|---
`broker`:处理来自外部客户端的查询请求 | http://node01:8888
`coordinator`:管理集群中数据的可用性<br/>`overlord`:控制数据摄取负载的分配	 | http://node01:8081/index.html
`middleManager`:摄取数据<br/>`historical`:存储可查询的数据 | http://node01:8090/console.html

## 4 Druid入门案例

> `需求`：
>> 使用Druid 分析 2019年5月8日 按照商品分类、商品区域的产品订单总额

> `实现步骤`：
>> 将测试数据源4.资料\Druid\0.druid测试数据源\0.商品订单数据\order.json

```
{"timestamp":"2019-05-08T01:03.00Z","category":"手机","areaName":"北京","money":"1450"}
{"timestamp":"2019-05-08T01:01.00Z","category":"手机","areaName":"北京","money":"1450"}
{"timestamp":"2019-05-08T01:03.00Z","category":"手机","areaName":"北京","money":"8412"}
{"timestamp":"2019-05-08T05:01.00Z","category":"电脑","areaName":"上海","money":"1513"}
{"timestamp":"2019-05-08T01:03.00Z","category":"家电","areaName":"北京","money":"1550"}
{"timestamp":"2019-05-08T01:01.00Z","category":"电脑","areaName":"杭州","money":"1550"}
{"timestamp":"2019-05-08T01:03.00Z","category":"电脑","areaName":"北京","money":"5611"}
{"timestamp":"2019-05-08T03:01.00Z","category":"家电","areaName":"北京","money":"4410"}
{"timestamp":"2019-05-08T01:03.00Z","category":"家具","areaName":"郑州","money":"1120"}
{"timestamp":"2019-05-08T01:01.00Z","category":"家具","areaName":"北京","money":"6661"}
{"timestamp":"2019-05-08T05:03.00Z","category":"家具","areaName":"杭州","money":"1230"}
{"timestamp":"2019-05-08T01:01.00Z","category":"书籍","areaName":"北京","money":"5550"}
{"timestamp":"2019-05-08T01:03.00Z","category":"书籍","areaName":"北京","money":"5550"}
{"timestamp":"2019-05-08T01:01.00Z","category":"电脑","areaName":"北京","money":"1261"}
{"timestamp":"2019-05-08T03:03.00Z","category":"电脑","areaName":"杭州","money":"6660"}
{"timestamp":"2019-05-08T01:01.00Z","category":"电脑","areaName":"天津","money":"6660"}
{"timestamp":"2019-05-08T01:03.00Z","category":"书籍","areaName":"北京","money":"9000"}
{"timestamp":"2019-05-08T05:01.00Z","category":"书籍","areaName":"北京","money":"1230"}
{"timestamp":"2019-05-08T01:03.00Z","category":"电脑","areaName":"杭州","money":"5551"}
{"timestamp":"2019-05-08T01:01.00Z","category":"电脑","areaName":"北京","money":"2450"}
{"timestamp":"2019-05-08T01:03.00Z","category":"食品","areaName":"北京","money":"5520"}
{"timestamp":"2019-05-08T01:01.00Z","category":"食品","areaName":"北京","money":"6650"}
{"timestamp":"2019-05-08T01:03.00Z","category":"服饰","areaName":"杭州","money":"1240"}
{"timestamp":"2019-05-08T01:01.00Z","category":"食品","areaName":"天津","money":"5600"}
{"timestamp":"2019-05-08T01:03.00Z","category":"食品","areaName":"北京","money":"7801"}
{"timestamp":"2019-05-08T01:01.00Z","category":"服饰","areaName":"北京","money":"9000"}
{"timestamp":"2019-05-08T01:03.00Z","category":"服饰","areaName":"杭州","money":"5600"}
{"timestamp":"2019-05-08T01:01.00Z","category":"食品","areaName":"北京","money":"8000"}
{"timestamp":"2019-05-08T02:03.00Z","category":"服饰","areaName":"北京","money":"7000"}
```

>> 上传到服务器的 /export/servers/tmp/druid 目录中

> 1、上传测试数据到每台Linux服务器

```
mkdir -p /export/servers/tmp/druid
```

> 2、摄取数据到Druid中,可以按照下面步骤操作,也可以直接导入”4.资料\Postman”中的.json请求配置文件
>> 2.1 打开postman，请求地址设置为
> 
>> ```http://node01:8090/druid/indexer/v1/task```
>
>> 2.2 请求方式选择为POST
> 
>> 2.3 body > raw > JSON(application/json)
> 
>> 2.4 将资料中的index_order.json文件 粘贴到 postman中

```json
{
  "spec": {
  	  "dataSchema": {
	    "dataSource": "demo_order",
	    "parser": {
	      "type": "String",
	      "parseSpec": {
	        "format": "json",
	        "dimensionsSpec": {
	          "dimensions": [
	            "category",
	            "areaName"
	          ]
	        },
	        "timestampSpec": {
	          "column": "timestamp",
	          "format": "auto"
	        }
	      }
	    },
	    "metricsSpec": [
	      {
	        "type": "count",
	        "name": "count"
	      },
	      {
	        "type": "longSum",
	        "name": "money",
	        "fieldName": "money",
	        "expression": null
	      }
	    ],
	    "granularitySpec": {
	      "type": "uniform",
	      "segmentGranularity": "DAY",
	      "queryGranularity": "HOUR",
	      "rollup": true,
	      "intervals": [
	        "2019-05-06T00:00:00.000Z/2019-05-09T00:00:00.000Z"
	      ]
	    },
	    "transformSpec": {
	      "filter": null,
	      "transforms": []
	    }
	  },
	  "ioConfig": {
	    "type": "index",
	    "firehose": {
	      "type": "local",
	      "baseDir": "/export/servers/tmp/druid",
	      "filter": "order.json",
	      "parser": null
	    },
	    "appendToExisting": false
	  },
	  "tuningConfig": {
	    "type": "index",
	    "maxRowsPerSegment": null,
	    "maxRowsInMemory": 1000000,
	    "maxBytesInMemory": 0,
	    "maxTotalRows": null,
	    "numShards": null,
	    "partitionDimensions": [],
	    "indexSpec": {
	      "bitmap": {
	        "type": "concise"
	      },
	      "dimensionCompression": "lz4",
	      "metricCompression": "lz4",
	      "longEncoding": "longs"
	    },
	    "maxPendingPersists": 0,
	    "buildV9Directly": true,
	    "forceGuaranteedRollup": false,
	    "reportParseExceptions": false,
	    "pushTimeout": 0,
	    "segmentWriteOutMediumFactory": null,
	    "logParseExceptions": false,
	    "maxParseExceptions": 2147483647,
	    "maxSavedParseExceptions": 0
	  }
  },
  "type": "index"
}
```
 
>> 2.5 发送请求

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/16.png)

> 3、稍等一会之后执行 SQL 查询
>> 3.1 打开 Druid 控制台 http://node01:8888
>> 3.2 打开 Query 选项卡，执行以下SQL实现 按照商品分类、商品区域的产品订单总额

```
-- 分析2019年5月8日，按照商品分类、商品区域的产品订单总额
SELECT
category,
areaName,
SUM(money) AS total_money,
SUM("count") AS total_count
FROM "demo_order"
WHERE TIME_FORMAT("__time", 'yyyyMMdd') = '20190508'
GROUP BY category, areaName
```

## 5 Druid数据摄取

> Druid支持流式和批量两种方式的数据摄入，针对不同类型的数据，Druid将外部数据源分为两种形式：
> `1.流式数据源`
>> 指的是持续不断地生产数据的数据源。例如：消息队列、日志、文件等
> 
> `2.静态数据源`
>> 指的是数据已经生产完毕，不会有新数据产生的数据源。例如：文件系统的文件

### 5.1 批量（离线）数据摄取

> 流式数据可以通过两种方式来摄入：本地文件和远程文件

#### 5.1.1 摄取本地文件

> `需求：`
>> 将摄取服务器本地上的 ad_event.json 数据到Druid中

> `操作步骤：`

> 1、在某一个服务器节点中创建 /export/servers/tmp/druid 文件夹

> 2、上传数据文件和摄取配置文件
>> 将资料：”druid测试数据源\广告点击数据中的 ad_event.json“ 上传到 /export/servers/tmp/druid目录中

```json
{"timestamp":"2018-12-01T01:03.00Z","city":"beijing","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:01.00Z","city":"guangzhou","platform":"pc","click":"1"}
{"timestamp":"2018-12-01T01:03.00Z","city":"beijing","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T05:01.00Z","city":"beijing","platform":"pc","click":"1"}
{"timestamp":"2018-12-01T01:03.00Z","city":"guangzhou","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:01.00Z","city":"beijing","platform":"mobile","click":"0"}
{"timestamp":"2018-12-01T01:03.00Z","city":"beijing","platform":"pc","click":"1"}
{"timestamp":"2018-12-01T03:01.00Z","city":"shanghai","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:03.00Z","city":"beijing","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:01.00Z","city":"guangzhou","platform":"pc","click":"1"}
{"timestamp":"2018-12-01T05:03.00Z","city":"beijing","platform":"mobile","click":"0"}
{"timestamp":"2018-12-01T01:01.00Z","city":"shanghai","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:03.00Z","city":"beijing","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:01.00Z","city":"guangzhou","platform":"pc","click":"1"}
{"timestamp":"2018-12-01T03:03.00Z","city":"beijing","platform":"mobile","click":"0"}
{"timestamp":"2018-12-01T01:01.00Z","city":"shanghai","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:03.00Z","city":"beijing","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T05:01.00Z","city":"beijing","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:03.00Z","city":"beijing","platform":"mobile","click":"1"}
{"timestamp":"2018-12-01T01:01.00Z","city":"shanghai","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:03.00Z","city":"beijing","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:01.00Z","city":"guangzhou","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:03.00Z","city":"beijing","platform":"mobile","click":"0"}
{"timestamp":"2018-12-01T01:01.00Z","city":"shanghai","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:03.00Z","city":"beijing","platform":"pc","click":"1"}
{"timestamp":"2018-12-01T01:01.00Z","city":"guangzhou","platform":"pc","click":"0"}
{"timestamp":"2018-12-01T01:03.00Z","city":"beijing","platform":"mobile","click":"0"}
{"timestamp":"2018-12-02T01:01.00Z","city":"shanghai","platform":"pc","click":"0"}
{"timestamp":"2018-12-02T02:03.00Z","city":"beijing","platform":"pc","click":"0"}
```

> 3、使用postman提交本地批量索引任务

>> 将index_ad_event.json文件中的内容拷贝到 postman 中

```json
{
  "spec": {
  	  "dataSchema": {
	    "dataSource": "ad_event",
	    "parser": {
	      "type": "String",
	      "parseSpec": {
	        "format": "json",
	        "dimensionsSpec": {
	          "dimensions": [
	            "city",
	            "platform"
	          ]
	        },
	        "timestampSpec": {
	          "column": "timestamp",
	          "format": "auto"
	        }
	      }
	    },
	    "metricsSpec": [
	      {
	        "type": "count",
	        "name": "count"
	      },
	      {
	        "type": "longSum",
	        "name": "click",
	        "fieldName": "click",
	        "expression": null
	      }
	    ],
	    "granularitySpec": {
	      "type": "uniform",
	      "segmentGranularity": "DAY",
	      "queryGranularity": "HOUR",
	      "rollup": true,
	      "intervals": [
	        "2018-12-01T00:00:00.000Z/2018-12-03T00:00:00.000Z"
	      ]
	    },
	    "transformSpec": {
	      "filter": null,
	      "transforms": []
	    }
	  },
	  "ioConfig": {
	    "type": "index",
	    "firehose": {
	      "type": "local",
	      "baseDir": "/export/servers/tmp/druid",
	      "filter": "ad_event.json",
	      "parser": null
	    },
	    "appendToExisting": false
	  },
	  "tuningConfig": {
	    "type": "index",
	    "maxRowsPerSegment": null,
	    "maxRowsInMemory": 1000000,
	    "maxBytesInMemory": 0,
	    "maxTotalRows": null,
	    "numShards": null,
	    "partitionDimensions": [],
	    "indexSpec": {
	      "bitmap": {
	        "type": "concise"
	      },
	      "dimensionCompression": "lz4",
	      "metricCompression": "lz4",
	      "longEncoding": "longs"
	    },
	    "maxPendingPersists": 0,
	    "buildV9Directly": true,
	    "forceGuaranteedRollup": false,
	    "reportParseExceptions": false,
	    "pushTimeout": 0,
	    "segmentWriteOutMediumFactory": null,
	    "logParseExceptions": false,
	    "maxParseExceptions": 2147483647,
	    "maxSavedParseExceptions": 0
	  }
  },
  "type": "index"
}
```

>> 发送post请求到

```
http://node01:8090/druid/indexer/v1/task
```
> 4、可以在Overlord（http://node01:8090/console.html） 中查看到任务信息

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/17.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/18.png)

> 5、在 http://node01:8888中测试查询数据

```
SELECT
*
FROM "ad_event"
LIMIT 1
```

#### 5.1.2 摄取HDFS文件

> Druid支持加载HDFS上的数据。它会使用 HadoopDruidIndexer 加载批量数据，将数据生成 segments 文件，存放在HDFS上，再从HDFS下载 segments 文件到本地。然后遍可从Druid中查询数据。

> `需求：`
>> 摄取HDFS上的wikiticker-2015-09-12-sampled.json文件到Druid中

> 操作步骤：

> 1、启动HDFS集群、YARN集群

> 2、上传 “druid测试数据源\维基百科访问日志数据” wikiticker-2015-09-12-sampled.json到任意服务器 /export/servers/tmp/druid 目录，再将 文件上传到HDFS

```
{"time":"2015-09-12T00:47:00.496Z","channel":"#ca.wikipedia","cityName":null,"comment":"Robot inserta {{Commonscat}} que enllaça amb [[commons:category:Rallicula]]","countryIsoCode":null,"countryName":null,"isAnonymous":false,"isMinor":true,"isNew":false,"isRobot":true,"isUnpatrolled":false,"metroCode":null,"namespace":"Main","page":"Rallicula","regionIsoCode":null,"regionName":null,"user":"PereBot","delta":17,"added":17,"deleted":0}
{"time":"2015-09-12T00:47:05.474Z","channel":"#en.wikipedia","cityName":"Auburn","comment":"/* Status of peremptory norms under international law */ fixed spelling of 'Wimbledon'","countryIsoCode":"AU","countryName":"Australia","isAnonymous":true,"isMinor":false,"isNew":false,"isRobot":false,"isUnpatrolled":false,"metroCode":null,"namespace":"Main","page":"Peremptory norm","regionIsoCode":"NSW","regionName":"New South Wales","user":"60.225.66.142","delta":0,"added":0,"deleted":0}
{"time":"2015-09-12T00:47:08.770Z","channel":"#vi.wikipedia","cityName":null,"comment":"fix Lỗi CS1: ngày tháng","countryIsoCode":null,"countryName":null,"isAnonymous":false,"isMinor":true,"isNew":false,"isRobot":true,"isUnpatrolled":false,"metroCode":null,"namespace":"Main","page":"Apamea abruzzorum","regionIsoCode":null,"regionName":null,"user":"Cheers!-bot","delta":18,"added":18,"deleted":0}
{"time":"2015-09-12T00:47:11.862Z","channel":"#vi.wikipedia","cityName":null,"comment":"clean up using [[Project:AWB|AWB]]","countryIsoCode":null,"countryName":null,"isAnonymous":false,"isMinor":false,"isNew":false,"isRobot":true,"isUnpatrolled":false,"metroCode":null,"namespace":"Main","page":"Atractus flammigerus","regionIsoCode":null,"regionName":null,"user":"ThitxongkhoiAWB","delta":18,"added":18,"deleted":0}
{"time":"2015-09-12T00:47:13.987Z","channel":"#vi.wikipedia","cityName":null,"comment":"clean up using [[Project:AWB|AWB]]","countryIsoCode":null,"countryName":null,"isAnonymous":false,"isMinor":false,"isNew":false,"isRobot":true,"isUnpatrolled":false,"metroCode":null,"namespace":"Main","page":"Agama mossambica","regionIsoCode":null,"regionName":null,"user":"ThitxongkhoiAWB","delta":18,"added":18,"deleted":0}
{"time":"2015-09-12T00:47:17.009Z","channel":"#ca.wikipedia","cityName":null,"comment":"/* Imperi Austrohongarès */","countryIsoCode":null,"countryName":null,"isAnonymous":false,"isMinor":false,"isNew":false,"isRobot":false,"isUnpatrolled":false,"metroCode":null,"namespace":"Main","page":"Campanya dels Balcans (1914-1918)","regionIsoCode":null,"regionName":null,"user":"Jaumellecha","delta":-20,"added":0,"deleted":20}
```

```
hdfs dfs -mkdir -p /tmp/druid/
hdfs dfs -put wikiticker-2015-09-12-sampled.json /tmp/druid/wikiticker-2015-09-12-sampled.json
```

> 3、修改 index_wikiticker-2015-9-12-sample.json 文件中配置 HDFS 的地址

```json
{
    "type": "index_hadoop",
	"spec": {
		"dataSchema": {
			"dataSource": "wikiticker",
			"parser": {
				"type": "hadoopyString",
				"parseSpec": {
					"format": "json",
					"dimensionsSpec": {
						"dimensions": [
							"channel",
							"cityName",
							"comment",
							"countryIsoCode",
							"countryName",
							"isAnonymous",
							"isMinor",
							"isNew",
							"isRobot",
							"isUnpatrolled",
							"metroCode",
							"namespace",
							"page",
							"regionIsoCode",
							"regionName",
							"user"
						]
					},
					"timestampSpec": {
						"format": "auto",
						"column": "time"
					}
				}
			},
			"metricsSpec": [
				{
					"name": "count",
					"type": "count"
				},
				{
					"name": "added",
					"type": "longSum",
					"fieldName": "added"
				},
				{
					"name": "deleted",
					"type": "longSum",
					"fieldName": "deleted"
				},
				{
					"name": "delta",
					"type": "longSum",
					"fieldName": "delta"
				}
			],
			"granularitySpec": {
				"type": "uniform",
				"segmentGranularity": "day",
				"queryGranularity": "none",
				"intervals": [
					"2015-09-12/2015-09-13"
				],
				"rollup": false
			}
		},
		"ioConfig": {
			"type": "hadoop",
			"inputSpec": {
				"type": "static",
				"paths": "/tmp/druid/wikiticker-2015-09-12-sampled.json"
			}
		},
		"tuningConfig": {
			"type": "hadoop",
			"partitionsSpec": {
				"type": "hashed",
				"targetPartitionSize": 5000000
			},
			"jobProperties": {
				"fs.default.name": "hdfs://node01:8020",
				"fs.defaultFS": "hdfs://node01:8020",
				"dfs.datanode.address": "node01",
				"dfs.client.use.datanode.hostname": "true",
				"dfs.datanode.use.datanode.hostname": "true",
				"yarn.resourcemanager.hostname": "node01",
				"yarn.nodemanager.vmem-check-enabled": "false",
				"mapreduce.map.java.opts": "-Duser.timezone=UTC -Dfile.encoding=UTF-8",
				"mapreduce.job.user.classpath.first": "true",
				"mapreduce.reduce.java.opts": "-Duser.timezone=UTC -Dfile.encoding=UTF-8",
				"mapreduce.map.memory.mb": 1024,
				"mapreduce.reduce.memory.mb": 1024
			}
		}
	},
    "hadoopDependencyCoordinates": [
        "org.apache.hadoop:hadoop-client:2.8.3"
    ]
}
```

> 4、使用 postman 提交索引任务
>> 将index_wikiticker-2015-9-12-sample.json文件中的内容拷贝到 postman 中

>> 发送post请求到
```
http://node01:8090/druid/indexer/v1/task
```

> 5、稍等一会到 Druid控制台中执行SQL查询

```
SELECT *
FROM "wikiticker"
LIMIT 1
```

### 5.2 流式（实时）数据摄取

#### 5.2.1 Kafka索引服务方式摄取

> `需求：`
>> 实时摄取Kafka中 metrics topic的数据到 Druid中

> `操作步骤：`

> 1、启动 Kafka 集群

```
cd /export/servers/kafka_2.11-1.0.0/
bin/kafka-server-start.sh config/server.properties >/dev/null 2>& 1 &
```

> 2、在Kafka集群上创建一个名为metrics的topic

```
bin/kafka-topics.sh --create --zookeeper node01:2181 --partitions 1 --replication-factor 1 --topic metrics
```

> 3、定义摄取配置文件
>> 修改 druid测试数据源\kafka实时摄取数据中的 index-metrics-kafka.json 文件中的kafka服务器地址

```json
{
	"type": "kafka",
	"dataSchema": {
		 "dataSource": "metrics-kafka",
		 "parser": {
			 "type": "string",
			 "parseSpec": {
				 "timestampSpec": {
					 "column": "time",
					 "format": "auto"
				 },
				 "dimensionsSpec": {
					 "dimensions": ["url", "user"]
				 },
				 "format": "json"
			 }
		 },
		 "granularitySpec": {
			 "type": "uniform",
			 "segmentGranularity": "HOUR",
			 "queryGranularity": "NONE"
		 },
		 "metricsSpec": [
			{
				 "type": "count",
				 "name": "views"
			 },
			 {
				 "name": "latencyMs",
				 "type": "doubleSum",
				 "fieldName": "latencyMs"
			 }
		 ]
	},
	"ioConfig": {
		 "topic": "metrics",
		 "consumerProperties": {
			 "bootstrap.servers": "node01:9092",
			 "group.id": "kafka-indexing-service"
		 },
		 "taskCount": 1,
		 "replicas": 1,
		 "taskDuration": "PT1H"
	},
	"tuningConfig": {
		 "type": "kafka",
		 "maxRowsInMemory": "100000",
		 "workerThreads": 2
	}
}
```

> 4、打开postman提交索引任务
> 将 index-metrics-kafka.json 文件中的内容拷贝到 postman 中

>发送post请求到

```
http://node01:8090/druid/indexer/v1/supervisor
```
 
> 在Overlord中可以看到

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/19.png)

> 6、在Kafka集群上开启一个控制台producer

```
bin/kafka-console-producer.sh --broker-list node01:9092 --topic metrics
```

> 7、在Kafka producer控制台中粘贴如下数据

```
{"time":"2019-07-23T17:57:58Z","url":"/foo/bar","user":"alice","latencyMs":32}
{"time":"2019-07-23T17:57:59Z","url":"/","user":"bob","latencyMs":11}
{"time":"2019-07-23T17:58:00Z","url": "/foo/bar","user":"bob","latencyMs":45}
```

> 8、在 Druid Console中执行以下SQL查询

```
SELECT *
from "metrics-kafka"
LIMIT 1
```

#### 5.2.2 Druid WebUI生成json文件-了解

> 1.准备主题

```
bin/kafka-topics.sh --create --zookeeper node01:2181 --partitions 1 --replication-factor 1 --topic metrics2
```

> 2.发送一些测试数据

```
bin/kafka-console-producer.sh --broker-list node01:9092 --topic metrics2
```

```    
{"time":"2019-07-23T17:57:58Z","url":"/foo/bar","user":"alice","latencyMs":32}
{"time":"2019-07-23T17:57:59Z","url":"/","user":"bob","latencyMs":11}
{"time":"2019-07-23T17:58:00Z","url": "/foo/bar","user":"bob","latencyMs":45}
```

```
http://node01:8888/unified-console.html#query
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/20.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/21.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/22.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/23.png)

> 发送一些数据

```
{"time":"2019-07-23T17:57:58Z","url":"/foo/bar","user":"alice","latencyMs":32} 
{"time":"2019-07-23T17:57:59Z","url":"/","user":"bob","latencyMs":11}
{"time":"2019-07-23T17:58:00Z","url": "/foo/bar","user":"bob","latencyMs":45}
```

> 查询

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/24.png)

> 最后暂停所有的实时摄取任务

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/25.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Druid/26.png)


### 5.3 摄取配置文件结构说明-了解

#### 5.3.1 主体结构

> 摄取配置文件主要由以下几个部分组成：
>> `type`：文件上传方式（index、index_hadoop、kafka）
> 
>> `spec`：
> 
>> `dataSchema`：数据解析模式
> 
>> `ioConfig`：数据源
> 
>> `turningConfig`：优化配置（分区规则、分区大小）

```
{
    // ① 文件拉取方式
    // 1.1 index 		- 拉取本地文件
    // 1.2 index_hadoop - 拉取HDFS文件
    // 1.3 kafka		- 拉取Kafka流数据
    "type": "index",
    "spec": {
        // ② 数据解析模式
        "dataSchema": {...},
        // ③ 摄取数据源
        "ioConfig": {...},
        // ④ 摄取过程优化配置
        "tuningConfig": {...}
    }
}
```

#### 5.3.2 数据解析模式

>数据解析模式，主要为针对数据文件，定义了一系列规则：
>> 1.`获取时间戳属性`
> 
>> 2.`维度属性`
> 
>> 3.`度量属性`
> 
>> 4.`定义如何进行指标计算`
> 
>> 5.`配置粒度规则`

```
// ② 数据摄取模式
"dataSchema": {
    // 2.1 数据源
    "dataSource": "ad_event_local",
    // 2.2 解析器
    "parser": {
        // 2.2.1 解析字符串文本
        "type": "String",
        "parseSpec": {
        // 2.2.1.1 字符串文本格式为JSON
        "format": "json",
        // 2.2.1.2 指定维度列名
        "dimensionsSpec": {
            "dimensions": [
                "city",
                "platform"
            ]
        },
        // 2.2.1.3 指定时间戳的列，以及时间戳格式化方式
        "timestampSpec": {
            "format": "auto",
            "column": "timestamp"
            }
        }
    },
    // 2.3 指标计算规则
    "metricsSpec": [
        {
            "name": "count",
            "type": "count"
        },
        {
            // 2.3.1 聚合计算后指标的列名
            "name": "click",
            // 2.3.2 聚合函数：count、longSum、doubleSum、longMin、doubleMin、doubleMax
            "type": "longSum",
            "fieldName": "click"
        }
    ]
    // 2.4 粒度规则
    "granularitySpec": {
        "type": "uniform",
        // 2.4.1 按天来生成 segment （每天生成一个segment）
        "segmentGranularity": "day",
        // 2.4.2 查询的最小粒度（最小粒度为小时）
        "queryGranularity": "hour",
        // 2.4.3 加载原始数据的时间范围，批量数据导入需要设置/流式导入无需设置
        "intervals": [
            "2018-12-01/2018-12-03"
        ]
    },
}
```

#### 5.3.3 数据源配置

> 数据源配置主要指定：

> 要加载数据的类型,从哪儿加载数据

```
"ioConfig": {
    "type": "index",
    "inputSpec": {
        // 3.1 本地文件 local/ HDFS使用 hadoop
        "type": "local",
        // 3.2 路径
        "baseDir": "/root/data/",
        // 3.3 只过滤出来哪个文件
        "filter": "ad_event.json"
    }
}
```

#### 5.3.4 优化配置

> 通常在优化配置中可以指定一些优化选项

```
"tuningConfig": {
    "type": "index",
    // 4.1 分区类型
        "partitionsSpec": {
        "type": "hashed",
        // 4.2 每个分区的目标行数（这里配置每个分区500W行）
        "targetPartitionSize": 5000000
    }
}
```

## 6 Druid数据查询

### 6.1 JSON API 方式

> 已经演示过

### 6.2 WEB UI 方式

> 已经演示过

### 6.3 JAVA API 方式

#### 6.3.1 pom

```xml
 <dependencies>
        <!-- druid驱动包-->
        <dependency>
            <groupId>org.apache.calcite.avatica</groupId>
            <artifactId>avatica</artifactId>
            <version>1.13.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.calcite.avatica</groupId>
            <artifactId>avatica-core</artifactId>
            <version>1.13.0</version>
        </dependency>
    </dependencies>
```

#### 6.3.2 java代码

```
import java.sql.*;

/**
 * Author xiaoma
 * Date 2020/10/26 9:54
 * Desc 演示使用ApacheDruid数据库API
 * 注意:
 * Java代码操作数据之前学习过JDBC规范!而Druid也是遵循JDBC规范的
 * 所以直接导入Druid的驱动包,然后按照JDBC规范进行编码即可
 */
public class DruidTest {
    public static void main(String[] args) throws Exception {
        //1.加载驱动
        //Class.forName("org.apache.calcite.avatica.remote.Driver");
        //2.获取连接
        Connection conn = DriverManager.getConnection("jdbc:avatica:remote:url=http://192.168.52.100:8888/druid/v2/sql/avatica/");
        System.out.println("获取到的连接对象为:" + conn);

        //3.创建语句对象
        String sql = "SELECT user,views FROM \"metrics-kafka\"";
        Statement statement = conn.createStatement();
        //4.执行查询
        ResultSet rs = statement.executeQuery(sql);
        //5.结果集处理
        while (rs.next()){
            String user = rs.getString("user");
            long views = rs.getLong("views");
            System.out.println(user+" : "+views);
        }
        //6.关闭资源
        rs.close();
        statement.close();
        conn.close();
    }
}

```

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
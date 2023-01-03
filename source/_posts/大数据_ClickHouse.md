---
title: ClickHouse 用于OLAP的列式数据库管理系统
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(70).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(70).jpg
tags:
  - 大数据
  - ClickHouse
category:
  - - 编程
    - 大数据
date: 2020-09-03 23:51:37
---

ClickHouse 的工作速度比传统方法快100-1000倍。

ClickHouse 的性能超过了市场上可供比较的面向列的数据库管理系统。它每秒处理数亿到数十亿行数据和数千兆字节的数据

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 实时OLAP分析需求

### 1.1 背景介绍

> 离线数仓的最大问题即：`慢`，数据无法实时的通过可视化页面展示出来，通常离线数仓分析的是“T+1”的数据，针对于时效性要求比较高的场景，则无法满足需求，例如：快速实时返回“分组+聚合计算+排序聚合指标”查询需求。

> 因此智慧物流系统的应用场景分为以下几个部分：
>> Spark-to-Kudu 小时/日级场景
> 
>> Spark-to-ClickHouse 秒级场景
> 
>> Spark-to-ElasticSearch 秒级场景

### 1.2 技术选型

> 目前市面上主流的开源OLAP引擎包含不限于：Hive、Presto、Kylin、Impala、Sparksql、Druid、Clickhouse、Greeplum等，可以说目前没有一个引擎能在数据量，灵活程度和性能上做到完美，用户需要根据自己的需求进行选型。

框架 | 描述
---|---
`Hive` | Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供完整的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。<br/><br/>优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析。<br/><br/>`缺点是慢`
`Spark SQL` | SparkSQL的前身是`Shark`，它将 SQL 查询与 Spark 程序无缝集成,可以将结构化数据作为 Spark 的 RDD 进行查询。SparkSQL作为Spark生态的一员继续发展，而不再受限于Hive，只是兼容Hive。
`Presto` | Presto 是由 Facebook 开源的大数据分布式 SQL 查询引擎，适用于交互式分析查询，可支持众多的数据源，包括 HDFS，RDBMS，KAFKA 等，而且提供了非常友好的接口开发数据源连接器。<br/><br/>Presto由于是基于内存的，而hive是在磁盘上读写的，因此presto比hive快很多，但是由于是基于内存的计算当多张大表关联操作时易引起内存溢出错误。
`Kylin` | Apache Kylin™是一个开源的分布式分析引擎，提供Hadoop/Spark之上的SQL查询接口及多维分析（OLAP）能力以支持超大规模数据，最初由eBay Inc. 开发并贡献至开源社区。它能在亚秒内查询巨大的Hive表。<br/><br/>所以适合Kylin的场景包括：<br/>1）用户数据存在于Hadoop HDFS中，利用Hive将HDFS文件数据以关系数据方式存取，数据量巨大，在500G以上<br/>2）每天有数G甚至数十G的数据增量导入<br/>3）有10个以内较为固定的分析维度
`Impala` | Impala不提供任何对序列化和反序列化的支持。<br/><br/>Impala只能读取文本文件，而不能读取自定义二进制文件。<br/><br/>每当新的记录/文件被添加到HDFS中的数据目录时，该表需要被刷新。这个缺点会导致正在执行的查询sql遇到刷新会挂起，查询不动。
`Druid` | Druid 是一种能对历史和实时数据提供亚秒级别的查询的数据存储。Druid 支持低延时的数据摄取，灵活的数据探索分析，高性能的数据聚合，简便的水平扩展。适用于数据量大，可扩展能力要求高的分析型查询系统。<br/><br/>与其他的时序数据库类似，Druid在查询条件命中大量数据情况下可能会有性能问题，而且排序、聚合等能力普遍不太好，灵活性和扩展性不够，比如缺乏Join、子查询等。
`Greeplum` | `Greenplum`是一个开源的大规模并行数据分析引擎。借助MPP（大规模并行处理）架构，在大型数据集上执行复杂SQL分析的速度比很多解决方案都要快。<br/><br/>Greenplum基于Postgresql，也就是说GreenPulm和TiDB的定位类似，想要在OLTP和OLAP上进行统一。
`ClickHouse` | Clickhouse由俄罗斯`yandex`公司开发。专为在线数据分析而设计。Yandex是俄罗斯搜索引擎公司。官方提供的文档表名，ClickHouse 日处理记录数"十亿级"。<br/><br/>特性:采用列式存储；数据压缩；支持分片，并且同一个计算任务会在不同分片上并行执行，计算完成后会将结果汇总；支持SQL；支持联表查询；支持实时更新；自动多副本同步；支持索引；分布式存储查询。<br/><br/>大家对Nginx应该不陌生，战斗民族开源的软件普遍的特点包括：轻量级，快。<br/><br/>ClickHouse最大的特点就是快，快，快，重要的话说三遍！与Hadoop、Spark这些巨无霸组件相比，ClickHouse很轻量级

> `总结`

> 上面给出了常用的一些OLAP引擎，各自有各自的特点，将其分组：
>> Hive，Impala - 基于SQL on Hadoop
> 
>> Presto和Spark SQL类似 - 基于内存解析SQL生成执行计划
> 
>> Kylin - 用空间换时间，预计算
> 
>> Druid - 一个支持数据的实时摄入
> 
>> ClickHouse - OLAP领域的Hbase，单表查询性能优势巨大
> 
>> Greenpulm - OLAP领域的Postgresql

### 1.3 设计方案

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/1.png)

## 2 ClickHouse入门

### 2.1 ClickHouse的概述

> ClickHouse是俄罗斯的`Yandex`于2016年开源的面向OLAP列式数据库管理系统（DBMS）
> 
> ClickHouse采用 `C++` 语言开发，以`卓越的查询性能`著称，在基准测试中`超过了目前很多主流的列式数据库`
> 
> ClickHouse集群的`每台服务器每秒能处理数亿到十亿多行和数十千兆字节的数据`
> 
> ClickHouse会充分利用所有可用的硬件，以尽可能快地处理每个查询
> 
> 单个查询（解压缩后，仅使用的列）的峰值处理性能超过`每秒2TB`
> 
> 允许使用`类SQL查询实时生成分析数据`报告，具有速度快、线性可扩展、硬件高效、容错、功能丰富、高度可靠、简单易用和支持跨数据中心部署等特性，号称在内存数据库领域是最快的
> 
> ClickHouse提供了丰富的`数据类型`、`数据库引擎`和`表引擎`，它所存储的表类似于关系型数据库中的表，默认情况下使用结构化方式在节点本地存储表的数据，同时支持多种数据压缩方式
> 
> ClickHouse独立于Hadoop生态系统，`不依赖Hadoop的HDFS`，但可以扩展HDFS进行数据查询，ClickHouse还支持查询Kafka和MySQL中的数据
> 
> ClickHouse目前已经`在很多大型企业中得到了充分的生产验证`，其在存储PB级别的数据规模时仍能很好的提供稳健的实时OLAP服务。

#### 2.1.1 ClickHouse的特性

> 真正面向列的DBMS

> 支持压缩

> 支持普通硬盘存储

> 支持多核并行处理

> 支持SQL

> 支持矢量引擎

> 支持实时数据更新

> 支持索引

> 支持在线查询

> 支持近似计算

> 支持数据辅助和数据完整性

#### 2.1.2 ClickHouse的优势

> 高性能

> 线性可扩展

> 硬件高效

> 容错

> 高度可靠

> 简单易用

#### 2.1.3 ClickHouse的劣势

> 缺少高频率，低延迟的修改或删除已存在数据的能力。仅能用于批量删除或修改数据。

> 没有完整的事务支持

> 不支持二级索引

> 有限的SQL支持，join实现与众不同

> 不支持窗口功能

> 元数据管理需要人工干预维护

#### 2.1.4 ClickHouse的基准测试

> ClickHouse提供了一个与其他列式数据库的基准测试，这个基准测试大多数是在单台服务器进行测试，该服务器的配置是：
>> 双CPU(Intel(R) Xeon(R) CPU E5-2650 v2@2.60GHZ)
> 
>> 内存128GB
> 
>> 在8个6TB SATA硬盘上安装MD RAID-5
> 
>> 文件系统为Ext4

> 这个测试中，有些结果可能是过时的

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/2.png)

### 2.2 ClickHouse的应用场景

> 绝大多数请求都是用于读访问的

> 数据需要以大批量（大于1000行）进行更新，而不是单行更新；或者根本没有更新操作

> 数据只是添加到数据库，没有必要修改

> 读取数据时，会从数据库中提取出大量的行，但只用到一小部分列

> 表很“宽”，即表中包含大量的列

> 查询频率相对较低（通常每台服务器每秒查询数百次或更少）

> 对于简单查询，允许大约50毫秒的延迟

> 列的值是比较小的数值和短字符串（例如，每个URL只有60个字节）

> 在处理单个查询时需要高吞吐量（每台服务器每秒高达数十亿行）

> 不需要事务

> 数据一致性要求较低

> 每次查询中只会查询一个大表。除了一个大表，其余都是小表

> 查询结果显著小于数据源。即数据有过滤或聚合。返回结果不超过单个服务器内存大小

### 2.3 ClickHouse的使用案例

#### 2.3.1 电信行业用于存储数据和统计数据使用

> 我国的`中国电信G网`数据分析应用采用`ClickHouse`作为数据存储引擎，主要存储`网络基站设备数据`、`监控设备和骨干网`等数据，这些数据日的增量500亿条左右，约700GB。并进行相应的分析处理，最终提供BI应用、数据挖掘等系统使用。

#### 2.3.2 新浪微博用于用户行为数据记录和分析工作

> 新浪微博APP监控系统采用`ClickHouse`作为数据存储引擎，使用Kafka存储实时产生的消息，Python消费数据存储到ClickHouse中，然后Superset连接ClickHouse作为可视化工作台。同时还使用Hangout消费Kafka的数据到ElasticSearch中，然后使用Kibana进行问题跟踪和问题排查。

#### 2.3.3 RTB网络广告

> Geniee是日本的一家广告公司，使用ClickHouse作为其RTB实时竞价服务的数据存储引擎。

#### 2.3.4 商业智能

> `今日头条`最早使用ClickHouse的是用户行为分析系统。该系统在使用 ClickHouse之前，engine（引擎）层已经有两个迭代。

> 尝试过Spark全内存方案还有一些其他的方案，都存在很多问题。主要因为产品需要比较强的交互能力，页面拖拽的方式能够给分析师展示不同的指标，查询模式比较多变，并且有一些查询的 DSL 描述，也不好用现成的SQL去表示，这就需要engine有比较好的定制能力。

> 行为分析系统的表可以打成一个大的宽表形式，join的形式相对少一点。系统的数据量比较大，因为产品要支持头条所有APP的用户行为分析，包含头条全量和抖音全量数据，用户的上报日志分析，面临不少技术挑战。

> 在使用ClickHouse做一些简单的POC测试工作后，综合来看ClickHouse的性能、功能和产品质量来说效果不错，因为开发ClickHouse的公司使用的场景实际上跟头条用户分析是比较类似的，因此有一定的借鉴意义。

> 目前头条 ClickHouse 集群的规模大概有几千个节点，最大的集群规模可能有1200个节点，这是一个单集群的最大集群节点数。数据总量大概是几十PB，日增数据100TB，落地到ClickHouse的日增数据总量大概是它的3倍，原始数据也就 300T 左右，大多数查询的响应时间是在几秒钟。从交互式的用户体验来说，一般希望把所有的响应控制在 30 秒之内返回，ClickHouse基本上能够满足大部分要求。覆盖的用户场景包括产品分析师做精细化运营，开发人员定位问题，也有少量的广告类客户。

#### 2.3.5 Yandex的统计分析服务Yandex.Metrica

> `Yandex.Metric`是Yandex提供的免费网络分析服务，可跟踪和报告网站流量。Yandex.Metrica使用一个简单的JavaScript标记，由网站站长在其网站上实现。标签收集网站的访问者，访问量和行为数据。Metrica也可以与Yandex.Direct在线广告平台链接以收集广告转化率。

### 2.4 ClickHouse快速入门

#### 2.4.1 安装ClickHouse(单机)

> `1 安装yum-utils工具包`

```
yum install yum-utils
```

> 2 `添加ClickHouse的yum源`

```
yum-config-manager --add-repo https://repo.yandex.ru/clickhouse/rpm/stable/x86_64
```

> 3 `安装ClickHouse的服务端和客户端`

```
yum install -y clickhouse-server clickhouse-client
```

> 如果安装时出现warning: rpmts_HdrFromFdno: Header V4 RSA/SHA1 Signature, key ID e0c56bd4: NOKEY错误导致无法安装，需要在安装命令中添加—nogpgcheck来解决。

```
yum install -y clickhouse-server clickhouse-client --nogpgcheck
```

> 4 `关于安装的说明`

> 默认的配置文件路径是：/etc/clickhouse-server/

> 默认的日志文件路径是：/var/log/clickhouse-server/

> 5 `查看ClickHouse的版本信息`

```
clickhouse-client -m --host node2.xiaoma.cn --user root --password 123456
select version();
```

#### 2.4.2 在命令行中操作ClickHouse

> ClickHouse安装包中提供了clickhouse-client工具，这个客户端在运行shell环境中，使用TCP方式连接clickhouse-server服务。要运行该客户端工具可以选择使用交互式与非交互式（批量）两种模式：使用非交互式查询时需要指定--query参数；在交互模式下则需要注意是否使用—mutiline参数来开启多行模式。

> clickhouse-client提供了很多参数可供使用，常用的参数如下表：

参数 | 介绍
---|---
--host,-h | 服务端的 host 名称, 默认是 'localhost'。 您可以选择使用 host 名称或者 IPv4 或 IPv6 地址。
--port | 连接服务端的端口，默认值9000
--user,-u | 访问的用户名，默认default
--password | 访问用户的密码，默认空字符串
--query,-q | 非交互模式下的查询语句
--database,-d | 连接的数据库，默认是default
--multiline,-m | 使用多行模式，在多行模式下，回车键仅表示换行。默认不使用多行模式。
--multiquery,-n | 使用”,”分割的多个查询，仅在非交互模式下有效
--format, -f | 使用指定格式化输出结果
--vertical, -E | 使用垂直格式输出，即每个值使用一行显示
--time, -t | 打印查询时间到stderr中
--stacktrace | 如果出现异常，会打印堆栈跟踪信息
--config-file | 使用指定配置文件
--use_client_time_zone | 使用服务端时区
quit,exit | 表示退出客户端
Ctrl+D,Ctrl+C | 表示退出客户端

> `登录`

```
clickhouse-client -m --host node2.xiaoma.cn --user root --password 123456
```

#### 2.4.3 样例数据导入

> `1 编写下载航班数据脚本`

> 创建名为 clickhouse-example-data-download.sh 的脚本文件

```
vim clickhouse-example-data-download.sh
for s in `seq 2017 2020`
do
for m in `seq 1 12`
do
wget https://transtats.bts.gov/PREZIP/On_Time_Reporting_Carrier_On_Time_Performance_1987_present_${s}_${m}.zip
done
done
```

> `2 下载航班数据`

```
chmod +x clickhouse-example-data-download.sh
./clickhouse-example-data-download.sh
```

> `3 创建ontime表`

```
CREATE TABLE `ontime` (
`Year` UInt16,
`Quarter` UInt8,
`Month` UInt8,
`DayofMonth` UInt8,
`DayOfWeek` UInt8,
`FlightDate` Date,
`UniqueCarrier` FixedString(7),
`AirlineID` Int32,
`Carrier` FixedString(2),
`TailNum` String,
`FlightNum` String,
`OriginAirportID` Int32,
`OriginAirportSeqID` Int32,
`OriginCityMarketID` Int32,
`Origin` FixedString(5),
`OriginCityName` String,
`OriginState` FixedString(2),
`OriginStateFips` String,
`OriginStateName` String,
`OriginWac` Int32,
`DestAirportID` Int32,
`DestAirportSeqID` Int32,
`DestCityMarketID` Int32,
`Dest` FixedString(5),
`DestCityName` String,
`DestState` FixedString(2),
`DestStateFips` String,
`DestStateName` String,
`DestWac` Int32,
`CRSDepTime` Int32,
`DepTime` Int32,
`DepDelay` Int32,
`DepDelayMinutes` Int32,
`DepDel15` Int32,
`DepartureDelayGroups` String,
`DepTimeBlk` String,
`TaxiOut` Int32,
`WheelsOff` Int32,
`WheelsOn` Int32,
`TaxiIn` Int32,
`CRSArrTime` Int32,
`ArrTime` Int32,
`ArrDelay` Int32,
`ArrDelayMinutes` Int32,
`ArrDel15` Int32,
`ArrivalDelayGroups` Int32,
`ArrTimeBlk` String,
`Cancelled` UInt8,
`CancellationCode` FixedString(1),
`Diverted` UInt8,
`CRSElapsedTime` Int32,
`ActualElapsedTime` Int32,
`AirTime` Int32,
`Flights` Int32,
`Distance` Int32,
`DistanceGroup` UInt8,
`CarrierDelay` Int32,
`WeatherDelay` Int32,
`NASDelay` Int32,
`SecurityDelay` Int32,
`LateAircraftDelay` Int32,
`FirstDepTime` String,
`TotalAddGTime` String,
`LongestAddGTime` String,
`DivAirportLandings` String,
`DivReachedDest` String,
`DivActualElapsedTime` String,
`DivArrDelay` String,
`DivDistance` String,
`Div1Airport` String,
`Div1AirportID` Int32,
`Div1AirportSeqID` Int32,
`Div1WheelsOn` String,
`Div1TotalGTime` String,
`Div1LongestGTime` String,
`Div1WheelsOff` String,
`Div1TailNum` String,
`Div2Airport` String,
`Div2AirportID` Int32,
`Div2AirportSeqID` Int32,
`Div2WheelsOn` String,
`Div2TotalGTime` String,
`Div2LongestGTime` String,
`Div2WheelsOff` String,
`Div2TailNum` String,
`Div3Airport` String,
`Div3AirportID` Int32,
`Div3AirportSeqID` Int32,
`Div3WheelsOn` String,
`Div3TotalGTime` String,
`Div3LongestGTime` String,
`Div3WheelsOff` String,
`Div3TailNum` String,
`Div4Airport` String,
`Div4AirportID` Int32,
`Div4AirportSeqID` Int32,
`Div4WheelsOn` String,
`Div4TotalGTime` String,
`Div4LongestGTime` String,
`Div4WheelsOff` String,
`Div4TailNum` String,
`Div5Airport` String,
`Div5AirportID` Int32,
`Div5AirportSeqID` Int32,
`Div5WheelsOn` String,
`Div5TotalGTime` String,
`Div5LongestGTime` String,
`Div5WheelsOff` String,
`Div5TailNum` String
) ENGINE = MergeTree(FlightDate, (Year, FlightDate), 8192)
```

> `4 导入数据 `

> 创建名为 import.sh 的脚本文件

```
vim import.sh
for i in *.zip; do echo $i; unzip -cq $i '*.csv' | sed 's/\.00//g' | clickhouse-client --host=node2.xiaoma.cn --query="INSERT INTO default.ontime FORMAT CSVWithNames"; done
chmod +x import.sh
./import.sh
```

### 2.5 ClickHouse的数据类型支持

> 官网文档： https://clickhouse.tech/docs/zh/

> ClickHouse与常用的关系型数据库MySQL或Oracle的数据类型类似，提供了丰富的数据类型支持。

#### 2.5.1 整形

> ClickHouse支持Int和Uint两种固定长度的整型，Int类型是符号整型，Uint类型是无符号整型。

分类 | 数据类型 | 取值范围
---|---|---
整型 | Int8 | -128 ~ 127
| Int16 | -32768 ~ 32767
| Int32 | -2147483648 ~ 2147483647
| Int64 | -9223372036854775808 ~ 223372036854775807
无符号整型 | UInt8 | 0 ~ 255
| Uint16 | 0 ~ 65535
| Uint32 | 0 ~ 4294967295
| Uint64 | 0 ~ 18446744073709551615

#### 2.5.2 浮点型

> ClickHouse支持Float32和Float64两种浮点类型，浮点型在运算时可能会导致一些问题，例如计算的结果取决于计算机的处理器和操作系统、可能是正无穷或负无穷等问题，官方建议尽量以整数形式存储数据。例如，将固定精度的数字转换为整数值，例如货币数量或页面加载时间用毫秒为单位表示。

分类 | 取值范围
---|---
Float32 | select 1-0.9 的结果是`0.09999999999999998`
Float64 | select 1/0的结果是 `inf（正无穷）`
| select -1/0的结果是 `-inf（负无穷）`
| select 0/0的结果是 `nan（非数字）`

#### 2.5.3 Decimal

> ClickHouse支持Decimal类型的有符号定点数，可在加、减和乘法运算过程中保持精度。对于除法，最低有效数字会被丢弃，但不会四舍五入。数据采用与自身位宽相同的有符号整数存储。这个数在内存中实际范围会高于上述范围，从 String 转换到十进制数的时候会做对应的检查。由于现代CPU不支持128位数字，因此 Decimal128 上的操作由软件模拟。

> 所以 Decimal128 的运算速度明显慢于 Decimal32/Decimal64。

> Decimal(P,S),P参数指的是精度，有效范围：[1:38]，决定可以有多少个十进制数字（包括分数）；S参数指的是小数长度，有效范围：[0：P]，决定数字的小数部分中包含的小数位数。

数据类型 | 十进制的范围
---|---
Decimal32(S) | Decimal32(S)：  ( -1 * 10^(9 - S), 1 * 10^(9 - S) )
Decimal64(S) | Decimal64(S)：  ( -1 * 10^(18 - S), 1 * 10^(18 - S) )
Decimal128(S) | Decimal128(S)： ( -1 * 10^(38 - S), 1 * 10^(38 - S) )

> 对Decimal的二进制运算导致更宽的结果类型，两个不同的Decimal类型在运算时精度的变化规则如下：

> 例子	

```
Decimal64(S1) Decimal32(S2) -> Decimal64(S)
Decimal128(S1) Decimal32(S2) -> Decimal128(S)
Decimal128(S1) Decimal64(S2) -> Decimal128(S)
```

> 小数变化规则

```
加法，减法：S = max(S1, S2)
乘法：S = S1 + S2
除法：S = S1
```

#### 2.5.4 布尔型

> ClickHouse中没有定义布尔类型，可以使用UInt8类型，取值限制为0或1。

#### 2.5.5 字符串类型

> ClickHouse中的String类型没有编码的概念。字符串可以是任意的字节集，按它们原本的方式进行存储和输出。若需存储文本，建议使用UTF-8编码。至少，如果你的终端使用UTF-8，这样读写就不需要进行任何的转换。对不同的编码文本ClickHouse会有不同处理字符串的函数。比如，length函数可以计算字符串包含的字节数组的长度，然而lengthUTF8函数是假设字符串以 UTF-8编码，计算的是字符串包含的Unicode字符的长度。

数据类型 | 十进制的范围
---|---
String | 字符串可以任意长度的。它可以包含任意的字节集，包含空字节。ClickHouse中的String类型可以代替其他DBMS中的VARCHAR、BLOB、CLOB等类型。
FixedString(N) | 固定长度 N 的字符串，N必须是严格的正自然数。当服务端读取长度小于N的字符串时候，通过在字符串末尾添加空字节来达到N字节长度。当服务端读取长度大于N的字符串时候，将返回错误消息。与String相比，极少会使用FixedString，因为使用起来不是很方便。<br/><br/>1）在插入数据时，如果字符串包含的字节数小于N,将对字符串末尾进行空字节填充。如果字符串包含的字节数大于N,将抛Too large value for FixedString(N)异常。<br/><br/>2）在查询数据时，ClickHouse不会删除字符串末尾的空字节。如果使用WHERE子句，则须要手动添加空字节以匹配FixedString的值（例如：where a=’abc\0’）。<br/><br/>`注意，FixedString(N)的长度是个常量。仅由空字符组成的字符串，函数length返回值为N,而函数empty的返回值为1。`

#### 2.5.6 UUID

> ClickHouse支持UUID类型（通用唯一标识符），该类型是一个16字节的数字，用于标识记录。ClickHouse内置generateUUIDv4函数来生成UUID值，UUID数据类型仅支持String数据类型也支持的函数（例如，min，max和count）。

#### 2.5.7 Date类型

> ClickHouse支持Date类型，这个日期类型用两个字节存储，表示从 1970-01-01 (无符号) 到当前的日期值。允许存储从 Unix 纪元开始到编译阶段定义的上限阈值常量（目前上限是2106年，但最终完全支持的年份为2105），最小值输出为0000-00-00。日期类型中不存储时区信息。

#### 2.5.8 DateTime类型

> ClickHouse支持DataTime类型，这个时间戳类型用四个字节（无符号的）存储Unix时间戳。允许存储与日期类型相同范围内的值，最小值为0000-00-00 00:00:00。时间戳类型值精确到（不包括闰秒）。

> 使用客户端或服务器时的系统时区，时间戳是从文本转换为二进制并返回。在文本格式中，有关夏令时的信息会丢失。

> 默认情况下，客户端连接到服务的时候会使用服务端时区。您可以通过启用客户端命令行选项`--use_client_time_zone` 来设置使用客户端时间。

> 因此，在处理文本日期时（例如，在保存文本转储时），请记住在夏令时更改期间可能存在歧义，如果时区发生更改，则可能存在匹配数据的问题。

#### 2.5.9 枚举类型

> ClickHouse支持`Enum8和Enum16`两种枚举类型。Enum保存的是'string'=integer的对应关系。在 ClickHouse中，尽管用户使用的是字符串常量，但所有含有Enum 数据类型的操作都是按照包含整数的值来执行，这在性能方面比使用String数据类型更有效。

> 在ORDER BY、GROUP BY、IN、DISTINCT等函数中，Enum 的行为与相应的数字作用相同。例如，按数字排序。对于等式运算符和比较运算符，Enum 的工作机制与它们在底层数值上的工作机制相同。

> Enum中的字符串和数值都不允许为NULL，当声明表字段时使用Nullable类型包含Enum类型时，在插入数据时允许NULL值。Enum类型提供toString函数来返回字符串值；toT函数可以转换为数值类型，T表示一个数值类型，如果T恰好对应Enum底层的数值类型则这个转换是0成本的。Enum类型可以使用Alter无成本修改对应集合的值，可以使用Alter来添加或删除Enum的成员（出于安全保障，如果改变之前用过的Enum会报异常），也可以用Alter将Enum8转换为Enum16或反之。

数据类型 | String=Integer对应关系 | 取值范围
---|---|---
Enum8 | 'String'= Int8 | -128 ~ 127
Enum16 | 'String'= Int16 | -32768 ~ 32767

> 创建tbl_test_enum表

```
create table tbl_test_enum(e1 Enum8('male'=1, 'female'=2),e2 Enum16('hello'=1,'word'=2), e3 Nullable(Enum8('A'=1, 'B'=2)),e4 Nullable(Enum16('a'=1,'b'=2))) engine=TinyLog;
```

> 插入字符串数据

```
insert into tbl_test_enum values('male', 'hello', 'A', null),('male', 'word', null, 'a');
insert into tbl_test_enum values(2, 1, 'C', null);
```

> 查询结果

```
select * from tbl_test_enum;
```

#### 2.5.10 数组类型

> ClickHouse支持Array(T)类型，T可以是任意类型，包括数组类型，但不推荐使用多维数组，因为对其的支持有限（MergeTree引擎表不支持存储多维数组）。T要求是兼容的数据类型，因为ClickHouse会自动检测并根据元素内容计算出存储这些数据的最小数据类型,如:不能使用array(1,’hello’)。

> 数组声明的两种方式

```
select array(1,3,5) as arr1,[2,4,6] as arr2, toTypeName(arr1) as arrType1, toTypeName(arr2) as arrType2;
```

#### 2.5.11 AggregateFunction类型

```
create table aggMT (whatever Date default '2019-12-18',key String,value String,first AggregateFunction(min, DateTime),last AggregateFunction(max, DateTime),total AggregateFunction(count,UInt64)) ENGINE=AggregatingMergeTree(whatever,(key,value),8192);
insert into aggMT (key,value,first,last,total) select 'test','1.2.3.4',minState(toDateTime(1576654217)),maxState(toDateTime(1576654217)),countState(cast(1 as UInt64));
insert into aggMT (key,value,first,last,total) select 'test','1.2.3.5',minState(toDateTime(1576654261)),maxState(toDateTime(1576654261)),countState(cast(1 as UInt64));
insert into aggMT (key,value,first,last,total) select 'test','1.2.3.6',minState(toDateTime(1576654273)),maxState(toDateTime(1576654273)),countState(cast(1 as UInt64));
select key, value,minMerge(first),maxMerge(last),countMerge(total) from aggMT group by key, value;
```

#### 2.5.12 元组类型

> ClickHouse提供Tuple类型支持，Tuple(T1,T2...)中每个元素都可以是单独的类型。除了内存表以外，元组中不可以嵌套元组，但可以用于临时列分组。在查询中，使用IN表达式和带特定参数的lambda函数可以来对临时列进行分组。元组可以是查询的结果。在这种情况下，对于JSON以外的文本格式，括号中的值是逗号分隔的。在JSON格式中，元组作为数组输出（在方括号中）。在动态创建元组时，ClickHouse 会自动为元组的每一个参数赋予最小可表达的类型。如果参数值为NULL则这个元组对应元素类型是Nullable。

> 使用元组的例子1：

```
select tuple(1, 'a') as x, toTypeName(x);
```

> 使用元组传入null值时自动推断类型例子2：

```
select tuple(1, null) as x, toTypeName(x);
```

#### 2.5.13 Nullable类型

> ClickHouse支持Nullable类型，该类型允许用NULL来表示缺失值。Nullable字段不能作为索引列使用，在ClickHouse的表中存储Nullable列时，会对性能产生一定影响。

> 默认情况下，字段是不允许为NULL的。例如有个Int8类型的字段，在插入数据时有可能为NULL，需要将字段类型声明为Nullable(Int8)。

> 创建测试表tbl_test_nullable

```
create table tbl_test_nullable(f1 String, f2 Int8, f3 Nullable(Int8)) engine=TinyLog;
```

> 插入非null值到tbl_test_nullable表(成功)

```
insert into tbl_test_nullable(f1,f2,f3) values('NoNull',1,1);
```

> f1字段为null值时插入到tbl_test_nullable表(失败)

```
insert into tbl_test_nullable(f1,f2,f3) values(null,2,2);
```

> f2字段为null值时插入到tbl_test_nullable表(失败)

```
insert into tbl_test_nullable(f1,f2,f3) values('NoNull2',null,2);
```

> f3字段为null值时插入到tbl_test_nullable表(成功)

```
insert into tbl_test_nullable(f1,f2,f3) values('NoNull2',2,null);
```

> 查询tbl_test_nullable表(有2条记录)

```
select * from tbl_test_nullable;
```

#### 2.5.14 嵌套数据结构

> ClickHouse支持嵌套数据结构，可以简单地把嵌套数据结构当做是所有列都是相同长度的多列数组。创建表时，可以包含任意多个嵌套数据结构的列，但嵌套数据结构的列仅支持一级嵌套。嵌套列在insert时，需要把嵌套列的每一个字段以[要插入的值]格式进行数据插入。

> 创建带嵌套结构字段的表

```
create table tbl_test_nested(uid Int64, ctime date, user Nested(name String, age Int8, phone Int64), Sign Int8) engine=CollapsingMergeTree(ctime,intHash32(uid),(ctime,intHash32(uid)),8192,Sign);
```

> 插入数据

```
insert into tbl_test_nested values(1,'2019-12-25',['zhangsan'],[23],[13800138000],1);
```

> 查询uid=1并且user嵌套列的age>=20的数据

```
select * from tbl_test_nested where uid=1 and arrayFilter(u -> u >= 20, user.age) != [];
```

> 查询user嵌套列name=zhangsan的数据

```
select * from tbl_test_nested where hasAny(user.name,['zhangsan']);
```

>模糊查询user嵌套列name=zhang的数据

```
select * from tbl_test_nested where arrayFilter(u -> u like '%zhang%', user.name) != [];
```

#### 2.5.15 interval

> Interval是ClickHouse提供的一种特殊的数据类型，此数据类型用来对Date和Datetime进行运算，不能使用Interval类型声明表中的字段。

> Interval支持的时间类型有SECOND、MINUTE、HOUR、DAY、WEEK、MONTH、QUARTER和YEAR。对于不同的时间类型参数，都有一个单独的数据类型，如下表格。

时间类型参数 | 查询Interval类型 | Interval类型
---|---|---
SECOND | SELECT toTypeName(INTERVAL 4 SECOND); | IntervalSecond
MINUTE | SELECT toTypeName(INTERVAL 4 MINUTE); | IntervalMinute
HOUR | SELECT toTypeName(INTERVAL 4 HOUR); | IntervalHour
DAY | SELECT toTypeName(INTERVAL 4 DAY); | IntervalDay
WEEK | SELECT toTypeName(INTERVAL 4 WEEK); | IntervalWeek
MONTH | SELECT toTypeName(INTERVAL 4 MONTH); | IntervalMonth
QUARTER | SELECT toTypeName(INTERVAL 4 QUARTER); | IntervalQuarter
YEAR | SELECT toTypeName(INTERVAL 4 YEAR); | IntervalYear

> 获取当前时间+3天的时间

```
select now() as cur_dt, cur_dt + interval 4 DAY plus_dt;
```

> 获取当前时间+4天+3小时的时间

```
select now() as cur_dt, cur_dt + interval 4 DAY + interval 3 HOUR as plus_dt;
```

#### 2.5.16 IPv4类型与IPv6类型

> ClickHouse支持IPv4和Ipv6两种Domain类型，Ipv4类型是与UInt32类型保持二进制兼容的Domain类型，其用于存储IPv4地址的值；

> IPv6是与FixedString(16)类型保持二进制兼容的Domain类型，其用于存储IPv6地址的值。

> 这两种Domain类型提供了更为紧凑的二进制存储的同时支持识别可读性更加友好的输入输出格式。

> 创建tbl_test_domain表

```
create table tbl_test_domain(url String, ip4 IPv4, ip6 IPv6) ENGINE = MergeTree() ORDER BY url;
```

> 插入IPv4和IPv6类型字段数据到tbl_test_domain表

```
insert into tbl_test_domain(url,ip4,ip6) values('http://www.xiaoma.cn','127.0.0.1','2a02:aa08:e000:3100::2');
```

> 查询tbl_test_domain表数据

```
select * from tbl_test_domain;
```

> 查询类型和二进制格式

```
select url,toTypeName(ip4) as ip4Type, hex(ip4) as ip4Hex,toTypeName(ip6) as ip6Type, hex(ip6) as ip6Hex from tbl_test_domain;
```

> 使用IPv4NumToString和IPv6NumToString将Domain类型转换为字符串

```
select url,IPv4NumToString(ip4) as ip4Str,IPv6NumToString(ip6) as ip6Str from tbl_test_domain;
```

#### 2.5.17 默认值处理

> 在ClickHouse中，对于某些类型的列，在没有显示插入值时，会自动填充默认值处理。

数据类型 | 默认值
---|---
Int和Uint | 0
String | 空字符串
Array | 空数组
Date | 0000-00-00
DateTime | 0000-00-00 00:00:00
NULL | 不支持

### 2.6 ClickHouse的引擎

> ClickHouse提供了多种不同的表引擎，表引擎可以简单理解为不同类型的表。

> 表引擎（即表的类型）决定了：
>> 数据的存储方式和位置，写到哪里以及从哪里读取数据
> 
>> 支持哪些查询以及如何支持
> 
>> 并发数据访问
> 
>> 索引的使用（如果存在）
> 
>> 是否可以执行多线程请求
> 
>> 数据复制参数

> 下面介绍其中几种，对其他引擎有兴趣的可以去查阅官方文档
>> https://clickhouse.tech/docs/zh/engines/table-engines/


#### 2.6.1 日志引擎

##### 2.6.1.1 TinyLog引擎

> 最简单的表引擎，用于将数据存储在磁盘上。每列都存储在单独的压缩文件中，写入时，数据将附加到文件末尾。

> 该引擎没有并发控制
>> 如果同时从表中读取和写入数据，则读取操作将抛出异常；
> 
>> 如果同时写入多个查询中的表，则数据将被破坏。

> 这种表引擎的典型用法是 `write-once`：首先只写入一次数据，然后根据需要多次读取。此引擎适用于相对较小的表（建议最多1,000,000行）。如果有许多小表，则使用此表引擎是适合的，因为它比需要打开的文件更少。当拥有大量小表时，可能会导致性能低下。不支持索引。

> 案例：创建一个TinyLog引擎的表并插入一条数据

```
create table user (id UInt16, name String) ENGINE=TinyLog;
insert into user (id, name) values (1, 'zhangsan');
```

> 此时我们到保存数据的目录/var/lib/clickhouse/data/default/user中可以看到如下目录结构：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/3.png)

> id.bin 和 name.bin 是压缩过的对应的列的数据，sizes.json 中记录了每个 *.bin 文件的大小：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/4.png)

#### 2.6.2 数据库引擎

> ClickHouse提供了本机、MySQL和Lazy这3种数据库引擎，但在默认情况下仅使用其本机数据库引擎，该引擎提供可配置的表引擎(MergeTree、Log和Intergation)和SQL方言(完整的SQL解析器，即递归下降解析器；数据格式解析器，即快速流解析器)。还可以使用MySQL和Lazy。

##### 2.6.2.1 MySQL引擎

> `MySQL引擎用于将远程的MySQL服务器中的表映射到ClickHouse中`，并允许您对表进行`INSERT`和`SELECT`查询，以方便您`在ClickHouse与MySQL之间进行数据交换`。

> MySQL数据库引擎会将对其的查询转换为MySQL语法并发送到MySQL服务器中，因此您可以执行诸如SHOW TABLES或SHOW CREATE TABLE之类的操作。

> 但您无法对其执行以下操作：
>> RENAME
> 
>> CREATE TABLE
> 
>> ALTER

> 语法结构：
>> 创建Mysql引擎的语法

```
CREATE DATABASE [IF NOT EXISTS] db_name [ON CLUSTER cluster]
ENGINE = MySQL('host:port', ['database' | database], 'user', 'password')
```

> MySQL数据库引擎参数

```
host:port— 链接的MySQL地址。
database— 链接的MySQL数据库。
user— 链接的MySQL用户。
password— 链接的MySQL用户密码。
```

> 使用示例：
>> 在MySQL中创建表:

```
mysql> USE test;
Database changed
mysql> CREATE TABLE `mysql_table` (
->   `int_id` INT NOT NULL AUTO_INCREMENT,
->   `float` FLOAT NOT NULL,
->   PRIMARY KEY (`int_id`));

Query OK, 0 rows affected (0,09 sec)

mysql> insert into mysql_table (`int_id`, `float`) VALUES (1,2);
Query OK, 1 row affected (0,00 sec)

mysql> select * from mysql_table;
+--------+-------+
| int_id | value |
+--------+-------+
|      1 |     2 |
+--------+-------+
1 row in set (0,00 sec)
```

> 在ClickHouse中创建MySQL类型的数据库，同时与MySQL服务器交换数据：

```
CREATE DATABASE mysql_db ENGINE = MySQL('node1.xiaoma.cn:3306', 'test', 'root', '123456')
SHOW DATABASES;
┌─name─────┐
│ default  │
│ mysql_db │
│ system   │
└──────────┘
SHOW TABLES FROM mysql_db
┌─name─────────┐
│  mysql_table │
└──────────────┘
SELECT * FROM mysql_db.mysql_table
┌─int_id─┬─value─┐
│      1 │     2 │
└────────┴───────┘
INSERT INTO mysql_db.mysql_table VALUES (3,4)
SELECT * FROM mysql_db.mysql_table
┌─int_id─┬─value─┐
│      1 │     2 │
│      3 │     4 │
└──────┴─────┘
```

#### 2.6.3 MergeTree系列引擎(重点)

> `MergeTree（合并树）`系列引擎是ClickHouse中最强大的表引擎，`是官方主推的存储引擎`，几乎支持ClickHouse所有的核心功能。

> 该系列引擎主要用于海量数据分析的场景，支持对表数据进行`分区、复制、采样、存储有序、主键索引、稀疏索引和数据TTL`等特性。

> MergeTree系列引擎的基本理念是`当有大量数据要插入到表中时，需要高效地一批一批的写入数据片段，并希望这些数据片段在后台按照一定规则合并，这种方法比插入期间连续重写存储中的数据效率更高。`

> 简而言之就是具有批量数据快速插入和后台并发处理的优势。

> MergeTree系列引擎支持ClickHouse所有的SQL语法，但还是有一些SQL语法和MySQL并不太一样。

> MergeTree系列引擎包括：
>> `ReplacingMergeTree`
>> `SummingMergeTree`
>> `AggregatingMergeTree`
>> `CollapsingMergeTree`
>> `VersionedCollapsingMergeTree`
>> `GraphiteMergeTree`

##### 2.6.3.1 MergeTree

> MergeTree引擎的表的允许插入主键重复的数据，`主键主要作用是生成主键索引来提升查询效率，而不是用来保持记录主键唯一`

> `创建MergeTree表的说明`

> 创建MergeTree引擎表的语法

```
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
...
INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = MergeTree()
[PARTITION BY expr]
[ORDER BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'], ...]
[SETTINGS name=value, ...]
```

子句说明 | 使用方式
---|---
ENGINE | ENGINE = MergeTree()  --说明：该引擎不需要参数。
PARTITION BY 字段名称 | PARTITION by to YYYYMM(cdt)
ORDER BY 字段名称(可以是元组) | ORDER BY cdt或ORDER BY (age,gender)
PRIMARY KEY 字段名称 | PRIMARY KEY age
SAMPLE BY 字段名称 | SAMPLE BY intHash64(userId)
TTL Date字段或DateTime字段 | TTL cdt + INTERVAL 1 DAY
SETTINGS param=value... |SETTINGS index_granularity=8192<br/>说明：索引粒度。即索引中相邻”标记”间的数据行数。设为 8192 可以适用大部分场景。<br/><br/>SETTINGS index_granularity_bytes=<br/>说明：设置数据粒度的最大大小(单位/字节)，默认10MB。从大行（数十和数百MB）的表中select数据时，此设置可提高ClickHouse的提高select性能。<br/><br/>enable_mixed_granularity_parts<br/>说明：启用或禁用过渡。<br/><br/>use_minimalistic_part_header_in_zookeeper<br/>说明：在ZK中存储数据部分标题，0是关闭，1是存储的少量数据。<br/><br/>min_merge_bytes_to_use_direct_io<br/>说明：使用对存储磁盘的直接I / O访问所需的最小合并操作数据量。合并数据部分时，ClickHouse会计算要合并的所有数据的总存储量。如果卷超过min_merge_bytes_to_use_direct_io字节，ClickHouse将使用直接I/O接口（O_DIRECT选项）读取数据并将数据写入存储磁盘。如果为min_merge_bytes_to_use_direct_io = 0，则直接I / O被禁用。<br/>默认值：10 * 1024 * 1024 * 1024字节。<br/><br/>merge_with_ttl_timeout<br/>说明：与TTL合并之前的最小延迟(单位/秒)，默认86400。<br/><br/>write_final_mark<br/>说明：启用或禁用在数据部分末尾写入最终索引标记，默认1。建议不关闭此设置。<br/><br/>storage_policy<br/>说明：存储策略。

> `创建MergeTree引擎的表`

> 创建MergeTree引擎表有两种方式，`一种是集群表，一种是本地表`。

> 创建使用MergeTree引擎的集群表test.tbl_testmergetree_users_all,集群表一般都携带_all后缀，而且必须所有节点都存在test数据库，这样所有节点的test库中都有tbl_testmergetree_users_all表。

```
CREATE TABLE test.tbl_test_mergetree_users_all ON cluster 'ch_cluster'(
id UInt64,
email String,
username String,
gender UInt8,
birthday Date,
mobile FixedString(13),
pwd String,
regDT DateTime,
lastLoginDT DateTime,
lastLoginIP String
) ENGINE=MergeTree() partition by toYYYYMMDD(regDT) order by id settings index_granularity=8192;
```


> 创建使用MergeTree引擎的本地表test.tbl_test_mergetree_users

```
CREATE TABLE tbl_test_mergetree_users(
id UInt64,
email String,
username String,
gender UInt8,
birthday DATE,
mobile FixedString(13),
pwd String,
regDT DateTime,
lastLoginDT DateTime,
lastLoginIP String
) ENGINE=MergeTree() partition by toYYYYMMDD(regDT) order by id settings index_granularity=8192;
```

> `插入数据到MergeTree引擎的表`

> 1、测试数据集

```
values (1,'wcfr817e@yeah.net','督咏',2,'1992-05-31','13306834911','7f930f90eb6604e837db06908cc95149','2008-08-06 11:48:12','2015-05-08 10:51:41','106.83.54.165'),(2,'xuwcbev9y@ask.com','上磊',1,'1983-10-11','15302753472','7f930f90eb6604e837db06908cc95149','2008-08-10 05:37:32','2014-07-28 23:43:04','121.77.119.233'),(3,'mgaqfew@126.com','涂康',1,'1970-11-22','15200570030','96802a851b4a7295fb09122b9aa79c18','2008-08-10 11:37:55','2014-07-22 23:45:47','171.12.206.122'),(4,'b7zthcdg@163.net','金俊振',1,'2002-02-10','15207308903','96802a851b4a7295fb09122b9aa79c18','2008-08-10 14:47:09','2013-12-26 15:55:02','61.235.143.92'),(5,'ezrvy0p@163.net','阴福',1,'1987-09-01','13005861359','96802a851b4a7295fb09122b9aa79c18','2008-08-12 21:58:11','2013-12-26 15:52:33','182.81.200.32'),(6,'juestiua@263.net','岑山裕',1,'1996-02-12','13008315314','96802a851b4a7295fb09122b9aa79c18','2008-08-14 05:48:16','2013-12-26 15:49:12','36.59.3.248'),(7,'oyyrzd@yahoo.com.cn','姚茗咏',2,'1977-02-06','13303203846','96e79218965eb72c92a549dd5a330112','2008-08-15 10:07:31','2013-12-26 15:55:05','106.91.23.177'),(8,'lhrwkwwf@163.com','巫红影',2,'1996-02-21','15107523748','96802a851b4a7295fb09122b9aa79c18','2008-08-15 14:37:41','2013-12-26 15:55:05','123.234.85.27'),(9,'v2zqak8kh@0355.net','空进',1,'1974-01-16','13903178080','96802a851b4a7295fb09122b9aa79c18','2008-08-16 03:24:44','2013-12-26 15:52:42','121.77.192.123'),(10,'mqqqmf@yahoo.com','西香',2,'1980-10-13','13108330898','96802a851b4a7295fb09122b9aa79c18','2008-08-16 04:42:08','2013-12-26 15:55:08','36.57.21.234'),(11,'sf8oubu@yahoo.com.cn','壤晶媛',2,'1976-03-05','15202615557','96802a851b4a7295fb09122b9aa79c18','2008-08-16 06:08:51','2013-12-26 15:55:03','182.83.220.201'),(12,'k6k21ce@qq.com','东平',1,'2005-04-25','13603648382','96802a851b4a7295fb09122b9aa79c18','2008-08-16 06:18:20','2013-12-26 15:55:05','210.34.111.155'),(13,'zguxgg@qq.com','夹影悦',2,'2002-08-23','15300056290','96802a851b4a7295fb09122b9aa79c18','2008-08-16 06:57:45','2013-12-26 15:55:02','61.232.211.180'),(14,'g2jqhbzrf@aol.com','生晓怡',2,'1974-06-22','13507515420','96802a851b4a7295fb09122b9aa79c18','2008-08-16 08:23:43','2013-12-26 15:55:02','182.86.5.162'),(15,'1evn3spn@126.com','咎梦',2,'1998-04-14','15204567060','060ed80051e6384b77ddfaa26191778b','2008-08-16 08:29:57','2013-12-26 15:55:02','210.30.171.70'),(16,'tqndz6l@googlemail.com','司韵',2,'1992-08-28','15202706709','96802a851b4a7295fb09122b9aa79c18','2008-08-16 14:34:25','2013-12-26 15:55:03','171.10.115.59'),(17,'3472gs22x@live.com','李言',1,'1997-09-08','15701526600','96802a851b4a7295fb09122b9aa79c18','2008-08-16 15:04:07','2013-12-26 15:52:39','171.14.80.71'),(18,'p385ii@gmail.com','詹芸',2,'2004-11-05','15001974846','96802a851b4a7295fb09122b9aa79c18','2008-08-16 15:26:06','2013-12-26 15:55:02','182.89.147.245'),(19,'mfbncfu@yahoo.com','蒙芬霞',2,'1990-09-10','15505788156','96802a851b4a7295fb09122b9aa79c18','2008-08-16 15:30:58','2013-12-26 15:55:05','182.91.15.89'),(20,'l24ffbn@ask.com','后冠',1,'2000-09-02','15608241150','96802a851b4a7295fb09122b9aa79c18','2008-08-17 01:15:55','2014-08-29 00:51:12','36.58.7.85'),(21,'m26lggpe@qq.com','宋美月',2,'2003-01-13','15606561425','96802a851b4a7295fb09122b9aa79c18','2008-08-17 01:24:09','2013-12-26 15:52:36','123.235.233.160'),(22,'ndmfm13qf@0355.net','邬玲',2,'2002-08-11','13207844859','96802a851b4a7295fb09122b9aa79c18','2008-08-17 03:31:11','2013-12-26 15:55:03','36.60.8.4'),(23,'5shmvnd@sina.com','乐心有',1,'1998-05-01','13201212693','96802a851b4a7295fb09122b9aa79c18','2008-08-17 03:33:41','2013-12-26 15:55:02','123.234.184.210'),(24,'pwa0hu@3721.net','任学诚',1,'1978-03-19','15802040152','7f930f90eb6604e837db06908cc95149','2008-08-17 07:24:01','2013-12-26 15:52:34','210.43.167.14'),(25,'1ybjhul@googlemail.com','巫纨',2,'1995-01-20','15900530105','96802a851b4a7295fb09122b9aa79c18','2008-08-17 07:48:06','2013-12-26 15:55:02','222.55.139.104'),(26,'b31me2i8b@yeah.net','石娅',2,'2000-02-25','13908735198','96802a851b4a7295fb09122b9aa79c18','2008-08-17 08:17:24','2013-12-26 15:52:45','123.235.99.123'),(27,'qgb2w4n7@163.net','柏菁',2,'1975-02-09','15306552661','96802a851b4a7295fb09122b9aa79c18','2008-08-17 08:47:39','2013-12-26 15:55:02','121.77.245.202'),(28,'cfb3ck@sohu.com','鲜梦',2,'1974-01-26','13801751668','96802a851b4a7295fb09122b9aa79c18','2008-08-17 08:55:47','2013-12-26 15:55:02','210.26.163.24'),(29,'nfrf6mp@msn.com','鄂珍',2,'1974-04-14','13300247433','96802a851b4a7295fb09122b9aa79c18','2008-08-17 09:02:14','2013-12-26 15:55:08','210.31.214.157'),(30,'o1isumh@126.com',' 法姬',2,'1978-06-16','15607848127','96802a851b4a7295fb09122b9aa79c18','2008-08-17 09:09:59','2013-12-26 15:55:08','222.24.34.19'),(31,'y2wrclkq@msn.com','太以',1,'1998-09-07','13608585923','96802a851b4a7295fb09122b9aa79c18','2008-08-17 11:35:39','2013-12-26 15:52:35','182.89.218.177'),(32,'fv9avnuo@263.net','庚姣欣',2,'1982-09-14','13004625187','96802a851b4a7295fb09122b9aa79c18','2008-08-17 12:50:36','2013-12-26 15:55:02','106.82.225.130'),(33,'o1e96z@yahoo.com','微伟',1,'1981-07-30','13707663880','96802a851b4a7295fb09122b9aa79c18','2008-08-17 15:12:05','2013-12-26 15:49:12','171.13.152.247'),(34,'cm3oz64ja@msn.com','那竹娜',2,'1989-01-09','13607294767','96802a851b4a7295fb09122b9aa79c18','2008-08-17 15:51:08','2013-12-26 15:55:02','171.13.110.67'),(35,'g7impl@msn.com','闾和栋',1,'1994-10-12','13907368366','96802a851b4a7295fb09122b9aa79c18','2008-08-17 16:51:02','2013-12-26 15:55:01','210.28.163.83'),(36,'jz2fjtt@163.com','夏佳悦',2,'2001-03-17','15102554998','7af1b63f0d1f37c35c1274339c12b6a8','1970-01-01 08:00:00','1970-01-01 08:00:00','222.91.138.221'),(37,'klwrtomws@yahoo.com','南义梁',1,'1981-03-19','15105745902','96802a851b4a7295fb09122b9aa79c18','2008-08-18 01:49:17','2013-12-26 15:55:01','36.62.155.17'),(38,'yhzs1nnlk@3721.net','牧元',1,'2001-06-07','13501780163','96802a851b4a7295fb09122b9aa79c18','2008-08-18 04:24:52','2013-12-26 15:55:01','171.15.184.130'),(39,'hem76ot33@gmail.com','凌伟文',1,'1988-03-04','13201417535','96802a851b4a7295fb09122b9aa79c18','2008-08-18 05:34:52','2013-12-26 15:55:14','61.237.105.3'),(40,'ndp40j@sohu.com','弘枝',2,'2000-09-05','13001236425','96802a851b4a7295fb09122b9aa79c18','2008-08-18 06:23:48','2013-12-26 15:55:01','106.82.172.45'),(41,'zeyacpr@gmail.com','台凡',2,'1998-05-26','15102913418','96802a851b4a7295fb09122b9aa79c18','2008-08-18 06:41:24','2013-12-26 15:55:07','123.233.69.218'),(42,'0ts0wiz@aol.com','任晓红',2,'1984-05-02','13502366778','96802a851b4a7295fb09122b9aa79c18','2008-08-18 06:55:16','2013-12-26 15:55:01','210.26.44.18'),(43,'zi7dhzo@googlemail.com','蔡艺艳',2,'1990-08-07','15603954810','96802a851b4a7295fb09122b9aa79c18','2008-08-18 06:57:30','2013-12-26 15:55:01','171.12.171.179'),(44,'b0yfzilu@hotmail.com','郎诚',1,'1994-05-18','13602127171','96802a851b4a7295fb09122b9aa79c18','2008-08-18 07:02:04','2013-12-26 15:55:02','171.8.22.92'),(45,'er9az5e9s@163.com','台翰',1,'1994-06-22','15900953220','96802a851b4a7295fb09122b9aa79c18','2008-08-18 07:05:08','2013-12-26 15:55:14','222.31.141.156'),(46,'e34jy2@yeah.net','彭筠',2,'1983-08-12','15106915420','96802a851b4a7295fb09122b9aa79c18','2008-08-18 07:09:37','2013-12-26 15:52:34','36.60.51.67'),(47,'1u0zc56h@163.net','包华婉',2,'1998-10-03','13102518450','96802a851b4a7295fb09122b9aa79c18','2008-08-18 07:47:24','2013-12-26 15:55:02','121.76.76.105'),(48,'cs8kyk3@ask.com','淳盛',1,'2002-06-19','13203151569','96802a851b4a7295fb09122b9aa79c18','2008-08-18 08:01:58','2013-12-26 15:55:02','36.60.76.111'),(49,'ibcgi5ll@yahoo.com','车珍枫',2,'1975-07-27','15605361319','96802a851b4a7295fb09122b9aa79c18','2008-08-18 08:12:45','2013-12-26 15:55:01','106.83.110.76'),(50,'gzxcx6vz@live.com','应冰红',2,'2004-04-19','15104154370','96802a851b4a7295fb09122b9aa79c18','2008-08-18 09:00:09','2013-12-26 15:55:01','182.88.181.216');
```

> 2、插入数据到`集群表`test.tbl_test_mergetree_users_all

> 使用SQL语句

```
insert into test.tbl_test_mergetree_users_all(id, email, username, gender, birthday, mobile, pwd, regDT, lastLoginDT, lastLoginIP)
```

> 追加上面的values语句进行数据插入。

> 50条数据是按照分区字段regDT划分片段，然后每个段中是根据排序字段id默认使用ASC进行排序的。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/5.png)

> 查询符合email模糊匹配gmail并且id>30并且分区是20080818条件的数据。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/6.png)

> 这个表的物理存储是在我们设置的数据路径`/export/data/clickhouse`中，该路径下的/data/data下是ClickHouse节点中维护的数据库，对应下图中的default、system和test三个文件夹。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/7.png)

> 然后在test文件夹下，有一个tbl_test_mergetree_users_all文件夹（是我们自己创建的表），该文件夹中有很多日期类型的文件夹（我们创建表时指定的分区字段的值），在此文件夹下有很多具体的数据文件。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/8.png)

> 这些数据文件中，columns.txt表示`tbl_test_mergetree_users_all`表20080816_29_29_0分区下所有数据的列信息，如图：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/9.png)

> 这些数据文件中，count.txt表示`tbl_test_mergetree_users_all`表20080816_29_29_0分区下所有数据的总条数，如图：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/10.png)

> 这些数据文件中的*.bin和*.mrk2指的是列字段的数据信息，*.idx指的是索引字段信息。

> 3、插入数据到`本地表`test.tbl_test_mergetree_users

> 使用SQL语句

```
insert into tbl_test_mergetree_users(id, email, username, gender, birthday, mobile, pwd, regDT, lastLoginDT, lastLoginIP)
```

> 追加上面的values语句进行数据插入。

> 这个本地表的物理存储与上面的集群表的存储路径和文件类似。

> 进入本地路径：cd /var/lib/clickhouse/data/default/tbl_test_mergetree_users

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/11.png)

> 随便进入一个目录：cd 20080818_9_9_0

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/12.png)

```
- *.bin是按列保存数据的文件
- *.mrk保存块偏移量
- primary.idx保存主键索引
```

> `删除MergeTree引擎的表`

> 1、删除集群中所有节点的tbl_test_mergetree_users_all表

```
drop table test.tbl_test_mergetree_users_all on cluster 'ch_cluster';
```

> 2、删除tbl_test_mergetree_users本地表

```
drop table tbl_test_mergetree_users;
```

##### 2.6.3.2 ReplacingMergeTree

> `为了解决MergeTree相同主键无法去重的问题`，ClickHouse提供了`ReplacingMergeTree`引擎，用来对主键重复的数据进行去重。

> 删除重复数据可以使用`optimize`命令手动执行，这个合并操作是在后台运行的，且无法预测具体的执行时间

> 在使用optimize命令执行合并时，如果表数据量过大，会导致耗时很长，此时表将是不可用的，因为optimize会通过读取和写入大量数据来完成合并操作。

> ReplacingMergeTree适合在后台清除重复数据以节省空间，但不能保证不存在重复数据。在没有彻底optimize之前，可能无法达到主键去重的效果，比如部分数据已经被去重，而另外一部分数据仍旧存在主键重复的情况。在分布式场景下，相同主键的数据可能被分片到不同节点上，不同分片间无法去重。ReplacingMergeTree更多的被用于确保数据最终被去重，而无法保证查询过程中主键不重复。

> `创建ReplacingMergeTree表的说明`

> 1、创建ReplacingMergeTree引擎表的语法

```
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
...
) ENGINE = ReplacingMergeTree([ver])
[PARTITION BY expr]
[ORDER BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]
```

子句说明 | 使用方式
---|---
ver参数 | ReplacingMergeTree([ver])中的ver参数是可选的，指带有版本的列，这个列允许使用UInt*、Date或DateTime类型。ReplacingMergeTree在合并时会把具有相同主键的所有行仅保留一个。如果不指定ver参数则保留最后一次插入的数据。

> `创建ReplacingMergeTree引擎的表`

> 1、创建ReplacingMergeTree引擎的本地表tbl_test_replacing_mergetree_users

```
CREATE TABLE tbl_test_replacingmergetree_users (
id UInt64,
email String,
username String,
gender UInt8,
birthday Date,
mobile FixedString(13),
pwd String,
regDT DateTime,
lastLoginDT DateTime,
lastLoginIP String
) ENGINE=ReplacingMergeTree(id) partition by toYYYYMMDD(regDT) order by id settings index_granularity=8192;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/13.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/14.png)

> `插入数据到ReplacingMergeTree引擎的表`

> 插入数据到表tbl_test_replacingmergetree_users

> 使用SQL语句插入数据：

```
insert into tbl_test_replacingmergetree_users select * from tbl_test_mergetree_users where id<=5;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/15.png)

> 插入重复数据（`使用lastLoginDT来区分数据插入的先后顺序`）：

```
insert into tbl_test_replacingmergetree_users(id,email,username,gender,birthday,mobile,pwd,regDT,lastLoginIP,lastLoginDT) select id,email,username,gender,birthday,mobile,pwd,regDT,lastLoginIP,now() as lastLoginDT from tbl_test_mergetree_users where id<=3;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/16.png)

> 再次查询表中全量数据:

```
select * from tbl_test_replacingmergetree_users order by id,lastLoginDT;
```

> `明显看到id是1、2、3的数据各有两条，说明目前已经存在3条重复数据，且新增的3条数据的lastLoginDT都是2019-12-04 14:55:56。`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/17.png)

> 现在使用optimize命令执行合并操作，使表中主键id字段的重复数据由现在的6条变成3条：

```
optimize table tbl_test_replacingmergetree_users final;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/18.png)

> 再次查询

```
select * from tbl_test_replacingmergetree_users;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/19.png)

> 发现主键id字段为1、2、3的重复数据已经合并了，且合并后保留了最后一次插入的3条数据，因为最后插入的3条记录的时间是2019-12-04 14:55:56。

> 删除表

```
drop table tbl_test_replacingmergetree_users;
```

##### 2.6.3.3 SummingMergeTree

> `ClickHouse通过SummingMergeTree来支持对主键列进行预聚合`。在后台合并时，会将主键相同的多行进行sum求和，然后使用一行数据取而代之，从而大幅度降低存储空间占用，提升聚合计算性能。

> ClickHouse只在后台`Compaction`时才会进行数据的预先聚合，而compaction的执行时机无法预测，所以可能会存在一部分数据已经被预先聚合，但仍有一部分数据尚未被聚合的情况。

> 因此在执行聚合计算时，SQL中仍需要使用GROUP BY子句来保证sum的准确。

> 在预聚合时，ClickHouse会对主键列以外的其他所有列进行预聚合。但这些列必须是数值类型才会计算sum（当sum结果为0时会删除此行数据）；如果是String等不可聚合的类型，则随机选择一个值。

> `通常建议将SummingMergeTree与MergeTree配合使用，使用MergeTree来存储明细数据，使用SummingMergeTree存储预聚合的数据来支撑加速查询。`

> 创建SummingMergeTree引擎表的的语法

```
语法结构
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
...
) ENGINE = SummingMergeTree([columns])
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]
```

> SummingMergeTree参数说明
>> SummingMergeTree([columns])中的[columns]参数是表中的列，是可选的，该列是要汇总值的列名称的元组。这些列必须是数字类型，并且不能在主键中。如果不指定该列参数，ClickHouse会使用数值数据类型汇总所有非主键列的sum值

> 创建SummingMergeTree引擎的tbl_test_summingmergetree表

```
create table tbl_test_summingmergetree(
key UInt64,
value UInt64
) engine=SummingMergeTree() order by key;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/20.png)

> 第一次插入数据

```
insert into tbl_test_summingmergetree(key,value) values(1,13);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/21.png)

> 查询第一次插入的数据

```
select * from tbl_test_summingmergetree;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/22.png)

> 第二次插入重复数据

```
insert into tbl_test_summingmergetree(key,value) values(1,13);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/23.png)

> 查询表数据（有2条key=1的重复数据）

```
select * from tbl_test_summingmergetree;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/24.png)

> 第三次插入重复数据

```
insert into tbl_test_summingmergetree(key,value) values(1,16);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/25.png)

> 查询表数据（有3条key=1的重复数据）

```
select * from tbl_test_summingmergetree;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/26.png)

> 使用sum和count查询数据
>> sum函数用于计算value的和，count函数用于查看插入次数，group by用于保证是否合并完成都是准确的计算sum

```
select key,sum(value),count(value) from tbl_test_summingmergetree group by key;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/27.png)

> 手动触发重复数据的合并

```
optimize table tbl_test_summingmergetree final;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/28.png)

> 再次使用sum和count查询数据

```
select key,sum(value),count(value) from tbl_test_summingmergetree group by key;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/29.png)

> 结果集中key=1的count值变成1了，sum(value)的值是38。说明手动触发合并生效了。我们再来使用非聚合查询：

```
select * from tbl_test_summingmergetree;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/30.png)

> 此时，key=1的这条数据的确是合并完成了，由原来的3条变成1条了，而且value值的求和是正确的38。

##### 2.6.3.4 AggregatingMergeTree

> `AggregatingMergeTree也是预聚合引擎的一种，是在MergeTree的基础上针对聚合函数计算结果进行增量计算用于提升聚合计算的性能。`

> 与SummingMergeTree的区别在于：SummingMergeTree对非主键列进行sum聚合，而AggregatingMergeTree则可以指定各种聚合函数。

> AggregatingMergeTree表适用于增量数据聚合，包括聚合的物化视图。

> AggregatingMergeTree的语法比较复杂，需要结合物化视图或ClickHouse的特殊数据类型AggregateFunction一起使用。在insert和select时，也有独特的写法和要求：写入时需要使用-State语法，查询时使用-Merge语法。


> 创建AggregatingMergeTree引擎表的的语法

```
语法结构
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
...
) ENGINE = AggregatingMergeTree()
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[TTL expr]
[SETTINGS name=value, ...]
```

> 创建2张表
>> 创建用户行为表
> 
>> MergeTree引擎的用户行为表用来存储所有的用户行为数据，是后边AggregatingMergeTree引擎的UV和PV增量计算表的数据源。
> 
>> `因为AggregatingMergeTree的UV和PV增量计算表无法使用insert into tableName values语句插入，只能使用insert into tableName select语句才可以插入数据。`

```
-- 用户行为表
create table tbl_test_mergetree_logs(
guid String,
url String,
refUrl String,
cnt UInt16,
cdt DateTime
) engine = MergeTree() partition by toYYYYMMDD(cdt) order by toYYYYMMDD(cdt);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/31.png)

> 插入数据到用户行为表

```
insert into tbl_test_mergetree_logs(guid,url,refUrl,cnt,cdt) values('a','www.xiaoma.com','www.xiaoma.cn',1,'2019-12-17 12:12:12'),('a','www.xiaoma.com','www.xiaoma.cn',1,'2019-12-17 12:14:45'),('b','www.xiaoma.com','www.xiaoma.cn',1,'2019-12-17 13:13:13');
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/32.png)

> 查询用户行为表所有数据

```
select * from tbl_test_mergetree_logs;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/33.png)

> 创建UV和PV增量计算表

```
-- UV和PV增量计算表
create table tbl_test_aggregationmergetree_visitor(
guid String,
cnt AggregateFunction(count, UInt16),
cdt Date
) engine = AggregatingMergeTree() partition by cdt order by cnt;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/34.png)

> 插入数据到UV和PV增量计算表

```
insert into tbl_test_aggregationmergetree_visitor select guid,countState(cnt),toDate(cdt) from tbl_test_mergetree_logs group by guid,cnt,cdt;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/35.png)

> 统计UV和PV增量计算表

```
select guid,count(cnt) from tbl_test_aggregationmergetree_visitor group by guid,cnt;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/36.png)

> 查询出的2条记录，guid列可以用来计算uv指标，count列可以用来计算pv值。

> 因为在插入数据的时候，根据guid执行group by计算，每个用户只有一条，所以统计guid列可得到uv指标；

> 使用countState(cnt)计算每个用户的所有访问次数，所以通过cnt列可以得到pv指标；

> cdt字段作为分区字段，即增量聚合每天的uv和pv指标。

##### 2.6.3.5 CollapsingMergeTree

> `在ClickHouse中不支持对数据update和delete操作（不能使用标准的更新和删除语法操作CK）`，但在增量计算场景下，状态更新是一个常见的现象，此时update操作似乎更符合这种需求。

> ClickHouse提供了一个`CollapsingMergeTree`表引擎，`它继承于MergeTree引擎，是通过一种变通的方式来实现状态的更新。`

> CollapsingMergeTree表引擎需要的建表语句与MergeTree引擎基本一致，惟一的区别是需要指定Sign列（必须是Int8类型）。`这个Sign列有1和-1两个值，1表示为状态行，当需要新增一个状态时，需要将insert语句中的Sign列值设为1；-1表示为取消行，当需要删除一个状态时，需要将insert语句中的Sign列值设为-1。`

> 这其实是插入了两行除Sign列值不同，但其他列值均相同的数据。因为有了Sign列的存在，当触发后台合并时，会找到存在状态行与取消行对应的数据，然后进行折叠操作，`也就是同时删除了这两行数据。`

> 状态行与取消行不折叠有两种情况。
>> 第一种是合并机制，由于合并在后台发生，且具体的执行时机不可预测，所以可能会存在状态行与取消行还没有被折叠的情况，这时会出现数据冗余；
> 
>> 第二种是当乱序插入时(CollapsingMergeTree仅允许严格连续插入)，ClickHouse不能保证相同主键的行数据落在同一个节点上，但不同节点上的数据是无法折叠的。为了得到正确的查询结果，需要将count(col)、sum(col)改写成sum (Sign)、sum(col * Sign)。

> 如果在业务系统中使用ClickHouse的CollapsingMergeTree引擎表，当状态行已经存在，要插入取消行来删除数据的时候，必须存储一份状态行数据来执行insert语句删除。这种情况下，就有些麻烦，因为同一个业务数据的状态需要我们记录上一次原始态数据，和当前最新态的数据，才能完成原始态数据删除，最新态数据存储到ClickHouse中。

> 创建CollapsingMergeTree引擎表的语法

```
语法结构
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
...
) ENGINE = CollapsingMergeTree(sign)
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]
```

> CollapsingMergeTree(sign)参数说明
>> Sign是列名称，必须是Int8类型，用来标志Sign列。Sign列值为1是状态行，为-1是取消行。

> 创建CollapsingMergeTree引擎的tbl_test_collapsingmergetree_day_mall_sale_all表

```
create table tbl_test_collapsingmergetree_day_mall_sale (
mallId UInt64,
mallName String,
totalAmount Decimal(32,2),
cdt Date,
sign Int8
) engine=CollapsingMergeTree(sign) partition by toYYYYMMDD(cdt) order by mallId;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/37.png)

> 第一次插入2条sign=1的数据
>> `注意：当一行数据的sign列=1时，是标记该行数据属于状态行。也就是说，我们插入了两条状态行数据。`

```
insert into tbl_test_collapsingmergetree_day_mall_sale(mallId,mallName,totalAmount,cdt,sign) values(1,'西单大悦城',17649135.64,'2019-12-24',1);
insert into tbl_test_collapsingmergetree_day_mall_sale(mallId,mallName,totalAmount,cdt,sign) values(2,'朝阳大悦城',16341742.99,'2019-12-24',1);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/38.png)

> 查询第一次插入的数据

```
select * from tbl_test_collapsingmergetree_day_mall_sale;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/39.png)

> 第二次插入2条sign=-1的数据
>> `注意：当一行数据的sign列=-1时，是标记该行数据属于取消行（取消行有一个要求：除了sign字段值不同，其他字段值必须是相同的。这样一来，就有点麻烦，因为我们在状态发生变化时，还需要保存着未发生状态变化的数据。这个场景类似于修改数据，但由于ClickHouse本身的特性不支持update，所以其提供了一种变通的方式，即通过CollapsingMergeTree引擎来支持这个场景）。取消行指的是当这一行数据有了新的状态变化，需要先取消原来存储的数据，使ClickHouse合并时来删除这些sign由1变成-1的数据，虽然合并发生时机不确定，但如果触发了合并操作就一定会被删除。这样一来，我们将有新状态变化的数据再次插入到表，就仍然是2条数据。`

```
insert into tbl_test_collapsingmergetree_day_mall_sale(mallId,mallName,totalAmount,cdt,sign) values(1,'西单大悦城',17649135.64,'2019-12-24',-1);
insert into tbl_test_collapsingmergetree_day_mall_sale(mallId,mallName,totalAmount,cdt,sign) values(2,'朝阳大悦城',16341742.99,'2019-12-24',-1);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/40.png)

> 对表执行强制合并

```
optimize table tbl_test_collapsingmergetree_day_mall_sale final;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/41.png)

> 然后发现查询数据时，表中已经没有了数据。这表示当触发合并操作时，会合并状态行与取消行同时存在的数据。

> 第三次插入3条数据

```
insert into tbl_test_collapsingmergetree_day_mall_sale(mallId,mallName,totalAmount,cdt,sign) values(1,'西单大悦城',17649135.64,'2019-12-24',1);
insert into tbl_test_collapsingmergetree_day_mall_sale(mallId,mallName,totalAmount,cdt,sign) values(2,'朝阳大悦城',16341742.99,'2019-12-24',1);
insert into tbl_test_collapsingmergetree_day_mall_sale(mallId,mallName,totalAmount,cdt,sign) values(1,'西单大悦城',17649135.64,'2019-12-24',-1);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/42.png)

```
select * from tbl_test_collapsingmergetree_day_mall_sale;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/43.png)

##### 2.6.3.6 VersionedCollapsingMergeTree

> 该引擎继承自 MergeTree 并将折叠行的逻辑添加到合并数据部分的算法中，这个引擎:
>> 允许快速写入不断变化的对象状态
> 
>> 删除后台中的旧对象状态，这显著降低了存储体积

> VersionedCollapsingMergeTree 用于相同的目的 折叠树，但使用不同的折叠算法，允许以多个线程的任何顺序插入数据。 特别是， Version 列有助于正确折叠行，即使它们以错误的顺序插入。

> 相比之下, CollapsingMergeTree 只允许严格连续插入。

> 创建VersionedCollapsingMergeTree引擎表的语法

```
语法结构
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
...
) ENGINE = VersionedCollapsingMergeTree(sign, version)
[PARTITION BY expr]
[ORDER BY expr]
[SAMPLE BY expr]
[SETTINGS name=value, ...]
```

> VersionedCollapsingMergeTree(sign)参数说明
>> Sign是列名称，必须是Int8类型，用来标志Sign列。Sign列值为1是状态行，为-1是取消行。

> 折叠数据
>> 考虑一种情况，您需要为某个对象保存不断变化的数据。对于一个对象有一行，并在发生更改时更新该行是合理的。但是，对于数据库管理系统来说，更新操作非常昂贵且速度很慢，因为它需要重写存储中的数据。 如果需要快速写入数据，则不能接受更新，但可以按如下顺序将更改写入对象。
> 
>> 使用 Sign 列写入行时。 如果 Sign = 1 这意味着该行是一个对象的状态（让我们把它称为 “state” 行）。 如果 Sign = -1 它指示具有相同属性的对象的状态的取消（让我们称之为 “cancel” 行）。 还可以使用 Version 列，它应该用单独的数字标识对象的每个状态。
> 
>> 例如，我们要计算用户在某个网站上访问了多少页面以及他们在那里的时间。

>在某个时间点，我们用用户活动的状态写下面的行:

```
┌───────────UserID───┬─PageViews─┬─Duration─┬─Sign─┬─Version─┐
│ 4324182021466249494   │         5   │      146 │     1 │       1   |
└───────────────────┴─────────┴───────┴─────┴────────┘
```

> 在稍后的某个时候，我们注册用户活动的变化，并用以下两行写入它。

```
┌────────────UserID─┬─PageViews─┬─Duration─┬─Sign─┬─Version─┐
│ 4324182021466249494 │         5    │      146 │   -1 │       1 |
│ 4324182021466249494 │         6    │      185 │    1 │       2 |
└─────────────────┴──────────┴────────┴────┴──────┘
```

> 第一行取消对象（用户）的先前状态。 它应该复制已取消状态的所有字段，除了 Sign.

> 第二行包含当前状态。因为我们只需要用户活动的最后一个状态行

```
┌────────────UserID─┬─PageViews─┬─Duration─┬─Sign─┬─Version─┐
│ 4324182021466249494 │         5    │      146 │    1  │       1 |
│ 4324182021466249494 │         5    │      146 │   -1  │       1 |
└─────────────────┴──────────┴────────┴─────┴──────┘
```

> 可以删除，折叠对象的无效（旧）状态。 VersionedCollapsingMergeTree 在合并数据部分时执行此操作。

> 使用示例

```
┌───────────UserID─┬─PageViews─┬─Duration─┬─Sign─┬─Version─┐
│ 4324182021466249494 │         5 │      146 │    1 │       1 |
│ 4324182021466249494 │         5 │      146 │   -1 │       1 |
│ 4324182021466249494 │         6 │      185 │    1 │       2 |
└─────────────────┴────────┴────────┴──────┴─────────┘
```

> 创建表:

```
CREATE TABLE UAct
(
UserID UInt64,
PageViews UInt8,
Duration UInt8,
Sign Int8,
Version UInt8
)
ENGINE = VersionedCollapsingMergeTree(Sign, Version)
ORDER BY UserID
```

> 插入数据:

```
INSERT INTO UAct VALUES (4324182021466249494, 5, 146, 1, 1)
INSERT INTO UAct VALUES (4324182021466249494, 5, 146, -1, 1),(4324182021466249494, 6, 185, 1, 2)
```

> 我们用两个 INSERT 查询以创建两个不同的数据部分。 如果我们使用单个查询插入数据，ClickHouse将创建一个数据部分，并且永远不会执行任何合并。

> 获取数据:

```
SELECT *  FROM UAct
┌────────────UserID─┬─PageViews─┬─Duration─┬─Sign─┬─Version─┐
│ 4324182021466249494 │         5 │      146 │    1 │       1 │
└─────────────────┴────────┴────────┴────┴──────┘
┌──────────────UserID─┬─PageViews─┬─Duration─┬─Sign─┬─Version─┐
│ 4324182021466249494 │         5 │      146 │   -1 │       1 │
│ 4324182021466249494 │         6 │      185 │    1 │       2 │
└─────────────────┴────────┴───────┴─────┴───────┘
```

> 我们在这里看到了什么，折叠的部分在哪里？

> 我们使用两个创建了两个数据部分 INSERT 查询。

> 该 SELECT 查询是在两个线程中执行的，结果是行的随机顺序。

> 由于数据部分尚未合并，因此未发生折叠。 ClickHouse在我们无法预测的未知时间点合并数据部分。

> 这就是为什么我们需要聚合:

```
SELECT
UserID,
sum(PageViews * Sign) AS PageViews,
sum(Duration * Sign) AS Duration,
Version
FROM UAct
GROUP BY UserID, Version
HAVING sum(Sign) > 0;
┌────────────UserID─┬─PageViews─┬─Duration┬─Version─┐
│ 4324182021466249494 │         6   │      185 │       2 │
└─────────────────┴──────────┴───────┴───────┘
```

> 如果我们不需要聚合，并希望强制折叠，我们可以使用 FINAL 修饰符 FROM 条款

```
SELECT * FROM UAct FINAL
┌────────────UserID─┬─PageViews─┬─Duration─┬─Sign─┬─Version─┐
│ 4324182021466249494 │         6   │      185 │    1 │       2 │
└─────────────────┴──────────┴───────┴────┴─────────┘
```

### 2.7 ClickHouse的SQL语法

#### 2.7.1 常用的SQL命令

作用 | SQL
---|---
列出所有数据库 | show databases;
进入某一个数据库 | use dbName;
列出数据库中所有的表 | show tables;
创建数据库 | create database [if not exists] dbName;
删除数据库 | drop database dbName;
创建表 | create [temporary] table [if not exists] tableName [ON CLUSTER cluster] (<br/>fieldName dataType<br/>) engine = EngineName(parameters);
清空表 | truncate table tableName;
删除表 | drop table tableName;
创建视图 | create view view_name as select ...
创建物化视图 | create [MATERIALIZED] view [if not exists] [db.]tableName [to [db.]name] [engine=engine] [populate] as select ...

#### 2.7.2 select查询语法

> ClickHouse中完整select的查询语法如下（除了SELECT关键字和expr_list以外，其他字句都是可选的）：

```
SELECT [DISTINCT] expr_list
[FROM [db.]table | (subquery) | table_function] [FINAL]
[SAMPLE sample_coeff]
[ARRAY JOIN ...]
[GLOBAL] ANY|ALL INNER|LEFT JOIN (subquery)|table USING columns_list
[PREWHERE expr]
[WHERE expr]
[GROUP BY expr_list] [WITH TOTALS]
[HAVING expr]
[ORDER BY expr_list]
[LIMIT [n, ]m]
[UNION ALL ...]
[INTO OUTFILE filename]
[FORMAT format]
[LIMIT n BY columns]
```

> 如果查询中不包含DISTINCT，GROUP BY，ORDER BY子句以及IN和JOIN子查询，那它将仅使用O(1)数量的内存来完全流式的处理查询，否则这个查询将消耗大量的内存，除非你指定了这些系统配置：max_memory_usage, max_rows_to_group_by, max_rows_to_sort, max_rows_in_distinct, max_bytes_in_distinct, max_rows_in_set, max_bytes_in_set, max_rows_in_join, max_bytes_in_join, max_bytes_before_external_sort, max_bytes_before_external_group_by。它们规定了可以使用外部排序（将临时表存储到磁盘中）以及外部聚合，目前系统不存在关于Join的配置。

> `DISTINCT子句`
>> 如果使用了DISTINCT子句，则会对结果中的完全相同的行进行去重。在GROUP BY不包括聚合函数，并对全部SELECT部分都包含在GROUP BY中时的作用一样。但该子句与GROUP BY子句存在以下几点不同：
> 
>> 可以与GROUP BY配合使用；
> 
>> 当不存在ORDER BY子句但存在LIMIT子句时，查询将在同时满足DISTINCT与LIMIT的情况下立即停止查询；
> 
>> 在处理数据的同时输出结果，并不是等待整个查询全部完成。
> 
>> 在SELECT表达式中存在Array类型的列时，不能使用DISTINCT。

> `FROM子句`
>> 如果查询中不包含FROM子句则会读取system.one。 system.one中仅包含一行数据（此表实现了与其他数据库管理系统中的DUAL相同的功能）。
> 
>> 下面两个句子是等价的。
> 
>> FROM子句规定了将从哪个表、或子查询、或表函数中读取数据；同时ARRAY JOIN子句和JOIN子句也可以出现在这里。
> 
>> 可以使用包含在括号里的子查询来替代表，在这种情况下，子查询的处理将会构建在外部的查询内。不同于SQL标准，子查询后无需指定别名。为了兼容，你可以在子查询后添加‘AS 别名’，但是指定的名字不能被使用在任何地方。
> 
>> 也可以使用表函数来代替表，有关信息，参见“表函数”。
> 
>> 执行查询时，在查询中列出的所有列都将从对应的表中提取数据；如果你使用的是子查询的方式，则任何在外部查询中没有使用的列，子查询将从查询中忽略它们；如果你的查询没有列出任何的列（如SELECT count() FROM t），则将额外的从表中提取一些列（最好的情况下是最小的列），以便计算行数。
> 
>> 最后的FINAL修饰符仅能够被使用在SELECT from CollapsingMergeTree场景中。当你为FROM指定了FINAL修饰符时，你的查询结果将会在查询过程中被聚合。需要注意的是，在这种情况下，查询将在单个流中读取所有相关的主键列，同时对需要的数据进行合并。这会导致查询更慢。在大多数情况下，你应该避免使用FINAL修饰符。

```
select 1 as id,'zhangsan' as name
select 1 as id,'zhangsan' as name from system.one
```

> `SAMPLE子句`
>> SAMPLE是ClickHouse中的近似查询处理，它只能工作在MergeTree*系列的表中，并且在创建表时需要显示指定采样表达式。
> 
>> SAMPLE子句可以使用SAMPLE k来表示，其中k可以是0到1的小数值，或者是一个足够大的正整数值。当k为0到1的小数时，查询将使用k作为百分比选取数据。
> 
>> 例如，SAMPLE 0.1查询只会检索数据总量的10%。当k为一个足够大的正整数时，查询将使用'k'作为最大样本数。例如，SAMPLE 1000查询只会检索最多1000行数据，使用相同的采样率得到的结果总是一致的。

> `ARRAY JOIN子句`
>> ARRAY JOIN子句可以帮助查询进行与数组和nested数据类型的连接。它有点类似arrayJoin函数，但它的功能更广泛。ARRAY JOIN本质上等同于INNERT JOIN数组。
> 
>> 创建tbl_test_array_join表：

```
create table tbl_test_array_join(str String, arr Array(Int8)) engine=Memory;
```

> 插入数据：

```
insert into tbl_test_array_join(str,arr) values('a',[1,3,5]),('b',[2,4,6]);
```

> `使用ARRAY JOIN：`

```
select str,arr,arrItem from tbl_test_array_join ARRAY JOIN arr as arrItem;
```

> `JOIN 子句`
>> JOIN子句用于连接数据，作用与SQL的JOIN的定义相同。需要注意的是JOIN与ARRAY JOIN没有任何关系。
> 
>> 可以使用具体的tableName来代替<left_subquery>与<right_subquery>。这与使用SELECT * FROM table子查询的方式相同。除非你的表是[Join](../operations/table_engines/join.md支持的JOIN类型：INNER JOIN、LEFT OUTER JOIN、RIGHT OUTER JOIN、FULL OUTER JOIN和CROSS JOIN。默认的OUTER关键字可以省略不写。
> 
>> 在使用ALL修饰符对JOIN进行修饰时，如果右表中存在多个与左表关联的数据，那么系统则将右表中所有可以与左表关联的数据全部返回在结果中。这与SQL标准的JOIN行为相同。
> 
>> 在使用ANY修饰符对JOIN进行修饰时，如果右表中存在多个与左表关联的数据，那么系统仅返回第一个与左表匹配的结果。如果左表与右表一一对应，不存在多余的行时，ANY与ALL的结果相同。
> 
>> 可以在会话中通过设置join_default_strictness来指定默认的JOIN修饰符。
> 
>> 当使用普通的JOIN时，查询将被发送给远程的服务器，并在这些远程服务器上生成右表并与它们关联，即右表来自于各个服务器本身。
> 
>>d 当使用GLOBAL ... JOIN，首先会在请求服务器上计算右表并以临时表的方式将其发送到所有服务器。这时每台服务器将直接使用它进行计算。建议从子查询中删除所有JOIN不需要的列。当执行JOIN查询时，因为与其他阶段相比没有进行执行顺序的优化：JOIN优先于WHERE与聚合执行。因此，为了显示的指定执行顺序，建议使用子查询的方式执行JOIN。子查询不允许设置别名或在其他地方引用它们。USING中指定的列必须在两个子查询中具有相同的名称，而其他列必须具有不同的名称。可以通过使用别名的方式来更改子查询中的列名。USING子句使用的是等值连接。
> 
>> 右表（子查询的结果）将会保存在内存中。如果没有足够的内存，则无法运行JOIN。
> 
>> 只能在查询中指定一个JOIN。若要运行多个JOIN，你可以将它们放入子查询中。每次运行相同的JOIN查询，都会重新计算（不缓存结果）。为了避免这种情况，可以使用‘Join’引擎，它是一个预处理的Join数据结构，总是保存在内存中。在一些场景下，使用IN代替JOIN将会得到更高的效率。在各种类型的JOIN中，最高效的是ANY LEFT JOIN，然后是ANY INNER JOIN，效率最差的是ALL LEFT JOIN以及ALL INNER JOIN。

> `WHERE子句`
>> 如果使用WHERE子句, 则在该子句中必须包含一个UInt8类型的表达式。这个表达是是一个带有比较和逻辑的表达式，它会在所有数据转换前用来过滤数据。如果在支持索引的数据库表引擎中，这个表达式将被评估是否使用索引。

> `PREWHERE子句`
>> PREWHERE子句与WHERE子句的意思大致相同，在一个查询中如果同时指定PREWHERE和WHERE，在这种情况下，PREWHERE优先于WHERE。
> 
>> 当使用PREWHERE时，首先只读取PREWHERE表达式中需要的列。然后在根据PREWHERE执行的结果读取其他需要的列。
> 
>> 如果在过滤条件中有少量不适合索引过滤的列，但是它们又可以提供很强的过滤能力。这时使用PREWHERE能减少数据的读取。但PREWHERE字句仅支持*MergeTree系列引擎，不适合用于已经存在于索引中的列，因为当列已经存在于索引中的情况下，只有满足索引的数据块才会被读取。如果将'optimize_move_to_prewhere'设置为1时，但在查询中不包含PREWHERE，则系统将自动的把适合PREWHERE表达式的部分从WHERE中抽离到PREWHERE中。

> `GROUP BY子句`
>> 如果使用了GROUP BY子句，则在该子句中必须包含一个表达式列表。其中每个表达式将会被称之为“key”。SELECT，HAVING，ORDER BY子句中的表达式列表必须来自于这些“key”或聚合函数。被选择的列中不能包含非聚合函数或key之外的其他列。如果查询表达式列表中仅包含聚合函数，则可以省略GROUP BY子句，这时会假定将所有数据聚合成一组空“key”。
> 
>> GROUP BY子句会为遇到的每一个不同的key计算一组聚合函数的值。在GROUP BY子句中不支持使用Array类型的列。常量不能作为聚合函数的参数传入聚合函数中，例如sum(1)。

> `WITH TOTALS修饰符`
>> 如果使用了WITH TOTALS修饰符，你将会在结果中得到一个被额外计算出的行。在这一行中将包含所有key的默认值（零或者空值），以及所有聚合函数对所有被选择数据行的聚合结果。

> `GROUP BY使用外部存储设备`
>> 你可以在GROUP BY中允许将临时数据转存到磁盘上来限制对内存的使用。 max_bytes_before_external_group_by这个配置确定了在GROUP BY中启动将临时数据转存到磁盘上的内存阈值。如果你将它设置为0（这是默认值），这项功能将被禁用。如果使用时建议把max_memory_usage设置为max_bytes_before_external_group_by的2倍。

> `LIMIT子句`
>> LIMIT m用于在查询结果中选择前m行数据；LIMIT n, m 用于在查询结果中选择从n行开始的m行数据，但n和m这两个参数必须是正整数。如果没有指定ORDER BY子句，则结果的顺序是不确定的。

> `LIMIT N BY子句`
>> LIMIT N BY子句和LIMIT没有关系，LIMIT N BY COLUMNS子句可以用来在每一个COLUMNS分组中求得最大的N行数据。我们可以将它们同时用在一个查询中。LIMIT N BY子句中可以包含任意多个分组字段表达式列表。

> `HAVING子句`
>> HAVING子句可以用来过滤GROUP BY之后的数据，类似于WHERE子句。WHERE与HAVING不同之处在于WHERE在聚合前(GROUP BY)执行，HAVING在聚合后执行。如果不存在聚合，则不能使用HAVING。

> `ORDER BY子句`
>> 如果使用ORDER BY子句，则该子句中必须存在一个表达式列表，表达式列表中每一个表达式都可以分配一个DESC（降序）或ASC（升序），如果没有显示指定则默认以ASC方式进行排序。当对浮点类型的列排序时，不管排序的顺序如何，如果使用升序排序时，NaNs好像比所有值都要大。如果使用降序排序时，NaNs好像比所有值都小。

> `FORMAT子句`
>> 'FORMAT format'子句用于指定返回数据的格式，使用它可以方便的转换或创建数据的转储。如果不存在FORMAT子句，则使用默认的格式，这将取决与DB的配置以及所使用的客户端。对于批量模式的HTTP客户端和命令行客户端而言，默认的格式是TabSeparated。对于交互模式下的命令行客户端，默认的格式是PrettyCompact（它有更加美观的格式）。
> 
>> 当使用命令行客户端时，数据以内部高效的格式在服务器和客户端之间进行传递。客户端将单独的解析FORMAT子句，以帮助数据格式的转换，会减轻网络和服务器的负载。

> `UNION ALL子句`
>> 仅支持UNION ALL，不支持其他UNION规则(如UNION DISTINCT)。如果需要使用UNION DISTINCT，可以使用UNION ALL中包含SELECT DISTINCT的子查询的方式。UNION ALL中的查询可以同时运行，它们的结果将被混合到一起，这些查询的结果必须相同（列的数量和类型）。列名不同也是允许的，在这种情况下最终结果的列名将从第一个查询中获取。UNION会为查询之间进行类型转换。例如，如果组合的两个查询中包含相同的字段，并且是类型兼容的Nullable和non-Nullable，则结果将会将该字段转换为Nullable类型的字段。
> 
>> 作为UNION ALL查询的部分不能包含在括号内。ORDER BY与LIMIT子句应该被应用在每个查询中，而不是最终的查询中。如果需要做最终结果转换则需要将UNION ALL作为一个子查询包含在FROM子句中。

#### 2.7.3 insert into语法

> ClickHouse中完整insert的主要用于向系统中添加数据, 语法如下:

> `语法1：`

```
INSERT INTO [db.]table [(c1, c2, c3)] VALUES (v11, v12, v13), (v21, v22, v23)...
```

> 使用语法1时，如果表存在但要插入的数据不存在，如果有DEFAULT表达式的列就根据DEFAULT表达式填充值。如果没有DEFAULT表达式的列则填充零或空字符串。如果`strict_insert_defaults=1`（开启了严格模式）则必须在insert中写出所有没定义DEFAULT表达式的列。

> `语法2：`

```
INSERT INTO [db.]table [(c1, c2, c3)] FORMAT format_name data_set
```

> 使用语法2时，数据可以是ClickHouse支持的任何输入输出格式传递给INSERT，但format_name必须显示的指定。

> `语法3：`

```
INSERT INTO [db.]table [(c1, c2, c3)] FORMAT Values (v11, v12, v13)...
```

> 语法3所用的输入格式就与语法1中INSERT ... VALUES的中使用的输入格式相同。

> `语法4：`

```
INSERT INTO [db.]table [(c1, c2, c3)] SELECT ...
```

> 语法4是使用SELECT的结果写入到表中，select中的列类型必须与table中的列类型位置严格一致，列名可以不同，但类型必须相同。

> 注意
>> 除了VALUES外，其他格式中的数据都不允许出现如now()、1 + 2等表达式。VALUES格式允许有限度的使用但不建议我们这么做，因为执行这些表达式的效率低下。
> 
>> 系统不支持的其他用于修改数据的查询：UPDATE、DELETE、REPLACE、MERGE、UPSERT和 INSERT UPDATE。但是可以使用ALTER TABLE ... DROP PARTITION查询来删除一些不需要的数据。
> 
>> 如果在写入的数据中包含多个月份的混合数据时，将会显著的降低INSERT的性能。为了避免这种情况，可以让数据总是以尽量大的batch进行写入，如每次写入100000行；数据在写入ClickHouse前预先的对数据进行分组。
> 
>> 在进行INSERT时将会对写入的数据进行一些处理，按照主键排序，按照月份对数据进行分区、数据总是被实时的写入、写入的数据已经按照时间排序，这几种情况下，性能不会出现下降。

#### 2.7.4 alter语法

> ClickHouse中的`ALTER`只支持`MergeTree系列，Merge和Distributed引擎的表`

```
基本语法：
ALTER TABLE [db].name [ON CLUSTER cluster] ADD|DROP|MODIFY COLUMN ...
```

> 参数解析：
>> ADD COLUMN – 向表中添加新列
>> DROP COLUMN – 在表中删除列
>> MODIFY COLUMN – 更改列的类型

> 案例演示：

```
创建一个MergerTree引擎的表
CREATE TABLE mt_table (
date  Date,
id UInt8,
name String
) ENGINE=MergeTree() partition by toYYYYMMDD(date) order by id settings index_granularity=8192;
```

> 向表中插入一些值

```
insert into mt_table values ('2020-09-15', 1, 'zhangsan');
insert into mt_table values ('2020-09-15', 2, 'lisi');
insert into mt_table values ('2020-09-15', 3, 'wangwu');
```

> 在末尾添加一个新列age

```
:)alter table mt_table add column age UInt8
:)desc mt_table
┌─name─┬─type───┬─default_type─┬─default_expression─┐
│ date │ Date   │              │                    │
│ id   │ UInt8  │              │                    │
│ name │ String │              │                    │
│ age  │ UInt8  │              │                    │
└──────┴────────┴──────────────┴────────────────────┘
:) select * from mt_table
┌───────date─┬─id─┬─name─┬─age─┐
│ 2019-06-01 │  2 │ lisi │   0 │
└────────────┴────┴──────┴─────┘
┌───────date─┬─id─┬─name─────┬─age─┐
│ 2019-05-01 │  1 │ zhangsan │   0 │
│ 2019-05-03 │  3 │ wangwu   │   0 │
└────────────┴────┴──────────┴─────┘
```

> 更改age列的类型

```
:)alter table mt_table modify column age UInt16
:)desc mt_table

┌─name─┬─type───┬─default_type─┬─default_expression─┐
│ date │ Date   │              │                    │
│ id   │ UInt8  │              │                    │
│ name │ String │              │                    │
│ age  │ UInt16 │              │                    │
└──────┴────────┴──────────────┴────────────────────┘
```

> 删除刚才创建的age列

```
:)alter table mt_table drop column age
:)desc mt_table
┌─name─┬─type───┬─default_type─┬─default_expression─┐
│ date │ Date   │              │                    │
│ id    │ UInt8  │              │                    │
│ name │ String │              │                    │
└─────┴──────┴─────────┴───────────────┘
```

### 2.8 ClickHouse的SQL函数

#### 2.8.1 类型检测函数

```
select toTypeName(0);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/44.png)

```
select toTypeName(-0);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/45.png)

```
select toTypeName(1000);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/46.png)

```
select toTypeName(-1000);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/47.png)

```
select toTypeName(10000000);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/48.png)

```
select toTypeName(-10000000);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/49.png)

```
select toTypeName(1.99);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/50.png)

```
select toTypeName(toFloat32(1.99));
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/51.png)

```
select toTypeName(toDate('2019-12-12')) as dateType, toTypeName(toDateTime('2019-12-12 12:12:12')) as dateTimeType;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/52.png)

```
select toTypeName([1,3,5]);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/53.png)

#### 2.8.2 数学函数

函数名称 | 作用 | 用法 | 结果
---|---|---|---
plus | 求和 | select plus(1, 1) | =2
minus | 差 | select minus(10, 5) | =5
multiply | 求积 | select multiply(2, 2) | =4
divide | 除法 | select divide(6, 2)<br/>select divide(10, 0)<br/>select divide(0, 0) | =3<br/>=inf<br/>=nan
intDiv | 整数除法 | select intDiv(10, 3) | =3
intDivOrZero | 计算商 | select intDivOrZero(5,2) | =2
modulo | 余数 | select modulo(10, 3) | =1
negate | 取反 | select negate(10) | =-10
abs | 绝对值 | select abs(-10) | =10
gcd | 最大公约数 | select gcd(12, 24) | =12
lcm | 最小公倍数 | select lcm(12, 24) | =24

#### 2.8.3 时间函数

```
select now() as curDT,toYYYYMM(curDT),toYYYYMMDD(curDT),toYYYYMMDDhhmmss(curDT);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/54.png)

```
select toDateTime('2019-12-16 14:27:30') as curDT;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/55.png)

```
select toDate('2019-12-12') as curDT;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/56.png)

### 2.9 ClickHouse的update/delete的使用

> 从使用场景来说，Clickhouse是个分析型数据库。这种场景下，数据一般是不变的，因此Clickhouse对`update、delete`的支持是比较弱的，实际上并不支持标准的update、delete操作。

> 下面介绍一下Clickhouse中update、delete的使用。

> 更新、删除语法：
>> Clickhouse通过alter方式实现更新、删除，它把update、delete操作叫做mutation(突变)。语法为：

```
ALTER TABLE [db.]table DELETE WHERE filter_expr
ALTER TABLE [db.]table UPDATE column1 = expr1 [, ...] WHERE filter_expr
```

> 那么，`mutation`与标准的update、delete有什么区别呢？

> 标准SQL的更新、删除操作是同步的，即客户端要等服务端返回执行结果（通常是int值）；

> 而Clickhouse的update、delete是通过异步方式实现的，当执行update语句时，服务端立即返回，但是实际上此时数据还没变，而是排队等着。

```
创建表:
CREATE TABLE tbl_test_users(
id UInt64,
email String,
username String,
gender UInt8,
birthday Date,
mobile FixedString(13),
pwd String,
regDT DateTime,
lastLoginDT DateTime,
lastLoginIP String
) ENGINE=MergeTree() partition by toYYYYMMDD(regDT) order by id settings index_granularity=8192;
```

> 插入数据到MergeTree引擎的表

```
insert into tbl_test_users(id, email, username, gender, birthday, mobile, pwd, regDT, lastLoginDT, lastLoginIP) values (1,'wcfr817e@yeah.net','督咏',2,'1992-05-31','13306834911','7f930f90eb6604e837db06908cc95149','2008-08-06 11:48:12','2015-05-08 10:51:41','106.83.54.165'),(2,'xuwcbev9y@ask.com','上磊',1,'1983-10-11','15302753472','7f930f90eb6604e837db06908cc95149','2008-08-10 05:37:32','2014-07-28 23:43:04','121.77.119.233'),(3,'mgaqfew@126.com','涂康',1,'1970-11-22','15200570030','96802a851b4a7295fb09122b9aa79c18','2008-08-10 11:37:55','2014-07-22 23:45:47','171.12.206.122'),(4,'b7zthcdg@163.net','金俊振',1,'2002-02-10','15207308903','96802a851b4a7295fb09122b9aa79c18','2008-08-10 14:47:09','2013-12-26 15:55:02','61.235.143.92'),(5,'ezrvy0p@163.net','阴福',1,'1987-09-01','13005861359','96802a851b4a7295fb09122b9aa79c18','2008-08-12 21:58:11','2013-12-26 15:52:33','182.81.200.32');
```

> 查询表中全量数据。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/57.png)

> 更新数据

```
ALTER TABLE tbl_test_users UPDATE username='张三' WHERE id=1;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/58.png)

```
select * from tbl_test_users;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/59.png)

> 删除数据

```
ALTER TABLE tbl_test_users DELETE WHERE id=1;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/60.png)

```
select * from tbl_test_users;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/61.png)

> 查看mutation队列
>> 那么，怎么查看数据是否更新完成了呢？
>> 可以通过system.mutations表查看相关信息：

```
SELECT
database,
table,
command,
create_time,
is_done
FROM system.mutations
ORDER BY create_time DESC
LIMIT 10;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ClickHouse/62.png)

```
database: 库名
table: 表名
command: 更新/删除语句
create_time: mutation任务创建时间，系统按这个时间顺序处理数据变更
is_done: 是否完成，1为完成，0为未完成
```

> 通过以上信息，可以查看当前有哪些mutation已经完成，is_done为1即表示已经完成。

> `Mutation具体过程`
>> 首先，使用where条件找到需要修改的分区；
> 
>> 然后，重建每个分区，用新的分区替换旧的，分区一旦被替换，就不可回退；
> 
>> 对于每个分区，可以认为是原子性的；但对于整个mutation，如果涉及多个分区，则不是原子性的。

> `注意事项`
>> 更新功能不支持更新有关主键或分区键的列
> 
>> 更新操作没有原子性，即在更新过程中select结果很可能是一部分变了，一部分没变，从上边的具体过程就可以知道
> 
>> 更新是按提交的顺序执行的
> 
>> 更新一旦提交，不能撤销，即使重启clickhouse服务，也会继续按照system.mutations的顺序继续执行
> 
>> 已完成更新的条目不会立即删除，保留条目的数量由finished_mutations_to_keep存储引擎参数确定。 超过数据量时旧的条目会被删除
> 
>> 更新可能会卡住，比如update intvalue='abc'这种类型错误的更新语句执行不过去，那么会一直卡在这里，此时，可以使用`KILL MUTATION`来取消，语法：

```
kill mutation where database='app' and table='test' // database、table是system.mutations表中的字段
```

> `使用建议`
>> 按照官方的说明，update/delete 的使用场景是一次更新大量数据，也就是where条件筛选的结果应该是一大片数据。
> 
>> 举例：`alter table test update status=1 where status=0 and day='2020-04-01'`，一次更新一天的数据。
> 
>> 那么，能否一次只更新一条数据呢？例如：`alter table test update pv=110 where id=100`
> 
>> 当然也可以，但频繁的这种操作，可能会对服务造成压力。这很容易理解，如上文提到，更新的单位是分区，如果只更新一条数据，那么需要重建一个分区；如果更新100条数据，而这100条可能落在3个分区上，则需重建3个分区；相对来说一次更新一批数据的整体效率远高于一次更新一行。
> 
>> 对于频繁单条更新的这种场景，建议使用`ReplacingMergeTree/CollapsingMergeTree`引擎来变相解决。

### 2.10 ClickHouse的使用

#### 2.10.1 使用Java操作ClickHouse

> `pom`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.xiaoma.clickhouse</groupId>
    <artifactId>xiaoma_clickhouse</artifactId>
    <version>1.0-SNAPSHOT</version>

    <repositories>
        <repository>
            <id>mvnrepository</id>
            <url>https://mvnrepository.com/</url>
            <layout>default</layout>
        </repository>
        <repository>
            <id>cloudera</id>
            <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
        </repository>
        <repository>
            <id>elastic.co</id>
            <url>https://artifacts.elastic.co/maven</url>
        </repository>
    </repositories>

    <properties>
        <scala.version>2.11</scala.version>
        <!-- Spark -->
        <spark.version>2.4.0</spark.version>
        <!-- Parquet -->
        <parquet.version>1.9.0</parquet.version>
        <!-- ClickHouse -->
        <clickhouse.version>0.2.2</clickhouse.version>
        <jtuple.version>1.2</jtuple.version>
    </properties>

    <dependencies>
        <!-- Spark -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql-kafka-0-10_2.11</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.parquet</groupId>
            <artifactId>parquet-common</artifactId>
            <version>${parquet.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-graphx_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>net.jpountz.lz4</groupId>
            <artifactId>lz4</artifactId>
            <version>1.3.0</version>
        </dependency>
        <dependency>
            <groupId>org.javatuples</groupId>
            <artifactId>javatuples</artifactId>
            <version>${jtuple.version}</version>
        </dependency>
        <!-- Clickhouse -->
        <dependency>
            <groupId>ru.yandex.clickhouse</groupId>
            <artifactId>clickhouse-jdbc</artifactId>
            <version>${clickhouse.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>com.fasterxml.jackson.core</groupId>
                    <artifactId>jackson-databind</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>com.fasterxml.jackson.core</groupId>
                    <artifactId>jackson-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
</project>
```

> 代码演示

```
package cn.xiaoma.clickhouse;

import java.sql.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 使用JDBC的方式操作Clickhouse
 * 使用JDBC操作Mysql
 * 使用JDBC操作Phoenix
 * 使用JDBC操作Druid
 * 使用JDBC操作Druid
 *
 * 操作步骤：
 * 1）注册驱动
 * 2）创建连接
 * 3）创建statement
 * 4）执行查询
 * 5）循环查询结果
 * 6）关闭连接，释放资源
 */
public class ClickHouseJdbcDemo {
    public static void main(String[] args) {
        String sqlDB = "show databases";//查询数据库
        String sqlTab = "show tables";//查看表
        String sqlCount = "select * from tbl_test_users";//查询ontime数据量
        String sqlAgg = "SELECT\n" +
                "    UserID,\n" +
                "    sum(PageViews * Sign) AS PageViews,\n" +
                "    sum(Duration * Sign) AS Duration,\n" +
                "    Version\n" +
                "FROM UAct\n" +
                "GROUP BY UserID, Version\n" +
                "HAVING sum(Sign) > 0";

        exeSql(sqlDB);
        exeSql(sqlTab);
        exeSql(sqlCount);
        exeSql(sqlAgg);
    }

    public static void exeSql(String sql){
        String address = "jdbc:clickhouse://node2.xiaoma.cn:8123/default";
        Connection connection = null;
        Statement statement = null;
        ResultSet results = null;
        try {
            //1）注册驱动
            Class.forName("ru.yandex.clickhouse.ClickHouseDriver");

            //2）创建连接
            connection = DriverManager.getConnection(address);

            //3）创建statement
            statement = connection.createStatement();

            //4）执行查询
            long begin = System.currentTimeMillis();
            results = statement.executeQuery(sql);
            long end = System.currentTimeMillis();
            System.out.println("执行（"+sql+"）耗时："+(end-begin)+"ms");

            //获取元数据信息
            ResultSetMetaData rsmd = results.getMetaData();
            List<Map> list = new ArrayList();
            //5）循环查询结果
            while(results.next()){
                Map map = new HashMap();
                for(int i = 1;i<=rsmd.getColumnCount();i++) {
                    map.put(rsmd.getColumnName(i), results.getString(rsmd.getColumnName(i)));
                }
                list.add(map);
            }
            for(Map map : list){
                System.err.println(map);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //6）关闭连接，释放资源
            try {
                if(results!=null){
                    results.close();
                }
                if(statement!=null){
                    statement.close();
                }
                if(connection!=null){
                    connection.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### 2.10.2 使用Spark操作ClickHouse

> 代码

```
package cn.xiaoma.clickhouse

import org.apache.spark.SparkConf
import org.apache.spark.sql.{DataFrame, SparkSession}

/**
 * 使用spark的DataFrame操作ClickHouse
 * 1）创建表（kafka的数据消费出来以后，需要写入到clickhouse表中，首先表存在才能写）
 * 2）插入数据
 * 3）更新数据
 * 4）删除数据
 *
 * 使用离线的方式，将dataFrame的数据同步到clickHouse
 * 不论是实时写入clickhouse还是离线写入clickhouse，都是基于DataFrame对象完成的数据同步
 */
object ClickHouseDemo {
  /**
   * 入口函数
   * @param args
   */
  def main(args: Array[String]): Unit = {
    /**
     * 实现步骤：
     * 1）创建sparkConf对象
     * 2）创建sparkSession对象
     * 3）加载一份离线数据，生成DataFrame对象
     * 4）基于生成的DataFrame对象完成创建ClickHouse表
     * 5）基于生成的DataFrame对象完成插入ClickHouse数据
     * 6）基于生成的DataFrame对象完成修改ClickHouse数据
     * 7）基于生成的DataFrame对象完成删除ClickHouse数据
     * 8）停止作业，释放资源
     */

    //TODO 1）创建sparkConf对象
    val sparkConf: SparkConf = new SparkConf().setAppName(this.getClass.getSimpleName).setMaster("local[*]")

    //TODO  2）创建sparkSession对象
    val sparkSession: SparkSession = SparkSession.builder().config(sparkConf).getOrCreate()

    //TODO 3）加载一份离线数据，生成DataFrame对象
    val dataFrame: DataFrame = sparkSession.read.json("E:\\input\\order.json")

    dataFrame.show()

    //TODO 4）基于生成的DataFrame对象完成创建ClickHouse表
    /**
     * 创建表：
     * 1：表名（每个DataFrame对象有一个对应的表名）
     * 2：表的引擎（指定表的引擎）
     * 3：表的字段数量和字段名称（Schema）
     * 4：表的字段类型（SparkSQL的字段类型，如：Long->UInt64）
     */
    val createTableSql: String = ClickHouseUtils.createTable("order", dataFrame)
    //执行建表语句
    ClickHouseUtils.executeUpdate(createTableSql)

    println(createTableSql)

    //TODO 5）基于生成的DataFrame对象完成插入ClickHouse数据
    ClickHouseUtils.insertToClickHouseWithDF("order", dataFrame)

    //TODO 6）基于生成的DataFrame对象完成修改ClickHouse数据
    //ClickHouseUtils.updateToClickHouseWithDF("order", dataFrame)

    //TODO 7）基于生成的DataFrame对象完成删除ClickHouse数据
    //ClickHouseUtils.deleteToClickHouseWithDF("order", dataFrame)
    //TODO 8）停止作业，释放资源
    sparkSession.stop()
  }
}

```

> Utils

```
package cn.xiaoma.clickhouse

import java.sql.{Date, PreparedStatement}
import java.text.SimpleDateFormat

import org.apache.spark.sql.{DataFrame, Row}
import org.apache.spark.sql.types.{DataType, DataTypes, DoubleType, FloatType, IntegerType, LongType, StringType, StructField, StructType}
import ru.yandex.clickhouse.{ClickHouseConnection, ClickHouseDataSource, ClickHouseStatement}
import ru.yandex.clickhouse.settings.ClickHouseProperties

import scala.collection.immutable
import scala.collection.mutable.ArrayBuffer

/**
 * ClickHouse操作的工具类
 * 1）创建表
 * 2）插入数据
 * 3）修改数据
 * 4）删除数据
 */
object ClickHouseUtils {


  /**
   * 创建clickhouse的连接对象
   */
  def createConnection = {
    //定义clickhouse的host
    val hostName = "node2.xiaoma.cn"
    //定义端口号
    val port = 8123
    //创建clickhouse的连接字符串
    val url: String = s"jdbc:clickhouse://${hostName}:${port}/default"
    //创建clickhouse的连接对象
    val clickHouseDataSource: ClickHouseDataSource = new ClickHouseDataSource(url, new ClickHouseProperties())
    clickHouseDataSource.getConnection
  }

  /**
   * 创建表
   * @param tableName
   * @param dataFrame
   */
  def createTable(tableName: String, dataFrame: DataFrame, primaryFieldKey:String = "id") = {
    /**
     * 1：表名（每个DataFrame对象有一个对应的表名）
     * 2：表的引擎（指定表的引擎）
     * 3：表的字段数量和字段名称（Schema）
     * 4：表的字段类型（SparkSQL的字段类型，如：Long->UInt64）
     */
    //获取dataFrame对象的schema作为创建clickhouse表的字段信息
    val tableFieldArr: Seq[String] = dataFrame.schema.map(field => {
      //获取到列名
      val fieldName: String = field.name
      //获取到schema的列的类型（spark-Sql的字段类型）
      val fieldDataType: DataType = field.dataType
      //根据sparksql的字段类型转换成clickhouse认识的字段类型
      val fieldType: String = fieldDataType match {
        case StringType => "String"
        case IntegerType => "Int32"
        case FloatType => "Float32"
        case DoubleType => "Float64"
        case LongType => "Int64"
        case _ => throw new Exception(s"未被识别的字段类型：${fieldDataType}")
      }

      //返回字段名称和字段类型的字段串
      s"${fieldName} ${fieldType}"
    })

    //生成的建表的sql语句
    s"CREATE TABLE IF NOT EXISTS ${tableName}(${tableFieldArr.mkString(",")}) ENGINE=MergeTree() order by ${primaryFieldKey} settings index_granularity=8192;"
  }

  /**
   * 基于DataFrame将数据写入到ClickHouse表中
   *
   * @param tableName
   * @param dataFrame
   * @return
   */
  def insertToClickHouseWithDF(tableName: String, dataFrame: DataFrame) = {
    /**
     * 1：生成写入数据的sql语句：INSERT INTO order(areaName,category,id,money,timestamp) VALUES(?,?,?,?,?)
     *  表名
     *  字段（来自schema）
     *  字段值
     * 2：根据生成的sql语句执行数据的更新操作
     */
    //1：生成写入数据的sql语句：INSERT INTO order(areaName,category,id,money,timestamp) VALUES(?,?,?,?,?)
    val insertSQL: String = createInsertStatementSQL(tableName, dataFrame)
    println(insertSQL)

    //遍历DataFrame中的每行数据，然后根据sql语句生成写入的待数据的value值
    dataFrame.foreachPartition(iters=>{
      //实例化connection的连接对象
      var connection: ClickHouseConnection = null
      var statement: PreparedStatement = null

      try {
        connection = createConnection
        statement = connection.prepareStatement(insertSQL)

        //遍历分区中的每一条数据
        iters.foreach(row=>{
          //获取到每条数据：|    北京|平板电脑|  1| 1450|2019-05-08T01:03.00Z|、|    北京|    手机|  2| 1450|2019-05-08T01:01.00Z|
          //循环当前行对象的所有的列
          row.schema.fields.foreach(filed=>{
            //获取到列的名称
            val fieldName: String = filed.name
            //根据列名获取列的下标
            val fieldIndex: Int = row.schema.fieldIndex(fieldName)
            //根据列名的下标获取到列值
            val fieldValue: Any = row.get(fieldIndex)
            //为参数化的sql语句赋值
            statement.setObject(fieldIndex+1, fieldValue)
          })
          //将每条数据追加批量数据集合中
          statement.addBatch()
        })
        //批量更新
        statement.executeBatch()
      } catch {
        case ex:Exception => ex.printStackTrace()
      } finally {
        if(statement != null && !statement.isClosed)  statement.close()
        if(connection!=null && !connection.isClosed) connection.close()
      }
    })

  }

  /**
   * 基于DataFrame生成插入的sql语句
   * @param tableName
   * @param dataFrame
   * @return
   */
  def createInsertStatementSQL(tableName: String, dataFrame: DataFrame) = {
    //insert into order(.....) values(.....)
    //insert into order(areaName,id,money,category,timestamp) values(?,?,?,?,?)
    //根据schema的列的描述信息生成要插入的列的字段
    val columns: Seq[String] = dataFrame.schema.map(field => field.name)
    //根据列名的数量生成value的占位符
    val values: immutable.Seq[String] = (1 to columns.length) map (i=> "?")

    //返回生成的插入表数据的sql语句
    s"INSERT INTO ${tableName}(${columns.mkString(",")}) VALUES(${values.mkString(",")})"
  }

  /**
   * 基于DataFrame修改数据
   *
   * @param tableName
   * @param dataFrame
   * @return
   */
  def updateToClickHouseWithDF(tableName: String, dataFrame: DataFrame,primaryFieldKey:String = "id") = {
      //ALTER TABLE order Update areaName='北京' where ID=1
      dataFrame.foreachPartition(rows=>{
        //创建连接对象
        var connection:ClickHouseConnection = null
        var statement :ClickHouseStatement= null

        try {
          connection = createConnection
          statement = connection.createStatement()

          //对每个分区的数据进行循环遍历
          rows.foreach(row=>{
            //每行数据生成数据的更新语句
            val updateSQL: String = createUpdateStatementSQL(tableName, row)
            //println(updateSQL)
            statement.executeUpdate(updateSQL)
          })
        } catch {
          case ex:Exception => ex.printStackTrace()
        } finally {
          if(statement != null && !statement.isClosed)  statement.close()
          if(connection!=null && !connection.isClosed) connection.close()
        }
      })
  }

  /**
   * 生成更新操作的sql语句
   * @param tableName
   * @param row
   * @return
   */
  def createUpdateStatementSQL(tableName: String, row: Row, primaryFieldKey:String = "id") = {
    //获取到所有得列：ALTER TABLE order Update areaName='北京', timestamp="2019-05-08T01:03.00Z" where ID=1
    val schema: StructType = row.schema
    //获取到需要被更新的列（主键列不能被更新，因此需要将该列排除表）
    val noPrimaryFieldKey: Array[StructField] = schema.fields.filter(field => field.name != primaryFieldKey)
    val sets: ArrayBuffer[String] = ArrayBuffer[String]()
    for(i<-0 until noPrimaryFieldKey.length){
      val field: StructField = noPrimaryFieldKey(i)
      sets += s"${field.name}='${getFieldValue(field.name,row).toString}'"
    }

    //返回生成的更新的sql语句
    s"ALTER TABLE ${tableName} UPDATE ${sets.mkString(",")} WHERE ${primaryFieldKey}=${getFieldValue(primaryFieldKey, row)}"
  }

  /**
   * 基于DataFrame完成clickhouse表的数据删除功能
   *
   * @param str
   * @param dataFrame
   * @return
   */
  def deleteToClickHouseWithDF(tableName: String, dataFrame: DataFrame, primaryFieldKey:String = "id") = {
    dataFrame.foreachPartition(rows=>{
      var connection:ClickHouseConnection = null;
      var statement:ClickHouseStatement = null;

      try {
        connection = createConnection
        statement = connection.createStatement()

        rows.foreach(row=>{
          val deleteSQL: String = createDeleteStatementSQL(tableName, row)
          statement.executeUpdate(deleteSQL)
        })
      } catch {
        case ex:Exception => ex.printStackTrace()
      } finally {
        if(statement != null && !statement.isClosed)  statement.close()
        if(connection!=null && !connection.isClosed) connection.close()
      }
    })
  }

  /**
   * 生成删除操作的sql语句
   * @param tableName
   * @param row
   * @return
   */
  def createDeleteStatementSQL(tableName: String, row: Row,primaryFieldKey:String = "id") = {
    //ALTER TABLE order DELETE WHERE id=1
    val primaryFieldValue: String = getFieldValue(primaryFieldKey, row)

    //生成删除操作的sql语句
    s"ALTER TABLE ${tableName} DELETE WHERE ${primaryFieldKey}=${primaryFieldValue}"
  }


  /**
   * 根据列名获取到列值
   * @param fieldName
   * @param data
   */
  def getFieldValue(fieldName:String, data:Row) ={
    //需要返回的列值
    var fieldValue:String = null
    //定义是否查找到了列值
    var flag = true

    //获取到所有的列
    val fields: Array[StructField] = data.schema.fields
    for(i <-0 until fields.length if flag){
      val field: StructField = fields(i)
      if(fieldName == field.name){
        //找到了需要查询的列
        fieldValue = field.dataType match {
          case DataTypes.DoubleType => if(data.isNullAt(i)) "NULL" else s"${data.getDouble(i)}"
          case DataTypes.FloatType => if(data.isNullAt(i)) "NULL" else s"${data.getFloat(i)}"
          case DataTypes.IntegerType => if(data.isNullAt(i)) "NULL" else s"${data.getInt(i)}"
          case DataTypes.LongType => if(data.isNullAt(i)) "NULL" else s"${data.getLong(i)}"
          case DataTypes.StringType => if(data.isNullAt(i)) "NULL" else s"${data.getString(i)}"
          case DataTypes.DateType => if(data.isNullAt(i)) "NULL" else {
            new SimpleDateFormat("yyyy-MM-dd").format(new Date(data.get(i).asInstanceOf[Date].getTime/1000))
          }
          case DataTypes.NullType => "NULL"
        }
        flag = false
      }
    }

    //返回列值
    fieldValue
  }

  /**
   * 传递sql语句，执行更新操作
   * @param sql
   */
  def executeUpdate(sql:String) ={
    //获取connection的连接对象
    val connection: ClickHouseConnection = createConnection
    //创建statement
    val statement: ClickHouseStatement = connection.createStatement()
    statement.executeUpdate(sql)

    //关闭连接释放资源
    statement.close()
    connection.close()
  }
}
```



## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)

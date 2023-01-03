---
title: HBase 分布式可扩展大数据存储
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(51).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(51).jpg
tags:
  - 大数据
  - Hadoop
  - HBase
category:
  - - 编程
    - 大数据
 
date: 2020-01-03 16:26:05
---

Apache HBase™是Hadoop数据库，这是一个`分布式`，`可扩展`的`大数据存储`。

当您需要对`大数据`进行`随机，实时的读/写访问`时，请使用Apache HBase™。

该项目的目标是在商品硬件群集上托管非常大的表（数十亿行X数百万列）。

Apache HBase是一种`开放源`，`分布式`，`版本化`，`非关系型`数据库，其仿照Chang等人的Google的Bigtable：结构化数据的分布式存储系统。

正如Bigtable利用Google文件系统提供的`分布式数据存储`一样，Apache HBase在Hadoop和HDFS之上提供类似于Bigtable的功能。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 简介

### 1.1 Hadoop

> 从 1970 年开始，大多数的公司数据存储和维护使用的是关系型数据库。<br/>
大数据技术出现后，很多拥有海量数据的公司开始选择像Hadoop的方式来存储海量数据。<br/>
Hadoop使用分布式文件系统HDFS来存储海量数据，并使用 MapReduce 来处理。Hadoop擅长于存储各种格式的庞大的数据，任意的格式甚至非结构化的处理

### 1.2 Hadoop的局限

> Hadoop主要是实现批量数据的处理，并且通过顺序方式访问数据。<br/>
要查找数据必须搜索整个数据集， 如果要进行随机读取数据，效率较低(压根就不支持)

### 1.3 HBase与NoSQL

> NoSQL是一个通用术语，泛指一个数据库并不是使用SQL作为主要语言的非关系型数据库。<br/>
HBase是BigTable的开源java版本。是建立在HDFS之上，提供高可靠性、高性能、列存储、可伸缩、实时读写NoSQL的数据库系统。<br/>
HBase仅能通过主键(row key)和主键的range来检索数据，仅支持单行事务。<br/>
主要用来存储结构化和半结构化的松散数据。<br/>
Hbase查询数据功能很简单，不支持join等复杂操作，不支持复杂的事务（行级的事务），从技术上来说，HBase更像是一个「数据存储」而不是「数据库」，因为HBase缺少RDBMS中的许多特性，例如带类型的列、二级索引以及高级查询语言等。<br/>
Hbase中支持的数据类型：byte[]。<br/>
与Hadoop一样，Hbase目标主要依靠横向扩展，通过不断增加廉价的商用服务器，来增加存储和处理能力，例如，把集群从10个节点扩展到20个节点，存储能力和处理能力都会加倍。<br/>
HBase中的表一般有这样的特点。
> 
```
大：一个表可以有上十亿行，上百万列
面向列:面向列(族)的存储和权限控制，列(族)独立检索
稀疏:对于为空(null)的列，并不占用存储空间，因此，表可以设计的非常稀疏
```

### 1.4 HBase应用场景

#### 1.4.1 `对象存储`

> 不少的头条类、新闻类的的新闻、网页、图片存储在HBase之中，一些病毒公司的病毒库也是存储在HBase之中

#### 1.4.2 `时序数据`

> HBase之上有OpenTSDB模块，可以满足时序类场景的需求

#### 1.4.3 `推荐画像`

> 用户画像，是一个比较大的稀疏矩阵，蚂蚁金服的风控就是构建在HBase之上

#### 1.4.4 `时空数据`

> 主要是轨迹、气象网格之类，滴滴打车的轨迹数据主要存在HBase之中，另外在技术所有大一点的数据量的车联网企业，数据都是存在HBase之中

#### 1.4.5 `CubeDB OLAP`

> Kylin一个cube分析工具，底层的数据就是存储在HBase之中，不少客户自己基于离线计算构建cube存储在hbase之中，满足在线报表查询的需求

#### 1.4.6 `消息/订单`

> 在电信领域、银行领域，不少的订单查询底层的存储，另外不少通信、消息同步的应用构建在HBase之上

#### 1.4.7 `Feeds流`

> 典型的应用就是xx朋友圈类似的应用，用户可以随时发布新内容，评论、点赞。

#### 1.4.8 `NewSQL`

> 之上有Phoenix的插件，可以满足二级索引、SQL的需求，对接传统数据需要SQL非事务的需求

#### 1.4.9 `其他`

> 存储爬虫数据<br/>
> 海量数据备份<br/>
> 短网址

### 1.5 发展历程

年份 | 重大事件
---|---
2006年11月 | Google发布BigTable论文.
2007年10月 | 发布第一个可用的HBase版本，基于Hadoop 0.15.0.
2008年1月 | HBase称为Hadoop的一个子项目.
2010年5月 | HBase称为Apache的顶级项目.

### 1.6 HBase特点

`强一致性读/写`

> HBASE不是“最终一致的”数据存储<br/>
它非常适合于诸如高速计数器聚合等任务

`自动分块`

> HBase表通过Region分布在集群上，随着数据的增长，区域被自动拆分和重新分布

`自动RegionServer故障转移`

`Hadoop/HDFS集成`

`HBase支持HDFS开箱即用作为其分布式文件系统`

`MapReduce`

> HBase通过MapReduce支持大规模并行处理，将HBase用作源和接收器

`Java Client API`

> HBase支持易于使用的 Java API 进行编程访问

`Thrift/REST API`  

`块缓存和布隆过滤器`

> HBase支持块Cache和Bloom过滤器进行大容量查询优化

`运行管理`

> HBase为业务洞察和JMX度量提供内置网页。

### 1.7 RDBMS与HBase的对比

#### 1.7.1 关系型数据库

##### 1.7.1.1 结构

> 数据库以表的形式存在<br/>
支持FAT、NTFS、EXT、文件系统<br/>
使用主键（PK）<br/>
通过外部中间件可以支持分库分表，但底层还是单机引擎<br/>
使用行、列、单元格

##### 1.7.1.2 功能

> 支持向上扩展（买更好的服务器）<br/>
使用SQL查询<br/>
面向行，即每一行都是一个连续单元<br/>
数据总量依赖于服务器配置<br/>
具有ACID支持<br/>
适合结构化数据<br/>
传统关系型数据库一般都是中心化的<br/>
支持事务<br/>
支持Join

#### 1.7.2 HBase

##### 1.7.2.1 结构

> 以表形式存在<br/>
支持HDFS文件系统<br/>
使用行键（row key）<br/>
原生支持分布式存储、计算引擎<br/>
使用行、列、列族和单元格

##### 1.7.2.2 功能

> 支持向外扩展<br/>
使用API和MapReduce、Spark、Flink来访问HBase表数据<br/>
面向列蔟，即每一个列蔟都是一个连续的单元<br/>
数据总量不依赖具体某台机器，而取决于机器数量<br/>
HBase不支持ACID（Atomicity、Consistency、Isolation、Durability）<br/>
适合结构化数据和非结构化数据<br/>
一般都是分布式的<br/>
HBase不支持事务，支持的是单行数据的事务操作<br/>
不支持Join

### 1.8 HDFS对比HBase

#### 1.8.1 HDFS

> HDFS是一个非常适合存储大型文件的分布式文件系统<br/>
HDFS它不是一个通用的文件系统，也无法在文件中快速查询某个数据

#### 1.8.2 HBase

> HBase构建在HDFS之上，并为大型表提供快速记录查找(和更新)<br/> 
HBase内部将大量数据放在HDFS中名为「StoreFiles」的索引中，以便进行高速查找<br/>
Hbase比较适合做快速查询等需求，而不适合做大规模的OLAP应用

### 1.9 Hive对比HBase

#### 1.9.1 Hive

> 数据仓库工具

Hive的本质其实就相当于将HDFS中已经存储的文件在Mysql中做了一个双射关系，以方便使用HQL去管理查询

> 用于数据分析、清洗

Hive适用于离线的数据分析和清洗，延迟较高

> 基于HDFS、MapReduce
> 
Hive存储的数据依旧在DataNode上，编写的HQL语句终将是转换为MapReduce代码执行

#### 1.9.2 HBase

> NoSQL数据库

是一种面向列存储的非关系型数据库。

> 用于存储结构化和非结构化的数据

适用于单表非关系型数据的存储，不适合做关联查询，类似JOIN等操作。

> 基于HDFS

数据持久化存储的体现形式是Hfile，存放于DataNode中，被ResionServer以region的形式进行管理

> 延迟较低，接入在线业务使用

面对大量的企业数据，HBase可以直线单表大量数据的存储，同时提供了高效的数据访问速度

#### 1.9.3 总结Hive与HBase

> Hive和Hbase是两种基于Hadoop的不同技术<br/> 
Hive是一种类SQL的引擎，并且运行MapReduce任务<br/>
Hbase是一种在Hadoop之上的NoSQL 的Key/value数据库<br/>
这两种工具是可以同时使用的。就像用Google来搜索，用FaceBook进行社交一样，Hive可以用来进行统计查询，HBase可以用来进行实时查询，数据也可以从Hive写到HBase，或者从HBase写回Hive

## 2 集群搭建

### 2.1 安装

#### 2.1.1 上传解压HBase安装包

```
tar -xvzf hbase-2.1.0.tar.gz -C ../servers/
```

#### 2.1.2 修改HBase配置文件

##### 2.1.2.1 hbase-env.sh

```
cd /export/servers/hbase-2.1.0/conf
vim hbase-env.sh
# 第28行
export JAVA_HOME=/export/servers/jdk1.8.0_241/
export HBASE_MANAGES_ZK=false
```

##### 2.1.2.2 hbase-site.xml

```
vim hbase-site.xml
------------------------------
<configuration>
        <!-- HBase数据在HDFS中的存放的路径 -->
        <property>
            <name>hbase.rootdir</name>
            <value>hdfs://node1.xiaoma.cn:8020/hbase</value>
        </property>
        <!-- Hbase的运行模式。false是单机模式，true是分布式模式。若为false,Hbase和Zookeeper会运行在同一个JVM里面 -->
        <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
        </property>
        <!-- ZooKeeper的地址 -->
        <property>
            <name>hbase.zookeeper.quorum</name>
            <value>node1.xiaoma.cn,node2.xiaoma.cn,node3.xiaoma.cn</value>
        </property>
        <!-- ZooKeeper快照的存储位置 -->
        <property>
            <name>hbase.zookeeper.property.dataDir</name>
            <value>/export/servers/apache-zookeeper-3.6.0-bin/data</value>
        </property>
        <!--  V2.1版本，在分布式情况下, 设置为false -->
        <property>
            <name>hbase.unsafe.stream.capability.enforce</name>
            <value>false</value>
        </property>
</configuration>
```

#### 2.1.3 配置环境变量

```
# 配置Hbase环境变量
vim /etc/profile
export HBASE_HOME=/export/servers/hbase-2.1.0
export PATH=$PATH:${HBASE_HOME}/bin:${HBASE_HOME}/sbin

#加载环境变量
source /etc/profile
```

#### 2.1.4 复制jar包到lib

```
cp $HBASE_HOME/lib/client-facing-thirdparty/htrace-core-3.1.0-incubating.jar $HBASE_HOME/lib/
```

#### 2.1.5 修改regionservers文件

```
vim regionservers 
node1.xiaoma.cn
node2.xiaoma.cn
node3.xiaoma.cn
```

#### 2.1.6 分发安装包与配置文件

```
cd /export/servers
scp -r hbase-2.1.0/ node2.xiaoma.cn:$PWD
scp -r hbase-2.1.0/ node3.xiaoma.cn:$PWD

#配置文件不建议这么改，自己手动去2号机和3号机改一下
scp -r /etc/profile node2.xiaoma.cn:/etc
scp -r /etc/profile node3.xiaoma.cn:/etc

在node2.xiaoma.cn和node3.xiaoma.cn配置加载环境变量
source /etc/profile
```

#### 2.1.7 启动HBase

> 启动ZK，三台服务器都执行

```
cd /export/servers/zookeeper-3.4.6/bin
./zkServer.sh start
./zkServer.sh status #查看启动状态，一个leader，两个follower就好了
```

> 启动hadoop

```
start-all.sh
jps 
```
查看启动成功没有,具体{% post_link 大数据_Hadoop集群搭建 去hadoop那篇文章看 %}
> 启动hbase

```
start-hbase.sh  
#要是中间报警告说连不到其他主机之类的，就先stop-hbase.sh，然后ssh node1.xiaoma.cn,ssh node2.xiaoma.cn,ssh node3.xiaoma.cn,ssh node1.xiaoma.cn，再去执行start-hbase.sh就好了
```

![启动hbase](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/1.png)

#### 2.1.8 验证Hbase是否启动成功

> 启动hbase shell客户端

```
hbase shell
```

> 输入status

```
[root@node1 onekey]# hbase shell
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/export/servers/hadoop-2.7.5/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/export/servers/hbase-2.1.0/lib/client-facing-thirdparty/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
HBase Shell
Use "help" to get list of supported commands.
Use "exit" to quit this interactive shell.
Version 2.1.0, re1673bb0bbfea21d6e5dba73e013b09b8b49b89b, Tue Jul 10 17:26:48 CST 2018
Took 0.0034 seconds                                                                                                                                           
Ignoring executable-hooks-1.6.0 because its extensions are not built. Try: gem pristine executable-hooks --version 1.6.0
Ignoring gem-wrappers-1.4.0 because its extensions are not built. Try: gem pristine gem-wrappers --version 1.4.0
2.4.1 :001 > status
1 active master, 0 backup masters, 3 servers, 0 dead, 0.6667 average load
Took 0.4562 seconds                                                                                                                                           
2.4.1 :002 >
```

报错的话可以去日志/export/servers/hbase-2.1.0/logs里查看具体的报错信息，里面有master和regionserver的日志

![验证Hbase是否启动成功](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/2.png)

### 2.2 [WebUI](http://node1.xiaoma.cn:16010/master-status)

http://node1.xiaoma.cn:16010/master-status

![WebUI验证](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/3.png)

### 2.3 安装目录说明

目录名 | 说明
--- | ---
bin | 所有hbase相关的命令都在该目录存放
conf | 所有的hbase配置文件
hbase-webapps | hbase的web ui程序位置
lib | hbase依赖的java库
logs | hbase的日志文件

### 2.4 参考硬件配置

> 针对大概800TB存储空间的集群中每个Java进程的典型内存配置：

进程 | 堆 | 描述
--- | --- | --- 
NameNode | 8 GB | 每100TB数据或每100W个文件大约占用NameNode堆1GB的内存
SecondaryNameNode | 8GB | 在内存中重做主NameNode的EditLog，因此配置需要与NameNode一样
DataNode | 1GB | 适度即可
ResourceManager | 4GB | 适度即可（注意此处是MapReduce的推荐配置）
NodeManager | 2GB | 适当即可（注意此处是MapReduce的推荐配置）
HBase HMaster | 4GB | 轻量级负载，适当即可
HBase RegionServer | 12GB | 大部分可用内存、同时为操作系统缓存、任务进程留下足够的空间
ZooKeeper | 1GB | 适度

推荐：

> Master机器要运行NameNode、ResourceManager、以及HBase HMaster，推荐24GB左右<bn/>
Slave机器需要运行DataNode、NodeManager和HBase RegionServer，推荐24GB（及以上）<bn/>
根据CPU的核数来选择在某个节点上运行的进程数，例如：两个4核CPU=8核，每个Java进程都可以独立占有一个核（推荐：8核CPU）<bn/>
内存不是越多越好，在使用过程中会产生较多碎片，Java堆内存越大， 会导致整理内存需要耗费的时间越大。例如：给RegionServer的堆内存设置为64GB就不是很好的选择，一旦FullGC就会造成较长时间的等待，而等待较长，Master可能就认为该节点已经挂了，然后移除掉该节点

## 3 HBase数据模型

### 3.1 简介

> 在HBASE中，数据存储在具有行和列的表中。这是看起来关系数据库(RDBMS)一样，但将HBASE表看成是多个维度的Map结构更容易理解。

![HBase数据模型简介](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/4.png)

```
{
  "zzzzz" : "woot",
  "xyz" : "hello",@
  "aaaab" : "world",
  "1" : "x",
  "aaaaa" : "y"
}
```

### 3.2 术语

#### 3.2.1 表（Table）

> HBase中数据都是以表形式来组织的<br/>
HBase中的表由多个行组成

在HBase WebUI（http://node1.xiaoma.cn:16010中可以查看到目前HBase中的表）

![表](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/5.png)

#### 3.2.2 行键（row key）

> HBASE中的行由一个rowkey（行键）和一个或多个列组成，列的值与rowkey、列相关联<br/>
行在存储时按行键按字典顺序排序<br/>
行键的设计非常重要，尽量让相关的行存储在一起

> 例如：存储网站域。如行键是域，则应该将域名反转后存储(org.apache.www、org.apache.mail、org.apache.jira)。这样，所有Apache域都在表中存储在一起，而不是根据子域的第一个字母展开

> 后续，我们会讲解rowkey的设计策略。

#### 3.2.3 列（Column）

> HBASE中的列由列族（Column Family）和列限定符（Column Qualifier）组成<br/>
表示如下——列族名:列限定符名。例如：C1:USER_ID、C1:SEX

#### 3.2.4 列族（Column Family）

![列族（Column Family）](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/6.png)

> 出于性能原因，列族将一组列及其值组织在一起<br/>
每个列族都有一组存储属性，例如：

是否应该缓存在内存中

数据如何被压缩或行键如何编码等
> 表中的每一行都有相同的列族，但在列族中不存储任何内容<br/>
所有的列族的数据全部都存储在一块（文件系统HDFS）<br/>
HBase官方建议所有的列蔟保持一样的列，并且将同一类的列放在一个列蔟中

#### 3.2.5 列标识符（Column Qualifier）

> 列族中包含一个个的列限定符，这样可以为存储的数据提供索引<br/>
列族在创建表的时候是固定的，但列限定符是不作限制的<br/>
不同的行可能会存在不同的列标识符

#### 3.2.6 单元格（Cell）

> 单元格是行、列系列和列限定符的组合<br/>
包含一个值和一个时间戳（表示该值的版本）<br/>
单元格中的内容是以二进制存储的

ROW | COLUMN+CELL
--- | ---
1250995 | column=C1:ADDRESS, timestamp=1588591604729, value=\xC9\xBD\xCE\xF7\xCA
1250995 | column=C1:LATEST_DATE, timestamp=1588591604729, value=2019-03-28
1250995 | column=C1:NAME, timestamp=1588591604729, value=\xB7\xBD\xBA\xC6\xD0\xF9
1250995 | column=C1:NUM_CURRENT, timestamp=1588591604729, value=398.5
1250995 | column=C1:NUM_PREVIOUS, timestamp=1588591604729, value=379.5
1250995 | column=C1:NUM_USEAGE, timestamp=1588591604729, value=19
1250995 | column=C1:PAY_DATE, timestamp=1588591604729, value=2019-02-26
1250995 | column=C1:RECORD_DATE, timestamp=1588591604729, value=2019-02-11
1250995 | column=C1:SEX, timestamp=1588591604729, value=\xC5\xAE
1250995 | column=C1:TOTAL_MONEY, timestamp=1588591604729, value=114

### 3.3 概念模型

Row Key | Time Stamp | ColumnFamily contents | ColumnFamily anchor | ColumnFamily people
--- | --- | --- | --- | ---
"com.cnn.www" | t9 |  | anchor:cnnsi.com = "CNN"	 |
"com.cnn.www" | t8 |  | anchor:my.look.ca = "CNN.com" | 
"com.cnn.www" | t6 | contents:html = "<html>…​" |  | 
"com.cnn.www" | t5 | contents:html = "<html>…​" |  | 
"com.cnn.www" | t3 | contents:html = "<html>…​" |  | 
"com.example.www" | t5 | contents:html = "<html>…​" |  | people:author = "John Doe"

> 上述表格有两行、三个列族（contens、ancho、people）<br/>
“com.cnn.www”这一行anchor列族两个列（anchor:cssnsi.com、anchor:my.look.ca）、contents列族有个1个列（html）<br/>
“com.cnn.www”在HBase中有 t3、t5、t6、t8、t9 5个版本的数据<br/>
HBase中如果某一行的列被更新的，那么最新的数据会排在最前面，换句话说同一个rowkey的数据是按照倒序排序的

## 4 常用shell操作

> 我们可以以shell的方式来维护和管理HBase。例如：执行建表语句、执行增删改查操作等等。

### 4.1 需求

> 有以下订单数据，我们想要将这样的一些数据保存到HBase中。

订单ID | 订单状态 | 支付金额 | 支付方式ID | 用户ID | 操作时间 | 商品分类 
--- | --- |--- |--- |--- |--- | --- |
001 | 已付款 | 200.5 | 1 | 001 | 2020-5-2 | 18:08:53 | 手机

> 接下来，我们将使用HBase shell来进行以下操作：

1. 创建表
2. 添加数据
3. 更新数据
4. 删除数据
5. 查询数据

### 4.2 创建表

> 在HBase中，所有的数据也都是保存在表中的。要将订单数据保存到HBase中，首先需要将表创建出来。

#### 4.2.1 启动HBase Shell

HBase的shell其实JRuby的IRB（交互式的Ruby），但在其中添加了一些HBase的命令。

启动HBase shell：

```
hbase shell
```

#### 4.2.2 创建表

语法：
```
create '表名','列蔟名'...
```

创建订单表，表名为ORDER_INFO，该表有一个列蔟为C1

```
create 'ORDER_INFO','C1';
```

注意：
> create要写成小写<br/>
一个表可以包含若干个列蔟<br/>
命令解析：调用hbase提供的ruby脚本的create方法，传递两个字符串参数<br/>
通过下面链接可以看到每个命令都是一个ruby脚本

https://github.com/apache/hbase/tree/branch-2.1/hbase-shell/src/main/ruby/shell/commands

#### 4.2.3 查看表

```
hbase(main):005:0> list
TABLE                                                                                                                                                                    
ORDER_INFO                                                                                                                                                               
1 row(s)
Took 0.0378 seconds                                                                                                                                                      
=> ["ORDER_INFO"]
```

#### 4.2.4 删除表

> 要删除某个表，必须要先禁用表

##### 4.2.4.1 禁用表

> 语法：disable "表名"

##### 4.2.4.2 删除表

> 语法：drop "表名"

##### 4.2.4.3 删除ORDER_INFO表

```
disable "ORDER_INFO"
drop "ORDER_INFO"
```

### 4.3 添加数据

#### 4.3.1 需求

> 接下来，我们需要往订单表中添加以下数据。

订单ID | 订单状态 | 支付金额 | 支付方式ID | 用户ID | 操作时间 | 商品分类
--- | --- | --- | --- | --- | --- | ---
ID | STATUS | PAY_MONEY | PAYWAY | USER_ID | OPERATION_DATE | CATEGORY
000001 | 已提交 | 1 | 4944191 | 2020-04-25 | 12:09:16 | 手机

#### 4.3.2 PUT操作

> HBase中的put命令，可以用来将数据保存到表中。但put一次只能保存一个列的值。以下是put的语法结构：

```
put '表名','ROWKEY','列蔟名:列名','值'
```

> 要添加以上的数据，需要使用7次put操作。如下：

```
put 'ORDER_INFO','000001','C1:ID','000001'
put 'ORDER_INFO','000001','C1:STATUS','已提交'
put 'ORDER_INFO','000001','C1:PAY_MONEY',4070
put 'ORDER_INFO','000001','C1:PAYWAY',1
put 'ORDER_INFO','000001','C1:USER_ID',4944191
put 'ORDER_INFO','000001','C1:OPERATION_DATE','2020-04-25 12:09:16'
put 'ORDER_INFO','000001','C1:CATEGORY','手机;'
```

### 4.4 查看添加的数据

#### 4.4.1 需求

> 要求将rowkey为：000001对应的数据查询出来。

#### 4.4.2 get命令

> 在HBase中，可以使用get命令来获取单独的一行数据。语法：

```
get '表名','rowkey'
```

#### 4.4.3 查询指定订单ID的数据

> get 'ORDER_INFO','000001'

COLUMN | CELL
--- | ---
C1:CATEGORY | timestamp=1588415690678, value=\xE6\x89\x8B\xE6\x9C\xBA;             
C1:OPERATION_DATE | timestamp=1588415689773, value=2020-04-25 12:09:16                   
C1:PAYWAY | timestamp=1588415689681, value=1                                     
C1:PAY_MONEY | timestamp=1588415689643, value=4070                                  
C1:STATUS | timestamp=1588415689591, value=\xE5\xB7\xB2\xE6\x8F\x90\xE4\xBA\xA4  
C1:USER_ID | timestamp=1588415689721, value=4944191

#### 4.4.4 显示中文

> 在HBase shell中，如果在数据中出现了一些中文，默认HBase shell中显示出来的是十六进制编码。要想将这些编码显示为中文，我们需要在get命令后添加一个属性：{FORMATTER => 'toString'}

##### 4.4.4.1

```
get 'ORDER_INFO','000001', {FORMATTER => 'toString'}
```


> { key => value}，这个是Ruby语法，表示定义一个HASH结构<br/>
get是一个HBase Ruby方法，’ORDER_INFO’、’000001’、{FORMATTER => 'toString'}是put方法的三个参数<br/>
FORMATTER要使用大写<br/>
在Ruby中用{}表示一个字典，类似于hashtable，FORMATTER表示key、’toString’表示值

### 4.5 更新操作

#### 4.5.1 需求

> 将订单ID为000001的状态，更改为「已付款」

#### 4.5.2 使用put来更新数据

> 同样，在HBase中，也是使用put命令来进行数据的更新，语法与之前的添加数据一模一样。

#### 4.5.3 更新指定的列

```
put 'ORDER_INFO', '000001', 'C1:STATUS', '已付款'
```

> HBase中会自动维护数据的版本<br/>
每当执行一次put后，都会重新生成新的时间戳

```
C1:STATUS   timestamp=1588748844082, value=已提交
C1:STATUS   timestamp=1588748952074, value=已付款
C1:STATUS   timestamp=1588748994244, value=已付款
```

### 4.6 删除操作

#### 4.6.1 删除状态列数据

##### 4.6.1.1 需求

> 将订单ID为000001的状态列删除。

##### 4.6.1.2 delete命令

> 在HBase中，可以使用delete命令来将一个单元格的数据删除。语法格式如下：

```
delete '表名', 'rowkey', '列蔟:列'
```

注意：此处HBase默认会保存多个时间戳的版本数据，所以这里的delete删除的是最新版本的列数据。

##### 4.6.1.3 删除指定的列

```
delete 'ORDER_INFO','000001','C1:STATUS'
```

#### 4.6.2 删除整行数据

##### 4.6.2.1 需求

> 将订单ID为000001的信息全部删除（删除所有的列）

##### 4.6.2.2 deleteall命令

> deleteall命令可以将指定rowkey对应的所有列全部删除。语法：

```
deleteall '表名','rowkey'
```

##### 4.6.2.3 删除指定的订单

```
deleteall 'ORDER_INFO','000001'
```

#### 4.6.3 清空表

##### 4.6.3.1 需求

> 将ORDER_INFO的数据全部删除

##### 4.6.3.2 truncate命令

> truncate命令用来清空某个表中的所有数据。语法：

```
truncate "表名"
```

##### 4.6.3.3 清空ORDER_INFO的所有数据

```
truncate 'ORDER_INFO'
```

### 4.7 导入测试数据集

#### 4.7.1 需求

> 在资料的 数据集/ ORDER_INFO.txt 中，有一份这样的HBase数据集，我们需要将这些指令放到HBase中执行，将数据导入到HBase中。

> 可以看到这些都是一堆的put语句。那么如何才能将这些语句全部执行呢？

![需求](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/7.png)

#### 4.7.2 执行command文件

##### 4.7.2.1 上传command文件

> 将该数据集文件上传到指定的目录中

##### 4.7.2.2 执行

使用以下命令执行：即可。

```
hbase shell /export/software/ORDER_INFO.txt
```

### 4.8 计数操作

#### 4.8.1 需求

> 查看HBase中的ORDER_INFO表，一共有多少条记录。

#### 4.8.2 count命令

> count命令专门用来统计一个表中有多少条数据。语法：

```
count ‘表名’
```

注意：这个操作是比较耗时的。在数据量大的这个命令可能会运行很久。

#### 4.8.3 获取订单数据

```
count 'ORDER_INFO'
```

### 4.9 大量数据的计数统计

当HBase中数据量大时，可以使用HBase中提供的MapReduce程序来进行计数统计。语法如下：

```
$HBASE_HOME/bin/hbase org.apache.hadoop.hbase.mapreduce.RowCounter '表名'
```

#### 4.9.1 启动YARN集群

> 启动yarn集群

```
start-yarn.sh
```

> 启动history server

```
mr-jobhistory-daemon.sh start historyserver
```

#### 4.9.2 执行MR JOB

```
$HBASE_HOME/bin/hbase org.apache.hadoop.hbase.mapreduce.RowCounter 'ORDER_INFO'
```

> 通过观察YARN的WEB UI，我们发现HBase启动了一个名字为rowcounter_ORDER_INFO的作业。

![执行MR JOB](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/8.png)

### 4.10 扫描操作

#### 4.10.1 需求一：查询订单所有数据

##### 4.10.1.1 需求

> 查看ORDER_INFO表中所有的数据

##### 4.10.1.2 scan命令

> 在HBase，我们可以使用scan命令来扫描HBase中的表。语法：

```
scan '表名'
```

##### 4.10.1.3 扫描ORDER_INFO表

```
scan 'ORDER_INFO',{FORMATTER => 'toString'}
```

注意：要避免scan一张大表！

#### 4.10.2 需求二：查询订单数据（只显示3条）

```
scan 'ORDER_INFO', {LIMIT => 3, FORMATTER => 'toString'}
```

#### 4.10.3 需求三：查询订单状态、支付方式

##### 4.10.3.1 需求

> 只查询订单状态以及支付方式，并且只展示3条数据

##### 4.10.3.2 命令

```
scan 'ORDER_INFO', {LIMIT => 3, COLUMNS => ['C1:STATUS', 'C1:PAYWAY'], FORMATTER => 'toString'}
```

注意：[‘C1:STATUS’, …]在Ruby中[]表示一个数组

#### 4.10.4 需求四：查询指定订单ID的数据并以中文展示

> 根据ROWKEY来查询对应的数据，ROWKEY为02602f66-adc7-40d4-8485-76b5632b5b53，只查询订单状态、支付方式，并以中文展示。

>要查询指定ROWKEY的数据，需要使用ROWPREFIXFILTER，用法为：

```
scan '表名', {ROWPREFIXFILTER => 'rowkey'}
```

> 实现指令：

```
scan 'ORDER_INFO', {ROWPREFIXFILTER => '02602f66-adc7-40d4-8485-76b5632b5b53', COLUMNS => ['C1:STATUS', 'C1:PAYWAY'], FORMATTER => 'toString'}
```

### 4.11 过滤器

#### 4.11.1 简介

> 在HBase中，如果要对海量的数据来进行查询，此时基本的操作是比较无力的。此时，需要借助HBase中的高级语法——Filter来进行查询。Filter可以根据列簇、列、版本等条件来对数据进行过滤查询。因为在HBase中，主键、列、版本都是有序存储的，所以借助Filter，可以高效地完成查询。当执行Filter时，HBase会将Filter分发给各个HBase服务器节点来进行查询。

> HBase中的过滤器也是基于Java开发的，只不过在Shell中，我们是使用基于JRuby的语法来实现的交互式查询。以下是HBase 2.2的JAVA API文档。

#### 4.11.2 HBase中的过滤器

> 在HBase的shell中，通过show_filters指令，可以查看到HBase中内置的一些过滤器。

```
hbase(main):028:0> show_filters
DependentColumnFilter                                                                                                                                                       
KeyOnlyFilter                                                                                                                                                               
ColumnCountGetFilter                                                                                                                                                        
SingleColumnValueFilter                                                                                                                                                     
PrefixFilter                                                                                                                                                                
SingleColumnValueExcludeFilter                                                                                                                                              
FirstKeyOnlyFilter                                                                                                                                                          
ColumnRangeFilter                                                                                                                                                           
ColumnValueFilter                                                                                                                                                           
TimestampsFilter                                                                                                                                                            
FamilyFilter                                                                                                                                                                
QualifierFilter                                                                                                                                                             
ColumnPrefixFilter                                                                                                                                                          
RowFilter                                                                                                                                                                   
MultipleColumnPrefixFilter                                                                                                                                                  
InclusiveStopFilter                                                                                                                                                         
PageFilter                                                                                                                                                                  
ValueFilter                                                                                                                                                                 
ColumnPaginationFilter
```
![HBase中的过滤器](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/9.png)

[Java API官方地址](https://hbase.apache.org/devapidocs/index.html)：https://hbase.apache.org/devapidocs/index.html

#### 4.11.3 过滤器的用法

> 过滤器一般结合scan命令来使用。打开HBase的JAVA API文档。找到RowFilter的构造器说明，我们来看以下，HBase的过滤器该如何使用。

```
scan '表名', { Filter => "过滤器(比较运算符, '比较器表达式')" }
```

##### 4.11.3.1 比较运算符

比较运算符 | 描述
---|---
= | 等于
> | 大于
>= | 大于等于
< | 小于
<= | 小于等于
!= | 不等于

##### 4.11.3.2 比较器
比较器 | 描述
---|---
BinaryComparator | 匹配完整字节数组
BinaryPrefixComparator | 匹配字节数组前缀
BitComparator | 匹配比特位
NullComparator | 匹配空值
RegexStringComparator | 匹配正则表达式
SubstringComparator | 匹配子字符串

##### 4.11.3.3 比较器表达式

> 基本语法：比较器类型:比较器的值

比较器 | 表达式语言缩写
---|---
BinaryComparator | binary:值
BinaryPrefixComparator | binaryprefix:值
BitComparator | bit:值
NullComparator | null
RegexStringComparator | regexstring:正则表达式
SubstringComparator | substring:值

#### 4.11.4 需求一：使用RowFilter查询指定订单ID的数据

##### 4.11.4.1 需求

> 只查询订单的ID为：02602f66-adc7-40d4-8485-76b5632b5b53、订单状态以及支付方式

分析

> 1.因为要订单ID就是ORDER_INFO表的rowkey，所以，我们应该使用rowkey过滤器来过滤<br/>
2.通过HBase的JAVA API，找到RowFilter构造器

![需求](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/10.png)

> 通过上图，可以分析得到，RowFilter过滤器接受两个参数，

> op——比较运算符<br/>
rowComparator——比较器

> 所以构建该Filter的时候，只需要传入两个参数即可

##### 4.11.4.2 命令

#### 4.11.5 需求二：查询状态为已付款的订单

##### 4.11.5.1 需求

> 查询状态为「已付款」的订单

分析

> 1.因为此处要指定列来进行查询，所以，我们不再使用rowkey过滤器，而是要使用列过滤器<br/>
2.我们要针对指定列和指定值进行过滤，比较适合使用SingleColumnValueFilter过滤器，查看JAVA API

需要传入四个参数：
> 列簇<br/>
列标识（列名）<br/>
比较运算符<br/>
比较器

注意：

> `列名STATUS的大小写一定要对！此处使用的是大写！`<br/>
`列名写错了查不出来数据，但HBase不会报错，因为HBase是无模式的`

##### 4.11.5.2 命令

```
scan 'ORDER_INFO', {FILTER => "SingleColumnValueFilter('C1', 'STATUS', =, 'binary:已付款')", FORMATTER => 'toString'}
```

#### 4.11.6 需求三：查询支付方式为1，且金额大于3000的订单

> 分析<br/>
此处需要使用多个过滤器共同来实现查询，多个过滤器，可以使用AND或者OR来组合多个过滤器完成查询<br/>
使用SingleColumnValueFilter实现对应列的查询

##### 4.11.6.1 命令

> 1.查询支付方式为1

```
SingleColumnValueFilter('C1', 'PAYWAY', = , 'binary:1')
```

> 2.查询金额大于3000的订单

```
SingleColumnValueFilter('C1', 'PAY_MONEY', > , 'binary:3000')
```

> 3.组合查询

```
scan 'ORDER_INFO', {FILTER => "SingleColumnValueFilter('C1', 'PAYWAY', = , 'binary:1') AND SingleColumnValueFilter('C1', 'PAY_MONEY', > , 'binary:3000')", FORMATTER => 'toString'}
```

> 注意：<br/>
HBase shell中比较默认都是字符串比较，所以如果是比较数值类型的，会出现不准确的情况<br/>
例如：在字符串比较中4000是比100000大的

### 4.12 INCR

#### 4.12.1 需求

> 某新闻APP应用为了统计每个新闻的每隔一段时间的访问次数，他们将这些数据保存在HBase中。<br/>
该表格数据如下所示：

新闻ID | 访问次数 | 时间段 | ROWKEY
---|---|---|---
0000000001 | 12 | 00:00-01:00 | 0000000001_00:00-01:00
0000000002 | 12 | 01:00-02:00 | 0000000002_01:00-02:00

> 要求：原子性增加新闻的访问次数值。

#### 4.12.2 incr操作简介

> incr可以实现对某个单元格的值进行原子性计数。语法如下：

```
incr '表名','rowkey','列蔟:列名',累加值（默认累加1）
```

> 如果某一列要实现计数功能，必须要使用incr来创建对应的列<br/>
使用put创建的列是不能实现累加的

#### 4.12.3 导入测试数据

![导入测试数据](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/11.png)

> 该脚本创建了一个表，名为NEWS_VISIT_CNT，列蔟为C1。并使用incr创建了若干个计数器，每个rowkey为：新闻的编号_时间段。CNT为count的缩写，表示访问的次数。

hbase shell /export/software/NEWS_VISIT_CNT.txt
scan 'NEWS_VISIT_CNT', {LIMIT => 5, FORMATTER => 'toString'}

#### 4.12.4 需求一：对0000000020新闻01:00 - 02:00访问计数+1

> 1.获取0000000020这条新闻在01:00-02:00当前的访问次数

```
get_counter 'NEWS_VISIT_CNT','0000000020_01:00-02:00','C1:CNT'
```

此处，如果用get获取到的数据是这样的：

```
base(main):029:0> get 'NEWS_VISIT_CNT','0000000020_01:00-02:00','C1:CNT'
COLUMN                                           CELL                                                                                                                                        
C1:CNT                                          timestamp=1599529533072, value=\x00\x00\x00\x00\x00\x00\x00\x06                                                                             
1 row(s)
Took 0.0243 seconds
```

> 2.使用incr进行累加

```
incr 'NEWS_VISIT_CNT','0000000020_01:00-02:00','C1:CNT'
```

> 3.再次查案新闻当前的访问次数

```
get_counter 'NEWS_VISIT_CNT','0000000020_01:00-02:00','C1:CNT'
```

### 4.13 更多的操作

> [以下连接可以查看到所有HBase中支持的SHELL脚本。](https://learnhbase.net/2013/03/02/hbase-shell-commands/)

```
https://learnhbase.net/2013/03/02/hbase-shell-commands/
```

## 5 shell管理操作

### 5.1 status

> 例如：显示服务器状态

```
2.4.1 :062 > status
1 active master, 0 backup masters, 3 servers, 0 dead, 1.0000 average load
Took 0.0034 seconds
```

### 5.2 whoami

> 显示HBase当前用户，例如：

```
2.4.1 :066 > whoami
root (auth:SIMPLE)
    groups: root
Took 0.0080 seconds
```
 
### 5.3 list

> 显示当前所有的表

```
2.4.1 :067 > list
TABLE                                                                                                                                                  
ORDER_INFO                                                                                                                                             
1 row(s)
Took 0.0266 seconds                                                                                                                                    
 => ["ORDER_INFO"]
 ```

### 5.4 count

> 统计指定表的记录数，例如：

```
2.4.1 :070 > count 'ORDER_INFO'
66 row(s)
Took 0.0404 seconds                                                                                                                                    
=> 66
```

### 5.5 describe

> 展示表结构信息

```
2.4.1 :074 > describe 'ORDER_INFO'
Table ORDER_INFO is ENABLED                                                                                                                            
ORDER_INFO                                                                                                                                             
COLUMN FAMILIES DESCRIPTION                                                                                                                            
{NAME => 'C1', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE =
> 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE =
> 'false', IN_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLO
CKSIZE => '65536'}                                                                                                                                     
1 row(s)
Took 0.0265 seconds
```

### 5.6 exists

> 检查表是否存在，适用于表量特别多的情况

```
2.4.1 :075 > exists 'ORDER_INFO'
Table ORDER_INFO does exist                                                                                                                            
Took 0.0050 seconds                                                                                                                                    
=> true
```

### 5.7 is_enabled，is_disable

> 检查表是否启用或禁用

```
2.4.1 :077 > is_enabled 'ORDER_INFO'
true                                                                                                                                                   
Took 0.0058 seconds                                                                                                                                    
=> true
2.4.1 :078 > is_disabled 'ORDER_INFO'
false                                                                                                                                                  
Took 0.0085 seconds                                                                                                                                    
=> 1
```

### 5.8 alter

> 该命令可以改变表和列族的模式，例如：

```
# 创建一个USER_INFO表，两个列蔟C1、C2
create 'USER_INFO', 'C1', 'C2'
# 新增列蔟C3
alter 'USER_INFO', 'C3'
# 删除列蔟C3
alter 'USER_INFO', 'delete' => 'C3'
```

注意：

```
'delete' => 'C3'，还是一个Map结构，只不过只有一个key，可以省略两边的{}
```

### 5.9 disable/enable

> 禁用一张表/启用一张表

### 5.10 drop 

> 删除一张表，记得在删除表之前必须先禁用

### 5.11 truncate

> 清空表的数据，禁用表-删除表-创建表

## 6 HBase Java操作

### 6.1 需求与数据集

> 某某自来水公司，需要存储大量的缴费明细数据。以下截取了缴费明细的一部分内容。

用户id | 姓名 | 用户地址 | 性别 | 缴费时间 | 表示数（本次） | 表示数（上次） | 用量（立方） | 合计金额 | 查表日期 | 最迟缴费日期 
---|---|---|---|---|---|---|---|---|---|---
4944191 | 登卫红 | 贵州省铜仁市德江县7单元267室 | 男 | 2020-05-10 | 308.1 | 283.1 | 25 | 150 | 2020-04-25 | 2020-06-09 

> 因为缴费明细的数据记录非常庞大，该公司的信息部门决定使用HBase来存储这些数据。并且，他们希望能够通过Java程序来访问这些数据。

### 6.2 依赖

```xml
<repositories><!-- 代码库 -->
        <repository>
            <id>aliyun</id>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
                <updatePolicy>never</updatePolicy>
            </snapshots>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>2.1.0</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>6.14.3</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <target>1.8</target>
                    <source>1.8</source>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### 6.3 需求代码

```
package cn.xiaoma.hbase.admin.api_test;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.filter.BinaryPrefixComparator;
import org.apache.hadoop.hbase.filter.SingleColumnValueFilter;
import org.apache.hadoop.hbase.util.Bytes;
import org.testng.annotations.AfterTest;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Test;

import java.util.List;

/**
 @desc 此类用于java操作hbase相关测试类
 @date 2021-01-18 14:48:07
 @author xiaoma
 */
public class TableAmdinTest {
    //需求一: 创建一个名为WATER_BILL的表，包含一个列蔟C1。
    @Test
    @SuppressWarnings("all")
    public void test01() throws Exception{

        //1. 获取hbase的连接对象:
        Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum","node1:2181,node2:2181,node3:2181");
        Connection connection = ConnectionFactory.createConnection(conf);

        //2. 获取执行操作的相关管理对象:  admin对象(对表的操作)  和  table对象 (对表数据的操作)

        Admin admin = connection.getAdmin();

        //3. 执行相关的操作: 创建表

        //3.1: 先判断要创建表是否存在
        boolean flag = admin.tableExists(TableName.valueOf("WATER_BILL")); // 如果存在 就为 true  如果不存在 为 false
        if(flag){
            return;
        }

        //3.2: 创建表
        TableDescriptorBuilder tableDescriptorBuilder = TableDescriptorBuilder.newBuilder(TableName.valueOf("WATER_BILL"));

        ColumnFamilyDescriptor familyDescriptor = ColumnFamilyDescriptorBuilder.newBuilder("C1".getBytes()).build();
        tableDescriptorBuilder.setColumnFamily(familyDescriptor);

        TableDescriptor tableDescriptor = tableDescriptorBuilder.build();
        admin.createTable(tableDescriptor);

        //4. 释放资源

        admin.close();
        connection.close();

    }
    private  String tableName = "WATER_BILL";
    //需求二: 插入一条数据:
    @Test
    public void test02() throws Exception {

        //1. 获取 hbase的客户端连接对象
        Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum","node1:2181,node2:2181,node3:2181");
        Connection connection = ConnectionFactory.createConnection(conf);
        //2. 根据连接对象, 获取相关的管理对象: admin  table
        Table table = connection.getTable(TableName.valueOf(tableName));
        //3. 执行相关的操作: 添加数据   put命令

        Put put = new Put("4944191".getBytes());
        put.addColumn("C1".getBytes(),"NAME".getBytes(),"登卫红".getBytes());

        table.put(put); // 增 删 该 不需要返回值

        //4. 释放资源
        table.close();
        connection.close();

    }
    private  Connection connection;
    private  Admin admin ;
    private Table table;
    @BeforeTest
    public  void before() throws Exception{
        //1. 获取 hbase的客户端连接对象
        Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum","node1:2181,node2:2181,node3:2181");
        connection = ConnectionFactory.createConnection(conf);

        //2. 根据连接对象, 获取相关的管理对象: admin  table
        admin = connection.getAdmin();
        table = connection.getTable(TableName.valueOf(tableName));
    }


    /*

        列名	说明	值
        ADDRESS	用户地址	贵州省铜仁市德江县7单元267室
        SEX	性别	男
        PAY_DATE	缴费时间	2020-05-10
        NUM_CURRENT	表示数（本次）	308.1
        NUM_PREVIOUS	表示数（上次）	283.1
        NUM_USAGE	用量（立方）	25
        TOTAL_MONEY	合计金额	150
        RECORD_DATE	查表日期	2020-04-25
        LATEST_DATE	最迟缴费日期	2020-06-09

     */
    // 需求3: 添加一行数据: 多列
    @Test
    public void test03() throws Exception{
        //1. 根据连接工厂, 创建连接对象: @Before

        //2. 根据连接对象, 创建相关的管理对象:  @Before

        //3. 执行相关的操作: 添加一行数据: 多列

        Put put = new Put("4944191".getBytes()); // 一个put表示封装一行数据
        put.addColumn("C1".getBytes(),"ADDRESS".getBytes(),"贵州省铜仁市德江县7单元267室".getBytes());
        put.addColumn("C1".getBytes(),"SEX".getBytes(),"男".getBytes());
        put.addColumn("C1".getBytes(),"PAY_DATE".getBytes(),"2020-05-10".getBytes());
        put.addColumn("C1".getBytes(),"NUM_CURRENT".getBytes(),"308.1".getBytes());
        put.addColumn("C1".getBytes(),"NUM_PREVIOUS".getBytes(),"283.1".getBytes());
        put.addColumn("C1".getBytes(),"NUM_USAGE".getBytes(),"25".getBytes());
        put.addColumn("C1".getBytes(),"TOTAL_MONEY".getBytes(),"150".getBytes());
        put.addColumn("C1".getBytes(),"RECORD_DATE".getBytes(),"2020-04-25".getBytes());
        put.addColumn("C1".getBytes(),"LATEST_DATE".getBytes(),"2020-06-09".getBytes());


        table.put(put);

        //4. 释放资源:  @After


    }

    // 需求4: 如何查询某一条数据:
    @Test
    public void test04() throws Exception{

        //1. 根据连接工厂, 创建连接对象: @Before

        //2. 根据连接对象, 创建相关的管理对象:  @Before

        //3. 执行相关的操作: 如何查询某一条数据: get
        Get get = new Get("4944191".getBytes());
        Result result = table.get(get);

        //4. 获取数据 : 一个 result对象 就是一行数据对象
        List<Cell> cellList = result.listCells();

        for (Cell cell : cellList) { // 每一个单元格: rowkey 列族 列名 列值
            // 对于单元格操作, 专门hbase提供了一个工具类 CellUtils
            byte[] rowkeyBytes = CellUtil.cloneRow(cell);
            String rowkey = Bytes.toString(rowkeyBytes);

            byte[] familyBytes = CellUtil.cloneFamily(cell);
            String family = Bytes.toString(familyBytes);

            byte[] qualifierBytes = CellUtil.cloneQualifier(cell);
            String qualifier = Bytes.toString(qualifierBytes);

            byte[] valueBytes = CellUtil.cloneValue(cell);
            String value = Bytes.toString(valueBytes);


            System.out.println("rowkey为:"+rowkey +";列族为:"+family+";列名为:"+qualifier+";列值为:"+value);
        }


        //5. 释放资源:  @After

    }

    //需求5: 如何删除一条数据
    @Test
    public void test05() throws Exception{

        //1. 根据连接工厂, 创建连接对象: @Before

        //2. 根据连接对象, 创建相关的管理对象:  @Before

        //3. 执行相关的操作: 如何删除一条数据 说明: delete 删除整个一行 删除某几列, 删除某几个列族数据
        Delete delete = new Delete("4944191".getBytes());
        //delete.addColumn("C1".getBytes(),"ADDRESS".getBytes());

        table.delete(delete);


        //4. 释放资源:  @After

    }

    // 需求6: 删除表的操作
    @Test
    public void test06() throws Exception{

        //1. 根据连接工厂, 创建连接对象: @Before

        //2. 根据连接对象, 创建相关的管理对象:  @Before


        //3. 执行相关的操作:删除表的操作
        //3.1: 禁用表
        admin.disableTable(TableName.valueOf(tableName));

        //3.2: 判断表是否已经禁用了呢
        boolean flag = admin.isTableDisabled(TableName.valueOf(tableName));
        if(flag){
            admin.deleteTable(TableName.valueOf(tableName));

        }

        //4. 释放资源:  @After

    }

    // 需求7: 查询2020年6月份所有用户的用水量
    @Test
    public void test07() throws Exception{

        //1. 根据连接工厂, 创建连接对象: @Before

        //2. 根据连接对象, 创建相关的管理对象:  @Before


        //3. 执行相关的操作:查询2020年6月份所有用户的用水量
        Scan scan = new Scan();  // 扫描全部数据

        SingleColumnValueFilter filter = new SingleColumnValueFilter(
                "C1".getBytes(), "RECORD_DATE".getBytes(), CompareOperator.EQUAL, new BinaryPrefixComparator("2020-06".getBytes()));
        scan.setFilter(filter);


        // 结果需要有: NAME NUM_USAGE
        scan.addColumn("C1".getBytes(),"LATEST_DATE".getBytes());

        ResultScanner results = table.getScanner(scan);

        //4. 获取数据:
        for (Result result : results) {

            List<Cell> cellList = result.listCells();

            for (Cell cell : cellList) {

                String rowkey = Bytes.toString(CellUtil.cloneRow(cell));
                String family = Bytes.toString(CellUtil.cloneFamily(cell));
                String qualifier = Bytes.toString(CellUtil.cloneQualifier(cell));

                //NUM_USAGE这一列用double类型展示，因为如果存的不是string，那么就会打印乱码
                if(qualifier.equals("NUM_USAGE")){
                    Double value = Bytes.toDouble(CellUtil.cloneValue(cell));
                    System.out.println("rowkey为:"+rowkey +";列族为:"+family+";列名为:"+qualifier+";列值为:"+value);
                }else{
                    String value = Bytes.toString(CellUtil.cloneValue(cell));
                    System.out.println("rowkey为:"+rowkey +";列族为:"+family+";列名为:"+qualifier+";列值为:"+value);
                }



            }

            System.out.println("---------------------------");
        }


        //5. 释放资源

    }

    @AfterTest
    public void close() throws Exception{
        table.close();
        admin.close();
        connection.close();
    }
}

```

## 7 HBase高可用

考虑关于HBase集群的一个问题，在当前的HBase集群中，只有一个Master，一旦Master出现故障，将会导致HBase不再可用。所以，在实际的生产环境中，是非常有必要搭建一个高可用的HBase集群的。

### 7.1 HBase高可用简介

HBase的高可用配置其实就是HMaster的高可用。要搭建HBase的高可用，只需要再选择一个节点作为HMaster，在HBase的conf目录下创建文件backup-masters，然后再backup-masters添加备份Master的记录。一条记录代表一个backup master，可以在文件配置多个记录。

### 7.2 HBase高可用搭建

> 1.在hbase的conf文件夹中创建backup-masters文件

```
cd/export/servers/hbase-2.1.0/conf
touch backup-masters
```

> 2.将node2.xiaoma.cn和node3.xiaoma.cn添加到该文件中

```
vim backup-masters
node2.xiaoma.cn
node3.xiaoma.cn
```

> 3.将backup-masters文件分发到所有的服务器节点中

```
scp backup-masters node2.xiaoma.cn:$PWD
scp backup-masters node3.xiaoma.cn:$PWD
```

> 4.重新启动hbase

```
stop-hbase.sh
start-hbase.sh
```

> 5.查看webui，检查BackupMasters中是否有node2.xiaoma.cn、node3.xiaoma.cn

```
http://node1.xiaoma.cn:16010/master-status
```

> 6.尝试杀掉node1.xiaoma.cn节点上的master

```
kill -9 HMaster进程id
```

> 7.访问http://node2.xiaoma.cn:16010和http://node3.xiaoma.cn:16010，观察是否选举了新的Master

> 8 再开启node1的HMaster

```
hbase-daemon.sh start master
```

## 8 HBase架构

### 8.1 系统架构

![HBase架构](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/12.png)

![HBase架构](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/13.png)

#### 8.1.1 Client

> 客户端，例如：发出HBase操作的请求。例如：之前我们编写的Java API代码、以及HBase shell，都是CLient

#### 8.1.2 Master Server

> 在HBase的Web UI中，可以查看到Master的位置。

- 监控RegionServer
- 处理RegionServer故障转移
- 处理元数据的变更
- 处理region的分配或移除
- 在空闲时间进行数据的负载均衡
- 通过Zookeeper发布自己的位置给客户端

#### 8.1.3 Region Server

> 处理分配给它的Region<br/>
负责存储HBase的实际数据<br/>
刷新缓存到HDFS<br/>
维护HLog<br/>
执行压缩<br/>
负责处理Region分片

> RegionServer中包含了大量丰富的组件，如下：

- Write-Ahead logs
- HFile(StoreFile)
- Store
- MemStore
- Region

### 8.2 逻辑结构模型

![逻辑结构模型](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/14.png)

#### 8.2.1 Region

> 在HBASE中，表被划分为很多「Region」，并由Region Server提供服务

![Region](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/15.png)

#### 8.2.2 Store

> Region按列族垂直划分为「Store」，存储在HDFS在文件中

#### 8.2.3 MemStore

> MemStore与缓存内存类似<br/>
当往HBase中写入数据时，首先是写入到MemStore<br/>
每个列族将有一个MemStore<br/>
当MemStore存储快满的时候，整个数据将写入到HDFS中的HFile中

#### 8.2.4 StoreFile

> 每当任何数据被写入HBASE时，首先要写入MemStore<br/>
当MemStore快满时，整个排序的key-value数据将被写入HDFS中的一个新的HFile中<br/>
写入HFile的操作是连续的，速度非常快<br/>
物理上存储的是HFile

#### 8.2.5 WAL

> WAL全称为Write Ahead Log，它最大的作用就是	故障恢复<br/>
WAL是HBase中提供的一种高并发、持久化的日志保存与回放机制<br/>
每个业务数据的写入操作（PUT/DELETE/INCR），都会保存在WAL中<br/>
一旦服务器崩溃，通过回放WAL，就可以实现恢复崩溃之前的数据<br/>
物理上存储是Hadoop的Sequence File

## 9 表结构设计

### 9.1 名称空间

#### 9.1.1 说明

- 在一个项目中，需要使用HBase保存多张表，这些表会按照业务域来划分
- 为了方便管理，不同的业务域以名称空间（namespace)来划分，这样管理起来会更加容易
- 类似于Hive中的数据库，不同的数据库下可以放不同类型的表
- HBase默认的名称空间是「default」，默认情况下，创建表时表都将创建在 default 名称空间下
- HBase中还有一个命名空间「hbase」，用于存放系统的内建表（namespace、meta）

#### 9.1.2 语法

##### 9.1.1 创建命名空间

```
create_namespace 'MOMO_CHAT'
```

##### 9.1.2 查看命名空间列表

```
list_namespace
```

##### 9.1.3 查看命名空间

```
describe_namespace 'MOMO_CHAT'
```

##### 9.1.4 命名空间创建表

> 在命令MOMO_CHAT命名空间下创建名为：MSG的表，该表包含一个名为C1的列蔟。<br/>
注意：带有命名空间的表，使用冒号将命名空间和表名连接到一起。

```
create 'MOMO_CHAT:MSG','C1'
```

##### 9.1.5 删除命名空间

> 删除命名空间，命名空间中必须没有表，如果命名空间中有表，是无法删除的

```
drop_namespace 'MOMO_CHAT'
```

### 9.2 列簇设计

> HBase列蔟的数量应该越少越好

- 两个及以上的列蔟HBase性能并不是很好
- 一个列蔟所存储的数据达到flush的阈值时，表中所有列蔟将同时进行flush操作
- 这将带来不必要的I/O开销，列蔟越多，对性能影响越大

`原则: 能少则少 原则上能用一个列族解决的, 坚决不使用两个  最多不超过5个`

### 9.3 版本设计

#### 9.3.1 说明

此处，我们需要保存的历史聊天记录是不会更新的，一旦数据保存到HBase中，就不会再更新
无需考虑版本问题
本次项目中只保留一个版本即可，这样可以节省大量空间
HBase默认创建表的版本为1，故此处保持默认即可

#### 9.3.2 查看表

> 通过以下输出可以看到：

- 版本是相对于列蔟而言
- 默认列蔟的版本数为1

```
hbase(main):015:0> describe "MOMO_CHAT:MSG"
Table MOMO_CHAT:MSG is ENABLED                                                                                                                                                                                                               
MOMO_CHAT:MSG                                                                                                                                                                                                                                
COLUMN FAMILIES DESCRIPTION                                                                                                                                                                                                                  
{NAME => 'C1', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE => 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLI
CATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE => 'false', IN_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}

1 row(s)
```

### 9.4 数据压缩

#### 9.4.1 压缩算法

> 在HBase可以使用多种压缩编码，包括LZO、SNAPPY、GZIP。只在硬盘压缩，内存中或者网络传输中没有压缩。

压缩算法 | 压缩后占比 | 压缩 | 解压缩
---|---|---|---
GZIP | 13.4% | 21 MB/s | 118 MB/s
LZO | 20.5% | 135 MB/s | 410 MB/s
Zippy/Snappy | 22.2% | 172 MB/s | 409 MB/s

- GZIP的压缩率最高，但是其实CPU密集型的，对CPU的消耗比其他算法要多，压缩和解压速度也慢；
- LZO的压缩率居中，比GZIP要低一些，但是压缩和解压速度明显要比GZIP快很多，其中解压速度快的更多；
- Zippy/Snappy的压缩率最低，而压缩和解压速度要稍微比LZO要快一些
- 本案例采用GZ算法，这样可以确保的压缩比最大化，更加节省空间

> 如何选择压缩算法:

- 如果数据只是写入, 不进行读取: 一般选择 gzip  或者  LZO  如果要求压缩比最高 选择 gzip
- 如果数据既要进行写入, 又要进行读取: 建议选择 snappy

#### 9.4.2 查看表数据压缩方式

> 通过以下输出可以看出，HBase创建表默认是没有指定压缩算法的

```
hbase(main):015:0> describe "MOMO_CHAT:MSG"
Table MOMO_CHAT:MSG is ENABLED                                                                                                                                                                                                               
MOMO_CHAT:MSG                                                                                                                                                                                                                                
COLUMN FAMILIES DESCRIPTION                                                                                                                                                                                                                  
{NAME => 'C1', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE => 'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLI
CATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE => 'false', IN_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}

1 row(s)
```

#### 9.4.3 设置数据压缩

> 本案例中，我们使用GZ压缩算法，语法如下：

> 创建新的表，并指定数据压缩算法

```
create "MOMO_CHAT:MSG", {NAME => "C1", COMPRESSION => "GZ"}
```

> 修改已有的表，并指定数据压缩算法

```
alter "MOMO_CHAT:MSG", {NAME => "C1", COMPRESSION => "GZ"}
```

### 9.5 ROWKEY设计原则

#### 9.5.1 HBase官方的设计原则

##### 9.5.1.1 避免使用递增行键/时序数据

> 如果ROWKEY设计的都是按照顺序递增（例如：时间戳），这样会有很多的数据写入时，负载都在一台机器上。我们尽量应当将写入大压力均衡到各个RegionServer

##### 9.5.1.2 避免ROWKEY和列的长度过大

> 在HBase中，要访问一个Cell（单元格），需要有ROWKEY、列蔟、列名，如果ROWKEY、列名太大，就会占用较大内存空间。所以ROWKEY和列的长度应该尽量短小

> ROWKEY的最大长度是64KB，建议越短越好，不宜过长，一般为100字节以内，大部分业务定义0-20字节之间

> 列的长度也是不宜过长，一般0-10

##### 9.5.1.3 使用long等类型比String类型更省空间

> long类型为8个字节，8个字节可以保存非常大的无符号整数，例如：18446744073709551615。如果是字符串，是按照一个字节一个字符方式保存，需要快3倍的字节数存储。

##### 9.5.1.4 ROWKEY唯一性

> 设计ROWKEY时，必须保证RowKey的唯一性

> 由于在HBase中数据存储是Key-Value形式，若向HBase中同一张表插入相同RowKey的数据，则原先存在的数据会被新的数据覆盖。

#### 9.5.2 避免数据热点

> 热点是指大量的客户端（client）直接访问集群的一个或者几个节点（可能是读、也可能是写）

> 大量地访问量可能会使得某个服务器节点超出承受能力，导致整个RegionServer的性能下降，其他的Region也会受影响

##### 9.5.2.1 预分区

> 默认情况，一个HBase的表只有一个Region，被托管在一个RegionServer中

![预分区](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/16.png)

> 每个Region有两个重要的属性：Start Key、End Key，表示这个Region维护的ROWKEY范围

> 如果只有一个Region，那么Start Key、End Key都是空的，没有边界。所有的数据都会放在这个Region中，但当数据越来越大时，会将Region分裂，取一个Mid Key来分裂成两个Region

> 预分区个数 = 节点的倍数。默认Region的大小为10G，假设我们预估1年下来的大小为10T，则10000G / 10G = 1000个Region，所以，我们可以预设为1000个Region，这样，1000个Region将均衡地分布在各个节点上

##### 9.5.2.2 ROWKEY避免热点设计

> 1.反转策略

- 如果设计出的ROWKEY在数据分布上不均匀，但ROWKEY尾部的数据却呈现出了良好的随机性，可以考虑`将ROWKEY的翻转`，或者直接将尾部的bytes提前到ROWKEY的开头。
- 反转策略可以使ROWKEY随机分布，但是牺牲了ROWKEY的有序性
- 缺点：利于Get操作，但不利于Scan操作，因为数据在原ROWKEY上的自然顺序已经被打乱

> 2.加盐策略

- Salting（加盐）的原理是在原ROWKEY的`前面`添加固定长度的随机数，也就是给ROWKEY分配一个随机前缀使它和之间的ROWKEY的开头不同
- 随机数能保障数据在所有Regions间的负载均衡
- 缺点：因为添加的是随机数，基于原ROWKEY查询时无法知道随机数是什么，那样在查询的时候就需要去各个可能的Regions中查找，加盐对比读取是无力的

> 3.哈希策略

- 基于 ROWKEY的完整或部分数据进行 Hash，而后将Hashing后的值完整替换或部分替换原ROWKEY的前缀部分
- 这里说的 hash 包含 MD5、sha1、sha256 或 sha512 等算法
- 缺点：Hashing 也不利于 Scan，因为打乱了原RowKey的自然顺序

#### 9.5.3 陌陌打招呼数据预分区

##### 9.5.3.1 预分区

> 在HBase中，可以通过指定start key、end key来进行分区，还可以直接指定Region的数量，指定分区的策略。

> 1.指定 start key、end key来分区

```
hbase> create 'ns1:t1', 'f1', SPLITS => ['10', '20', '30', '40']
hbase> create 't1', 'f1', SPLITS => ['10', '20', '30', '40']
hbase> create 't1', 'f1', SPLITS_FILE => 'splits.txt', OWNER => 'johndoe'
```

> 2.指定分区数量、分区策略

```
hbase> create 't1', 'f1', {NUMREGIONS => 15, SPLITALGO => 'HexStringSplit'}
```

> 分区策略

- HexStringSplit: ROWKEY是十六进制的字符串作为前缀的
- DecimalStringSplit: ROWKEY是10进制数字字符串作为前缀的
- UniformSplit: ROWKEY前缀完全随机

> Region的数量可以按照数据量来预估。本次案例，因为受限于虚拟机，所以我们设计为6个Region。因为ROWKEY我们是使用多个字段拼接，而且前缀不是完全随机的，所以需要使用HexStringSplit。

##### 9.5.3.2 ROWKEY设计

- 为了确保数据均匀分布在每个Region，需要以MD5Hash作为前缀
- `ROWKEY = MD5Hash_发件人账号_收件人账号_时间戳`

![ROWKEY设计](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/17.png)

##### 9.5.3.3 业务分区脚本

```
create 'MOMO_CHAT:MSG', {NAME => "C1", COMPRESSION => "GZ"}, { NUMREGIONS => 6, SPLITALGO => 'HexStringSplit'}
```

> 执行完命令后，我们发现该表已经分为6个分区。这样将来数据就可以均匀地分布到不同的分区中了

`注意：勾选ShowDetailName&Start/EndKey，点击Recorder`

![业务分区脚本](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/18.png)

HDFS中，也有对应的6个文件夹。
URL：/hbase/data/MOMO_CHAT/MSG

![HDFS查看](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/19.png)

## 10 性能问题

> Hbase默认只支持对行键的索引，那么如果要针对其它的列来进行查询，就只能全表扫描

> 上述的查询是使用scan + filter组合来进行查询的，但查询地效率不高，因为要进行顺序全表扫描而没有其他索引。如果数据量较大，只能在客户端（client）来进行处理，如果要传输到Client大量的数据，然后交由客户端处理

- `网络传输压力很大`
- `客户端的压力很大`

> 如果表存储的数据量很大时，效率会非常低下，此时需要使用二级索引

> 也就是除了ROWKEY的索引外，还需要人为添加其他的方便查询的索引

> 如果每次需要我们开发二级索引来查询数据，这样使用起来很麻烦。再者，查询数据都是HBase Java API，使用起来不是很方便。为了让其他开发人员更容易使用该接口。如果有一种SQL引擎，通过SQL语句来查询数据会更加方便。

此时，使用Apache Phoenix就可以解决我们上述问题，{% post_link 大数据_Phoenix 点击前往查看Phoenix相关知识 %}。

![Phoenix](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/20.png)

## 11 重要工作机制

### 11.1 读取数据流程

> 从zookeeper找到meta表的region的位置，然后读取meta表中的数据。而meta中又存储了用户表的region信息

```
ZK：/hbase/meta-region-server，该节点保存了meta表的region server数据
```

> 根据namespace、表名和rowkey根据meta表中的数据找到对应的region信息

```
hbase(main):014:0> scan "hbase:meta", { FILTER => "PrefixFilter('ORDER_DTL')"}
ORDER_DTL,,1599542264340.30b90c560200da7819da10dc27d8a6ca. column=info:state, timestamp=1599542721810, value=OPEN
ORDER_DTL,,1599542264340.30b90c560200da7819da10dc27d8a6ca.  column=info:regioninfo, timestamp=1599542721810, value={ENCODED => 30b90c560200da7819da10dc27d8a6ca, NAME => 'ORDER_DTL,,1599542264340.30b90c560200da7819da10dc27d8a6ca.', STARTKEY => '', ENDKEY => '\x01'}
ORDER_DTL,,1599542264340.30b90c560200da7819da10dc27d8a6ca. column=info:server, timestamp=1599542721810, value=node3.itcast.cn:16020
```

> 找到对应的regionserver，查找对应的region

![读取数据流程](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/21.png)

> 从MemStore找数据，再去BlockCache中找，如果没有，再到StoreFile上读

> 可以把MemStore理解为一级缓存，BlockCache为二级缓存，但注意scan的时候BlockCache意义不大，因为scan是顺序扫描

![读取数据流程](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/22.png)

> 第一步: client首先连接zookeeper, 获取hbase:meta表对应的region被那个regionServer所管理了

> 第二步: client连接meta表对应的regionServer, 在mata表获取要查询的表有那些region, 以及这些 region被那个regionServer所管理, 将对应regionServer列表返回客户端

> 第三步: client连接对应的regionServer, 开始进行并行的读取数据: 先到memstore --> blockCache --> storeFile --> 大Hfile

> 第四步: client接收到各个region返回来的数据后, 对数据进行排序操作, 将排序后的数据展示给用户即可

![读取数据流程](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/23.png)

### 11.2 存储数据流程

`客户端的流程`:

> 第一步: client首先连接zookeeper, 获取hbase:meta表对应的region被那个regionServer所管理了

> 第二步: client连接meta表对应的regionServer,在meta表获取要写入表有那些region, 以及根据rowkey
以及每一个region的范围确定要写入到那个region中, 将这个region对应的regionServer返回给客户端

> 第三步: client连接对应的regionServer, 开始进行写入数据:  首先将数据写regionServer的HLog中, 然后
将数据写入到对应的region的memstore中

> 第四步: 当这两个地方都写入成功后, 客户端认为数据已经写入成功, 此时客户端写入操作执行完成

`服务端内部流程`:

> 第五步: 随着上述四步不断写入, 在memstore中也会越来越多, 当memstore中数据达到一定的阈值(128M|1h)后, 就会
执行flush刷新机制, 将内存中数据刷新到HDFS中, 形成一个小的Hfile文件

> 第六步: 随着第五步不断的刷新, 在HDFS上形成多个小Hfile文件, 到小的Hfile文件达到一定的阈值(默认3个)后,
此时就会执行compact的合并机制, 将多个小Hfile合并为一个大的HFile文件

> 第七步:随着第六步不断的合并, 大的Hfile也会变的越来越大, 当大Hfile达到一定的阈值后, 就会执行split分裂
机制, 将对应region进行一分二,形成两个新的region, 此时对应大Hfile也会也进行一分二的操作, 然后让每
一个region管理一个Hfile文件, 原有就得region和旧的Hfile就会被下线和删除操作

`注意: 读写操作 与master无关, 所以master短暂宕机, 并不会影响hbase的数据读写`

`master宕机会影响关于元数据的修改操作, 以及region的分配操作`

![存储数据流程](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/24.png)

#### 11.2.1 写入MemStore

![写入MemStore](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/25.png)

> Client访问zookeeper，从ZK中找到meta表的region位置

> 读取meta表中的数据，根据namespace、表名、rowkey获取对应的Region信息

> 通过刚刚获取的地址访问对应的RegionServer，拿到对应的表存储的RegionServer

> 去表所在的RegionServer进行数据的添加

> 查找对应的region，在region中寻找列族，先向MemStore中写入数据

#### 11.2.2 MemStore溢写合并

![MemStore溢写合并](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/26.png)

##### 11.2.2.1 说明

> 当MemStore写入的值变多，触发溢写操作（flush），进行文件的溢写，成为一个StoreFile

> 当溢写的文件过多时，会触发文件的合并（Compact）操作，合并有两种方式（major，minor）

##### 11.2.2.2 触发条件

> 一旦MemStore达到128M时，则触发Flush溢出（Region级别）

```
<property>
<name>hbase.hregion.memstore.flush.size</name>
<value>134217728</value>
<source>hbase-default.xml</source>
</property>
```

> MemStore的存活时间超过1小时（默认），触发Flush溢写（RegionServer级别）

```
<property>
<name>hbase.regionserver.optionalcacheflushinterval</name>
<value>3600000</value>
<source>hbase-default.xml</source>
</property>
```

#### 11.2.3 模拟数据查看MemStore使用情况

![模拟数据查看MemStore使用情况](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/27.png)

`注意，此处小数是无法显示的，只显示整数位的MB`

> 在资料/测试程序中有一个GenWaterBill代码文件，将它导入到之前创建的Java操作HBase中，然后运行。

> 打开HBase WebUI > Table Details > 「WATER_BILL」

> 打开Region所在的Region Server

![模拟数据查看MemStore使用情况](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/28.png)

> 点击Memory查看内存占用情况

![模拟数据查看MemStore使用情况](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/29.png)

#### 11.2.4 In-memory合并

##### 11.2.4.1 In-memory compaction介绍

> In-memory合并是HBase 2.0之后添加的。它与默认的MemStore的区别：实现了在内存中进行compaction（合并）。

> 在CompactingMemStore中，数据是以段（Segment）为单位存储数据的。MemStore包含了多个segment。

> 当数据写入时，首先写入到的是Active segment中（也就是当前可以写入的segment段）

> 在2.0之前，如果MemStore中的数据量达到指定的阈值时，就会将数据flush到磁盘中的一个StoreFile

> 2.0的In-memory compaction，active segment满了后，将数据移动到pipeline中。这个过程跟以前不一样，以前是flush到磁盘，而这次是将Active segment的数据，移到称为pipeline的内存当中。一个pipeline中可以有多个segment。而In-memory compaction会将pipeline的多个segment合并为更大的、更紧凑的segment，这就是compaction

> HBase会尽量延长CompactingMemStore的生命周期，以达到减少总的IO开销。当需要把CompactingMemStore flush到磁盘时，pipeline中所有的segment会被移动到一个snapshot中，然后进行合并后写入到HFile

![In-memory compaction介绍](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/30.png)

##### 11.2.4.2 compaction策略

> 但Active segment flush到pipeline中后，后台会触发一个任务来合并pipeline中的数据。合并任务会扫描pipeline中所有的segment，将segment的索引合并为一个索引。有三种合并策略：

> basic（基础型）

1. Basic compaction策略不清理多余的数据版本，无需对cell的内存进行考核
2. basic适用于所有大量写模式

> eager（饥渴型）

1. eager compaction会过滤重复的数据，清理多余的版本，这会带来额外的开销
2. eager模式主要针对数据大量过期淘汰的场景，例如：购物车、消息队列等

> adaptive（适应型）

1. adaptive compaction根据数据的重复情况来决定是否使用eager策略
2. 该策略会找出cell个数最多的一个，然后计算一个比例，如果比例超出阈值，则使用eager策略，否则使用basic策略

##### 11.2.4.3 配置

> 1.可以通过hbase-site.xml来配置默认In Memory Compaction方式

```
<property>
<name>hbase.hregion.compacting.memstore.type</name>
<value><none|basic|eager|adaptive></value>
</property>
```

> 2.在创建表的时候指定

```
create "test_memory_compaction", {NAME => 'C1', IN_MEMORY_COMPACTION => "BASIC"}
```

#### 11.2.5 StoreFile合并

> 当MemStore超过阀值的时候，就要flush到HDFS上生成一个StoreFile。因此随着不断写入，HFile的数量将会越来越多，根据前面所述，StoreFile数量过多会降低读性能

> 为了避免对读性能的影响，需要对这些StoreFile进行compact操作，把多个HFile合并成一个HFile

> compact操作需要对HBase的数据进行多次的重新读写，因此这个过程会产生大量的IO。可以看到compact操作的本质就是以IO操作换取后续的读性能的提高

##### 11.2.5.1 minor compaction

###### 11.2.5.1.1 说明

> Minor Compaction操作只用来做部分文件的合并操作，包括minVersion=0并且设置ttl的过期版本清理，不做任何删除数据、多版本数据的清理工作

> 小范围合并，默认是3-10个文件进行合并，不会删除其他版本的数据

> Minor Compaction则只会选择数个StoreFile文件compact为一个StoreFile

> Minor Compaction的过程一般较快，而且IO相对较低

###### 11.2.5.1.2 触发条件

> 在打开Region或者MemStore时会自动检测是否需要进行Compact（包括Minor、Major）

> minFilesToCompact由hbase.hstore.compaction.min控制，默认值为3

> 即Store下面的StoreFile数量减去正在compaction的数量 >=3时，需要做compaction

> http://node1.itcast.cn:16010/conf

```
<property>
<name>hbase.hstore.compaction.min</name>
<value>3</value>
<final>false</final>
<source>hbase-default.xml</source>
</property>
```

##### 11.2.5.2 major compaction

###### 11.2.5.2.1 说明

> Major Compaction操作是对Region下的Store下的所有StoreFile执行合并操作，最终的结果是整理合并出一个文件

> 一般手动触发，会删除其他版本的数据（不同时间戳的）

###### 11.2.5.2.2 触发条件

> 如果无需进行Minor compaction，HBase会继续判断是否需要执行Major Compaction

> 如果所有的StoreFile中，最老（时间戳最小）的那个StoreFile的时间间隔大于Major Compaction的时间间隔（hbase.hregion.majorcompaction——默认7天）

```
<property>
<name>hbase.hregion.majorcompaction</name>
<value>604800000</value>
<source>hbase-default.xml</source>
</property>
```

604800000毫秒 = 604800秒 = 168小时 = 7天
 
### 11.3 Region管理

#### 11.3.1 region分配

> 任何时刻，`一个region只能分配给一个region server`

> Master记录了当前有哪些可用的region server，以及当前哪些region分配给了哪些region server，哪些region还没有分配。当需要分配的新的region，并且有一个region server上有可用空间时，master就给这个region server发送一个装载请求，把region分配给这个region server。region server得到请求后，就开始对此region提供服务。

#### 11.3.2 region server上线

> Master使用ZooKeeper来跟踪region server状态

> 当某个region server启动时

1. 首先在zookeeper上的server目录下建立代表自己的znode
2. 由于Master订阅了server目录上的变更消息，当server目录下的文件出现新增或删除操作时，master可以得到来自zookeeper的实时通知
3. 一旦region server上线，master能马上得到消息。

#### 11.3.3 region server下线

> 当region server下线时，它和zookeeper的会话断开，ZooKeeper而自动释放代表这台server的文件上的独占锁

> Master就可以确定

1. region server和zookeeper之间的网络断开了
2. region server挂了

> 无论哪种情况，region server都无法继续为它的region提供服务了，此时master会删除server目录下代表这台region server的znode数据，并将这台region server的region分配给其它还活着的节点

#### 11.3.4 Region分裂

> 当region中的数据逐渐变大之后，达到某一个阈值，会进行裂变

1. 一个region等分为两个region，并分配到不同的RegionServer
2. 原本的Region会下线，新Split出来的两个Region会被HMaster分配到相应的HRegionServer上，使得原先1个Region的压力得以分流到2个Region上。

```
<-- Region最大文件大小为10G -->
<property>
<name>hbase.hregion.max.filesize</name>
<value>10737418240</value>
<final>false</final>
<source>hbase-default.xml</source>
</property>
```

> HBase只是增加数据，所有的更新和删除操作，都是在Compact阶段做的

> 用户写操作只需要进入到内存即可立即返回，从而保证I/O高性能读写

##### 11.3.4.1 自动分区

> 之前，我们在建表的时候，没有涉及过任何关于Region的设置，由HBase来自动进行分区。也就是Region达到一定大小就会自动进行分区。最小的分裂大小和table的某个region server的region 个数有关，当store file的大小大于如下公式得出的值的时候就会split，公式如下:

```
Min (R^2 * “hbase.hregion.memstore.flush.size”, “hbase.hregion.max.filesize”) R为同一个table中在同一个region server中region的个数。
```

> 如果初始时R=1,那么Min(128MB,10GB)=128MB,也就是说在第一个flush的时候就会触发分裂操作

> 当R=2的时候Min(22128MB,10GB)=512MB ,当某个store file大小达到512MB的时候，就会触发分裂

> 如此类推，当R=9的时候，store file 达到10GB的时候就会分裂，也就是说当R>=9的时候，store file 达到10GB的时候就会分裂

> split 点都位于region中row key的中间点

##### 11.3.4.2 手动分区

> 在创建表的时候，就可以指定表分为多少个Region。默认一开始的时候系统会只向一个RegionServer写数据，系统不指定startRow和endRow，可以在运行的时候提前Split，提高并发写入。

### 11.4 Master工作机制

#### 11.4.1 Master上线

> Master启动进行以下步骤:

> 1.从zookeeper上`获取唯一一个代表active master的锁`，用来阻止其它master成为master

> 2.一般hbase集群中总是有一个master在提供服务，还有一个以上的‘master’在等待时机抢占它的位置。

> 3.扫描zookeeper上的server父节点，获得当前可用的region server列表

> 4.和每个region server通信，获得当前已分配的region和region server的对应关系

> 5.扫描.META.region的集合，计算得到当前还未分配的region，将他们放入待分配region列表

#### 11.4.2 Master下线

> 由于`master只维护表和region的元数据`，而不参与表数据IO的过程，master下线仅导致所有元数据的修改被冻结

```
无法创建删除表
无法修改表的schema
无法进行region的负载均衡
无法处理region 上下线
无法进行region的合并
唯一例外的是region的split可以正常进行，因为只有region server参与
表的数据读写还可以正常进行
```

> 因此`master下线短时间内对整个hbase集群没有影响`。

> 从上线过程可以看到，master保存的信息全是可以冗余信息（都可以从系统其它地方收集到或者计算出来）

## 12 Hbase批量装载-Bulk load

### 12.1 简介

> 很多时候，我们需要将外部的数据导入到HBase集群中，例如：将一些历史的数据导入到HBase做备份。我们之前已经学习了HBase的Java API，通过put方式可以将数据写入到HBase中，我们也学习过通过MapReduce编写代码将HDFS中的数据导入到HBase。但这些方式都是基于HBase的原生API方式进行操作的。这些方式有一个共同点，就是需要与HBase连接，然后进行操作。HBase服务器要维护、管理这些连接，以及接受来自客户端的操作，会给HBase的存储、计算、网络资源造成较大消耗。此时，在需要将海量数据写入到HBase时，通过Bulk load（大容量加载）的方式，会变得更高效。可以这么说，进行大量数据操作，Bulk load是必不可少的。

> 我们知道，HBase的数据最终是需要持久化到HDFS。HDFS是一个文件系统，那么数据可定是以一定的格式存储到里面的。例如：Hive我们可以以ORC、Parquet等方式存储。而HBase也有自己的数据格式，那就是HFile。Bulk Load就是直接将数据写入到StoreFile（HFile）中，从而绕开与HBase的交互，HFile生成后，直接一次性建立与HBase的关联即可。使用BulkLoad，绕过了Write to WAL，Write to MemStore及Flush to disk的过程

> 更多可以参考官方对Bulk load的描述：https://hbase.apache.org/book.html#arch.bulk.load

### 12.2 Bulk load MapReduce程序开发

> Bulk load的流程主要分为两步：

1. 通过MapReduce准备好数据文件（Store Files）
2. 加载数据文件到HBase

### 12.3 银行转账记录海量冷数据存储案例

> 银行每天都产生大量的转账记录，超过一定时期的数据，需要定期进行备份存储。本案例，在MySQL中有大量转账记录数据，需要将这些数据保存到HBase中。因为数据量非常庞大，所以采用的是Bulk Load方式来加载数据。

> 项目组为了方便数据备份，每天都会将对应的转账记录导出为CSV文本文件，并上传到HDFS。我们需要做的就将HDFS上的文件导入到HBase中。

> 因为我们只需要将数据读取出来，然后生成对应的Store File文件。所以，我们编写的MapReduce程序，只有Mapper，而没有Reducer。

#### 12.3.1 上传数据

> 将资料中的数据集 bank_record.csv 上传到HDFS的 /bank/input 目录。该文件中包含50W条的转账记录数据。

```
hadoop fs -mkdir -p /bank/input
hadoop fs -put bank_record.csv /bank/input
```

> 然后，执行MapReduce程序。

### 12.3.2 编写mapperReduce程序

#### 12.3.2.1 pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>bigdata42_parent</artifactId>
        <groupId>org.itheima</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>day04_bankrecord_bulkload</artifactId>

    <repositories><!-- 代码库 -->
        <repository>
            <id>aliyun</id>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
                <updatePolicy>never</updatePolicy>
            </snapshots>
        </repository>
    </repositories>

    <dependencies>

        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>2.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-mapreduce</artifactId>
            <version>2.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-jobclient</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-core</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-auth</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <target>1.8</target>
                    <source>1.8</source>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

#### 12.3.2.2 entity

```
package cn.itcast.bank_record.entity;

public class TransferRecord {

    private String id;
    private String code;
    private String rec_account;
    private String rec_bank_name;
    private String rec_name;
    private String pay_account;
    private String pay_name;
    private String pay_comments;
    private String pay_channel;
    private String pay_way;
    private String status;
    private String timestamp;
    private String money;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public String getRec_account() {
        return rec_account;
    }

    public void setRec_account(String rec_account) {
        this.rec_account = rec_account;
    }

    public String getRec_bank_name() {
        return rec_bank_name;
    }

    public void setRec_bank_name(String rec_bank_name) {
        this.rec_bank_name = rec_bank_name;
    }

    public String getRec_name() {
        return rec_name;
    }

    public void setRec_name(String rec_name) {
        this.rec_name = rec_name;
    }

    public String getPay_account() {
        return pay_account;
    }

    public void setPay_account(String pay_account) {
        this.pay_account = pay_account;
    }

    public String getPay_name() {
        return pay_name;
    }

    public void setPay_name(String pay_name) {
        this.pay_name = pay_name;
    }

    public String getPay_comments() {
        return pay_comments;
    }

    public void setPay_comments(String pay_comments) {
        this.pay_comments = pay_comments;
    }

    public String getPay_channel() {
        return pay_channel;
    }

    public void setPay_channel(String pay_channel) {
        this.pay_channel = pay_channel;
    }

    public String getPay_way() {
        return pay_way;
    }

    public void setPay_way(String pay_way) {
        this.pay_way = pay_way;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

    public String getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(String timestamp) {
        this.timestamp = timestamp;
    }

    public String getMoney() {
        return money;
    }

    public void setMoney(String money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "TransferRecord{" +
                "id='" + id + '\'' +
                ", code='" + code + '\'' +
                ", rec_account='" + rec_account + '\'' +
                ", rec_bank_name='" + rec_bank_name + '\'' +
                ", rec_name='" + rec_name + '\'' +
                ", pay_account='" + pay_account + '\'' +
                ", pay_name='" + pay_name + '\'' +
                ", pay_comments='" + pay_comments + '\'' +
                ", pay_channel='" + pay_channel + '\'' +
                ", pay_way='" + pay_way + '\'' +
                ", status='" + status + '\'' +
                ", timestamp='" + timestamp + '\'' +
                ", money='" + money + '\'' +
                '}';
    }

    public static TransferRecord parse(String line) {
        TransferRecord transferRecord = new TransferRecord();
        String[] fields = line.split(",");

        transferRecord.setId(fields[0]);
        transferRecord.setCode(fields[1]);
        transferRecord.setRec_account(fields[2]);
        transferRecord.setRec_bank_name(fields[3]);
        transferRecord.setRec_name(fields[4]);
        transferRecord.setPay_account(fields[5]);
        transferRecord.setPay_name(fields[6]);
        transferRecord.setPay_comments(fields[7]);
        transferRecord.setPay_channel(fields[8]);
        transferRecord.setPay_way(fields[9]);
        transferRecord.setStatus(fields[10]);
        transferRecord.setTimestamp(fields[11]);
        transferRecord.setMoney(fields[12]);

        return transferRecord;
    }

    public static void main(String[] args) {
        String str = "7e59c946-b1c6-4b04-a60a-f69c7a9ef0d6,SU8sXYiQgJi8,6225681772493291,杭州银行,丁杰,4896117668090896,卑文彬,老婆，节日快乐,电脑客户端,电子银行转账,转账完成,2020-5-13 21:06:92,11659.0";
        TransferRecord tr = parse(str);

        System.out.println(tr);
    }
}

```

#### 12.3.2.3 mapper

```
package cn.itcast.bank_record.bulkload.mr;

import cn.itcast.bank_record.entity.TransferRecord;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class BulkLoadMapperTask extends Mapper<LongWritable,Text,ImmutableBytesWritable,Put> {
    private   ImmutableBytesWritable k2 = new ImmutableBytesWritable();
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        //1. 读取一行数据
        String line = value.toString();

        if(line != null && !"".equals(line)) {
            //2. 封装数据
            TransferRecord transferRecord = TransferRecord.parse(line);

            //3. 封装 k2 和 v2
            k2.set(transferRecord.getId().getBytes()); // 封装 rowkey

            Put put = new Put(transferRecord.getId().getBytes());

            put.addColumn("C1".getBytes(),"code".getBytes(),transferRecord.getCode().getBytes());
            put.addColumn("C1".getBytes(),"rec_account".getBytes(),transferRecord.getRec_account().getBytes());

            put.addColumn("C1".getBytes(),"rec_bank_name".getBytes(),transferRecord.getRec_bank_name().getBytes());
            put.addColumn("C1".getBytes(),"rec_name".getBytes(),transferRecord.getRec_name().getBytes());

            put.addColumn("C1".getBytes(),"pay_account".getBytes(),transferRecord.getPay_account().getBytes());
            put.addColumn("C1".getBytes(),"pay_name".getBytes(),transferRecord.getPay_name().getBytes());

            put.addColumn("C1".getBytes(),"pay_comments".getBytes(),transferRecord.getPay_comments().getBytes());
            put.addColumn("C1".getBytes(),"pay_channel".getBytes(),transferRecord.getPay_channel().getBytes());

            put.addColumn("C1".getBytes(),"pay_way".getBytes(),transferRecord.getPay_way().getBytes());
            put.addColumn("C1".getBytes(),"status".getBytes(),transferRecord.getStatus().getBytes());

            put.addColumn("C1".getBytes(),"timestamp".getBytes(),transferRecord.getTimestamp().getBytes());
            put.addColumn("C1".getBytes(),"money".getBytes(),transferRecord.getMoney().getBytes());

            //4. 写出去
            context.write(k2,put);

        }
    }
}

```
#### 12.3.3.4 job

```
package cn.itcast.bank_record.bulkload.mr;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Table;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.hbase.mapreduce.HFileOutputFormat2;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;

public class BulkLoadJobMain {

    public static void main(String[] args) throws Exception {

        //1. 创建job对象
        Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum","node1.itcast.cn:2181,node2.itcast.cn:2181,node3.itcast.cn:2181");
        Job job = Job.getInstance(conf, "BulkLoadJobMain");

        //2. 上传到集群中运行的必备配置
        job.setJarByClass(BulkLoadJobMain.class);

        //3. 封装 job任务的详细配置:  天龙八大步骤
        //3.1: 封装输入类和输入路径
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job,new Path("hdfs://node1:8020/bank/input"));

        //3.2: 封装 mapper类 和 mapper的输出 k2 和 v2的类型
        job.setMapperClass(BulkLoadMapperTask.class);

        job.setMapOutputKeyClass(ImmutableBytesWritable.class);
        job.setMapOutputValueClass(Put.class);

        //3.3: 封装shuffle: 分区 排序 规约 分组

        //3.7: 封装reduceTask类 和 输出 k3 和 v3  默认有一个原样输出的reduce
        job.setOutputKeyClass(ImmutableBytesWritable.class);
        job.setOutputValueClass(Put.class);


        //3.8: 封装输出类 和 输出路径
        job.setOutputFormatClass(HFileOutputFormat2.class);

        Connection connection = ConnectionFactory.createConnection(conf);
        Table table = connection.getTable(TableName.valueOf("ITCAST_BANK:TRANSFER_RECORD"));

        HFileOutputFormat2.configureIncrementalLoad(job,table,connection.getRegionLocator(TableName.valueOf("ITCAST_BANK:TRANSFER_RECORD")));

        HFileOutputFormat2.setOutputPath(job,new Path("hdfs://node1:8020/bank/output"));

        //4. 提交任务
        boolean flag = job.waitForCompletion(true);

        //5. 退出程序
        System.exit(flag ?0:1 );
    }
}

```

### 12.3.3 加载数据文件到HBase

```
hbase org.apache.hadoop.hbase.tool.LoadIncrementalHFiles /bank/output ITCAST_BANK:TRANSFER_RECORD
```

## 13 Hbase协处理器(Coprocessor)

### 13.1 大麦

> http://hbase.apache.org/book.html#cp

### 13.1 起源

> Hbase 作为列族数据库最经常被人诟病的特性包括：

1. 无法轻易建立“二级索引”
2. 难以执 行求和、计数、排序等操作

比如，在旧版本的(<0.92)Hbase 中，统计数据表的总行数，需要使用 Counter 方法，执行一次 MapReduce Job 才能得到。虽然 HBase 在数据存储层中集成了 MapReduce，能够有效用于数据表的分布式计算。然而在很多情况下，做一些简单的相加或者聚合计算的时候， 如果直接将计算过程放置在 server 端，能够减少通讯开销，从而获 得很好的性能提升

> 于是， HBase 在 0.92 之后引入了协处理器(coprocessors)，实现一些激动人心的新特性：能够轻易建立二次索引、复杂过滤器(谓词下推)以及访问控制等。

### 13.2 两种协处理器：observer和endpoint

#### 13.2.1 observer协处理器

> Observer 类似于传统数据库中的触发器，当发生某些事件的时候这类协处理器会被 Server 端调用。Observer Coprocessor 就是一些散布在 HBase Server 端代码中的 hook 钩子， 在固定的事件发生时被调用。比如： put 操作之前有钩子函数 prePut，该函数在 put 操作
执行前会被 Region Server 调用；在 put 操作之后则有 postPut 钩子函数

> 以 Hbase2.0.0 版本为例，它提供了三种观察者接口：

1. RegionObserver：提供客户端的数据操纵事件钩子： Get、 Put、 Delete、 Scan 等
2. WALObserver：提供 WAL 相关操作钩子。
3. MasterObserver：提供 DDL-类型的操作钩子。如创建、删除、修改数据表等。
4. 到 0.96 版本又新增一个 RegionServerObserver

> 下图是以 RegionObserver 为例子讲解 Observer 这种协处理器的原理：

![observer协处理器](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/31.png)

> 1.客户端发起get请求

> 2.该请求被分派给合适的RegionServer和Region

> 3.coprocessorHost拦截该请求，然后在该表上登记的每个RegionObserer上调用preGet()

> 4.如果没有被preGet拦截，该请求继续送到Region，然后进行处理

> 5.Region产生的结果再次被coprocessorHost拦截，调用posGet()处理

> 6.加入没有postGet()拦截该响应，最终结果被返回给客户端

#### 13.2.2 endpoint协处理器

> Endpoint 协处理器类似传统数据库中的存储过程，客户端可以调用这些 Endpoint 协处理器执行一段 Server 端代码，并将 Server 端代码的结果返回给客户端进一步处理，最常见的用法就是进行聚集操作

> 如果没有协处理器，当用户需要找出一张表中的最大数据，即max 聚合操作，就必须进行全表扫描，在客户端代码内遍历扫描结果，并执行求最大值的操作。这样的方法无法利用底层集群的并发能力，而将所有计算都集中到 Client 端统一执 行，势必效率低下。

> 利用 Coprocessor，用户可以将求最大值的代码部署到 HBase Server 端，HBase 将利用底层 cluster 的多个节点并发执行求最大值的操作。即在每个 Region 范围内 执行求最大值的代码，将每个 Region 的最大值在 Region Server 端计算出，仅仅将该 max 值返回给客户端。在客户端进一步将多个 Region 的最大值进一步处理而找到其中的最大值。这样整体的执行效率就会提高很多

> 下图是 EndPoint 的工作原理：

![endpoint协处理器](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/32.png)

#### 13.2.3 总结

> Observer 允许集群在正常的客户端操作过程中可以有不同的行为表现

> Endpoint 允许扩展集群的能力，对客户端应用开放新的运算命令

> observer 类似于 RDBMS 中的触发器，主要在服务端工作

> endpoint 类似于 RDBMS 中的存储过程，主要在 服务器端、client 端工作

> observer 可以实现权限管理、优先级设置、监控、 ddl 控制、 二级索引等功能

> endpoint 可以实现 min、 max、 avg、 sum、 distinct、 group by 等功能

### 13.3 协处理器加载方式

> 协处理器的加载方式有两种：

1. 静态加载方式（ Static Load）
2. 动态加载方式 （ Dynamic Load）

> 静态加载的协处理器称之为 System Coprocessor，动态加载的协处理器称 之为 Table Coprocessor。

#### 13.3.1 静态加载

> 通过修改 hbase-site.xml 这个文件来实现
> 启动全局 aggregation，能过操纵所有的表上的数据。只需要添加如下代码：

```
<property>
<name>hbase.coprocessor.user.region.classes</name>
<value>org.apache.hadoop.hbase.coprocessor.AggregateImplementation</value>
</property>
```

为所有 table 加载了一个 cp class，可以用” ,”分割加载多个 class

#### 13.3.2 动态加载

> 启用表 aggregation，只对特定的表生效

> 通过 HBase Shell 来实现，disable 指定表

```
hbase> disable 'mytable'
```

> 添加 aggregation

```
hbase> alter 'mytable', METHOD => 'table_att','coprocessor'=>
'|org.apache.Hadoop.hbase.coprocessor.AggregateImplementation||'
```

> 重启启用表

```
hbase> enable 'mytable'
```

#### 13.3.3 协处理器卸载

> 只需三步：

```
disable ‘test’
alter ‘test’, METHOD => ‘table_att_unset’, NAME => ‘coprocessor$1’
enable ‘test’
```

## 14 Hbase事务

> HBase 支持特定场景下的 ACID，即当对同一行进行 Put 操作时保证完全的 ACID。可以简单理解为针对一行的操作，是有事务性保障的。HBase也没有混合读写事务。也就是说，我们无法将读操作、写操作放入到一个事务中。

## 15 Hbase数据结构

> 在讲解HBase的LSM合并树之前，我们需要来了解一些常用的数据结构知识。

### 15.1 跳表

![跳表](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/33.png)

> 上图是一个有序链表，我们要检索一个数据就挨个遍历。如果想要再提升查询效率，可以变种为以下结构：

![跳表](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/34.png)

> 现在，我们要查询11，可以跳着来查询，从而加快查询速度。

### 15.2 常见树结构

#### 15.2.1 二叉搜索树（Binary Search Tree）

##### 15.2.1.1 什么是二叉搜索树？

二叉搜索树也叫二叉查找树。它是一种比较特殊的二叉树。

![二叉搜索树](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/35.png)

##### 15.2.1.2 树的高度、深度、层数

> 深度：节点的深度是根节点到这个节点所经历的边的个数，深度是从上往下数的

> 高度：节点的高度是该节点到叶子节点的最长路径（边数），高度是从下往上数的

> 层数：根节点为第一层，往下依次递增

> 上图：节点12的深度为0，高度为4，在第1层 ；节点15的深度为2，高度为2，在第3层

##### 15.2.1.3 二叉搜索树的特点

> 树中的每个节点，它的左子树中所有关键字值小于该节点关键字值，右子树中所有关键字值大于该节点关键字值

##### 15.2.1.4 二叉搜索树的查询方式

> 首先和根节点进行比较，如果等于根节点，则返回

> 如果小于根节点，则在根节点的左子树进行查找

> 如果大于根节点，则在根节点的右子树进行查找

##### 15.2.1.5 二叉搜索树的缺点

> 因为二叉搜索树是一种二叉树，每个节点只能有两个子节点，但有较多节点时，整棵树的高度会比较大，树的高度越大，搜索的性能开销也就越大

#### 15.2.2 平衡二叉树（Balance Binary Tree）

#### 15.2.2.1 简介

> 平衡二叉树也称为AVL数

> 它是一颗空数，或者它的任意节点左右两个子树的高度差绝对值不超过1

> 平衡二叉树很好地解决了二叉查找树退化成链表的问题

![平衡二叉树](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/36.png)

上图：

1. 两棵树都是二叉查找树
2. 左边的不是平衡二叉树：节点6的子节点：节点2的高度为：2，节点7的高度为：0，| 2 – 0 | = 2 > 1）
3. 右边的是平衡二叉树：节点6的子节点：节点3的高度为：1，节点7的高度为：0，| 1 – 0 | = 1 = 1 ）

#### 15.2.2.2 平衡二叉树的特点

> AVL树是高度平衡的（严格平衡），频繁的插入和删除，会引起频繁的rebalance，导致效率下降，它比较使用与插入/删除较少，查找较多的场景

#### 15.2.3 红黑树

#### 15.2.3.1 简介

> 红黑树是一种含有红黑节点并能自平衡的二叉搜索树，它满足以下性质：

- 每个节点要么是黑色，要么是红色
- 根节点是黑色 
- 每个叶子节点（NIL）是黑色
- 每个红色结点的两个子结点一定都是黑色
- 任意一结点到每个叶子结点的路径都包含数量相同的黑结点,

![红黑树](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/37.png)

#### 15.2.3.2 红黑树的特点

> 和AVL树不一样，红黑树是一种弱平衡的二叉树，它的插入/删除效率更高，所以对于插入、删除较多的情况下，就用红黑树，而且查找效率也不低。例如：Java中的TreeMap就是基于红黑树实现的。

#### 15.2.4 B树

#### 15.2.4.1 什么是B树

> B树是一种平衡多路搜索树

> 与二叉搜索树不同的是，B树的节点可以有多个子节点，不限于最多两个节点

> 它的子节点可以是几个或者是几千个

![B树](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/38.png)

#### 15.2.4.2 B树的特点

> 所有节点关键字是按递增次序排列，并遵循左小右大原则

> B-树有个最大的特点是有多个查找路径，而不像二叉搜索树，只有两路查找路径。

> 所有的叶子节点在同一层

> `逐层查找，找到节点后返回`

#### 15.2.4.3 B-树的查找方式

> 从根节点的关键字开始比较，例如：上图为13，判断大于还是小于

> 继续往下查找，因为节点可能会有多个节点，所以需要判断属于哪个区间

> 不断往下查找，直到找到为止或者没有找到返回Null

#### 15.2.5 B+树结构

##### 15.2.5.1 B+树简介

> B+树是B树的升级版。B+树常用在`文件系统和数据库`中，B+树通过对每个节点存储数据的个数进行扩展，`可以让连续的数据进行快速访问，有效减少查询时间`，减少IO操作。它能够保持数据稳定有序，其插入与修改拥有较稳定的对数时间复杂度
例如：Linux的Ext3文件系统、Oracle、MySQL、SQLServer都会使用到B+树。

![B+树](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/39.png)

- B+ 树是一种树数据结构，是一个n叉树
- 每个节点通常有多个孩子
- 一颗B+树包含根节点、内部节点和叶子节点
- `只有叶子节点包含数据（所有数据都是在叶子节点中出现）`

##### 15.2.5.2 B+树的特点

> 所有关键字都出现在叶子结点的链表中（稠密索引），且链表中的关键字恰好是有序的
如果执行的是：select * from user order by id，要全表扫描数据，那么B树就比较费劲了，但B+树就容易了，只要遍历最后的链表就可以了。

> 只会在叶子节点上搜索到数据

> 非叶子结点相当于是叶子结点的索引（稀疏索引），叶子结点相当于是存储

> 数据库的B+树高度大概在 2-4 层，也就是说查询到某个数据最多需要2到4次IO，相当于0.02到0.04s

##### 15.2.5.3 稠密索引和稀疏索引

> 稠密索引文件中的每个搜索码值都对应一个索引值

> 稀疏索引文件只为索引码的某些值建立索引项

> 稠密索引：

![稠密索引](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/40.png)

> 稀疏索引：

![稀疏索引](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/41.png)

### 15.3 LSM树数据结构

#### 15.3.1 简介

> 传统关系型数据库，一般都选择使用B+树作为索引结构，而在大数据场景下，HBase、Kudu这些存储引擎选择的是LSM树。LSM树，即日志结构合并树(Log-Structured Merge-Tree)。

- LSM树主要目标是快速建立索引
- B+树是建立索引的通用技术，但如果并发写入压力较大时，B+树需要大量的磁盘随机IO，而严重影响索引创建的速度，在一些写入操作非常频繁的应用场景中，就不太适合了
- LSM树通过磁盘的顺序写，来实现最好的写性能

#### 15.3.2 LSM树设计思想

![LSM树设计思想](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/42.png)

> LSM	的主要思想是划分不同等级的结构，换句话来理解，就是LSM中不止一个数据结构，而是存在多种结构

> 一个结构在内存、其他结构在磁盘（HBase存储结构中，有内存——MemStore、也有磁盘——StoreFile）

> 内存的结构可以是B树、红黑树、跳表等结构（HBase中是跳表），磁盘中的树就是一颗B+树

> C0层保存了最近写入的数据，数据都是有序的，而且可以随机更新、随机查询

> C1到CK层的数据都是存在磁盘中，每一层中key都是有序存储的

#### 15.3.3 LSM的数据写入操作

> 首先将数据写入到WAL（Write Ahead log），写日志是顺序写，效率相对较高（PUT、DELETE都是顺序写）

> 数据项写入到内存中的C0结构中

> 只有内存中的C0结构超过一定阈值的时候，将内存中的C0、和C1进行合并。这个过程就是Compaction（合并）

> 合并后的新的C1顺序写磁盘，替换之前的C1

> 但C1层达到一定的大小，会继续和下层合并，合并后旧的文件都可以删除，只保留最新的

> 整个写入的过程只用到了内存结构，C ompaction由后台异步完成，不阻塞写入

#### 15.3.4 LSM的数据查询操作

> 先在内存中查C0层

> 如果C0层中不存在数据，则查询C1层

> 不断逐层查询，最早的数据在CK层

> C0层因为是在内存中的结构中查询，所以效率较高。因为数据都是分布在不同的层结构中，所以一次查询，可能需要多次跨层次结构查询，所以读取的速度会慢一些。

> 根据以上，LSM树结构的程序适合于写密集、少量查询(scan)的场景

### 15.4 布隆过滤器

#### 15.4.1 简介

> 客户端：这个key存在吗？<br/>
服务器：不存在/不知道

> 本质上，布隆过滤器是一种数据结构，是一种比较巧妙的概率型数据结构。它的特点是高效地插入和查询。但我们要检查一个key是否在某个结构中存在时，通过使用布隆过滤器，我们可以快速了解到「这个key一定不存在或者可能存在」。相比于以前学习过的List、Set、Map这些数据结构，它更加高效、占用的空间也越少，但是它返回的结果是概率性的，是不确切的。

#### 15.4.2 应用场景

> 判断某个数据是否在海量数据中存在

> HBase中存储着非常海量数据，要判断某个ROWKEYS、或者某个列是否存在，使用布隆过滤器，可以快速获取某个数据是否存在。但有一定的误判率。但如果某个key不存在，一定是准确的。

#### 15.4.3 理解布隆过滤器

![理解布隆过滤器](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/43.png)

> 布隆过滤器是一个bit数组或者称为一个bit二进制向量

> 这个数组中的元素存的要么是0、要么是1

> k个hash函数都是彼此独立的，并将每个hash函数计算后的结果对数组的长度m取模，并将对一个的bit设置为1（蓝色单元格）

> 我们将每个key都按照这种方式设置单元格，就是「布隆过滤器」

#### 15.4.4 根据布隆过滤器查询元素

> 假设输入一个key，我们使用之前的k个hash函数求哈希，得到k个值

> 判断这k个值是否都为蓝色，如果有一个不是蓝色，那么这个key一定不存在

> 如果都有蓝色，那么key是可能存在（布隆过滤器会存在误判）

> 因为如果输入对象很多，而集合比较小的情况，会导致集合中大多位置都会被描蓝，那么检查某个key时候为蓝色时，刚好某个位置正好被设置为蓝色了，此时，会错误认为该key在集合中

### 15.5 StoreFiles(HFile) 结构

> StoreFile是HBase存储数据的文件格式。

#### 15.5.1 HFile的逻辑结构

##### 15.5.1.1 HFile逻辑结构图

![HFile逻辑结构图](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/44.png)

##### 15.5.1.2 逻辑结构说明

4大部分
> Scanned block section

- 扫描StoreFile时，所有的Data Block（数据块）都将会被读取
- Leaf Index（LSM + C1树索引）、Bloom block（布隆过滤器）都会被读取

> Non-scanned block section

- 扫描StoreFile时，不会被读取
- 包含MetaBlock和Intermediate Level Data Index Blocks

> Opening-time data section

- 在RegionServer启动时，需要将数据加载到内存中，包括数据块索引、元数据索引、布隆过滤器、文件信息。

> Trailer

- 记录了HFile的基本信息
- 各个部分的偏移值和寻址信息

#### 15.5.2 StoreFile物理结构

> StoreFile是以Hfile的形式存储在HDFS上的。Hfile的格式为下图：

![StoreFile物理结构](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/45.png)

> HFile文件是不定长的，长度固定的只有其中的两块：Trailer和FileInfo。正如图中所示的，Trailer中有指针指向其他数 据块的起始点。

> File Info中记录了文件的一些Meta信息，例如：AVG_KEY_LEN, AVG_VALUE_LEN, LAST_KEY, COMPARATOR, MAX_SEQ_ID_KEY等

> Data Index和Meta Index块记录了每个Data块和Meta块的起始点。

> Data Block是HBase I/O的基本单元，为了提高效率，HRegionServer中有基于LRU的Block Cache机制。每个Data块的大小可以在创建一个Table的时候通过参数指定，大号的Block有利于顺序Scan，小号Block利于随机查询。 每个Data块除了开头的Magic以外就是一个个KeyValue对拼接而成, Magic内容就是一些随机数字，目的是防止数据损坏。

> HFile里面的每个KeyValue对就是一个简单的byte数组。但是这个byte数组里面包含了很多项，并且有固定的结构。我们来看看里面的具体结构：

![StoreFile物理结构](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/46.png)

> 1.开始是两个固定长度的数值，分别表示Key的长度和Value的长度<br/>
2.紧接着是Key，开始是固定长度的数值，表示RowKey的长度<br/>
3.紧接着是 RowKey，然后是固定长度的数值，表示Family的长度<br/>
4.然后是Family，接着是Qualifier<br/>
5.然后是两个固定长度的数值，表示Time Stamp和Key Type（Put/Delete）——每一种操作都会生成一个Key-Value。Value部分没有这么复杂的结构，就是纯粹的二进制数据了。

![StoreFile物理结构](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/47.png)

> `Data Block段`：保存表中的数据，这部分可以被压缩<br/>
`Meta Block段 (可选的)`：保存用户自定义的kv对，可以被压缩。<br/>
`File Info段`：Hfile的元信息，不被压缩，用户也可以在这一部分添加自己的元信息。<br/>
`Data Block Index段`：Data Block的索引。每条索引的key是被索引的block的第一条记录的key。<br/>
`Meta Block Index段 (可选的)`：Meta Block的索引。<br/>
`Trailer`：这一段是定长的。保存了每一段的偏移量，读取一个HFile时，会首先 读取Trailer，Trailer保存了每个段的起始位置(段的Magic Number用来做安全check)，然后，DataBlock Index会被读取到内存中，这样，当检索某个key时，不需要扫描整个HFile，而只需从内存中找到key所在的block，通过一次磁盘io将整个 block读取到内存中，再找到需要的key。DataBlock Index采用LRU机制淘汰

## 16 Hbase调优

### 16.1 通用优化

#### 16.1.1 NameNode的元数据备份使用SSD

![NameNode的元数据备份使用SSD](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/48.png)

#### 16.1.2 定时备份NameNode上的元数据

> 每小时或者每天备份，如果数据极其重要，可以5~10分钟备份一次。备份可以通过定时任务复制元数据目录即可。

#### 16.1.3 为NameNode指定多个元数据目录

> 使用dfs.name.dir或者dfs.namenode.name.dir指定。一个指定本地磁盘，一个指定网络磁盘。这样可以提供元数据的冗余和健壮性，以免发生故障。

> 设置dfs.namenode.name.dir.restore为true，允许尝试恢复之前失败的dfs.namenode.name.dir目录，在创建checkpoint时做此尝试，如果设置了多个磁盘，建议允许。

#### 16.1.4 NameNode节点配置为RAID1（镜像盘）结构

![NameNode节点配置为RAID1（镜像盘）结构](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/49.png)

#### 16.1.5 补充：什么是Raid0、Raid0+1、Raid1、Raid5

![Raid0、Raid0+1、Raid1、Raid5](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HBase/50.png)

> `Standalone`：最普遍的单磁盘储存方式。

> `Cluster`：集群储存是通过将数据分布到集群中各节点的存储方式,提供单一的使用接口与界面,使用户可以方便地对所有数据进行统一使用与管理。

> `Hot swap`：用户可以再不关闭系统,不切断电源的情况下取出和更换硬盘,提高系统的恢复能力、拓展性和灵活性。

> `Raid0`：Raid0是所有raid中存储性能最强的阵列形式。其工作原理就是在多个磁盘上分散存取连续的数据,这样,当需要存取数据是多个磁盘可以并排执行,每个磁盘执行属于它自己的那部分数据请求,显著提高磁盘整体存取性能。但是不具备容错能力,适用于低成本、低可靠性的台式系统。

> `Raid1`：又称镜像盘,把一个磁盘的数据镜像到另一个磁盘上,采用镜像容错来提高可靠性,具有raid中最高的数据冗余能力。存数据时会将数据同时写入镜像盘内,读取数据则只从工作盘读出。发生故障时,系统将从镜像盘读取数据,然后再恢复工作盘正确数据。这种阵列方式可靠性极高,但是其容量会减去一半。广泛用于数据要求极严的应用场合,如商业金融、档案管理等领域。只允许一颗硬盘出故障。

> `Raid0+1`：将Raid0和Raid1技术结合在一起,兼顾两者的优势。在数据得到保障的同时,还能提供较强的存储性能。不过至少要求4个或以上的硬盘，但也只允许一个磁盘出错。是一种三高技术。

> `Raid5`：Raid5可以看成是Raid0+1的低成本方案。采用循环偶校验独立存取的阵列方式。将数据和相对应的奇偶校验信息分布存储到组成RAID5的各个磁盘上。当其中一个磁盘数据发生损坏后,利用剩下的磁盘和相应的奇偶校验信息 重新恢复/生成丢失的数据而不影响数据的可用性。至少需要3个或以上的硬盘。适用于大数据量的操作。成本稍高、储存性强、可靠性强的阵列方式。
RAID还有其他方式，请自行查阅。

#### 16.1.6 保持NameNode日志目录有足够的空间，有助于帮助发现问题

#### 16.1.7 Hadoop是IO密集型框架，所以尽量提升存储的速度和吞吐量

### 16.2 Linux优化

#### 16.2.1 开启文件系统的预读缓存可以提高读取速度

```
$ sudo blockdev --setra 32768 /dev/sda
```
> （尖叫提示：ra是readahead的缩写）

#### 16.2.2 最大限度使用物理内存

```
$ sudo sysctl -w vm.swappiness=0
```

> swappiness，Linux内核参数，控制换出运行时内存的相对权重

> swappiness参数值可设置范围在0到100之间，低参数值会让内核尽量少用交换，更高参数值会使内核更多的去使用交换空间

> 默认值为60（当剩余物理内存低于40%（40=100-60）时，开始使用交换空间）

> 对于大多数操作系统，设置为100可能会影响整体性能，而设置为更低值（甚至为0）则可能减少响应延迟

#### 16.2.3 调整ulimit上限，默认值为比较小的数字

```
$ ulimit -n 查看允许最大进程数
$ ulimit -u 查看允许打开最大文件数
```
> 修改：

```
$ sudo vi /etc/security/limits.conf 修改打开文件数限制
末尾添加：
*                soft    nofile          1024000
*                hard    nofile          1024000
Hive             -       nofile          1024000
hive             -       nproc           1024000
$ sudo vi /etc/security/limits.d/20-nproc.conf 修改用户打开进程数限制
修改为：
#*          soft    nproc     4096
#root       soft    nproc     unlimited
*          soft    nproc     40960
root       soft    nproc     unlimited
```

#### 16.2.4 开启集群的时间同步NTP

#### 16.2.5 更新系统补丁

> 更新补丁前，请先测试新版本补丁对集群节点的兼容性

### 16.3 Hdfs优化(hdfs-site.xml)

#### 16.3.1 保证RPC调用会有较多的线程数

```
属性：dfs.namenode.handler.count
```
> 解释：该属性是NameNode服务默认线程数，的默认值是10，根据机器的可用内存可以调整为50~100

```
属性：dfs.datanode.handler.count
```
> 解释：该属性默认值为10，是DataNode的处理线程数，如果HDFS客户端程序读写请求比较多，可以调高到15~20，设置的值越大，内存消耗越多，不要调整的过高，一般业务中，5~10即可。

#### 16.3.2 副本数的调整

```
属性：dfs.replication
```
> 解释：如果数据量巨大，且不是非常之重要，可以调整为2~3，如果数据非常之重要，可以调整为3~5。

#### 16.3.3 文件块大小的调整

```
属性：dfs.blocksize
```
> 解释：块大小定义，该属性应该根据存储的大量的单个文件大小来设置，如果大量的单个文件都小于100M，建议设置成64M块大小，对于大于100M或者达到GB的这种情况，建议设置成256M，一般设置范围波动在64M~256M之间。

### 16.4 HBase优化

#### 16.4.1 优化DataNode允许的最大文件打开数

```
属性：dfs.datanode.max.transfer.threads
文件：hdfs-site.xml
```
> 解释：HBase一般都会同一时间操作大量的文件，根据集群的数量和规模以及数据动作，设置为4096或者更高。默认值：4096

#### 16.4.2 优化延迟高的数据操作的等待时间

```
属性：dfs.image.transfer.timeout
文件：hdfs-site.xml
```
> 解释：如果对于某一次数据操作来讲，延迟非常高，socket需要等待更长的时间，建议把该值设置为更大的值（默认60000毫秒），以确保socket不会被timeout掉。

#### 16.4.3 优化数据的写入效率

```
属性：
mapreduce.map.output.compress
mapreduce.map.output.compress.codec
文件：mapred-site.xml
```
> 解释：开启这两个数据可以大大提高文件的写入效率，减少写入时间。第一个属性值修改为true，第二个属性值修改为：org.apache.hadoop.io.compress.GzipCodec

#### 16.4.4 优化DataNode存储

```
属性：dfs.datanode.failed.volumes.tolerated
文件：hdfs-site.xml
```
> 解释：默认为0，意思是当DataNode中有一个磁盘出现故障，则会认为该DataNode shutdown了。如果修改为1，则一个磁盘出现故障时，数据会被复制到其他正常的DataNode上。

#### 16.4.5 设置RPC监听数量

```
属性：hbase.regionserver.handler.count
文件：hbase-site.xml
```
> 解释：默认值为30，用于指定RPC监听的数量，可以根据客户端的请求数进行调整，读写请求较多时，增加此值。

#### 16.4.6 优化HStore文件大小

```
属性：hbase.hregion.max.filesize
文件：hbase-site.xml
```

> 解释：默认值10737418240（10GB），如果需要运行HBase的MR任务，可以减小此值，因为一个region对应一个map任务，如果单个region过大，会导致map任务执行时间过长。该值的意思就是，如果HFile的大小达到这个数值，则这个region会被切分为两个Hfile。

#### 16.4.7 优化hbase客户端缓存

```
属性：hbase.client.write.buffer
文件：hbase-site.xml
```
> 解释：用于指定HBase客户端缓存，增大该值可以减少RPC调用次数，但是会消耗更多内存，反之则反之。一般我们需要设定一定的缓存大小，以达到减少RPC次数的目的。

#### 16.4.8 指定scan.next扫描HBase所获取的行数

```
属性：hbase.client.scanner.caching
文件：hbase-site.xml
```
> 解释：用于指定scan.next方法获取的默认行数，值越大，消耗内存越大。

### 16.5 内存优化

> HBase操作过程中需要大量的内存开销，毕竟Table是可以缓存在内存中的，一般会分配整个可用内存的70%给HBase的Java堆。但是不建议分配非常大的堆内存，因为GC过程持续太久会导致RegionServer处于长期不可用状态，一般16~48G内存就可以了，如果因为框架占用内存过高导致系统内存不足，框架一样会被系统服务拖死。

#### 16.5.1 JVM优化

```
涉及文件：hbase-env.sh
```

#### 16.5.2 并行GC

```
参数：-XX:+UseParallelGC
解释：开启并行GC
```

#### 16.5.3 同时处理垃圾回收的线程数

```
参数：-XX:ParallelGCThreads=cpu_core – 1
解释：该属性设置了同时处理垃圾回收的线程数。
```

#### 16.5.4 禁用手动GC

```
参数：-XX:DisableExplicitGC
解释：防止开发人员手动调用GC
```

### 16.6 Zookeeper优化

#### 16.6.1 优化Zookeeper会话超时时间

```
参数：zookeeper.session.timeout
文件：hbase-site.xml
```
> 解释：In hbase-site.xml, set zookeeper.session.timeout to 30 seconds or less to bound failure detection (20-30 seconds is a good start).该值会直接关系到master发现服务器宕机的最大周期，默认值为30秒，如果该值过小，会在HBase在写入大量数据发生而GC时，导致RegionServer短暂的不可用，从而没有向ZK发送心跳包，最终导致认为从节点shutdown。一般20台左右的集群需要配置5台zookeeper。

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)

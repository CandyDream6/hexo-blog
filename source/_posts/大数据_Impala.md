---
title: Impala Hadoop的开源本地分析数据库
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(68).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(68).jpg
tags:
  - 大数据
  - 标签一
category:
  - - 编程
    - 大数据
date: 2020-08-17 17:50:59
---

Apache Impala是Apache Hadoop的开源原生分析数据库。

mpala 为 Hadoop 上的 BI/解析查询提供了低延迟和高并发性(不是由诸如 Apache Hive 这样的批处理框架提供的)。即使在多租户环境中，Impala 也是线性扩展的。

使用相同的文件和数据格式以及元数据、安全和资源管理框架作为您的 Hadoop 部署ーー没有冗余的基础设施或数据转换/重复。

对于 Apache Hive 用户，Impala 使用相同的元数据和 ODBC 驱动程序。与 Hive 一样，Impala 支持 SQL，因此您不必担心重新发明实现轮。

使用 Impala，更多的用户，无论是使用 SQL 查询还是 BI 应用程序，都可以通过单个存储库和元数据存储库通过分析与更多的数据进行交互。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 即席查询背景介绍

> 在快递业务运营过程中，经常会收到用户的投诉或者咨询，用户向咨询服务人员提供账号信息，客服人员可以根据用户的账号查询出来该用户的画像，以便对该用户的情况作为了解，因此数据开发人员需要编写sql语句实现用户画像开发，sql语句的开发牵扯到多表的关联会比较复杂，所以需要开发者平台实现sql的编写，Hue即实现了sql的开发

> 即席查询的作用：
>> 数据分析师可以根据数据表随意组合查询不同指标
> 
>> 在开发业务代码之前编写sql语句进行测试
> 
>> 开发人员可以根据现有的表结构进行自定义查询

## 2 业务处理流程

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/1.png)

## 3 impala

### 3.1 impala基本介绍

> `impala`是cloudera提供的一款高效率的sql查询工具，提供实时的查询效果，官方测试性能比hive快10到100倍，其sql查询比sparkSQL还要更加快速，号称是当前大数据领域最快的查询sql工具，

> impala是参照谷歌的新三篇论文（`Caffeine--网络搜索引擎、Pregel--分布式图计算、Dremel--交互式分析工具`）当中的`Dremel`实现而来，其中旧三篇论文分别是（`BigTable，GFS，MapReduce`）分别对应我们即将学的HBase和已经学过的HDFS以及MapReduce。

> impala是基于hive并使用内存进行计算，兼顾数据仓库，具有实时，批处理，多并发等优点

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/2.png)

> Kudu与Apache Impala （孵化）紧密集成，impala天然就支持兼容kudu，允许开发人员使用Impala的SQL语法从Kudu的tablets 插入，查询，更新和删除数据；

### 3.2 impala与hive的关系

> impala是基于hive的大数据分析查询引擎，直接使用hive的元数据库metadata，意味着impala元数据都存储在hive的metastore当中，并且impala兼容hive的绝大多数sql语法。所以需要安装impala的话，必须先安装hive，保证hive安装成功，并且还需要启动hive的metastore服务。

> Hive元数据包含用Hive创建的database、table等元信息。元数据存储在关系型数据库中，如Derby、MySQL等。

> 客户端连接metastore服务，metastore再去连接MySQL数据库来存取元数据。有了metastore服务，就可以有多个客户端同时连接，而且这些客户端不需要知道MySQL数据库的用户名和密码，只需要连接metastore 服务即可

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/3.png)

> Hive适合于长时间的批处理查询分析，而Impala适合于实时交互式SQL查询。可以先使用hive进行数据转换处理，之后使用Impala在Hive处理后的结果数据集上进行快速的数据分析。

### 3.3 impala与hive的异同

> Impala 与Hive都是构建在Hadoop之上的数据查询工具各有不同的侧重适应面，但从客户端使用来看Impala与Hive有很多的共同之处，如数据表元数据、ODBC/JDBC驱动、SQL语法、灵活的文件格式、存储资源池等。

> 但是Impala跟Hive`最大的优化区别`在于：`没有使用 MapReduce进行并行计算`，虽然MapReduce是非常好的并行计算框架，但它更多的面向批处理模式，而不是面向交互式的SQL执行。与 MapReduce相比，Impala把整个查询分成一`执行计划树`，而不是一连串的MapReduce任务，在分发执行计划后，Impala使用拉式获取数据的方式获取结果，把结果数据组成按执行树流式传递汇集，减少的了把中间结果写入磁盘的步骤，再从磁盘读取数据的开销。Impala使用服务的方式避免每次执行查询都需要启动的开销，即相比Hive没了MapReduce启动时间。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/4.png)

> `Impala使用的优化技术`
>> 使用LLVM产生运行代码，针对特定查询生成特定代码，同时使用Inline的方式减少函数调用的开销，加快执行效率。(C++特性)
> 
>> 充分利用可用的硬件指令（SSE4.2）。
> 
>> 更好的IO调度，Impala知道数据块所在的磁盘位置能够更好的利用多磁盘的优势，同时Impala支持直接数据块读取和本地代码计算checksum。
> 
>> 通过选择合适数据存储格式可以得到最好性能（Impala支持多种存储格式）。
> 
>> 最大使用内存，中间结果不写磁盘，及时通过网络以stream的方式传递。

> `执行计划`
>> Hive: 依赖于MapReduce执行框架，执行计划分成 map->shuffle->reduce->map->shuffle->reduce…的模型。如果一个Query会 被编译成多轮MapReduce，则会有更多的写中间结果。由于MapReduce执行框架本身的特点，过多的中间过程会增加整个Query的执行时间。
> 
>> Impala: 把执行计划表现为一棵完整的执行计划树，可以更自然地分发执行计划到各个Impalad执行查询，而不用像Hive那样把它组合成管道型的 map->reduce模式，以此保证Impala有更好的并发性和避免不必要的中间sort与shuffle。

> `数据流`
>> Hive: 采用推的方式，每一个计算节点计算完成后将数据主动推给后续节点。
> 
>> Impala: 采用拉的方式，后续节点通过getNext主动向前面节点要数据，以此方式数据可以流式的返回给客户端，且只要有1条数据被处理完，就可以立即展现出来，而不用等到全部处理完成，更符合SQL交互式查询使用

> `内存使用`
>> Hive: 在执行过程中如果内存放不下所有数据，则会使用外存，以保证Query能顺序执行完。每一轮MapReduce结束，中间结果也会写入HDFS中，同样由于MapReduce执行架构的特性，shuffle过程也会有写本地磁盘的操作。
> 
>> Impala: 在遇到内存放不下数据时，版本1.0.1是直接返回错误，而不会利用外存，以后版本应该会进行改进。这使用得Impala目前处理Query会受到一定的限制，最好还是与Hive配合使用

> `调度`
>> Hive: 任务调度依赖于Hadoop的调度策略。
> 
>> Impala: 调度由自己完成，目前只有一种调度器simple-schedule，它会尽量满足数据的局部性，扫描数据的进程尽量靠近数据本身所在的物理机器。调度器 目前还比较简单，在SimpleScheduler::GetBackend中可以看到，现在还没有考虑负载，网络IO状况等因素进行调度。但目前 Impala已经有对执行过程的性能统计分析，应该以后版本会利用这些统计信息进行调度吧。

> `容错`
>> Hive: 依赖于Hadoop的容错能力。
> 
>> Impala: 在查询过程中，没有容错逻辑，如果在执行过程中发生故障，则直接返回错误（这与Impala的设计有关，因为Impala定位于实时查询，一次查询失败， 再查一次就好了，再查一次的成本很低）。

> `适用面`
>> Hive: 复杂的批处理查询任务，数据转换任务。
> 
>> Impala：实时数据分析，因为不支持UDF，能处理的问题域有一定的限制，与Hive配合使用,对Hive的结果数据集进行实时分析

### 3.4 impala的优缺点

#### 3.4.1 优点

> 基于内存运算，不需要把中间结果写入磁盘，省掉了大量的I/O开销。

> 无需转换为Mapreduce，直接访问存储在HDFS，HBase中的数据进行作业调度，速度快。

> 使用了支持`Data locality（数据本地化）`的I/O调度机制，尽可能地将数据和计算分配在同一台机器上进行，减少了网络开销。

> 支持各种文件格式，如TEXTFILE 、SEQUENCEFILE 、RCFile、Parquet。

> 可以访问hive的metastore，对hive数据直接做数据分析。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/5.png)

#### 3.4.2 缺点

> 对内存的依赖大，且完全依赖于hive。

> 实践中，分区超过1万，性能严重下降。

> 只能读取文本文件，而不能直接读取自定义二进制文件。

> 每当新的记录/文件被添加到HDFS中的数据目录时，该表需要被刷新。

### 3.5 impala支持的文件格式

> Impala可以对Hadoop中大多数格式的文件进行查询。它能通过create table和insert的方式将一部分格式的数据加载到table中，但值得注意的是，有一些格式的数据它是无法写入的（write to）。对于Impala无法写入的数据格式，我们只能通过Hive建表，通过Hive进行数据的写入，然后使用Impala来对这些保存好的数据执行查询操作。

文件类型 | 文件格式 | 压缩编码 | 能否Create？ | 能否Insert？
---|---|---|---|---
Parquet | 结构化 | Snappy、GZIP | 能 | 能
Text | 非结构化 | LZO | 能。<br/><br/>如果建表时没有指定存储类型，默认采用未压缩的text，字段由ASCII编码的0x01字符串分割 | 能<br/><br/>如果使用了LZO压缩，则只能通过Hive建表和插入数据。
Avro | 结构化 | Snappy<br/><br/>GZIP<br/><br/>Deflate<br/><br/>BZIP2 | 在Impala 1.4.0 或者更高的版本上支持，之前的版本只能通过Hive来建表。 | 不能<br/><br/>只能通过LOAD DATA的方式将已经转换好格式的数据加载进去，或者使用Hive来插入数据
RCFile | 结构化 | Snappy<br/><br/>GZIP<br/><br/>Deflate<br/><br/>BZIP2 | 能 | 不能<br/><br/>只能通过LOAD DATA的方式将已经转换好格式的数据加载进去，或者使用Hive来插入数据
SequenceFile | 结构化 | Snappy<br/><br/>GZIP<br/><br/>Deflate<br/><br/>BZIP2 | 能 | 不能<br/><br/>只能通过LOAD DATA的方式将已经转换好格式的数据加载进去，或者使用Hive来插入数据

> Impala支持以下压缩编码：
>> `Snappy` – 推荐的编码，因为它在压缩率和解压速度之间有很好的平衡性，Snappy压缩速度很快，但是不如GZIP那样能节约更多的存储空间。Impala不支持Snappy压缩的text file
> 
>> `GZIP` – 压缩比很高能节约很多存储空间，Impala不支持GZIP压缩的text file
> 
>> `Deflate` – Impala不支持GZIP压缩的text file
> 
>> `BZIP2` - Impala不支持BZIP2压缩的text file
> 
>> `LZO` – 只用于text file，Impala可以查询LZO压缩的text格式数据表，但是不支持insert数据，只能通过Hive来完成数据的insert

### 3.6 impala的架构

> Impala是Cloudera在受到Google的Dremel启发下开发的实时交互SQL大数据查询工具（实时SQL查询引擎Impala），通过使用与商用并行关系数据库中类似的分布式查询引擎（由`Query Planner`、`Query Coordinator`和`Query Exec Engine`三部分组成），可以直接从HDFS或HBase中用SELECT、JOIN和统计函数查询数据，从而大大降低了延迟。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/6.png)

> Impala主要由`Impalad`、 `State Store`、`Catalogd`和`CLI`组成。

> `Impalad`
>> ⻆⾊名称为Impala Daemon,是在每个节点上运⾏的进程，是Impala的核⼼组件，进程名是Impalad;
> 
>> 负责读写数据⽂件，接收来⾃Impala-shell，JDBC,ODBC等的查询请求，与集群其它Impalad分布式并⾏完成查询任务，并将查询结果返回给中⼼协调者。
> 
>> 为了保证Impalad进程了解其它Impalad的健康状况，Impalad进程会⼀直与statestore保持通信。
> 
>> Impalad服务由三个模块组成：`Query Planner`、`Query Coordinator`和`Query Executor`，前两个模块组成前端，负责接收SQL查询请求，解析SQL并转换成执⾏计划，交由后端执⾏。

> `Impala State Store`
>> statestore监控集群中Impalad的健康状况，并将集群健康信息同步给Impalad。
> 
>> statestore进程名为statestored。

> `catalogd`
>> Impala执⾏的SQL语句引发元数据发⽣变化时，catalog服务负责把这些元数据的变化同步给其它Impalad进程(⽇志验证,监控statestore进程⽇志)
> 
>> catalogd会在Impala集群启动的时候加载hive元数据信息到Impala，其他时候不会主动加载，需要使用invalidate metadata，refresh命令。
> 
>> catalog服务对应进程名称是catalogd
> 
>> 由于⼀个集群需要⼀个catalogd以及⼀个statestored进程，⽽且catalogd进程所有请求都是经过statestored进程发送，所以官⽅建议让statestored进程与catalogd进程安排同个节点。

> `CLI`
>> 提供给用户查询使用的命令行工具（Impala Shell使用python实现），同时Impala还提供了Hue，JDBC， ODBC使用接口

### 3.7 impala如何执行查询

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/7.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/8.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/9.png)

> 1 客户端通过ODBC、JDBC、或者Impala shell向Impala集群中的任意节点发送SQL语句，这个节点的impalad实例作为这个查询的协调器（coordinator）

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/10.png)

> 2 Impala解析和分析这个查询语句来决定集群中的哪个impalad实例来执行某个任务，HDFS和HBase给本地的impalad实例提供数据访问

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/11.png)

> 3 各个impalad向协调器impalad返回数据，然后由协调器impalad向client发送结果集

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/12.png)


### 3.8 浏览器页面访问

> 访问impalad的管理界面http://node2:25000/

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/13.png)

> 访问statestored的管理界面http://node2:25010/

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/14.png)

> 访问catalogd 的管理界面http://node2:25020/

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/15.png)

## 4 impala-shell命令参数

### 4.1 impala-shell外部命令

> 所谓的外部命令指的是不需要进入到impala-shell交互命令行当中即可执行的命令参数。impala-shell后面执行的时候可以带很多参数。你可以在启动 impala-shell 时设置，用于修改命令执行环境。

```
impala-shell –h可以帮助我们查看帮助手册。也可以参考课程附件资料
```

> 比如几个常见的：

```
impala-shell –f 文件路径 执行指的的sql查询文件。
impala-shell –i 指定连接运行 impalad 守护进程的主机。默认端口是 21000。你可以连接到集群中运行 impalad 的任意主机。
impala-shell –o 保存执行结果到文件当中去。
```

### 4.2 impala-shell内部命令

> 所谓内部命令是指，进入impala-shell命令行之后可以执行的语法。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/16.png)

> connect hostname 连接到指定的机器impalad上去执行

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/17.png)

> refresh dbname.tablename增量刷新，刷新某一张表的元数据，主要用于刷新hive当中数据表里面的数据改变的情况

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/18.png)

> invalidate metadata全量刷新，性能消耗较大，主要用于hive当中新建数据库或者数据库表的时候来进行刷新。

> quit/exit命令 从Impala shell中弹出

> explain 命令 用于查看sql语句的执行计划。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/19.png)

> explain的值可以设置成0,1,2,3等几个值，其中3级别是最高的，可以打印出最全的信息

```
set explain_level=3;
```

> profile命令执行sql语句之后执行，可以打印出更加详细的执行步骤，主要用于查询结果的查看，集群的调优等。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/20.png)

> `注意`: 如果在hive窗口中插入数据或者新建的数据库或者数据库表，那么在impala当中是不可直接查询，需要执行invalidate metadata以通知元数据的更新；

> 在impala-shell当中插入的数据，在impala当中是可以直接查询到的，不需要刷新数据库，其中使用的就是catalog这个服务的功能实现的，catalog是impala1.2版本之后增加的模块功能，主要作用就是同步impala之间的元数据。

> 更新操作通知Catalog，Catalog通过广播的方式通知其它的Impalad进程。默认情况下Catalog是异步加载元数据的，因此查询可能需要等待元数据加载完成之后才能进行（第一次加载）

## 5 impala sql语法

### 5.1 数据库特定语言

> `创建数据库`

> CREATE DATABASE语句用于在Impala中创建新数据库。

```
CREATE DATABASE IF NOT EXISTS database_name;
这里，IF NOT EXISTS是一个可选的子句。如果我们使用此子句，则只有在没有具有相同名称的现有数据库时，才会创建具有给定名称的数据库。
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/21.png)

> impala默认使用impala用户执行操作，会报权限不足问题，解决办法：
>> 一：给HDFS指定文件夹授予权限

```
hadoop fs -chmod -R 777 hdfs://node2:9000/user/hive
```

>> 二：haoop 配置文件中hdfs-site.xml 中设置权限为false

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/22.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/23.png)

> 默认就会在hive的数仓路径下创建新的数据库名文件夹

```
/user/hive/warehouse/ittest.db
```

> 也可以在创建数据库的时候指定hdfs路径。需要注意该路径的权限。

```
hadoop fs -mkdir -p /input/impala
hadoop fs -chmod -R 777 /input/impala
create  external table  t3(id int ,name string ,age int )  row  format  delimited fields terminated  by  '\t' location  '/input/impala/external';
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/24.png)

> `删除数据库`

> Impala的DROP DATABASE语句用于从Impala中删除数据库。 在删除数据库之前，建议从中删除所有表。

> 如果使用级联删除，Impala会在删除指定数据库中的表之前删除它。

```
DROP database sample cascade
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/25.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/26.png)

### 5.2 表特定语句

#### 5.2.1 `create table 语句`

> CREATE TABLE语句用于在Impala中的所需数据库中创建新表。 需要指定表名字并定义其列和每列的数据类型

> impala支持的数据类型和hive类似，除了sql类型外，还支持java类型

```
基本格式:
create table IF NOT EXISTS database_name.table_name (
column1 data_type,
column2 data_type,
column3 data_type,
………
columnN data_type
);
```

> 例子:

```
CREATE TABLE IF NOT EXISTS my_db.student(name STRING, age INT, contact INT );
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/27.png)

> 默认建表的数据存储路径跟hive一致。也可以在建表的时候通过location指定具体路径，需要注意hdfs权限问题。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/28.png)

#### 5.2.2 `insert语句`

> Impala的INSERT语句有两个子句: into和overwrite。into用于插入新记录数据，overwrite用于覆盖已有的记录

```
insert into table_name (column1, column2, column3,...columnN) values (value1, value2, value3,...valueN);
Insert into table_name values (value1, value2, value2);
```

> 这里，column1，column2，... columnN是要插入数据的表中的列的名称。还可以添加值而不指定列名，但是，需要确保值的顺序与表中的列的顺序相同。

> 举个例子：

```
create table employee (Id INT, name STRING, age INT,address STRING, salary BIGINT);
insert into employee VALUES (1, 'Ramesh', 32, 'Ahmedabad', 20000 );
insert into employee values (2, 'Khilan', 25, 'Delhi', 15000 );
Insert into employee values (3, 'kaushik', 23, 'Kota', 30000 );
Insert into employee values (4, 'Chaitali', 25, 'Mumbai', 35000 );
Insert into employee values (5, 'Hardik', 27, 'Bhopal', 40000 );
Insert into employee values (6, 'Komal', 22, 'MP', 32000 );
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/29.png)

#### 5.2.3 `overwrite`

> 覆盖子句覆盖表当中全部记录。 覆盖的记录将从表中永久删除。

```
Insert overwrite employee values (1, 'Ram', 26, 'Vishakhapatnam', 37000 );
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/30.png)

#### 5.2.4 `select语句`

> Impala SELECT语句用于从数据库中的一个或多个表中提取数据。 此查询以表的形式返回数据

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/31.png)

#### 5.2.5 `describe语句`

> impala中的describe语句用于提供表的描述。 此语句的结果包含有关表的信息，例如列名称及其数据类型。

```
Describe table_name;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/32.png)

> 此外，还可以使用hive的查询表元数据信息语句。

```
desc formatted table_name;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/33.png)

#### 5.2.6 `alter table`

> Impala中的Alter table语句用于对给定表执行更改。使用此语句，我们可以添加，删除或修改现有表中的列，也可以重命名它们。

> 表重命名：

```
ALTER TABLE [old_db_name.]old_table_name RENAME TO [new_db_name.]new_table_name
```

> 向表中添加列：

```
ALTER TABLE name ADD COLUMNS (col_spec[, col_spec ...])
```

> 从表中删除列：

```
ALTER TABLE name DROP [COLUMN] column_name
```

> 更改列的名称和类型：

```
ALTER TABLE name CHANGE column_name new_name new_type
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/34.png)

#### 5.2.7 `delete、truncate table`

> Impala drop table语句用于删除Impala中的现有表。此语句还会删除内部表的底层HDFS文件。

> 注意：使用此命令时必须小心，因为删除表后，表中可用的所有信息也将永远丢失。

```
DROP table database_name.table_name;
```

> Impala的Truncate Table语句用于从现有表中删除所有记录。保留表结构。

> 您也可以使用DROP TABLE命令删除一个完整的表，但它会从数据库中删除完整的表结构，如果您希望存储一些数据，您将需要重新创建此表。

```
truncate table_name;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/35.png)

#### 5.2.8 `view视图`

> 视图仅仅是存储在数据库中具有关联名称的Impala查询语言的语句。 它是以预定义的SQL查询形式的表的组合。

> 视图可以包含表的所有行或选定的行。

```
Create View IF NOT EXISTS view_name as Select statement
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/36.png)

> 创建视图view、查询视图view

```
CREATE VIEW IF NOT EXISTS employee_view AS select name, age from employee;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/37.png)

> 修改视图

```
ALTER VIEW database_name.view_name为Select语句
```

> 例如

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/38.png)

> 以下是Alter View语句的示例

```
Alter view employee_view as select id, name, age, address from employee;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/39.png)

> 删除视图

```
DROP VIEW database_name.view_name;
```

> `order by 子句`

> Impala ORDER BY子句用于根据一个或多个列以升序或降序对数据进行排序。 默认情况下，一些数据库按升序对查询结果进行排序。

```
select * from table_name ORDER BY col_name [ASC|DESC] [NULLS FIRST|NULLS LAST]
```

> 可以使用关键字ASC或DESC分别按升序或降序排列表中的数据。

> 如果我们使用NULLS FIRST，表中的所有空值都排列在顶行; 如果我们使用NULLS LAST，包含空值的行将最后排列。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/40.png)

#### 5.2.9 `group by 子句`

> Impala GROUP BY子句与SELECT语句协作使用，以将相同的数据排列到组中。

```
select data from table_name Group BY col_name;
```

#### 5.2.10 `having子句`

> Impala中的Having子句允许您指定过滤哪些组结果显示在最终结果中的条件。 一般来说，Having子句与group by子句一起使用; 它将条件放置在由GROUP BY子句创建的组上。

#### 5.2.11 `limit ,offset`

> Impala中的limit子句用于将结果集的行数限制为所需的数，即查询的结果集不包含超过指定限制的记录。一般来说，select查询的resultset中的行从0开始。使用offset子句，我们可以决定从哪里考虑输出。

#### 5.2.12 `with 子句`
![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/41.png)


> 如果查询太复杂，我们可以为复杂部分定义别名，并使用Impala的with子句将它们包含在查询中。

```
格式: with x as (select 1), y as (select 2) (select * from x union y);
```

> 例如：使用with子句显示年龄大于25的员工和客户的记录。

```
with t1 as (select * from customers where age>25), t2 as (select * from employee where age>25)  (select * from t1 union select * from t2);
```

#### 5.2.13 `distinct`

> Impala中的distinct运算符用于通过删除重复值来获取唯一值。

```
select distinct columns… from table_name;
```

## 6 impala数据导入方式

### 6.1 load data

> 首先创建一个表：

```
create table user(id int ,name string,age int ) row format delimited fields terminated by "\t";
```
> 准备数据user.txt并上传到hdfs的 /user/impala路径下去

> 数据内容: user.txt

```
1	allen	18
2	kobe	19
```

> 加载数据

```
load data inpath '/user/impala/user.txt' into table user;
```

> 查询加载的数据:

```
select  *  from  user;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/42.png)

> 如果查询不不到数据，那么需要刷新一遍数据表。

```
refresh user;
```

### 6.2 insert into values

> 这种方式非常类似于RDBMS的数据插入方式。

```
create table t_test2(id int,name string);
insert into table t_test2 values(1,”zhangsan”);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/43.png)

### 6.3 insert into select

> 插入一张表的数据来自于后面的select查询语句返回的结果

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/44.png)

### 6.4 create as select

> 建表的字段个数、类型、数据来自于后续的select查询语句

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/45.png)

## 7 impala的java开发

### 7.1 pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.xiaoma</groupId>
    <artifactId>xiaoma_impala</artifactId>
    <version>1.0-SNAPSHOT</version>

    <repositories>
        <repository>
            <id>cloudera</id>
            <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.apache.kudu</groupId>
            <artifactId>kudu-client</artifactId>
            <version>1.6.0</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>

        <!--impala的jdbc操作-->
        <dependency>
            <groupId>com.cloudera</groupId>
            <artifactId>ImpalaJDBC41</artifactId>
            <version>2.6.3</version>
        </dependency>

        <!--Caused by : ClassNotFound : thrift.protocol.TPro-->
        <dependency>
            <groupId>org.apache.thrift</groupId>
            <artifactId>libfb303</artifactId>
            <version>0.9.3</version>
            <type>pom</type>
        </dependency>

        <!--Caused by : ClassNotFound : thrift.protocol.TPro-->
        <dependency>
            <groupId>org.apache.thrift</groupId>
            <artifactId>libthrift</artifactId>
            <version>0.9.3</version>
            <type>pom</type>
        </dependency>
    </dependencies>

</project>
```

### 7.2 代码演示

```
package cn.xiaoma;

import java.sql.*;

public class ImpalaJdbcDemo {

    public static void main(String[] args) {
        Connection con = null;
        ResultSet rs = null;
        PreparedStatement ps = null;
        String JDBC_DRIVER = "com.cloudera.impala.jdbc41.Driver";
        String CONNECTION_URL = "jdbc:impala://node2.xiaoma.cn:21050";
        try
        {
            // 注册步骤
            Class.forName(JDBC_DRIVER);
            // 获取连接
            con = (Connection) DriverManager.getConnection(CONNECTION_URL);
            ps = con.prepareStatement("select * from xiaoma.employee;");
            rs = ps.executeQuery();
            while (rs.next())
            {
                System.out.println(rs.getInt(1));
                System.out.println(rs.getString(2));
                System.out.println(rs.getInt(3));
                System.out.println(rs.getString(4));
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                rs.close();
                ps.close();
                con.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 8 使用impala操作kudu

> 需要先启动`hdfs`、`hive`、`kudu`、`impala`

> 使用impala的shell控制台

> 执行命令impala-shell

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/46.png)

> 1：使用该impala-shell命令启动Impala Shell。默认情况下，impala-shell 尝试连接到localhost端口21000 上的Impala守护程序。要连接到其他主机，请使用该-i <host:port>选项。要自动连接到特定的Impala数据库，请使用该-d <database>选项。例如，如果您的所有Kudu表都位于数据库中的Impala中impala_kudu，则-d impala_kudu可以使用此数据库。

> 2：要退出Impala Shell，请使用以下命令： quit;

### 8.1 创建kudu表

> 使用Impala创建新的Kudu表时，可以将该表创建为`内部表`或`外部表`。

#### 8.1.1 内部表

> `内部表`由Impala管理，当您从Impala中删除时，数据和表确实被删除。当您使用Impala创建新表时，它通常是内部表。

> 使用impala创建内部表：

```
CREATE TABLE `my_first_table`
(
id BIGINT,
name STRING,
PRIMARY KEY(id)
)
PARTITION BY HASH PARTITIONS 16
STORED AS KUDU
TBLPROPERTIES (
'kudu.num_tablet_replicas' = '1'
);
```

> 在 CREATE TABLE 语句中，必须首先列出构成主键的列。

> 此时创建的表是内部表，从impala删除表的时候，在底层存储的kudu也会删除表

```
drop table if exists my_first_table;
```

#### 8.1.2 外部表

> `外部表`（创建者CREATE EXTERNAL TABLE）不受Impala管理，并且删除此表不会将表从其源位置（此处为Kudu）丢弃。相反，它只会去除Impala和Kudu之间的映射。这是Kudu提供的用于将现有表映射到Impala的语法

> 使用java创建一个kudu表

```
package cn.xiaoma;

import org.junit.Before;
import org.junit.Test;

import java.sql.*;

/**
 * 使用impala操作kudu表
 * 1）创建表
 * 2）加载数据
 * 3）查询表的数据
 * 4) 修改数据
 * 5）删除数据
 */
public class ImpalaKuduDemo {

    //定义数据库的连接地址
    private String url;

    //定义数据库的连接
    private Connection connection;

    private PreparedStatement ps;

    /**
     * 初始化impala的连接地址
     */
    @Before
    public void init() throws SQLException {
        //初始化连接
        url = "jdbc:impala://node2.xiaoma.cn:21050/default";
        connection = DriverManager.getConnection(url);
    }

    /**
     *  1）创建表
     */
    @Test
    public void createTable() throws SQLException {
        String sql = "CREATE TABLE impala_student" +
                "(" +
                "CompanyId INT," +
                "WorkId INT," +
                "Name STRING," +
                "Gender STRING," +
                "Photo STRING," +
                "PRIMARY KEY(CompanyId)" +
                ") " +
                "PARTITION BY HASH PARTITIONS 16 " +
                "STORED AS KUDU " +
                "TBLPROPERTIES (" +
                "'kudu.num_tablet_replicas' = '1'" +
                ");";
        ps = connection.prepareStatement(sql);
        ps.execute();
    }

    /**
     * 2）加载数据
     */
    @Test
    public void insertData() throws SQLException {
        String sql = "insert into table impala_student(CompanyId, WorkId, Name,Gender,Photo) values(?,?,?,?,?)";

        for (int i = 1; i <=10 ; i++) {
            ps = connection.prepareStatement(sql);
            ps.setInt(1, i);
            ps.setInt(2, i*10);
            ps.setString(3, "zhang"+i );
            ps.setString(4, i%2+"");
            ps.setString(5, "Phone"+i);
            ps.execute();
        }
        ps.close();
        connection.close();
    }

    /**
     * 3）查询表的数据
     */
    @Test
    public void queryData() throws SQLException {
        String sql = "select * from impala_student where Gender=?";
        ps = connection.prepareStatement(sql);

        ps.setString(1, "0");

        ResultSet rs = ps.executeQuery();
        while(rs.next()){
            int CompanyId = rs.getInt(1);
            System.out.println("CompanyId="+CompanyId);
        }
    }

    /**
     * 4) 修改数据
     */
    @Test
    public void updateData() throws SQLException {
        String sql ="update impala_student set name=?,photo=? where companyid=?";

        ps = connection.prepareStatement(sql);
        ps.setString(1, "xiaoming");
        ps.setString(2, "phone-110");
        ps.setInt(3, 1);
        ps.execute();
    }

    /**
     * 5）删除数据
     */
    @Test
    public void deleteData() throws SQLException {
        String sql = "delete from impala_student where gender=?";

        ps = connection.prepareStatement(sql);
        ps.setString(1, "0");
        ps.execute();
    }

    @Test
    public void dropTable() throws SQLException {
        String sql = "drop table impala_student";

        ps = connection.prepareStatement(sql);
        ps.execute();
    }
}
```

> 在kudu的页面上可以观察到如下信息：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/47.png)

> 在impala的命令行查看表:

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/48.png)

> 当前在impala中并没有person这个表

> 使用impala创建外部表，将kudu的表映射到impala上：

> 在impala-shell执行

```
CREATE EXTERNAL TABLE `person` STORED AS KUDU
TBLPROPERTIES(
'kudu.table_name' = 'person',
'kudu.master_addresses' = 'node2.itcast.cn:7051')
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/49.png)

### 8.2 使用impala对kudu进行DML操作

#### 8.2.1 将数据插入Kudu表

> impala 允许使用标准 SQL 语句将数据插入 Kudu

##### 8.2.1.1 插入单个值

> 创建表

```
CREATE TABLE `my_first_table`
(
id BIGINT,
name STRING,
PRIMARY KEY(id)
)
PARTITION BY HASH PARTITIONS 16
STORED AS KUDU
TBLPROPERTIES (
'kudu.num_tablet_replicas' = '1'
);
```

> 此示例插入单个行

```
INSERT INTO my_first_table VALUES (50, "zhangsan");
```

> 查看数据

```
select * from my_first_table
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/50.png)

> 使用单个语句插入三行

```
INSERT INTO my_first_table VALUES (1, "john"), (2, "jane"), (3, "jim");
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/51.png)

##### 8.2.1.2 批量插入Batch Insert

> 从 Impala 和 Kudu 的角度来看，通常表现最好的方法通常是使用 Impala 中的 SELECT FROM 语句导入数据

```
INSERT INTO my_first_table SELECT * FROM temp1;
```

#### 8.2.2 更新数据

```
UPDATE my_first_table SET name="xiaowang" where id =1 ;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/52.png)

#### 8.2.3 删除数据

```
Delete from my_first_table where id =2;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/53.png)

### 8.3 更改表属性

> 开发人员可以通过更改表的属性来更改 Impala 与给定 Kudu 表相关的元数据。这些属性包括表名， Kudu 主地址列表，以及表是否由 Impala （内部）或外部管理。

#### 8.3.1 重命名 Impala 映射表

```
ALTER TABLE PERSON RENAME TO person_temp;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/54.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/55.png)

> `重命名impala表的时候，只会修改impala表的名字，不会修改kudu表名的名字，如果想修改kudu表名的话，需要使用impala3.3及以上版本`

#### 8.3.2 重新命名内部表的基础 Kudu 表

> 创建内部表：

```
CREATE TABLE kudu_student
(
CompanyId INT,
WorkId INT,
Name STRING,
Gender STRING,
Photo STRING,
PRIMARY KEY(CompanyId)
)
PARTITION BY HASH PARTITIONS 16
STORED AS KUDU
TBLPROPERTIES (
'kudu.num_tablet_replicas' = '1'
);
```

> 如果表是内部表，则可以通过更改 kudu.table_name 属性重命名底层的 Kudu 表

```
ALTER TABLE kudu_student SET TBLPROPERTIES('kudu.table_name' = 'new_student');
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/56.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/57.png)

> `注意：kudu.table_name属性的设置与版本有关`

> `在impala3.2版本中是无法修改底层的kudu表的名字的，从impala3.3开始可以修改`

> `在Impala 2.11及更低版本中，可以通过更改kudu.table_name属性来重命名基础Kudu表：`

> `结论：在impala2.11及impala3.2之间的版本是无法修改kudu.table_name属性的`

#### 8.3.3 将外部表重新映射到不同的 Kudu 表

> 果用户在使用过程中发现其他应用程序重新命名了kudu表，那么此时的外部表需要重新映射到kudu上

> 创建一个外部表：

```
CREATE EXTERNAL TABLE external_table
STORED AS KUDU
TBLPROPERTIES (
'kudu.master_addresses' = 'node2.itcast.cn:7051',
'kudu.table_name' = 'person'
);
```

> 重新映射外部表，指向不同的kudu表：

```
ALTER TABLE external_table
SET TBLPROPERTIES('kudu.table_name' = 'hashTable')
```

> `上面的操作是：将external_table映射的PERSON表重新指向hashTable表`

#### 8.3.4 更改 Kudu Master 地址

```
ALTER TABLE my_table
SET TBLPROPERTIES('kudu.master_addresses' = 'kudu-new-master.example.com:7051');
```

#### 8.3.5 将内部管理的表更改为外部

```
ALTER TABLE my_table SET TBLPROPERTIES('EXTERNAL' = 'TRUE');
```

> 将内部表更改成外部表以后，删除外部表，不会影响底层的kudu表，反之如果是内部表的话，删除内部表，则底层的kudu表也会同步删除

## 9 impala优化

### 9.1 impala关键配置

> 分配给此角色的内存软限制，由 Linux 内核强制执行。当达到此限制时，内核将只在主机面临内存压力时回收已分配给进程的页面。如果回收失败，内核可能会停止这些进程。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/58.png)

> 分配给此角色的内存硬限制，由 Linux 内核强制执行。当达到此限制时，内核将会回收已分配给进程的页面。如果回收失败，内核可能会停止这些进程。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/59.png)

> Impala Daemon 服务的内存限制（以字节为单位）。如果达到该限制，Impalad Daemon 上运行的查询会被停止。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/60.png)

### 9.2 impala查询分析

> 从 CM 主页进入 Impala 服务页面，点击查询按钮。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/61.png)

> 选择执行查询的时间范围，例如，昨天的上午 9 点到今天中午 12 点，使用 Impala 执行过 SQL 查询。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/62.png)

> 选择一个查询，并点击右侧的“查询详细信息”。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/63.png)

> “查询详细信息”中的 SQL 脚本。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/64.png)

> “查询详细信息”中的 SQL 脚本对应的执行计划。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/65.png)

> “查询详细信息”中的 SQL 脚本对应的查询耗时。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/66.png)

> “查询详细信息”中的 SQL 脚本对应的查询实例。

> 每一个查询段实例名称，与查询计划中是一一对应的，查询段中包括总耗时、缓冲池内存使用、入队和出队的内存和耗时开销。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Impala/67.png)

### 9.3 impala优化思路

> 查看执行计划：explain sql;

> 当SQL执行完成后， 使用profile输出底层的执行计划详细信息 ：profile;

> 当SQL执行完成后，使用summary输出查询时间和占用内存信息 ：summary;

> 大表和小表JOIN时，确保大表在左侧，小表在右侧( Impala 会广播小表到所有节点);

> 大表和大表JOIN时， 需要使用partitioned join。

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)

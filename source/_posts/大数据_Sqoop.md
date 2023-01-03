---
title: Sqoop 分布式数据迁移工具
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(44).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(44).jpg
tags:
  - 大数据
  - Hadoop
  - Sqoop
category:
  - - 编程
    - 大数据
 
date: 2019-10-22 15:42:27
---

sqoop是一款用于hadoop和关系型数据库之间数据导入导出的工具。

你可以通过sqoop把数据从数据库（比如mysql,oracle）导入到hdfs中；

也可以把数据从hdfs中导出到关系型数据库中。

sqoop通过Hadoop的MapReduce导入导出，因此提供了很高的并行性能以及良好的容错性。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 概述

> sqoop是apache旗下一款“`Hadoop和关系数据库服务器之间传送数据`”的工具。

```
导入数据：MySQL，Oracle导入数据到Hadoop的HDFS、HIVE、HBASE等数据存储系统；
导出数据：从Hadoop的HDFS、HIVE中导出数据到关系数据库mysql等
```

![概述](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Sqoop/1.png)

> `RDBMS`:`Relational Database Management System`(关系型数据库管理系统)

## 2 sqoop1与sqoop2架构对比

### 2.1 sqoop1架构

> Sqoop1以Client`客户`端的形式存在和运行。没有任务时是`没有进程存在`的。

![sqoop1架构](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Sqoop/2.png)

### 2.2 sqoop2架构

> sqoop2是以`B/S服务器`的形式去运行的，`始终会有Server服务端进程在运行`。

![sqoop2架构](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Sqoop/3.png)

## 3 工作机制

> 将导入或导出命令翻译成mapreduce程序来实现。

## 4 sqoop抽取的两种方式

> 对于Mysql数据的采集，通常使用Sqoop来进行。

> 通过Sqoop将关系型数据库数据到Hive有两种方式，一种是`原生Sqoop API`，一种是使用`HCatalog API`。两种方式略有不同。

> HCatalog方式与Sqoop方式的参数基本都是相同，只是个别不一样，都是可以实现Sqoop将数据抽取到Hive。

### 4.1 区别

#### 4.1.1 数据格式支持

> `Sqoop`支持的数据格式`较少`,不支持ORCFile。

> `HCatalog`支持数据格式的`较多`，包括RCFile, ORCFile, CSV, JSON和SequenceFile等格式。

#### 4.1.2 数据覆盖

> `Sqoop`方式`允许数据覆盖`。

> `HCatalog`不允许数据覆盖，`每次都只是追加`。

#### 4.1.3 字段名

> `Sqoop`方式比较随意，`不要求源表和目标表字段相同`(字段名称和个数都可以不相同)，它抽取的方式是`将字段按顺序插入`，比如目标表有3个字段，源表有一个字段，它会将数据插入到Hive表的第一个字段，其余字段为NULL。

> `HCatalog`不同，`源表和目标表字段名需要相同`，字段个数可以不相等，如果字段名不同，抽取数据的时候会报NullPointerException错误。HCatalog抽取数据时，会`按照字段名进行匹配插入`，哪怕字段个数不相等。

### 4.2 Sqoop方式

```
sqoop import \
--hive-import \
--connect 'jdbc:mysql://localhost:3306/test' \
--username 'root' \
--password '123456789' \
--query " select order_no from driver_action where  \$CONDITIONS" \
--hive-database test \
--hive-table driver_action \
--hive-partition-key pt \
--hive-partition-value 20190901 \
--null-string '' \
--null-non-string '' \
--num-mappers 1 \
--target-dir /tmp/test \
--delete-target-dir
```

### 4.3 HCatalog方式

```
sqoop import \
--connect jdbc:mysql://localhost:3306/test\
--username 'root' \
--password 'root' \
--query "SELECT order_no FROM driver_action  WHERE \$CONDITIONS" \
--hcatalog-database test \
--hcatalog-table driver_action \
--hcatalog-partition-keys pt \
--hcatalog-partition-values 20200104 \
--hcatalog-storage-stanza 'stored as orcfile tblproperties ("orc.compress"="SNAPPY")' \
--num-mappers 1
```

> 针对`不同字段名`，想要使用`HCatalog方式`将数据插入，可以使用下面的方式：

```
sqoop import \
--connect jdbc:mysql://localhost:3306/test\
--username 'root' \
--password 'root' \
--query "SELECT order_no_src as order_no_target  FROM driver_action WHERE \$CONDITIONS" \
--hcatalog-database test \
--hcatalog-table driver_action \
--hcatalog-partition-keys pt \
--hcatalog-partition-values 20200104 \
--hcatalog-storage-stanza 'stored as orc tblproperties ("orc.compress"="SNAPPY")' \
--num-mappers 1
```
## 5 项目选型

> 因为项目采用的是`ORC File`文件格式，`sqoop原始方式并不支持`，因此使用`HCatalog方式`来进行数据的导入导出。

## 6 Sqoop的数据导入

### 6.1 手册使用

> 查看手册

```
/usr/bin/sqoop help
```

> 命令行查看帮助

```
/usr/bin/sqoop list-databases --help
```

> 列出主机所有的数据库

```
/usr/bin/sqoop \
list-databases \
--connect jdbc:mysql://192.168.52.150:3306/ \
--username root \
--password 123456
```

> 查看某一个数据库下面的所有数据表

```
/usr/bin/sqoop \
list-tables \
--connect jdbc:mysql://192.168.52.150:3306/hive \
--username root \
--password 123456
```

### 6.2 完整数据导入

#### 6.2.1 导入数据库表数据到HDFS

```
/usr/bin/sqoop import \
--connect jdbc:mysql://192.168.52.150:3306/test \
--password 123456 \
--table emp \
-m 1
```

`注意`：mysql地址必须为服务器IP，不能是localhost或者机器名。

> 查看导入的数据

```
hdfs  dfs  -ls  /user/root/emp
```

默认使用逗号(,)分隔表的字段

#### 6.2.2 导入到HDFS指定目录

```
/usr/bin/sqoop \
--connect jdbc:mysql://192.168.52.150:3306/test \
--username root \
--password 123456 \
--table emp  \
--delete-target-dir \
--target-dir /sqoop/emp \
-m 1
```

使用参数 `--target-dir`来`指定导出目的地`，

使用参数`--delete-target-dir`来`判断导出目录是否已存在`，如果`存在就删掉`

> 查看导入的数据

```
hdfs dfs -text /sqoop/emp/part-m-00000
```

#### 6.2.3 导入到hdfs指定目录并指定字段之间的分隔符

```
/usr/bin/sqoop import  \
--connect jdbc:mysql://192.168.52.150:3306/test \
--username root \
--password 123456 \
--delete-target-dir \
--table emp  \
--target-dir /sqoop/emp2 \
-m 1 \
--fields-terminated-by '\t'
```

> 查看文件内容

```
hdfs dfs -text /sqoop/emp2/part-m-00000
```

#### 6.2.4 导入到HIVE

##### 6.2.4.1 第一步：准备hive数据库与表

> 将我们mysql当中的数据导入到hive表当中来

```
create database sqooptohive;

use sqooptohive;

create table sqooptohive.emp_hive(id int,name string,deg string,salary int ,dept string) \
row format delimited fields terminated by '\t' \
stored as orc;
```

![准备hive数据库与表](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Sqoop/4.png)

##### 6.2.4.2 第二步：开始导入

```
/usr/bin/sqoop import \
--connect jdbc:mysql://192.168.52.150:3306/test \
--username root \
--password 123456 \
--table emp \
--fields-terminated-by '\t' \
--hcatalog-database sqooptohive \
--hcatalog-table emp_hive \
-m 1
```

##### 6.2.4.3 第三步：hive表数据查看

```sql
select * from sqooptohive.emp_hive;
```

![hive表数据查看](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Sqoop/5.png)

### 6.3 条件部分导入

#### 6.3.2 where导入到HDFS

```
/usr/bin/sqoop import \
--connect jdbc:mysql://192.168.52.150:3306/test \
--username root \
--password 123456 --table emp_add \
--target-dir /sqoop/emp_add \
--delete-target-dir \
-m 1  \
--where "city = 'sec-bad'"
```

#### 6.3.2 sql语句查找导入hdfs

```
/usr/bin/sqoop import \
--connect jdbc:mysql://192.168.52.150:3306/test \
--username root \
--password 123456 \
--target-dir /sqoop/emp_conn
--delete-target-dir \
-m 1 \
--query 'select phno from emp_conn where 1=1 and  $CONDITIONS' \
```

> 查看hdfs数据内容

```
hdfs dfs -text /sqoop/emp_conn/part*
```

![查看hdfs数据内容](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Sqoop/6.png)

#### 6.3.3 增量导入数据到Hive表

```
/usr/bin/sqoop import \
--connect jdbc:mysql://192.168.52.150:3306/test \
--username root \
--password 123456 \
--fields-terminated-by '\t' \
--hcatalog-database sqooptohive \
--hcatalog-table emp_hive \
-m 1
--query "select * from emp where id>1203 and  \$CONDITIONS" \
```

![增量导入数据到Hive表](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Sqoop/7.png)

## 7 Sqoop的数据导出

### 7.1 第一步：创建mysql表

```sql
CREATE TABLE `emp_out` (
  `id` INT(11) DEFAULT NULL,
  `name` VARCHAR(100) DEFAULT NULL,
  `deg` VARCHAR(100) DEFAULT NULL,
  `salary` INT(11) DEFAULT NULL,
  `dept` VARCHAR(10) DEFAULT NULL
) ENGINE=INNODB DEFAULT CHARSET=utf8;
```

### 7.2 第二步：执行导出命令

通过`export`来实现数据的导出，将hive的数据导出到mysql当中去

```
/usr/bin/sqoop export \
--connect jdbc:mysql://192.168.52.150:3306/test \
--username root \
--password 123456 \
--table emp_out \
--hcatalog-database sqooptohive \
--hcatalog-table emp_hive \
-m 1
```

### 7.3 第三步：验证mysql表数据

![验证mysql表数据](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Sqoop/8.png)

## 8 Sqoop一些常用参数

参数 |说明
---|---
`--connect` | 连接关系型数据库的URL
`--username` | 连接数据库的用户名
`--password` | 连接数据库的密码
\-\-driver  | JDBC的driver class
`--query`或\-\-e <statement> | 将查询结果的数据导入，使用时必须伴随参--target-dir(导入hdfs)，--hcatalog-table(导入hive)，如果查询中有where条件，则条件后必须加上$CONDITIONS关键字。<br/><br/>如果使用双引号包含sql，则$CONDITIONS前要加上\\以完成转义：\\$CONDITIONS
`--hcatalog-database`  | 指定HCatalog表的数据库名称。如果未指定，default则使用默认数据库名称。提供 --hcatalog-database不带选项--hcatalog-table是错误的。
`--hcatalog-table`  | 此选项的参数值为HCatalog表名。该--hcatalog-table选项的存在表示导入或导出作业是使用HCatalog表完成的，并且是HCatalog作业的必需选项。
\-\-create-hcatalog-table | 此选项指定在导入数据时是否应自动创建HCatalog表。表名将与转换为小写的数据库表名相同。
\-\-hcatalog-storage-stanza 'stored as orc tblproperties ("orc.compress"="SNAPPY")' \\ | 建表时追加存储格式到建表语句中，tblproperties修改表的属性，这里设置orc的压缩格式为SNAPPY
`-m` | 指定并行处理的MapReduce任务数量。-m不为1时，需要用split-by指定分片字段进行并行导入，尽量指定int型。
`--split-by id` | 如果指定-split by, 必须使用$CONDITIONS关键字, 双引号的查询语句还要加\\
\-\-hcatalog-partition-keys<br/>\-\-hcatalog-partition-values | keys和values必须同时存在，相当于指定静态分区。允许将多个键和值提供为静态分区键。多个选项值之间用，（逗号）分隔。比如：<br/>--hcatalog-partition-keys year,month,day<br/>--hcatalog-partition-values 1999,12,31
`--null-string '\\N'`<br/>`--null-non-string '\\N'` | 指定mysql数据为空值时用什么符号存储，null-string针对string类型的NULL值处理，--null-non-string针对非string类型的NULL值处理
\-\-hive-drop-import-delims | 设置无视字符串中的分割符（hcatalog默认开启）
`--fields-terminated-by '\t'` | 设置字段分隔符

## 9 \-\-split-by详解

![--split-by](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Sqoop/9.png)

> 假设有一张表test，sqoop命令中`-–split-by 'id' -m 10`，会发生怎样奇特的事情。<br/>
> 首先呢，sqoop会去查表的元数据等等，重点说一下sqoop是如何根据–split-by进行分区的。<br/>
> 首先sqoop会向关系型数据库比如mysql发送一个命令:`select max(id),min(id) from test`。
> <br/>然后会把max、min之间的区间平均分为10分，最后`10个并行的map`去找数据库，同时也会分成`10个sql`给`10个map`去进行导入操作。
> 加入有1000条数据，则10个sql分别为`select XXX from table where split-by>=1 and split-by<100` , `select XXX from table where split-by>=101 and split-by<=200`.......最后`每个map各自获取各自SQL中的数据`进行导入工作。
> <br/>导数据就正式开始啦！

> `split-by`即便是`int`型，若`不是连续有规律递增`的话，各个map分配的数据是不均衡的，可能会`有些map很忙`，`有些map几乎没有数据处理`的情况，就是`数据倾斜`



## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
---
title: 浅谈Impala之INVALIDATE和REFRESH
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(95).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(95).jpg
tags:
  - 大数据
  - Impala
category:
  - - 编程
    - 大数据
  - - 编程
    - Impala
date: 2021-09-27 09:09:09
---

最近在搞数据到`Impala`，是先通过数据`插入到Hive`，然后`刷新Impala`元数据实现的。

`刷新Impala元数据`有两种方式，一种是`INVALIDATE`，一种是`REFRESH`。

以前一直理解`INVALIDATE是刷新结构信息同时刷新数据信息`，而`REFRESH是刷新数据信息`。

然后碰到一个`比较特殊的情况`，下面开始聊聊。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 区别

> 先看看`网友对两者的解释`，几乎全网通篇解释都如下。

>`如果数仓中发生了增删表或改变表结构的行为`，如create table、drop table、alter table add column等，就使用invalidate metadata [table]语句。
> 
> `如果数仓中某表加入了新数据，或者有分区的改动`，如load data、alter table add partition等，就使用refresh [table] (partition [partition])语句。
> 
> `invalidate metadata比起refresh而言要重量级得多`。因此，也几乎禁止使用不带表名的invalidate metadata语句。
> 
> 如果数仓中涉及到非常大批量的元数据更改，那么建议直接重启catalogd和statestored，这总比使用不带表名的invalidate metadata来得高效一些。

## 2 个人疑惑

>操作
>> 1：在Hive中创建一个表
>
>> 2：在Impala中执行INVALIDATE METADATA操作。
>
>> 3：在Hive中插入一个数据。
>
>> 4：在Impala中查询这条数据。

> 疑惑点：
>> `1：照理说第四步应该查不到数据，因为插入数据后没有执行任何刷新操作，但是这里就可以查到数据，奇怪了。`
> 
>> `2：如果我在第二步和第三步之间，在impala中执行一下查询操作，那么跑完第三步后，第四步又查不到这条数据，奇怪了。`
> 
>> `3：如果第二步用REFRESH的话，第四步impala就查不到数据，符合预期。`

## 3 官网溜溜

> 感觉水友们的解释都不是很明确，于是我们`去官网看看`。

> 官网地址
>> https://impala.apache.org/docs/build/html/topics/impala_invalidate_metadata.html
>
>> https://impala.apache.org/docs/build/html/topics/impala_refresh.html

### 3.1 INVALIDATE

```shell
INVALIDATE METADATA [[db_name.]table_name]
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/浅谈Impala之INVALIDATE和REFRESH/1.png)

### 3.2 REFRESH

```shell
REFRESH [db_name.]table_name [PARTITION (key_col1=val1 [, key_col2=val2...])]
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/浅谈Impala之INVALIDATE和REFRESH/2.png)

## 4 总结

> 可以看到，官网基本和水友们说的差不多，`增删表和结构变更用INVALIDATE`
 
> 但是有一点水友们没有说清楚，也是我2中的那个疑惑问题。
>> `INVALIDATE严格意义上并不是刷新元数据，如其名，他是将一个或所有表的元数据标记为陈旧`。下次 Impala 服务对元数据失效的表执行查询时，Impala 会在查询进行之前重新加载相关的元数据。
>
>> `REFRESH 语句从元存储数据库重新加载表的元数据`，并增量重新加载文件并从 HDFS NameNode读取元数据
 
> 所以俺的问题也得到了答案
>> `疑惑1是因为impala INVALIDATE标记元数据过期后，我在hive中插入数据，然后在impala中查询，这时候他回去加载所有的元数据，所以可以查到。`
>
>> `疑惑2是因为impala INVALIDATE标记元数据过期后，我执行查询操作，这时候会加载所有元数据，然后在hive中插入数据，再去impala中查询的话，不会去加载所有的元数据，因为刚才那次查询已经加载了一次了。`

## 5 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
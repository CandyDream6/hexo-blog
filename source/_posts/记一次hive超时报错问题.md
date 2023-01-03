---
title: 记一次hive超时报错问题
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(94).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(94).jpg
tags:
  - 大数据
  - hive
category:
  - - 编程
    - 大数据
  - - 编程
    - hive
date: 2021-09-12 20:40:52
---

最近在开发`RDBMS`到`Hive`/`Impala`/`Kudu`/`HBase`的载数功能。

碰到了一个`hive一直超时`的问题，可把人`烦死了`，就是`悟空那个表情`。

然后排查了一下问题，记录一下。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 场景

> 我们使用`hive做数仓`，将RDBMS的数据`摄取到hive`中，或者再到`Kudu/HBase`，这个一波操作的话，我们需要建两个表，一个`hive表`，一个`hive和RDBMS的StorageHandler表`。
 
> 当然如果是到Kudu或者HBase的话，还需要一张`hive到Kudu或者HBase的StorageHandler表`
 
> 也就是一波操作至少建立2-3个表。

## 2 事故现场

> 那么在这个建表加导入数据的过程中，经常出现`hive连接超时的情况`，恼火。

> 平时因为`开发任务很重`，所以每次碰到这个问题都是`直接重启Hive赶紧接着干活`。

> 终于有一天，内心的另一个我`忍受不了了`，决定把`这个狗东西的真面目`给他揭开。 

## 3 问题排查

### 3.1 现象查看

> 先去看了下`hive`，一切`显示正常`，这就`操蛋`了。

> 进到`hive的日志`中，看到了一点`端倪`，说是有`Hive Metastore Server运行不良`。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/记一次hive超时报错问题/3.png)

### 3.2 hive连接数查看

```shell
SELECT hive_open_connections,hive_open_operations FROM ENTITY_DATA
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/记一次hive超时报错问题/4.png)

### 3.3 配置查看

> 然后去看了下hive配置，原来CDH的hive`中hiveServer2`的`并发连接数`默认是`3`，`hiveServer2`和`Hive Metastore Server`的`最长等待时间`默认是`5分钟`。

```shell
hive.driver.parallel.compilation.global.limit
hive.metastore.client.socket.timeout
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/记一次hive超时报错问题/1.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/记一次hive超时报错问题/2.png)

> 所以问题原因就是，我们`同时`会有`很多载数任务进行`，这些任务`都比较耗费时间`。
> 
> 当有`三个任务`在执行的时候，`其他任务`就得`等待`，`超过5分钟`的话就`超时`了。

## 4 事故处理

> 解决问题的话，无非两个方向
>> 一是`增大并发数`
> 
>> 二是`延长超时时间 `

> 我们的话，因为载数任务比较多，3个并发肯定是不行的，所以先将载数`并发数调整到30`，`超时时间调整到15分钟`。

> 目前程序都`正常运行`，`没有出现问题`，以后要是这个配置扛不住的话，往上加就好了，当然也不是越大越好，`根据自己的实际场景`在这两者之间做一个`权衡`。

## 5 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
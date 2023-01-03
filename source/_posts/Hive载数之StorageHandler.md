---
title: Hive载数之StorageHandler
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(96).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(96).jpg
tags:
  - 大数据
  - Hive
category:
  - - 编程
    - 大数据
  - - 编程
    - Hive
  - - 编程
    - StorageHandler
date: 2021-09-29 09:09:01
---

最近项目在做`离线载数`，是`从RDBMS载数到Hive，Impala，Kudu和HBase`，这里整理一下过程，`给自己留个可以看的东西`。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 架构

> `StorageHandler官网`：https://cwiki.apache.org/confluence/display/Hive/StorageHandlers

> 我们整体的载数流程大概是这个样子

> 整个流程分为`quick模式`和`stable模式`
>> `quick模式`：数据从RDBMS直接到目标，中间不进行存储
> 
>> `stable模式`：数据从RDBMS到目标时，中间都先走一层Hive存储。

> 一般`quick模式是用来做性能测试`，`生产一般都使用stable模式`，确保`数据链路可追踪`。 

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Hive载数之StorageHandler/1.png)

## 2 流程介绍

### 2.1 quick模式

#### 2.1.1 RDBMS-> Hive

> RDBMS的数据通过`JDBC StorageHandler`直接存储到`Hive`。

#### 2.1.2 RDBMS-> Impala

> RDBMS的数据通过`JDBC StorageHandler`存储到`Hive`后，`刷新Impala元数据`。

#### 2.1.3 RDBMS-> Kudu

> RDBMS的数据通过`JDBC StorageHandler`和K`udu StorageHandler`两个`外联表`，前者连接Hive，后者连接Kudu，然后把前者的数据插入后者。

#### 2.1.478 RDBMS-> HBase

> RDBMS的数据通过`JDBC StorageHandler`和`HBase StorageHandler`两个`外联表`，前者连接Hive，后者连接HBase，然后把前者的数据插入后者。

### 2.2 stable模式

#### 2.2.1 RDBMS-> Hive

> RDBMS的数据通过`JDBC StorageHandler`直接存储到`Hive`。

#### 2.2.2 RDBMS-> Impala

> RDBMS的数据通过`JDBC StorageHandler`存储到`Hive`后，`刷新Impala元数据`。

#### 2.2.3 RDBMS-> Kudu

> RDBMS的数据通过`JDBC StorageHandler`存储到`Hive`后，`刷新Impala元数据`，再通过Impala插入数据到Kudu。

#### 2.2.4 RDBMS-> HBase

> RDBMS的数据通过`JDBC StorageHandler`存储到`Hive`后，通过`HBase StorageHandler`将Hive的数据插入到HBase。

## 3 StorageHandler安装

### 3.1 jar包下载

> 我们去maven仓库搜索`Hive JDBC Handler`、`Hive kudu Handler`、`Hive HBase Handler`，然后把jar包下载下来。

> 将jar包放在hive集群中，这里有两种方式，推荐第一种 

### 3.2 jar包导入

#### 3.2.1 方式一(推荐)

> 将jar拷贝到Hive集群配置的【`Hive 辅助 JAR 目录`】，英文叫【`Hive Auxiliary JARs Directory`】。

> 这样Hive`启动的时候`就会把这个目录下的jar包加载进来。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Hive载数之StorageHandler/2.png)

#### 3.2.2 方式二(不推荐)：

> 拷贝到下面目录。

> CDH是把集群所有jar包放在这里，然后用到得地方做`软连接`，我们也仿照他。

```shell
cd /opt/cloudera/parcels/CDH/jars
拷贝进去
```

> 到下面目录做`软连接`

```shell
cd /opt/cloudera/parcels/CDH/lib/hive/lib
ln -s ../../../jars/hex-hive-jdbc-handler.jar  hex-hive-jdbc-handler.jar
...其他StorageHandler，比如Kudu和HBase
```

### 3.3 其他包导入

> 1 需要导入`RDBMS的数据库驱动包`比如mysql需要导入
>> 比如`RDBMS`是`mysql`的话，导入`mysql-connector-java-x.y.z.jar`
>
>> `RDBMS`是`Oracle`的话，导入`ojdbc8-x.y.z.jar `
>
>> ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Hive载数之StorageHandler/3.png)

> 2 `kudu-StorageHandler`的还得导入`kudu-client-x.y.z.jar`等

> 我目前这个版本需要导入下面几个包，后续版本可能就慢慢融合进去了，不用导这么多
>> ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Hive载数之StorageHandler/4.png)

> 贴个链接，以后用到就不用到处找了  
>> 链接：https://pan.baidu.com/s/1sE-J4J8VRl_dYCGw-5CmUQ 提取码：6666

### 3.4 重启Hive组件

> `重启一下Hive`，这个`StorageHandler`就生效了。

> 随便写个demo测试一下，在Hive中执行下面命令，这个命令用的是公司内部定制的JDBC StorageHandler，大家用的话参考官网那个写法就可以

```shell
CREATE EXTERNAL TABLE hive_db.mysql_to_hive_jdbc_storage_handler_tb
(
    app_code string,
    username string
) 
STORED BY "com.hex.hive.jdbc.JdbcStorageHandler" 
TBLPROPERTIES 
(
    "mapred.jdbc.driver.class" = "com.mysql.jdbc.Driver",
    "mapred.jdbc.url" = "jdbc:mysql://ip:port",
    "mapred.jdbc.username" = "root",
    "mapred.jdbc.password" = "root",
    "mapred.jdbc.input.table.name" = "app_user",
    "mapred.jdbc.output.table.name" = "app_user",
    "mapred.jdbc.hive.lazy.split" = "false"
);
```

> 还有示例在上面3.3那个`网盘链接`里，可以`照着操作操作`，把里面数据库相关地址换成自己的

## 4 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)

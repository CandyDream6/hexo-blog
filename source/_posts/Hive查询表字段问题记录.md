---
title: Hive查询表字段问题记录
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(100).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(100).jpg
tags:
  - 大数据
  - 标签一
category:
  - - 编程
    - 大数据
  - - 编程
    - 大数据1
date: 2021-11-12 13:58:27
---

> 最近在做`Hive数据导出`的功能。

> 在操作过程中，出现了`表字段信息不符`和`字段描述信息不符`的情况。

> 解决之后记录一下。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 业务场景

> 我们要做的是将`Hive中的数据`导出到`文件`或者`DB`中，需要可根据`指定字段`和规则等导出。

> 那么这样第一步就是需要`获取Hive目标表的字段信息`供用户选择。

## 2 事故现场

> 我`查询表字段`使用的是下面语法,结果如图所示

```shell
Describe db.table;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Hive查询表字段问题记录/1.png)

> 可以看到，不光`字段集合有问题`，而且`字段的注释也是中文乱码`的。

## 3 解决方案

### 3.1 字段列表不符问题

> 图中`红色框内`的信息其实不是hive的表字段信息，因为没有找到其他的方式去做这个功能，所以目前依旧是使用`Describe语法`，然后进行`过滤`

> 先过滤掉`字段名不合适`的那几个，`再按字段名去重`，就可以实现需求了，如果其他人有更好的办法，可以推荐以下。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Hive查询表字段问题记录/2.png)

### 3.2 字段描述中文乱码问题

> 这个原因是`Hive的元数据库编码`和`Hive使用JDBC连接的配置编码`问题。

#### 3.2.1 修改元数据库的编码

> 我们`hive的元数据库`使用的是`Mysql`，所以执行以下语句

```shell
# 修改字段注释字符集
alter table COLUMNS_V2 modify column COMMENT varchar(256) character set utf8;
# 修改表注释字符集
alter table TABLE_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;
# 修改分区表参数，以支持分区键能够用中文表示
alter table PARTITION_PARAMS  modify column PARAM_VALUE varchar(4000) character set utf8;
alter table PARTITION_KEYS  modify column PKEY_COMMENT varchar(4000) character set utf8;
#  修改索引注解
alter table  INDEX_PARAMS  modify column PARAM_VALUE  varchar(4000) character set utf8;
```

#### 3.2.2 修改Hive连接JDBC的配置编码

> 修改`hive的配置文件``hive-site.xml`，`CDH`的话配置文件是在`/etc/hive/conf.cloudera.hive`目录下。

> 添加或修改以下配置

```shell
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://host:port/dbname?createDatabaseIfNotExist=true&amp;characterEncoding=UTF-8</value>         
    <description>JDBC connect string for a JDBC metastore</description>
</property>
```

## 4 结果校验

> 我们`新建一个Hive表`，然后`查询一下字段信息`测试一下

```shell
CREATE  TABLE test1
(
    app_code string COMMENT 'appcode注释',
    username string COMMENT 'username注释'
)
PARTITIONED BY (DT string)
STORED AS PARQUET;

Describe test1;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Hive查询表字段问题记录/3.png)

> 可以看到，`字段注释的乱码问题已经解决`。

## 5 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
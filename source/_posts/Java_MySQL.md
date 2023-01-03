---
title: MySQL 存储原理
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(76).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(76).jpg
tags:
  - Java
  - MySQL
category:
  - - 编程
    - MySql
date: 2021-03-24 09:06:54
---

`MySQL` 是最流行的`关系型数据库管理系统`。

在 WEB 应用方面 MySQL 是最好的 `RDBMS`(Relational Database Management System：`关系数据库管理系统`)应用软件之一。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
> 我画的`visio原图`链接，有兴趣的同学可以在这个图上自己改

```
链接：https://pan.baidu.com/s/1BMOk-anyl_XSSPxLtaDcMQ 
提取码：6666 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

## 1 MySQL底层存储

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_MySQL/1.png)

> `页 = 页目录 + 用户数据区域 + 页头`

> 插入的数据存在`用户数据区域`里，插入后会`按照主键排序`。(如果没有主键:MySQL会默认为每条数据生成一个rowId，如果没有主键，那么就按这个rowId排序)

> 每一页会对数据进行`分组`，创建`页目录`

> 会为所有页的页目录再抽取一层形成目录页。重新用一个页记录这个。 

> 就好比 `1`、`1.1`、`1.1.1` 
>> 假如`1.1.1`里存的是`数据`，那么`1.1`就是`页目录`，`1.1.1`就是`目录页`

```
Page是Innodb存储的最基本结构，也是Innodb磁盘管理的最小单位，与数据库相关的所有内容都存储在Page结构里。
每个数据页的大小为16kb。
查询数据库的时候，你查一条数据，他都是把这一页的数据都加载到内存里，而不是你查那一条，他把哪一条加载到内存。
每个Page使用一个32位（一位表示的就是0或1）的int值来表示，正好对应Innodb最大64TB的存储容量(16kb * 2^32=64tib)：64T
```

## 2 索引

### 2.1 主键索引

> 根据主键创建的索引：对上图简化一下，如下

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_MySQL/2.png)

> `根据主键查询的时候会走这个索引`
> 
> `或者根据其他索引查询完后，没有拿到全部数据(查询的字段不在)，只能查到数据的索引，这时候就得来主键索引里查了`

### 2.2 联合索引(辅助索引)

```
create index_tb1_bcd on t1(b,c,d);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_MySQL/3.png)

```
select * from t1 where c=2 and d=3;
```

> `这个查询不会走索引，会走全表扫描`
>> 因为要遵循`最左前缀原则`,联合索引是bcd，查询根据 ?cd 在索引第一步就卡住了，没法比较查询哪个分支。所以只能走全部扫描。

```
select * from t1 where b > 1;
```

> `回表`:根据某个字段去数据里查数据。如果用`联合索引`查询的话，`先根据联合索引查到数据的主键`，然后再`回表`根据主键去`查询数据`。
>
> `这个查询不走索引`,条件 b > 1 ，结果能查出来很多数据的索引，要进行很多次`回表`，所以不如，全部扫描

```
select b from t1 where b > 1;
```

> `覆盖索引。这个会走索引，因为要查询的字段b在联合索引里直接有。`

```
select b from t1;
```

> `这个走联合索引`

> 因为主键索引里，每页存储的数据肯定比辅助索引里每页存储的数据多(主键索引存的是全部数据)。所以走辅助索引查询的页数就少一点。(比如主键索引一共存了4页，而辅助索引里可能只存了3页)

## 3 扯皮

### 3.1 为什么主键建议用自增数字，不用uuid

> 1.uuid占用空间大，到时候页数很多。

> 2.在插入数据的时候，会根据主键进行排序，主键自增的话就自动排好序了。

### 3.2 如果没有表没有主键，那么B+树怎么建立。

> 1.MySQL自动会为每条数据加上一个rowId，当没有主键的时候，会用这个rowId去建索引(B+树)

### 3.3 MySQL中有哪些存储引擎

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_MySQL/4.png)

> `5.5`之前默认`MylSAM`
> `5.5`之后默认`InnoDB`，主要是因为InnoDB`支持事务`，其他不太了解


### 3.4 MySQL中有哪些类型索引

#### 3.4.1 普通索引

> `最基本的索引，它没有任何限制，用于加速查询。`

```
1:建表的时候一起创建
CREATE TABLE mytable ( name VARCHAR(32) , INDEX index_mytable_name (name) );
2:建表后，直接创建索引
CREATE INDEX index_mytable_name ON mytable(name);
3:修改表结构
ALTER TABLE mytable ADD INDEX index_mytable_name (name);
```

#### 3.4.2 主键索引

> `是一种特殊的唯一索引，一个表只能有一个主键，不允许有空值。一般是在建表的时候同时创建主键索引。`

```
1:建表的时候一起创建
CREATE TABLE mytable ( `id` int(11) NOT NULL AUTO_INCREMENT , `name` VARCHAR(32) , PRIMARY KEY (`id`) );
2:修改表结构
ALTER TABLE test.t1 ADD CONSTRAINT t1_pk PRIMARY KEY (id);

注：如果是字符串字段，还可以指定索引的长度，在列命令后面加上索引长度就可以了（例如:name(11)）
```

#### 3.4.3 唯一索引

> `索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。`

```
1:建表的时候一起创建
CREATE TABLE mytable ( `name` VARCHAR(32) , UNIQUE index_unique_mytable_name (`name`) );
2:建表后，直接创建索引
CREATE UNIQUE INDEX index_mytable_name ON mytable(name);
3:修改表结构
ALTER TABLE mytable ADD UNIQUE INDEX index_mytable_name (name);

注：如果是字符串字段，还可以指定索引的长度，在列命令后面加上索引长度就可以了（例如:name(11)）
```

#### 3.4.4 组合索引

> `指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用`。使用组合索引时遵循`最左前缀集合`。

```
1:建表的时候一起创建
CREATE TABLE mytable ( `id` int(11) , `name` VARCHAR(32) , INDEX index_mytable_id_name (`id`,`name`) );
2:建表后，直接创建索引
CREATE INDEX index_mytable_id_name ON mytable(id,name);
3:修改表结构
ALTER TABLE mytable ADD INDEX index_mytable_id_name　(id,name);
```

#### 3.4.5 全文索引

> `主要用来查找文本中的关键字，而不是直接与索引中的值相比较。`

> fulltext索引跟其它索引大不相同，它更像是一个搜索引擎，而不是简单的where语句的参数匹配。

> fulltext索引配合`match against`操作使用，而不是一般的where语句加like。

> 它可以在create table，alter table ，create index使用，不过目前只有char、varchar，text 列上可以创建全文索引。

```
1:建表的时候一起创建
CREATE TABLE `article` ( `id` int(11) NOT NULL AUTO_INCREMENT , `title` char(250) NOT NULL , `contents` text NULL , `create_at` int(10) NULL DEFAULT NULL , PRIMARY KEY (`id`), FULLTEXT (contents) );
2:建表后，直接创建索引
CREATE FULLTEXT INDEX index_article_contents ON article(contents);
3:修改表结构
ALTER TABLE article ADD FULLTEXT INDEX index_article_contents　(contents);
```

### 3.5 MySQL为什么采用B+树而不是B树

> 两个数都是特殊的二叉搜索树。一个节点可以放多个数据。

> `B树与B+树的最大区别就是`:
>> `B树可以在非叶结点中存储数据`
> 
>> `B+树只能在叶子节点存数据,其他节点存的都是索引`
>
>> `B+树的叶子节点之间有指针相连`

> 假如查询 6 < id < 9 这种区间的话
>> B树：如果根节点是8，那么需要先遍历8左边的子树去找数据，然后遍历8右边的子树去找
>
>> B+ ：遍历左边的树，然后根据找到的数据的指针直接找其他数据即可，不用遍历右边的树

## 4 SQL

> 一个表里有10亿条数据，给你10000个主键id，把这一万条数据查出来。

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)

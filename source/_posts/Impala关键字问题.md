---
title: Impala关键字问题
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(108).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(108).jpg
tags:
  - 大数据
  - 标签一
category:
  - - 编程
    - 大数据
  - - 编程
    - 大数据1
date: 2021-12-03 11:11:05
---

> 最近在操作hive和impala和过程中，`出现了impala关键字的问题`，太恶心了。

> 在用`hue等书写sql和impala-shell查询`的时候，情况还不一样，恶心心，记录一下。 

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 场景

> 我们是需要`从RDBMS载数到Impala`中，采用的方式是，通过`JDBCStorageHandler`，将数据装载到`Hive`中，然后刷新`Impala`指定表元数据，这样就可以在`Impala`中查询了。

## 2 Sql层面解决

> 首先我在Hive中建立了`一张和RDBMS数据结构一样的表`，然后`载数到Hive成功`，如下

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Impala关键字问题/1.png)

> 而我在`Impala查询`的时候，发现`desc是关键字`，`impala报错`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Impala关键字问题/2.png)

> 解决方案是对`关键字字段`加**``**

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Impala关键字问题/3.png)

## 3 Impala-shell层面解决

> 在`impala-shell`中，对于关键字，**``**`加或者不加都会报错`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Impala关键字问题/4.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Impala关键字问题/5.png)

> `正确的解决方案是`加上 **\\`**

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Impala关键字问题/6.png)

## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`
__
## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
---
title: DataGrip连接不到数据库，但Navicat可以连接
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(118).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(118).jpg
tags:
  - 大数据
  - 数据库
category:
  - - 编程
    - 数据库
  - - 编程
    - DataGrip
date: 2022-03-03 11:22:12
---

最近在用`DataGrip`连接公司的一个`MYSQL数据库`时，发现`连不上`，同事很多用`navicat`说是可以连接。

自己测了一下也是，这就比较`蛋疼`了，解决了记录一下。

我还是一如既往的`摒弃Navicat`，用D`ataGrip`不香么，`做出改变，与时俱进，突破自我`(自从当年`小阳哥`推荐我用`Datagrip`之后，到现在为止，`两年了`，我`始终直呼内行`，`吃水不忘挖井人，感谢小杨哥`，`respect！`)。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 报错提示

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/DataGrip连接不到数据库_但Navicat可以连接/1.png)

## 2 解决方案

> 是`SSL`的问题，把`以下两个配置`改为`false`就好了

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/DataGrip连接不到数据库_但Navicat可以连接/2.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/DataGrip连接不到数据库_但Navicat可以连接/3.png)


## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
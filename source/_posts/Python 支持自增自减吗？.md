---
title: Python 支持自增自减吗？
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(23).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(23).jpg
tags:
  - AI
  - Python
category:
  - - 编程
    - Python
 
date: 2018-12-19 21:35:38
---

python中到底支不支持自增(++)自减(--)操作呢？

例如：i++,++i。

如果不可以，下面代码为什么不报错'a++ + ++a'

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 历史

python到底支不支持自增自减呢

![python下标](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/pythonPlusPlus/pythonPlusPlus.png)

看上图，python是不支持自增自减的。

至于为什么要去掉这个操作，我们可以看下[Stack Overflow上的一个回答](https://stackoverflow.com/questions/3654830/why-are-there-no-and-operators-in-python)

![为什么不要下标](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/pythonPlusPlus/pythonPlusPlus1.png)


大致意思就是说有了for-in循环，很少会用到++操作了，而且++操作需要在语言中添加操作码，是完全没必要的

## 疑惑

既然python不支持自增自减操作，那么下图代码为什么可以运行呢

![疑惑](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/pythonPlusPlus/pythonPlusPlus2.png)

[我们来看下google研发总监的观点](http://norvig.com/python-iaq.html)

![google研发总监观点](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/pythonPlusPlus/pythonPlusPlus3.png)


这样就一目了然了，哈哈哈，太有趣了

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
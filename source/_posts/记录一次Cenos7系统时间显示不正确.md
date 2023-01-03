---
title: 记录一次Cenos7系统时间显示不正确
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(25).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(25).jpg
tags:
  - Cenos7
  - Linux
category:
  - - 编程
    - Linux
 
date: 2019-01-18 20:26:17
---

前几天刚装的虚拟机，配置好网络什么的之后就没有管了。

今天闲了打开一看，系统时间不合适。

去网上查了查，是时区设置问题。

记录一下，就自己看，大佬别喷。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1.安装ntpdate

```
yum install -y ntpdate
```

## 2.同步时间

```
ntpdate us.pool.ntp.org
```

## 3.查看时间

```
date
```

## 4.问题修复

如果上述操作没有解决问题，那么就覆盖下时区文件，再查看就OK了

```
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## 5.完工

以上内容也只是做了一下网上的搬运工，方便我自己以后查看。好了，吃饭走！走着！！

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
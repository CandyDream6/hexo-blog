---
title: 记录一次Cenos7虚拟机ping不到主机和外网
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(18).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(18).jpg
tags:
  - VMware
  - Cenos7
  - Linux
category:
  - - 编程
    - Linux
 
date: 2018-11-08 10:57:20
---

今天想打开虚拟机玩玩。

但是就是ping不到主机和外网，主机可以ping到虚拟机。

以前都好好的，于是看了ip、网卡、DNS，都没毛病。

最后........

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1.事故现场

可把人头皮搞麻，就怕这种本来好好的，突然不合适。先看下【事故现场】

![事故现场](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Cenos7CannotPingNet/Cenos7CannotPingNet1.png)

## 2.事故责任

原来是自己手欠，在火绒的设置里关闭了 **VMware** 的 **NET** 服务，如下图

![事故责任](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Cenos7CannotPingNet/Cenos7CannotPingNet2.png)

## 3.事故处理

1. 解决方案就是火绒里打开这个上图这个启动，这样以后每次开机都不用管了

2. win + r 输入 services.msc，启动 VMware NAT Service

![事故处理](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Cenos7CannotPingNet/Cenos7CannotPingNet3.png)

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
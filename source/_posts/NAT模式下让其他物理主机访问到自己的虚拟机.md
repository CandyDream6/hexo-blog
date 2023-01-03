---
title: NAT模式下让其他物理主机访问到自己的虚拟机
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(103).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(103).jpg
tags:
  - 虚拟机
  - 服务
category:
  - - 虚拟机
    - 服务
date: 2021-11-23 09:22:38
---

> 平时工作学习中，经常需要`本地启动虚拟机`来进行学习或者开发测试。

> 这时候同事需要连接我`虚拟机里的Mysql`，这里记录一下如何去做。`让其他物理主机访问到自己的虚拟机`.

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 环境准备

> 我的虚拟机的地址是`192.168.88.56`

> 我在虚拟机装了个`MySQL`，端口是`3306`，那么现在需要设置一个`主机到虚拟机的端口映射`，这样同事就可以访问到了。

## 2 网络配置

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/NAT模式下让其他物理主机访问到自己的虚拟机/1.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/NAT模式下让其他物理主机访问到自己的虚拟机/2.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/NAT模式下让其他物理主机访问到自己的虚拟机/3.png)

## 3 结果校验

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/NAT模式下让其他物理主机访问到自己的虚拟机/4.png)

## 4 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
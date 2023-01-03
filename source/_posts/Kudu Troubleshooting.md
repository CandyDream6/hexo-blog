---
title: Kudu Troubleshooting 
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(107).jpg 
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(107).jpg 
tags:
  - 大数据
  - Kudu 
category:
  - - 编程
    - 大数据
  - - 编程
    - Kudu 
date: 2021-12-02 11:00:30
---

> 以前`记录问题比较松散`，多了之后，想`汇总起来`。

> 这篇主要用来汇总平时在`使用Kudu过程中`碰到的一些问题和解决思路。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 kudu主节点tsServer启动失败

> `报错`

```shell
Check failed: _s.ok() Bad status: IO error: Failed to load FS layout: Could not lock /kudu/fs_data/data/block_manager_instance: Could not lock /kudu/fs_data/data/block_manager_instance: lock /kudu/fs_data/data/block_manager_instance: Resource temporarily unavailable (error 11)
```

```shell
Check failed: _s.ok() Bad status: Network error: error binding socket to 0.0.0.0:7050: Address already in use (error 98)
```

> `解决方案`：杀死占用的端口进程，然后在CM界面重启启动失败的TsServer，或者重启整个Kudu

```shell
fuser -k 7050/tcp
```

## 2 kudu日志暴增，导致磁盘每天都会爆满

> kudu日志激增，排查发现/var/log/kudu目录下生成了很多大文件

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/KuduTroubleshooting/1.png)

> `解决方案`：调整kudu日志级别，将master和tsServer的日志级别由0改为2

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/KuduTroubleshooting/2.png)

## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路

![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
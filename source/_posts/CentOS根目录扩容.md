---
title: CentOS根目录扩容
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(114).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(114).jpg
tags:
  - 大数据
  - CentOS
category:
  - - 编程
    - CentOS
  - - 编程
    - CentOS
date: 2022-01-20 18:14:32
---

最近`搭建CDP`，申请的`服务器磁盘没有挂载`，记录一下把未挂载的`散修磁盘``挂载到根目录`的方法。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 查看本机磁盘容量

```shell
df -h

lsblk
```

> 可以看到本机根目录`37G`，已使用`5%`。

> 有一块名为`sda`的磁盘容量为`200G`，没有`MOUNTPOINT(挂载点)`。所以我们计划把这200G挂载到根目录下。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CentOS根目录扩容/1.png)

## 2 添加磁盘分区

```shell
fdisk /dev/sda

依次输入 n p 回车 回车 回车 w
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CentOS根目录扩容/2.png)

> 看分区是否创建
>> 可以看到，分区`sda1`已经创建

```shell
lsblk
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CentOS根目录扩容/3.png)

## 3 开始扩容

### 3.1 创建物理卷

```shell
lvm

pvcreate /dev/sda1
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CentOS根目录扩容/4.png)

### 3.2 查看物理卷和卷组

```shell
pvdisplay
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CentOS根目录扩容/5.png)

### 3.3 将物理卷加入到卷组

```shell
vgextend centos /dev/sda1

pvdisplay
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CentOS根目录扩容/6.png)

### 3.4 将卷组空间添加到逻辑卷/dev/centos/root :

```shell
lvextend -l +100%FREE /dev/centos/root
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CentOS根目录扩容/7.png)

### 3.5 同步到文件系统

> 之前只是对逻辑卷扩容，还要`同步到文件系统`，实现对根目录的扩容

```shell
xfs_growfs /dev/centos/root
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CentOS根目录扩容/8.png)

## 4 成果查看

```shell
df -h
```

> 可以看到，`磁盘由原来的40G变成了240G`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CentOS根目录扩容/9.png)

## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

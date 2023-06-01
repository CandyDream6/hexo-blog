---
title: shell脚本:$'\r':command not found的解决方法
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(98).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(98).jpg
tags:
  - 大数据
  - Shell
category:
  - - 编程
    - Shell
date: 2021-10-18 09:13:56
---

最近想写一些常用的shell脚本，是在windows编写然后传到linux的。

然后碰到了`init.sh: line 1: $'\r': command not found 1`。

记录一下解决方案。

在Linux系统中，运行Shell脚本，出现了如下错误： one-more.sh: line 1: $'\r': command not found 1 出现这样的错误，是因为Shell脚本在Windows系统编写时，每行结尾是\r\n，而在Linux系统中行每行结尾是\n，所以在Linux系统中运行脚本时，会认为\r是一个字符，导致运行错误。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 事故现场

在Linux系统中，运行Shell脚本，出现了如下错误：

```shell
init.sh: line 1: $'\r': command not found 1
```

## 2 事故分析

> 出现这样的错误，是因为Shell脚本在Windows系统编写时，每行结尾是\r\n， 而在Linux系统中行每行结尾是\n，所以在Linux系统中运行脚本时，会认为\r是一个字符，导致运行错误。

## 3 事故处理

> 去除Shell脚本的\r字符

### 3.1 方案一

```shell
sed -i 's/\r//' init.sh
```

### 3.2 方案二

```shell
yum install -y dos2unix
```

```shell
dos2unix init.sh
```

## 8 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

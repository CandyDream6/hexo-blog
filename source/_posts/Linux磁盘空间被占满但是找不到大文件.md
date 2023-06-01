---
title: Linux磁盘空间被占满但是找不到大文件
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(109).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(109).jpg
tags:
  - 大数据
  - Linux
category:
  - - 编程
    - 大数据
  - - 编程
    - Linux
date: 2021-12-07 09:29:27
---

最近有个问题`恼火`的很，服务器`磁盘`动不动就`爆满`，`清理`了之后，`几个小时`也就差不多爆满了。

搞了半天，`发现了问题`，记录一下。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 事故现场

> 发现`磁盘空间发现占用接近100%`了

```shell
df -h 
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Linux磁盘空间被占满但是找不到大文件/1.png)

## 2 排查过程

> 但是逐级排查`没有发现大日志文件`，该清理的都已经清理了。
>> {% post_link 记一次CDH磁盘不足排查 点击跳转到`记一次CDH磁盘不足排查` %}。 

> 最后在网上查了一番后才`发现有这么一个说法`
>> 当我们`使用rm在linux上删除了大文件`，`但是如果有进程打开了这个大文件`，`却没有关闭这个文件的句柄`，`那么linux内核还是不会释放这个文件的磁盘空间`。

## 3 解决方案

> 查看已删除文件但是进程没有释放该文件的进程

```shell
lsof -n | grep deleted
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Linux磁盘空间被占满但是找不到大文件/2.png)

> 可使用下面命令杀死大部分上述进程，剩下的一些不知道怎么的删不了，自己通过kill -9 一个一个删除

```shell
lsof -n | grep 'deleted' | grep -v grep | awk '{print $2}'  | xargs kill -s SIGINT
```

```shell
kill -9 20440
```

## 4 结果查看 

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Linux磁盘空间被占满但是找不到大文件/3.png)

## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

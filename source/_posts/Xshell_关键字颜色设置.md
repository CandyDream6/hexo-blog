---
title: Xshell6 绿色版下载及关键字颜色设置
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(47).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(47).jpg
tags: 
  - 软件
  - windows
  - Xshell6
  - 关键字颜色
category:
  - - 软件
 
date: 2019-11-19 16:53:03
---

作为开发人员，经常用到的一个软件是`Xshell`。

在平时操作或者看日志的过程中，满屏的`黑纸白字`看起来总有一点`不太清晰`。

今天我们设置一下里面的`关键字颜色`，这样`开发和排错效率`就会提升一截，`使用舒适度`也会提高很多。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 效果图

> 先放个`效果图`，这就是本文的`最终效果`，我是在命令行打的，在日志中显示也是一样

![图片](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Xshell_关键字颜色设置/1.png)

## 2 Xshell6 Plus 绿色版安装

> 软件下载

```
链接：https://pan.baidu.com/s/1HpJubLEZo9dduyqU15CgAQ 
提取码：6666 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

> 软件激活 

```
安装教程:下载后解压，运行绿化.bat ，绿化后直接使用即可
如遇到缺少dll文件，可以下载wrcyyxk_v2020.09.11.exe进行安装
```

## 3 修改关键字颜色

> 设置方法

![图片](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Xshell_关键字颜色设置/2.png)

> 用到的正则表达式

```
# 正确规则 
[^A-Za-z_&-](accepted|allowed|enabled|connected|successfully|成功|正确|successful|succeeded|success)[^A-Za-z_-]|[=>"':.,;({\[][ ]*(true|yes|ok)[ ]*[]=>"':.,;)} ]

# 错误规则 
[^A-Za-z_&-]((bad|wrong|incorrect|improper|invalid|unsupported|bad)( file| memory)? (descriptor|alloc(ation)?|addr(ess)?|owner(ship)?|arg(ument)?|param(eter)?|setting|length|filename)|not properly|improperly|(operation |connection |authentication |access |permission )?(denied|disallowed|not allowed|refused|problem|failed|failure|not permitted)|no [A-Za-z]+( [A-Za-z]+)? found|invalid|unsupported|not supported|seg(mentation )?fault|错误|corruption|corrupted|corrupt|overflow|underrun|not ok|unimplemented|unsuccessfull|not implemented|errors?|\(ee\)|\(ni\))[^A-Za-z_-]|[=>"':.,;({\[][ ]*(false|no|ko)[ ]*[]=>"':.,;)} ]

# 警告规则 
[^A-Za-z_&-](\[\-w[A-Za-z-]+\]|caught signal [0-9]+|警告|cannot|(connection (to (remote host|[a-z0-9.]+) )?)?(closed|terminated|stopped|not responding)|exited|no more [A-Za-z] available|unexpected|(command |binary |file )?not found|(o)+ps|out of (space|memory)|low (memory|disk)|unknown|disabled|disconnected|deprecated|refused|disconnect(ion)?|attention|warnings?|exclamation|alerts?|\(ww\)|\(\?\?\)|could not|unable to)[^A-Za-z_-]

# 主机规则 
[^0-9A-Za-z_&-](localhost|([1-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-4])\.[0-9]+\.[0-9]+\.[0-9]+|null|none)[^0-9A-Za-z_-]

# 信息规则 
[^A-Za-z_&-](last (failed )?login:|launching|checking|loading|creating|building|important|booting|starting|notice|informational|informations?|info|信息|note|\(ii\)|\(\!\!\))[^A-Za-z_-]

```

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
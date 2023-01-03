---
title: windows10睡眠与休眠
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(24).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(24).jpg
tags:
  - windows
  - 休眠
  - 睡眠
category:
  - [博客, 技术] 
 
date: 2019-01-10 18:06:41
---

在日常工作生活中，我们经常碰到下面这样一个场景：

下班需要带电脑回家，一部分人选择电脑息屏待机，一部分人选择关机，回家后再打开。

后者面临的是需要重新打开所有需要的软件，就很麻烦。

今天就和大家分享一下windows的**休眠**功能(感谢我刚哥指导)。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 开篇

windows在关机处有下图四个选项，有些小同志只有三个选项(没有**休眠**，后面教大家设置)


主要介绍下**睡眠**和**休眠**的区别。

> 睡眠：这种模式，系统会将正在处理的所有数据保存到内存中，除内存以外的所有设备都停止供电。存在的风险就是，如果发生断电或者其他原因导致电脑关机，那么所有数据都会丢失，如果工作没有保存，那么芜湖~~~~起飞！！！！

> 休眠：这种模式，系统会将内存中的所有数据存储到硬盘里，这样的话，就算你关机，下次开机后，你的电脑还是刚才的工作状态，也就是所有以前启动的应用都在，所有你编辑的东西，一字不差的还原回来了。就是即使完全断电，工作状态也可以还原。

```
从上面来看，休眠可能对我来说是一种更好的选择，我再也不用每次回家都关机，然后再一个个打开一堆软件了。
```

## 设置休眠

上面讲到有的小同志在关机选项这里没有**休眠**选项，下面教大家如何设置出来。

![休眠](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/win10XiuMian/win10XiuMian1.png)

打开控制面板

```
win + r ，输入control(或者你知道的任何一种打开控制面板的方式打开它)
```

![打开控制面板](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/win10XiuMian/win10XiuMian2.png)

打开电源选项

![打开电源选项](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/win10XiuMian/win10XiuMian3.png)

开启休眠

![开启休眠](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/win10XiuMian/win10XiuMian4.png)

这样的话，就可以在关机那儿看到休眠了。但是一部分小同志这里没有**休眠**这个选项，我们不要慌，走着

右击屏幕左下角win图标，打开Windows PowerShell(管理员)

![打开Windows PowerShell](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/win10XiuMian/win10XiuMian5.png)

运行下面命令

```
powercfg -h on
```

这时候再打开上面的电源选项，就可以看到休眠选项了。

## 完工

齐活，果断睡眠，下班，回家走起~~~~~~~~~~~

特此感谢我刚哥指导！！！


## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)









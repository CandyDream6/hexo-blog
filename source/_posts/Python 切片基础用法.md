---
title: Python 切片基础用法
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(22).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(22).jpg
tags:
  - AI
  - Python
category:
  - - 编程
    - Python
 
date: 2018-12-14 11:02:27
---

今天略微看了下python中的切片，

瞬间觉得，这也太强大了吧，

爱了爱了。

看的过程中稍微有一点点绕(也不算绕)，

记录一下，自己以后看。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 下标

python中支持下标，同时也支持反向下标，

不管是**字符串**，还是**列表**或**元组**，都符合这个特征，

如下图

![python下标](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/pythonSlice/pythonSlice.png)

## 切片

> 切片是指对操作的对象截取其中一部分的操作。字符串、列表、元组都支持切片操作。

**切片的语法：[start_index:end_index:step]**

> 步长(step)：正负数均可，其绝对值大小决定了切取数据时的‘‘步长”，而正负号决定了“切取方向”，正表示“从左往右”取值，负表示“从右往左”取值。当step省略时，默认为1，即从左往右以步长1取值。**切取方向非常重要！切取方向非常重要！切取方向非常重要，重要的事情说三遍！**



>start_index：表示起始索引（包含该索引对应值）；该参数省略时，表示从对象“**端点**”开始取值，至于是从“**起点**”还是从“**终点**”开始，则由step参数的正负决定，step为正从“**起点**”开始，为负从“**终点**”开始。


>end_index：表示终止索引（不包含该索引对应值）；该参数省略时，表示一直取到数据“**端点**”，至于是到“**起点**”还是到“**终点**”，同样由step参数的正负决定，step为正时直到“**终点**”，为负时直到“**起点**”



### 1.切割字符串

![python下标](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/pythonSlice/pythonSlice1.png)

### 2.切割列表

对于列表和元素也可以用这样的方式进行切割，下面用列表举个例子

![python下标](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/pythonSlice/pythonSlice2.png)

### 3.字符串反转

![python下标](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/pythonSlice/pythonSlice3.png)

还有很多比较有趣的用法，以后遇到了再慢慢补充吧。

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
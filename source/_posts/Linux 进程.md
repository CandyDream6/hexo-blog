---
title: Linux 进程
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(31).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(31).jpg
tags:
  - Cenos7
  - Linux
category:
  - - 编程
    - Linux
 
date: 2019-04-18 21:57:03
---

在操作linux的过程中，我认为进程是极其极其重要的，今天初步了解了一下有关的知识点，记录一下，以后加深理解了再慢慢修补吧。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 基本介绍

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJinCheng/linuxJinCheng1.png)

## 2 查看进程

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJinCheng/linuxJinCheng2.png)

---

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJinCheng/linuxJinCheng3.png)

---

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJinCheng/linuxJinCheng4.png)

> 查看sshd进程，就是登录服务器的进程

```
ps -aux | grep sshd
```

> 查看java进程

```
jps
```

## 3 查看父进程

```
ps -ef | more

```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJinCheng/linuxJinCheng5.png)

## 4 终止进程

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJinCheng/linuxJinCheng6.png)

1. 案例1：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJinCheng/linuxJinCheng7.png)

2. 案例2：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJinCheng/linuxJinCheng8.png)

3. 案例3：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJinCheng/linuxJinCheng9.png)

4. 案例4：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJinCheng/linuxJinCheng10.png)

## 5 查看进程树

```
 yum -y install psmisc
```


## 6 查看进程输出标题

```
$ ps aux | head -1; ps aux | grep mysql

$ ps aux | grep -E 'USER|nginx'
$ ps aux | grep -E 'PID|nginx'
$ ps aux | egrep 'USER|nginx'
$ ps aux | egrep 'PID|nginx'

这里的USER和PID可以是表头的任意一个字段
这里用了-E去正则匹配，因为表头可以匹配到，所以就展示了
```

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
---
title: Centos7 配置阿里云yum源
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(26).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(26).jpg
tags:
  - Cenos7
  - Linux
category:
  - - 编程
    - Linux
 
date: 2019-02-09 21:45:58
---

CenOS7 默认拉取国外镜像源，种种原因速度特别慢，我们可以配置为阿里数据源，让他访问快一点点。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1.备份原有镜像源

```
mkdir /etc/yum.repos.d/back
mv /etc/yum.repos.d/* /etc/yum.repos.d/back
```

上面提示back不能移动到back不用管，往下走。

## 2.拉取阿里云的镜像源

```
yum install wget
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

## 3.重新设置缓存

```
yum clean all 
yum makecache 
yum repolist
yum update
```

## 4.完工

好了，这下拉取镜像就能快一点点了。

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
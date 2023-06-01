---
title: Cenos7 虚拟机网络配置 (让虚拟机访问因特网)
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(11).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(11).jpg
tags:
  - 软件
  - 安装
  - Linux
  - Cenos7
  - VMware
category:
  - - 编程
    - Linux
 
date: 2018-10-23 02:58:16
---

{% post_link Cenos7_安装_(超级超级详细的一篇教程) %}上一篇我们介绍了如何安装Cenos虚拟机，这篇介绍下虚拟机网络设置，可以让虚拟机连接利用主机的网络进行因特网的访问。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1.网络模式选择

> 先看下vmware的虚拟网卡VMnet8的网关IP

![网络模式选择](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Cenos7NetWorkSetting/CenosNetWorkSetting0.png)

> 确认虚拟机选择的是 **NAT** 网络模式

![网络模式选择](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Cenos7NetWorkSetting/CenosNetWorkSetting1.png)

## 2.设置虚拟网卡

> 把默认网关和IP地址设置在同一个网段内，还有DNS和网关设置成一样的就可以

![设置虚拟网卡](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Cenos7NetWorkSetting/CenosNetWorkSetting2.png)

## 3.虚拟机网络配置

> 注意的都在图里标了，大家自己看

编辑这个文件，有的系统叫ens33，差不多都是和ens33有关的这个文件
```
/etc/sysconfig/network-scripts/ifcfg-ens33
```

![虚拟机网络配置](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Cenos7NetWorkSetting/CenosNetWorkSetting3.png)

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
IPADDR=192.168.199.55
GATEWAY=192.168.199.2
DNS1=192.168.199.2
NAME=ens33
UUID=59738a3e-a489-43e0-8284-248329bd3c8f
DEVICE=ens33
ONBOOT=yes
```

## 4.重启网卡服务

> 修改了配置之后，只有重启网卡服务才可以生效

```
centos6: service network restart
centos7: systemctl restart network
```

检测一下，如果如下图，就表示成功了。

![网络检测](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Cenos7NetWorkSetting/CenosNetWorkSetting4.png)

## 5.安装**网络工具**和**vim**编辑

> 现在系统还是不可以使用 **ifconfig** 和 **vim** 命令的，我们依次执行下面三个命令安装下。

```
yum update
yum install net-tools
yum install vim
```

## 6.完工

到目前为止，我们有关Cenos7虚拟机的安装就全部结束了，祝大家学习、生活、工作愉快！


## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
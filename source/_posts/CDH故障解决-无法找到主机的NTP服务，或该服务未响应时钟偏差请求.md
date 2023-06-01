---
title: CDH故障解决:无法找到主机的NTP服务，或该服务未响应时钟偏差请求
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(106).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(106).jpg
tags:
  - 大数据
  - CDH
category:
  - - 编程
    - 大数据
  - - 编程
    - CDH
date: 2021-12-01 22:50:02
---

> 最近CDH由于`磁盘爆满`原因，导致`整个CDH集群`直接挂掉了。

> 在`清理完磁盘`，然后`重启CDH集群`后。出现了【`无法找到主机的NTP服务，或该服务未响应时钟偏差请求`】问题.

> 解决完之后，记录一下这个过程。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 事故现场

> 日常工作中，`CDH集群`有可能意外宕机，原因可能是`机房断电或者其他场景`。那么在`重新启动集群后`，有可能出现一些问题。

> 我就是在`CDH集群重启`之后，碰到了【`无法找到主机的NTP服务，或该服务未响应时钟偏差请求`】，导致`几乎整个集群全部爆红`。

## 2 解决方案

> 出现这个问题，有可能是`时钟同步问题`，也有可能是`cm-agent`或者`cm-server`的问题

### 2.1 检查时钟同步

> 1 `检查每台机器时钟同步是否开启`

```shell
systemctl status ntpd
systemctl is-enabled ntpd     #查看是否在开机启动项
systemctl  start  ntpd        #启动ntpd服务
systemctl enable  ntpd        #将ntpd添加到开机启动项
```

> 2 `查看NTP服务器主机是否正确`（查看该主机使用的ntp-server情况和同步延时等信息；）

```shell
ntpq -p
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CDH故障解决-无法找到主机的NTP服务，或该服务未响应时钟偏差请求/1.png)

> 3 `查看几台服务器时间是否一致`（查看ntpd服务或chronyd服务是否生效。只有centos7、redhat7、ubuntu14等以上（含）才有该命令；）

```shell
timedatectl
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CDH故障解决-无法找到主机的NTP服务，或该服务未响应时钟偏差请求/2.png)

> `以上操作都进行完成之后，我以为好了，结果发现CM里问题还是同样存在，没有任何变化`

### 2.2 尝试重启CM

> `各个节点重启cm-agent`

```shell
systemctl restart cloudera-scm-agent
systemctl status cloudera-scm-agent
```

> `主节点重启cm-server`

```shell
systemctl restart cloudera-scm-server.service
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```

## 3 结果

> 一开始我一直在看NTP的问题，看来看去`NTP是没问题的`，可纳闷了，后来`重启cm-agent和cm-server`后问题得到解决

> 可能由于集群故障，`CDH无法接收到NTP的时间同步`，而`不是NTP本身的问题`。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CDH故障解决-无法找到主机的NTP服务，或该服务未响应时钟偏差请求/3.png)

## 4 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
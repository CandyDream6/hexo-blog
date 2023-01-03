---
title: CDH集群迁移后的配置修改
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(104).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(104).jpg
tags:
  - 大数据
  - CDH
category:
  - - 编程
    - 大数据
  - - 编程
    - CDH
date: 2021-11-30 18:35:35
---

最近需要公司需要像客户`演示`我们的`产品`，目前内部是使用`项目组自己的CDH`。

为了`保证演示环境的稳定性`，我们决定将现有的`CDH集群`完全拷贝一份到`新的服务器`上。

因为`整个虚拟机`都是拷贝过去的，所以需要对一些`配置进行修改`，下面记录一下。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 前言

> `假设集群一共三台服务器，ip分别如下：`

```shell
192.168.99.111
192.168.99.222
192.168.99.333
```

## 2 配置修改

### 2.1 修改hosts文件

> `所有节点修改`

```shell
vim /etc/hosts
```

```shell
192.168.99.111 node1.company.cn
192.168.99.222 node2.company.cn
192.168.99.333 node3.company.cn
```

> `分发到每个节点`

### 2.2 修改ClouderaManager的数据库

> `我用的是MySQL数据`

> `修改HOSTS表`

```shell
update HOSTS SET IP_ADDRESS = '192.168.99.111' WHERE NAME='node1.company.cn';
update HOSTS SET IP_ADDRESS = '192.168.99.222' WHERE NAME='node2.company.cn';
update HOSTS SET IP_ADDRESS = '192.168.99.333' WHERE NAME='node3.company.cn';
```

### 2.3 修改ClouderaManagerAgent地址

> `所有节点修改`

```shell
vim /etc/cloudera-scm-agent/config.ini
```

```shell
server_host=192.168.99.111
```

### 2.4 重启cm-agent

> `主节点执行`

```shell
systemctl restart cloudera-scm-agent
```

### 2.5 重启cm-server

> `主节点执行`

```shell
systemctl start cloudera-scm-server.service
```

## 3 结果验收

> `进入CM管理界面，重启CDH集群和cloudera management Service服务`

```shell
192.168.99.111:7180
```

## 4 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)

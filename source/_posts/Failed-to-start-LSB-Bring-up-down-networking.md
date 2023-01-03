---
title: Failed_to_start_LSB_Bring_up/down_networking
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(110).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(110).jpg
tags:
  - 大数据
  - 标签一
category:
  - - 编程
    - 大数据
  - - 编程
    - 大数据1
date: 2021-12-10 09:40:40
---

> 今天`启动虚拟机`的时候，发现`网络不通`了。

> 查看了下，`ip和网卡等的配置都是合适`的。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 事故现场

```shell
[root@node3 ~]# systemctl restart network
Job for network.service failed because the control process exited with error code. See "systemctl status network.service" and "journalctl -xe" for details.
```

```shell
[root@node3 ~]# systemctl status network
● network.service - LSB: Bring up/down networking
   Loaded: loaded (/etc/rc.d/init.d/network; bad; vendor preset: disabled)
   Active: failed (Result: exit-code) since 四 2020-09-10 08:39:43 CST; 6min ago
     Docs: man:systemd-sysv-generator(8)
  Process: 7314 ExecStart=/etc/rc.d/init.d/network start (code=exited, status=1/FAILURE)

9月 10 08:39:43 test network[7314]: RTNETLINK answers: File exists
9月 10 08:39:43 test network[7314]: RTNETLINK answers: File exists
9月 10 08:39:43 test network[7314]: RTNETLINK answers: File exists
9月 10 08:39:43 test network[7314]: RTNETLINK answers: File exists
9月 10 08:39:43 test network[7314]: RTNETLINK answers: File exists
9月 10 08:39:43 test network[7314]: RTNETLINK answers: File exists
9月 10 08:39:43 test systemd[1]: network.service: control process exited, code=exited status=1
9月 10 08:39:43 test systemd[1]: Failed to start LSB: Bring up/down networking.
9月 10 08:39:43 test systemd[1]: Unit network.service entered failed state.
9月 10 08:39:43 test systemd[1]: network.service failed.
```

> 关键信息

```shell
Failed to start LSB: Bring up/down networking.
```

## 2 解决方案

> `我是方案一解决问题的`。

### 2.1 方案一：禁用NetworkManager

```shell
systemctl stop NetworkManager
systemctl disable NetworkManager
reboot
```

### 2.2 方案二：修改mac地址

> `ip addr show`查看当前`mac地址`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Failed-to-start-LSB-Bring-up-down-networking/1.png)

> 修改/etc/sysconfig/network-scripts/下以ifcfg开头的网络链接文件

> `添加或修改下面语句`

```shell
HWADDR="00:50:56:3a:f9:77"
```

## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)

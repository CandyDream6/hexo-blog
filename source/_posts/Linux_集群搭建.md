---
title: Linux 集群搭建
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(32).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(32).jpg
tags:
  - Cenos7
  - 大数据
category:
  - - 编程
    - 大数据
 
date: 2019-05-05 14:30:55
---

在学习linux的过程中，难免需要搭建个集群玩，今天就记录下搭建linux集群最基础的一些设置，当然，需要你先安装好一个linux虚拟机，可以查看{% post_link Cenos7_安装_(超级超级详细的一篇教程) %} 和 {% post_link Cenos7_虚拟机网络配置_(让虚拟机访问因特网) %}。接下来，我们开始吧！

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 克隆虚拟机

![克隆虚拟机1](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/1.png)

![克隆虚拟机2](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/2.png)

![克隆虚拟机3](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/3.png)

![克隆虚拟机4](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/4.png)

![克隆虚拟机5](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/5.png)

## 2 配置MAC地址

![配置MAC地址](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/6.png)

## 3 配置ip

```shell
vim /etc/sysconfig/network-scripts/ifcfg-ens33
```

![配置ip](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/7配置ip.png)

重启网卡

```
systemctl restart network
```

## 4 配置主机名

```
vim /etc/hostname
```

![配置主机名](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/8主机名配置.png)

## 5 配置域名映射

修改下面文件，不要修改原文件内容，添加图中所示内容即可(每个虚拟机配置了两个别名)
```
vim /etc/hosts
```

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/9配置域名映射.png)

## 6 关闭防火墙和Selinux

### 6.1 关闭防火墙

```shell
systemctl start firewalld.service         # 开启防火墙
systemctl stop firewalld.service          # 关闭防火墙
systemctl status firewalld.service        # 查看防火墙状态
systemctl enable firewalld.service        # 设置开机启动
systemctl disable firewalld.service       # 禁用开机启动
```

```shell
yum -y install iptables                      # 安装iptables
yum -y install iptables-services             # 安装iptables-services
systemctl start iptables.service          # 开启防火墙
systemctl stop iptables.service           # 关闭防火墙
systemctl status iptables.service         # 查看防火墙状态
systemctl enable iptables.service         # 设置开机启动
systemctl disable iptables.service        # 禁用开机启动
```

![关闭防火墙](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/10关闭防火墙.png)

### 6.2 关闭SELinux

#### 6.2.1 什么是SELinux ?

> 1. SELinux是Linux的一种安全子系统<br/>
> 2. Linux中的权限管理是针对于文件的, 而不是针对进程的, 也就是说, 如果root启动了某个进程,  则这个进程可以操作任何一个文件。<br/>
> 3. SELinux在Linux的文件权限之外, 增加了对进程的限制, 进程只能在进程允许的范围内操作资源

#### 6.2.2 为什么要关闭SELinux ?

> 如果开启了SELinux, 需要做非常复杂的配置, 才能正常使用系统, 在学习阶段, 在非生产环境, 一般不使用SELinux

#### 6.2.3 SELinux的工作模式

```
enforcing  强制模式
permissive 宽容模式
disabled   关闭
```

#### 6.2.4 关闭SELinux

下面文件改为图中所示：SELINUX=disable

```
vim /etc/selinux/config 
```

![关闭SELinux](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/11关闭SELinux.png)

#### 6.2.5 分别重启三台虚拟机

```
reboot
```

## 7 三台服务器免密码登录

### 7.1 为什么要免密登录

> Hadoop 节点众多, 所以一般在主节点启动从节点, 这个时候就需要程序自动在主节点登录到从节点中, 如果不能免密就每次都要输入密码, 非常麻烦。

### 7.2 免密 SSH 登录的原理

> 1. 需要先在 B节点 配置 A节点 的公钥
> 2. A节点 请求 B节点 要求登录
> 3. B节点 使用 A节点 的公钥, 加密一段随机文本
> 4. A节点 使用私钥解密, 并发回给 B节点
> 5. B节点 验证文本是否正确

![免密码登录原理](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/12免密码登录原理.png)

### 7.3 实现步骤

#### 7.3.1 第一步：三台机器生成公钥与私钥

> 在三台机器执行以下命令，生成公钥与私钥

```
ssh-keygen -t rsa
```

执行该命令之后，按下三个回车即可，然后敲（三个回车），就会生成两个文件**id_rsa（私钥）**、**id_rsa.pub（公钥）**，默认保存在/root/.ssh目录。

![生成公钥与私钥](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/13生产公钥和私钥.png)

#### 7.3.2 第二步：拷贝公钥到同一台机器

> 三台机器将拷贝公钥到第一台机器<br/>
三台机器都执行命令：

```
ssh-copy-id node1
```

在执行该命令之后，需要输入yes和node1的密码:

![拷贝公钥到同一台机器](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/14拷贝公钥到node1.png)

第一台机器在/root/.ssh下会生产authorized_keys文件

#### 7.3.3 第三步：复制第一台机器的认证到其他机器

> 将第一台机器的公钥拷贝到其他机器上<br/>
在第一台机器上指行以下命令

```
scp /root/.ssh/authorized_keys node2:/root/.ssh
scp /root/.ssh/authorized_keys node3:/root/.ssh
```

执行命令时，需要输入yes和对方的密码

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/15复制第一台机器的认证到其他机器.png)

其他机器在/root/.ssh下会生产authorized_keys文件

#### 7.3.4 第四步：测试SSH免密登录

```
ssh node2
exit
```

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxJiQunDaJian/16测试免密登录.png)

## 8 设置时区

> 查看时区

```shell
timedatectl
```

> 设置时区为上海

```shell
timedatectl set-timezone Asia/Shanghai
```

## 9 时钟同步

> 目标环境，`3台linux centos7`， `一台作为NTPD服务与外部公共NTP服务同步时间`，`同时作为内网的NTPD服务器`，`其他机器与这台服务做时间同步`。

| 服务器IP       | 角色          | 说明                                                         | 同步方式         |
| -------------- | ------------- | ------------------------------------------------------------ | ---------------- |
| 192.168.88.101 | NTPD服务      | 1、负责与外部公共NTPD服务同步标准时间<br />2、作为内外网络的NTPD服务 | NTPD服务平滑同步 |
| 192.168.88.102 | 内外NTP客户端 | 内网设备与192.168.88.101同步时间                             | NTPD服务平滑同步 |
| 192.168.88.103 | 内外NTP客户端 | 内网设备与192.168.88.101同步时间                             | NTPD服务平滑同步 |

### 9.1 为什么需要时间同步（掌握）

> 因为很多分布式系统是有状态的, 比如说存储一个数据, A节点 记录的时间是1, B节点 记录的时间是2, 就会出问题

### 9.2 NTP时间同步方式选择

> NTP同步方式在linux下一般两种：使用`ntpdate命令直接同步`和使用N`TPD服务平滑同步`。有什么区别呢，简单说下，免得时间长了，概念又模糊。

> 现有一台设备，系统时间是 13:00 , 真实的当前时间(在空中，也许卫星上，这里假设是在准备同步的上级目标NTP服务器)是: 12:30  。
> 
> 如果我们使用`ntpdate`同步（ntpdate -u 目标NTP服务器IP），操作系统的时间立即更新为12:30,假如，我们的系统有一个定时应用，是在每天12:40运行，那么实际今天这个的任务已经运行过了（当前时间是13:00嘛），现在被`ntpdate`修改为12:30，那么意味作10分钟后，又会执行一次任务，这就糟糕了，这个任务只能执行一次的嘛！！
> 
> 我想你（其实是我）已经懂了`ntpdate时间同步的隐患`，当然这个例子有些极端，但的确是有风险的，生产环境我不打算这么干，还是稳妥点好。
> 
> 所以解决该问题的办法就是`时间平滑更改`，不会让一个时间点在一天内经历两次，这就是NTPD服务方式平滑同步时间，它每次同步时间的偏移量不会太陡，是慢慢来的（问：怎么来，没有细究，只晓得一次一点的同步，完全同步好需要较长时间，所以一般`开启NTPD服务同步前先用ntpdate先手动同步一次`）。

### 9.3 安装配置

#### 9.3.1 安装NTPD服务（所有服务器执行）

> `所有服务器都执行，安装NTP软件包`

```
# yum install -y ntp  
```

```
# rpm -q ntp
ntp-4.2.6p5-29.el7.centos.2.x86_64    // 这表示已安装了，如果没有安装，这是空白。     
```

> `配置NTP服务为自启动`

```
# systemctl enable ntpd.service
# systemctl list-unit-files | grep ntpd
ntpd.service                                  enabled 
ntpdate.service                               disabled
```

>  在配置前，`先使用ntpdate手动同步下时间`，免得本机与外部时间服务器时间差距太大，让ntpd不能正常同步。

```
# ntpdate -u 202.112.10.36
8 Dec 21:51:57 ntpdate[1628]: adjust time server 202.112.10.36 offset 0.003856 sec
```

#### 9.3.2 配置内网NTP-Server(192.168.88.101)

> 下面主要是配置内网的`NPTD服务器`(192.168.88.101), NTPD服务配置核心就在`/etc/ntp.conf`文件，配置好了就OK。
>
> 贴出来的是修改的部分，其他默认。

```
vim /etc/ntp.conf
```

```
# 允许内网其他机器同步时间（新增项）
restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

# 中国这边最活跃的时间服务器 : http://www.pool.ntp.org/zone/cn（修改）
server 210.72.145.44 perfer      # 中国国家受时中心
server 202.112.10.36             # 1.cn.pool.ntp.org
server 59.124.196.83             # 0.asia.pool.ntp.org
 
# allow update time by the upper server 
# 允许上层时间服务器主动修改本机时间 （新增）
restrict 210.72.145.44 nomodify notrap noquery
restrict 202.112.10.36 nomodify notrap noquery
restrict 59.124.196.83 nomodify notrap noquery

# 外部时间服务器不可用时，以本地时间作为时间服务 （新增）
server  127.127.1.0     # local clock
fudge   127.127.1.0 stratum 10
```

> 启动服务

```
systemctl start ntpd.service
```

> 启动后，一般需要`5-10分钟左右`的时候才能与外部时间服务器开始同步时间。可以通过命令查询`NTPD服务`情况。

> 查看`服务连接和监听`

```
# netstat -tlunp | grep ntp
udp        0      0 192.168.88.101:123      0.0.0.0:*                           1753/ntpd           
udp        0      0 127.0.0.1:123           0.0.0.0:*                           1753/ntpd           
udp        0      0 0.0.0.0:123             0.0.0.0:*                           1753/ntpd           
udp6       0      0 fe80::cef6:8c02:cd8:123 :::*                                1753/ntpd           
udp6       0      0 ::1:123                 :::*                                1753/ntpd           
udp6       0      0 :::123                  :::*                                1753/ntpd                                   23103/ntpd  
看第一行，表示连接和监听已正确，采用UDP方式    
```

> 查看网络中的`NTP服务器`，同时显示客户端和每个服务器的关系

```
# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 210.72.145.44   .INIT.          16 u    -   64    0    0.000    0.000   0.000
 gus.buptnet.edu 202.112.10.60    2 u   47   64    3  274.042  116.766  61.129
 59.124.196.83   .INIT.          16 u    -   64    0    0.000    0.000   0.000
```

> 查看`时间同步状态`，这个一般需要`5-10分钟后`才能成功连接和同步。所以，服务器启动后需要稍等下。

```
# ntpstat

刚启动的时候，一般是：
unsynchronised
  time server re-starting
   polling server every 8 s
   
连接并同步后:
synchronised to NTP server (202.112.10.36) at stratum 3
   time correct to within 1036 ms
   polling server every 64 s
```

> OK，`内网的NTPD服务已经配置完成`，如果所有正常后，开始配置内网的其他设备与这台服务器作为时间同步服务。

#### 9.3.3 配置内网NTP-Clients

> 内网`其他设备作为NTP的客户端配置`，相对就比较简单，而且所有设备的配置都相同。
>
> 修改其中一台配置`/etc/ntp.conf`文件(我这里修改node2)，配置完成后，`拷贝到其他客户端机器`即可。
>
> 贴出来的是修改的部分，其他默认。

```
vim /etc/ntp.conf
```

```
# 配置时间服务器为本地的时间服务器
server 192.168.88.101
 
restrict 192.168.88.101 nomodify notrap noquery
 
server  127.127.1.0     # local clock
fudge   127.127.1.0 stratum 10
```

> `拷贝到其他服务器`

```
scp /etc/ntp.conf node3:/etc
```

> `先使用ntpdate手动同步下时间`(服务器2和3)

```
# ntpdate -u 192.168.88.101
 8 Dec 22:19:29 ntpdate[1777]: adjust time server 192.168.88.101 offset -0.024243 sec

这里有可能出现同步失败，一般情况下原因都是本地的NTPD服务器还没有正常启动起来，一般需要几分钟时间后才能开始同步。
```

> `启动服务`(服务器2和3)

```
systemctl start ntpd.service
```

> `查看同步情况`(服务器2和3)

```
# ntpq -p
# ntpstat

因为是内网，一般ntpstat很快就可以同步上，几分钟需要等下。
```


## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
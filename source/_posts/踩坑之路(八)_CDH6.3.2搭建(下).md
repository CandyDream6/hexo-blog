---
title: '踩坑之路(八) CDH6.3.2搭建(下)'
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(84).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(84).jpg
tags:
  - 大数据
  - 踩坑之路
category:
  - - 编程
    - 大数据
    
date: 2021-05-24 11:10:57
---

上一篇把Cloudera Manager安装好了，这一篇开始安装CDH

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 版本选择

### 1.1 使用默认用户名密码登录

```shell
用户名: admin
密码: admin
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/1.png)

### 1.2 登录成功来到环境界面

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/2.png)

### 1.3 接受许可条款

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/3.png)

### 1.4 根据需要选择版本

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/4.png)

## 2 创建集群

### 2.1 欢迎界面

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/5.png)

### 2.2 集群命名

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/6.png)

### 2.3 选择主机

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/7.png)

### 2.4 配置仓库

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/8.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/9.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/10.png)

### 2.5 安装Parcels

> 这是一个拼人品的的环节

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/11.png)

> `出错了就去这两个地方看日志`

```shell
/var/log/cloudera-scm-agent
/var/log/cloudera-scm-server
```

### 2.6 集群检查

> `看我喜庆的界面`，搞了我大半天，哎，搞来搞去，最后是端口的问题，好烦，CDH要开好多好多端口，乖乖照着官网一个个开吧

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/12.png)

> 如果没关防火墙，这里大多是端口问题，我们公司就不能关防火墙，所以乖乖照着上一篇文章看哪些端口需要开放吧。

> 再执行下这个，这是解决第二个检查里的那个警告，之后就见下图了。

```shell
# 查看当前内核参数
sysctl vm.swappiness
# 临时修改
sysctl vm.swappiness=10
# 查看修改后内核参数
sysctl vm.swappiness
# 永久修改
sed -i s/"vm.swappiness = 30"/"vm.swappiness = 10"/g  /usr/lib/tuned/virtual-guest/tuned.conf
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/13.png) 

## 3 安装服务

### 3.1 安装HDFS和YARN

> `放行端口`

```
https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/cm_ig_ports.html#concept_k5z_vwy_4j
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/14.png)

#### 3.1.1 角色分配

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/15.png)

#### 3.1.2 集群配置

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/16.png)

#### 3.1.3 开始安装

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/17.png)

##### 3.1.3.1 Failed to format NameNode.

> `错误`

```shell
Failed to format NameNode.

是由于之前初始化 namenode 在 /dfs/nn 留下了残留数据（失效数据），从而影响再次初始化
```

> 解决办法
>> 清空残留数据后，重新初始化

> namenode节点：

```
rm -rf /dfs/nn
```

> datanode节点：

```
rm -rf /dfs/dn
```

##### 3.1.3.2 First failure: Command (Create /tmp Directory (762)) has failed

> `错误`

```shell
First failure: Command (Create /tmp Directory (762)) has failed
Safe mode will be turned off automatically once the thresholds have been reached
```

> 按正常命令启动HDFS之后,HDFS一直处于安全模式。他会一会儿自己退出，我们目前先手动退出一下

```shell
sudo -u hdfs hdfs dfsadmin -safemode leave
```

##### 3.1.3.2 Failed to upload YARN MapReduce Framework JARs.

> `报错`

```shell
Failed to install YARN MapReduce Framework JARs.
Failed to upload YARN MapReduce Framework JARs.
```

> 解决
>> 这个我是端口的问题，去把需要的端口都放开吧(当然如果你关了防火墙就不会有这个问题)

```shell
需要开放的端口：
https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_ig_ports.html
```

#### 3.1.4 完成

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/18.png)

### 3.2 安装Zookeeper

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/20.png)

#### 3.2.1 角色分配

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/21.png)

#### 3.2.2 集群配置

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/22.png)

#### 3.2.3 开始安装

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/23.png)

### 3.3 安装Kafka

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/24.png)

#### 3.3.1 角色分配

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/25.png)

#### 3.3.2 集群配置

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/26.png)

#### 3.3.3 开始安装

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/27.png)

## 4 警告修复

### 4.1 DNS Resolution

```shell
The hostname and canonical name for this host are not consistent when checked from a Java process.
```

> 解决

```shell
vim /etc/sysconfig/network
NETWORKING=yes
HOSTNAME= node1.lankr.cn               ##每台服务器配置自己的
systemctl restart network
```

```shell
问题缘由是在配置hostname的时候配置了域名，服务器根据/etc/resolv.conf DNF解析是，提示找不到改域名。
处理方式：在每台服务器上添加本地域名解析：(根据上述hosts配置，须要添加以下配置，domain 是本地域名的意思)

echo "domain lankr.cn" >> /etc/resolv.conf
```

### 4.2 Erasure Coding Policy Verification Test

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/19.png)

### 4.3 ERROR：The health test result for NAME_NODE_HA_CHECKPOINT_AGE  has become bad: The filesystem checkpoint is 4 hour(s) old. This is 401.25% of the configured checkpoint period of 1 hour(s).

> 通过真正的错误的描述，发现主要是版本不匹配，说明在重新安装CDH的时候，保留了以前版本的CDH的数据，导致不一致的版本问题，所以导致secondarynamenode不执行检查点的操作。
 
> `那么解决办法就是删除之前的数据`，所以通过删除secondarynamenode执行检查点是的目录，即hdfs-site.xml中参数fs.checkpoint.dir, dfs.namenode.checkpoint.dir的值的路径，默认是`/dfs/snn`。

```shell
rm -rf /dfs/snn
```

### 4.4 Bad : This host is in contact with the Cloudera Manager Server. This host is not in contact with the Host Monitor.

> 这个问题，卡了一天多，到最后，发现有`一个端口漏掉了`，CDH组件非常多，需要开放的端口也就非常多了，一定要细致细致再细致。

> 照着官网一步一步来吧

```shell
https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_ig_ports.html#concept_k5z_vwy_4j
```

## 5 配置本地域名映射

```shell
C:\Windows\System32\drivers\etc\hosts
```

```shell
192.168.1.101 node1.lankr.cn node1
192.168.1.102 node2.lankr.cn node2
192.168.1.103 node3.lankr.cn node3
192.168.1.104 node4.lankr.cn node4
192.168.1.105 node5.lankr.cn node5
192.168.1.106 node6.lankr.cn node6
192.168.1.107 node7.lankr.cn node7
```

> 这样windows电脑就可以通过node1或者node1.lankr.cn来访问CM集群了，而不用ip了

## 6 老泪纵横

> `不容易呀不容易`，终于可以`头戴一片绿`了，这`青青草原`可真好看。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/8_CDH6.3.2搭建(下)/28.png)

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

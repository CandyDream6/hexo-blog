---
title: '踩坑之路(三) Zookeeper集群搭建'
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(79).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(79).jpg
tags:
  - 大数据
  - 踩坑之路
category:
  - - 编程
    - 大数据
    
date: 2021-05-07 18:10:57
---

在我们`大数据的这个坑`里，最主要的概念当然就是`分布式存储`和`分布式计算`。

一旦提到`分布式`，那么`注册中心`的概念就又用到了。

大数据生态圈里，有很多组件的`元数据信息`都是借助`ZooKeeper`去管理和调度的，包括Kafka等等(虽然Kafka扬言要脱离Zookeeper，但我去官网看了下最新版的release，发现还没有实现，希望Kafka能尽早实现吧，尽量减少组件之间的依赖性)。

那我们也不等不搭建一个ZooKeeper集群了，来实现分布式协调服务。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 下载上传Zookeeper安装包

https://zookeeper.apache.org/releases.html

```
链接：https://pan.baidu.com/s/1uBuiIO3c-UgkbVLm_mBD5g 
提取码：8888 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

> `apache-zookeeper-3.4.6.tar.gz`，把他上传到/lankr/software

## 2 解压配置zookeeper

> 创建脚本init_zookeeper.sh

```
cd /lankr/script/zookeeper
vim init_zookeeper.sh
```

> 写入下面内容

```shell
#!/bin/bash

# 判断/lankr/application是否已经由zookeeper3.6.3的包
ZOOKEEPER_APP_NAME="/lankr/application/zookeeper-3.4.6"

if [ -d $ZOOKEEPER_APP_NAME ]; then
    echo "zookeeper3.4.6的安装包已经存在，可能以前安装过，现在直接退出"
    exit 1
else
    echo "zookeeper的安装包不存在，现在开始安装和配置操作"
fi

echo

# 解压zookeeper到/lankr/application/
echo "开始解压zookeeper到/lankr/application" 
tar -zxvf /lankr/software/zookeeper-3.4.6.tar.gz -C /lankr/application
echo "完成解压zookeeper"

echo

# 备份配置文件
echo "开始备份配置文件"
cd /lankr/application/zookeeper-3.4.6/conf
cp zoo_sample.cfg zoo.cfg
echo "完成备份配置文件"

echo

# 创建数据存储目录
echo "开始创建数据存储目录"
mkdir -p /lankr/application/zookeeper-3.4.6/zkdatas/
echo "完成创建数据存储目录"

echo

echo "开始书写配置文件"
echo '#Zookeeper的数据存放目录,默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里
dataDir=/lankr/application/zookeeper-3.4.6/zkdatas/
# 保留多少个快照
autopurge.snapRetainCount=3
# 日志多少小时清理一次
initLimit=10
syncLimit=5
clientPort=2181
autopurge.purgeInterval=8928
# 集群中服务器地址
server.3=node3:2888:3888
server.4=node4:2888:3888
server.5=node5:2888:3888
server.6=node6:2888:3888
server.7=node7:2888:3888' > /lankr/application/zookeeper-3.4.6/conf/zoo.cfg
echo "完成书写配置文件"

echo

# echo "我用的这个版本，zookeeper识别不了JAVA_HOME，所以下面强制写入一下"

# sed -i '2i JAVA_HOME="/lankr/application/jdk1.8.0_291"' /lankr/application/zookeeper-3.4.6/bin/zkEnv.sh

# 配置zookeeper集群的myid
echo "开始配置zookeeper集群的myid"
echo 3 > /lankr/application/zookeeper-3.4.6/zkdatas/myid
echo "完成配置zookeeper集群的myid"

echo

# 分发zookeeper到各个服务器
echo "开始分发zookeeper到各个服务器"
for i in {4..7}
do 
    scp -r /lankr/application/zookeeper-3.4.6/ node$i:/lankr/application/
    # 修改各个服务器的zookeeper的myid的值
    ssh node$i "echo $i > /lankr/application/zookeeper-3.4.6/zkdatas/myid"
done
echo "完成分发zookeeper到各个服务器"
```

> 执行安装

```shell
chmod +x /lankr/script/zookeeper/init_zookeeper.sh
/lankr/script/zookeeper/init_zookeeper.sh
```


## 3 创建集群启动脚本

> 创建脚本start_zookeeper.sh

```
vim /lankr/script/zookeeper/start_zookeeper.sh
```

> 写入下面内容

```shell
#!/bin/bash

# 启动zookeeper集群
echo "准备启动zookeeper集群"
for i in {1..7}
do
    # 启动zookeeper
    echo "准备启动node$i上的zookeeper"
    ssh node$i "export JAVA_HOME=/lankr/application/jdk1.8.0_291 && /lankr/application/zookeeper-3.4.6/bin/zkServer.sh start"

    # 查看zookeeper状态
    echo "node$i上的zookeeper状态如下:"
    ssh node$i "/lankr/application/zookeeper-3.4.6/bin/zkServer.sh status"

    echo
done
```

> 执行脚本启动

```shell
chmod +x /lankr/script/zookeeper/bin/satrt_zookeeper.sh
/lankr/script/zookeeper/satrt_zookeeper.sh
```

> 查看下详细的日志

```shell
/lankr/application/zookeeper-3.4.6/bin/zkServer.sh start-foreground
```

## 4 客户端工具连接

> 可以使用客户端可视化工具连接Zookeeper。我使用`ZooInspector`

```
链接：https://pan.baidu.com/s/1mW_Fp2MsxsQCfjLvS4-A6A 
提取码：8888 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

> 如图所示，双击start.bat，然后输入zookeeper的ip:host就可以连接到了

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/3_Zookeeper集群搭建/1.png)

## 5 坑

> `默默留下没有技术的泪水`

### 5.1 启动集群失败

> 查看状态

```shell
/lankr/application/zookeeper-3.4.6/bin/zkServer.sh status

发现报错
Using config: /lankr/application/zookeeper-3.4.6/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.
```

> `事故原因`
>> 又是排查了半下午，心态崩了。
> 
>> 原来是防火墙的原因，以前我自己都是在本地搞，防火墙都是关掉的，可是服务器上的防火墙不能关怎么办,只能是开放对应的端口

```shell
# 查看防火墙状态，确保他是开着的
systemctl status firewalld

# 查看已经开放的端口，看有没有我们需要的那个
firewall-cmd --list-ports

# 开启zookeeper需要的三个端口
firewall-cmd --permanent --zone=public --add-port=2181/tcp
firewall-cmd --permanent --zone=public --add-port=2888/tcp
firewall-cmd --permanent --zone=public --add-port=3888/tcp

# 重启防火墙
systemctl restart firewalld.service

# 重载配置文件
firewall-cmd --reload
 
# 再次查看已经开放的端口
firewall-cmd --list-ports
```

### 5.2 远程启动集群失败

> `这个问题困了我一整个下午，人都麻了，后来问了文杰老哥解决的，记录一下，太难了。`
>> 注意：`远程启动zookeeper集群`的时候，写一下`export JAVA_HOME=/lankr/application/jdk1.8.0_291`，因为shell脚本执行的过程中，`登录shell`和`非登录shell`读取的环境变量配置文件不同。
> 
>> `登录shell`会读取`/etc/profile`,`~、。bash_profile`,`~/.bash_login`,`~/.profile`等文件，而`非登录shell`读取的脚本有`/etc/bashrc`和`~/.bashrc`,像`java，path`这些环境变量是`读取不到`的。
> 
>> 解决办法两个：`一是将/etc/profile文件中的环境变量复制到~/.bashrc就解决了`。`二是在执行时暴露一下环境变量，像我上面那样`。

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
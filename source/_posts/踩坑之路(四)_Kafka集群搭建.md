---
title: '踩坑之路(四) Kafka集群搭建'
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(80).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(80).jpg
tags:
  - 大数据
  - 踩坑之路
category:
  - - 编程
    - 大数据
    
date: 2021-05-08 14:10:57
---

Kafka在当前`MQ领域`简直就是`一骑绝尘`。

我搭这个集群，是为了以后将`采集的数据`直接`怼入Kafka`，做一次`消息缓存`，之后用计划用`flink去消费`，处理后续的逻辑业务。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 下载上传Kafka安装包

https://kafka.apache.org/downloads

```
链接：https://pan.baidu.com/s/1UxgjhBQ6mytqZp4bhpDs7w 
提取码：8888 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

> `kafka_2.11-1.0.0.tgz`，把他上传到/lankr/software

## 2 解压配置Kafka

> 创建脚本init_kafka.sh

```shell
touch /lankr/script/kafka/init_kafka.sh
chmod +x /lankr/script/kafka/init_kafka.sh
vim /lankr/script/kafka/init_kafka.sh
```

> 输入下面内容

```shell
#!/bin/bash

# 判断kafka包是否存在，如果存在的话，直接退出
KAFKA_APP_NAME="/lankr/application/kafka_2.11-1.0.0"	
if [ -d $KAFKA_APP_NAME ]; then
    echo "kafka的解压包已经存在，可能之前已经安装过kafka，现在直接退出:$KAFKA_APP_NAME"
    exit 1
else
    echo "kafka的解压包不存在，现在开始安装和配置操作"
fi

echo

# 解压kafka到/lankr/application
echo "开始解压Kafka到/lankr/application"
tar -zxvf /lankr/software/kafka_2.11-1.0.0.tgz -C /lankr/application
echo "完成解压Kafka"

echo 

# 修改配置文件
echo "开始修改配置文件"

echo 'broker.id=3
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/export/data/kafka/kafka-logs
num.partitions=2
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.flush.interval.messages=10000
log.flush.interval.ms=1000
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=node3:2181,node4:2181,node5:2181,node6:2181,node7:2181
zookeeper.connection.timeout.ms=6000
group.initial.rebalance.delay.ms=0
delete.topic.enable=true
host.name=node3' > /lankr/application/kafka_2.11-1.0.0/config/server.properties 

echo "完成修改配置文件"

echo

# 配置环境变量(先备份)
echo "备份/etc/profile到/etc/profile_init_back"

echo 

cp -r /etc/profile /etc/profile_init_back
echo "判断KAFKA_HOME是否添加到环境变量中"
FIND_FILE="/etc/profile"
FIND_STR_KAFKA_HOME="export KAFKA_HOME=/lankr/application/kafka_2.11-1.0.0"
if [ `grep -c "$FIND_STR_KAFKA_HOME" $FIND_FILE` -ne '0' ]; then
    echo "环境变量KAFKA_HOME已经配置，不再进行配置"
else
    echo "环境变量KAFKA_HOME还未配置，现在进行配置"
    echo '#KAFKA_HOME
export KAFKA_HOME=/lankr/application/kafka_2.11-1.0.0' >> /etc/profile
fi

echo

echo "判断KAFKA_HOME是否PATH中"
FIND_STR_KAFKA_HOME_IN_PATH='export PATH=$PATH:$KAFKA_HOME/bin'
if [ `grep -c "$FIND_STR_KAFKA_HOME_IN_PATH" $FIND_FILE` -ne '0' ]; then
    echo "环境变量KAFKA_HOME在PATH中已经配置，不再进行配置"
else
    echo "环境变量KAFKA_HOME在PATH中还未配置，现在进行配置"
    echo 'export PATH=$PATH:$KAFKA_HOME/bin' >> /etc/profile
fi

echo

source /etc/profile
echo "完成配置环境变量"

echo

# 分发安装包，循环发送，不同集群配置改下个数
echo "开始分发安装包和配置文件"
for i in {4..7}
do
    scp -r /lankr/application/kafka_2.11-1.0.0 node$i:/lankr/application
    ssh node$i "sed -i 's/\bbroker.id=3\b/broker.id=$i/g;s/\bhost.name=node3\b/host.name=node$i/g' /lankr/application/kafka_2.11-1.0.0/config/server.properties"
    scp /etc/profile node$i:/etc
    scp /etc/profile_init_back node$i:/etc
done 
echo "完成安装包和配置文件分发"
```

> 执行安装

```shell
/lankr/script/kafka/init_kafka.sh
```

## 3 开放需要端口

> kafka默认使用9092端口，那么我们放行一下

```shell
# 查看已经开放的端口，看有没有我们需要的那个
firewall-cmd --list-ports

# 开启zookeeper需要的三个端口
firewall-cmd --permanent --zone=public --add-port=9092/tcp

# 重启防火墙
systemctl restart firewalld.service

# 重载配置文件
firewall-cmd --reload
 
# 再次查看已经开放的端口
firewall-cmd --list-ports
```
 
## 4 创建一键启动脚本

> 创建脚本start_kafka.sh

```shell
touch /lankr/script/kafka/start_kafka.sh
chmod +x /lankr/script/kafka/start_kafka.sh
vim /lankr/script/kafka/start_kafka.sh
```

> 输入下面内容

```shell
#!/bin/bash

# 启动kafka集群
echo "准备启动kafka集群"
for i in {3..7}
do
    # 启动zookeeper
    echo "准备启动node$i上的kafka"
    ssh node$i "export JAVA_HOME=/lankr/application/jdk1.8.0_291 && nohup /lankr/application/kafka_2.11-1.0.0/bin/kafka-server-start.sh /lankr/application/kafka_2.11-1.0.0/config/server.properties >/dev/null 2>&1 &"
    echo "node$i kafka is running"
    echo
done
```

> 执行启动脚本

```shell
/lankr/script/kafka/start_kafka.sh
```

## 5 可视化工具

> 我用的Kafka可视化工具是Kafka tool

https://www.kafkatool.com/

```
链接：https://pan.baidu.com/s/1zFVjrs7woAZ61JvmF_DqqQ 
提取码：8888 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/4_Kafka集群搭建/1.png)

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)

---
title: '踩坑之路(六) Canal搭建'
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(82).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(82).jpg
tags:
  - 大数据
  - 踩坑之路
category:
  - - 编程
    - 大数据
    
date: 2021-05-10 19:10:57
---

`实时同步``MySQL数据到Kafka`方式有挺多。

这里我采用`阿里开源`的`MySQL binlog` `增量订阅&消费`组件`Canal`去做这件事

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 概述

> 官网地址：https://github.com/alibaba/canal
> 
> 具体情况去官网看吧

## 2 安装部署

### 2.1 下载最新版本包

```shell
https://github.com/alibaba/canal/releases
```

> 上传到服务器/lankr/software

### 2.2 解压

```shell
mkdir /lankr/application/canal-1.1.15
tar -zxvf canal.deployer-1.1.5.tar.gz -C /lankr/application/canal-1.1.15
```

### 2.3 开启binlog写入功能

> mysql开启 Binlog 写入功能，配置 binlog-format 为 ROW 模式，my.cnf 中配置如下

```shell
vim /lankr/application/mysqlCanal/conf/my.cnf
```

> 添加以下内容

```shell
# 下面是canal配置
log-bin=mysql-bin # 开启 binlog
binlog-format=ROW # 选择 ROW 模式
server_id=1 # 配置 MySQL replaction 需要定义，不要和 canal 的 slaveId 重复
```

### 2.4 配置canal账号

```shell
docker exec -it mysqlCanal /bin/bash
mysql -u root -p
```

```shell
CREATE USER canal IDENTIFIED BY 'canal';  
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
FLUSH PRIVILEGES;

ALTER USER 'canal'@'localhost' IDENTIFIED BY 'canal' PASSWORD EXPIRE NEVER;
ALTER USER 'canal'@'localhost' IDENTIFIED WITH mysql_native_password BY 'canal';
ALTER USER 'canal'@'%' IDENTIFIED BY 'canal' PASSWORD EXPIRE NEVER;
ALTER USER 'canal'@'%' IDENTIFIED WITH mysql_native_password BY 'canal';
FLUSH PRIVILEGES;
```

### 2.5 修改配置文件

#### 2.5.1 修改canal.properties

```shell
vim /lankr/application/canal-1.1.15/conf/canal.properties
```

> 修改部分如下：

```shell
# tcp, kafka, rocketMQ, rabbitMQ
canal.serverMode = kafka
canal.destinations = example
kafka.bootstrap.servers = 192.168.88.10:9092;192.168.88.20:9092;192.168.88.30:9092
```

#### 2.5.2 修改instance.properties

```shell
vim /lankr/application/canal-1.1.15/conf/example/instance.properties
```

> 修改内容如下

```shell
# 这个源码是注释的，自己复制下。不要和mysql my.cnf的server_id重复
canal.instance.mysql.slaveId = 1234

# 你的数据库地址信息
canal.instance.master.address=192.168.1.101:3306

# 用于同步的数据库的用户名和密码，我下面创建的是canal，canal
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal

# 没有匹配到的数据都发送到这个分区下
canal.mq.topic=a_default_topic

# 你自己的要同步的表的匹配规则，我要同步全部，所以这里就用默认的
canal.instance.filter.regex=.*\\..*

# 这样每个表都会发到一个主题里，这样就好管理了。然后先这样吧，后续再根据业务去看怎么弄
canal.mq.dynamicTopic=.*\\..*
```

### 2.6 启动：起飞

> 启动

```shell
/lankr/application/canal-1.1.15/bin/startup.sh
```

> 查看canal和example日志

```shell
tail -1000f /lankr/application/canal-1.1.15/logs/canal/canal.log
tail -1000f /lankr/application/canal-1.1.15/logs/example/example.log
```

### 2.7 关闭集群

```shell
/lankr/application/canal-1.1.15/bin/stop.sh
```

### 3 总结

> canal的官方文档确实坑很多，写的很乱，很多配置、参数都没有解释，哎，万能的google`滚啊趴啊`，`趴啊滚啊`，慢慢来吧。

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
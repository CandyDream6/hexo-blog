---
title: Canal 基于MySQL数据库增量日志解析
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(65).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(65).jpg
tags:
  - 大数据
  - Canal
category:
  - - 编程
    - 大数据
date: 2020-07-02 15:36:45
---

canal [kə'næl]，译意为水道/管道/沟渠。

主要用途是基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费.

canal 作为 MySQL binlog 增量获取和解析工具，可将变更记录投递到 MQ 系统中，比如 Kafka/RocketMQ，可以借助于 MQ 的多语言能力。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 MySQL

### 1.1 Mysql部署

> Docker hub上查找mysql镜像

```
docker search mysql
```

> 从docker hub上(阿里云加速器)拉取mysql镜像到本地标签为5.7

```
docker pull mysql:5.7
```

> 创建容器

```
docker run -di --name=mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
-p 代表端口映射，格式为宿主机映射端口:容器运行端口
-e 代表添加环境变量，MYSQL_ROOT_PASSWORD是root用户的登录密码
```

> 进入MySQL容器

> 使用mysql客户端连接

```
docker exec -it mysql bash
```

### 1.2 Mysql开启binlog日志

> 用来记录mysql中的`增加、删除、修改、清空`操作`select操作 不会 保存到binlog中`

> 必须要 打开 mysql中的binlog功能，才会生成binlog日志

> binlog日志就是一系列的二进制文件

```
-rw-rw---- 1 mysql mysql 669 11⽉月 10 21:29 mysql-bin.000001
-rw-rw---- 1 mysql mysql 126 11⽉月 10 22:06 mysql-bin.000002
-rw-rw---- 1 mysql mysql 11799 11⽉月 15 18:17 mysql-bin.00000
```

> 配置my.cnf

```
root@dfbf3fdefbdf:/# vim /etc/mysql/my.cnf
```

```
[mysqld] 
log-bin=mysql-bin  #添加这一行就ok 
binlog-format=ROW #选择row模式 
server_id=1  #配置mysql replaction需要定义，不能和canal的slaveId重复
expire_logs_days=7 # binlog文件保存7天 
max_binlog_size = 500m # 每个binlog日志文件大小
```

> 重启mysql容器（dfbf3fdefbdf是容器id）

```
docker restart dfbf3fdefbdf
```

> 进入sql命令行

```
docker exec -it mysql bash
mysql -uroot -p123456
```

> 验证 my.cnf 配置是否生效:

```
show variables like 'binlog_format';
show variables like 'log_bin';
show master status;
```

### 1.3 可能遇到的问题

> 创建容器时报错：

```
WARNING: IPv4 forwarding is disabled. Networking will not work.
ab71b3f089ff3a7538811e79fd3d989b5958383968b1f1eb8ba1dc6f1da3335b
```

> 解决方式：

```
vim /usr/lib/sysctl.d/00-system.conf
添加如下代码：net.ipv4.ip_forward=1
重启网络：systemctl restart network
完成以后，删除当前容器，重新创建新容器。
```

## 2 Canal介绍

### 2.1 Canal简介

> canal [kə'næl]，译意为水道/管道/沟渠，`主要用途是基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费`

> 早期阿里巴巴因为杭州和美国双机房部署，存在跨机房同步的业务需求，实现方式主要是基于业务 trigger 获取增量变更。从 2010 年开始，业务逐步尝试数据库日志解析获取增量变更进行同步，由此衍生出了大量的数据库增量订阅和消费业务。

> 基于日志增量订阅和消费的业务包括
>> 数据库镜像
>
>> 数据库实时备份
>
>> 索引构建和实时维护(拆分异构索引、倒排索引等)
>
>> 业务 cache 刷新
>
>> 带业务逻辑的增量数据处理
>
>> 当前的 canal 支持源端 MySQL 版本包括 5.1.x , 5.5.x , 5.6.x , 5.7.x , 8.0.x

### 2.2 Mysql的主备复制原理

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Canal/1.png)

> MySQL master 将数据变更写入二进制日志( binary log, 其中记录叫做二进制日志事件binary log events，可以通过 show binlog events 进行查看)

> MySQL slave 将 master 的 binary log events 拷贝到它的中继日志(relay log)

> MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据

### 2.3 Canal的工作原理

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Canal/2.png)

> canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送dump 协议

> MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )

> canal 解析 binary log 对象(原始为 byte 流)

### 2.4 Canal的架构

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Canal/3.png)

> server 代表一个 canal 运行实例，对应于一个 jvm

> instance 对应于一个数据队列 （1个 canal server 对应 1..n 个 instance )

> instance 下的子模块
>> eventParser: 数据源接入，模拟 slave 协议和 master 进行交互，协议解析
> 
>> eventSink: Parser 和 Store 链接器，进行数据过滤，加工，分发的工作
> 
>> eventStore: 数据存储
> 
>> metaManager: 增量订阅 & 消费信息管理器

> EventParser在向mysql发送dump命令之前会先`从Log Position中获取上次解析成功的位置(如果是第一次启动，则获取初始指定位置或者当前数据段binlog位点)`。mysql接受到dump命令后，由EventParser从mysql上pull binlog数据进行解析并传递给EventSink(传递给EventSink模块进行数据存储，是一个阻塞操作，直到存储成功 )，传送成功之后更新Log Position。流程图如下：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Canal/4.png)

> EventSink起到一个类似channel的功能，可以对数据进行过滤、分发/路由(1:n)、归并(n:1)和加工。EventSink是连接EventParser和EventStore的桥梁。

> EventStore实现模式是内存模式，内存结构为环形队列，由三个指针(Put、Get和Ack)标识数据存储和读取的位置。

> MetaManager是增量订阅&消费信息管理器，增量订阅和消费之间的协议包括get/ack/rollback，分别为：
>> Message getWithoutAck(int batchSize)，允许指定batchSize，一次可以获取多条，每次返回的对象为Message，包含的内容为：batch id[唯一标识]和entries[具体的数据对象]
> 
>> void rollback(long batchId)，顾名思义，回滚上次的get请求，重新获取数据。基于get获取的batchId进行提交，避免误操作
> 
>> void ack(long batchId)，顾名思议，确认已经消费成功，通知server删除数据。基于get获取的batchId进行提交，避免误操作

## 3 Canal安装部署

### 3.1 Canal部署

操作步骤 | 说明
---|---
1 | 安装canalserver镜像<br/><br/>docker pull canal/canal-server:v1.1.2
2 | 通过镜像生成canal-server容器<br/><br/>docker run -d --name canal-server \<br/>-e canal.instance.master.address=192.168.88.10:3306 \<br/>-e canal.instance.dbUsername=root \<br/>-e canal.instance.dbPassword=123456 \<br/>-p 11111:11111 \<br/>-d canal/canal-server:v1.1.2
3 | 进入canal-server容器<br/><br/>docker exec -it canal-server bash
4 | 执行/export/servers/canal/bin目录中的 startup.sh 启动canal<br/><br/>cd /home/admin/canal-server/bin<br/><br/>[root@a9ec635e5c35 bin]# ./startup.sh<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Canal/5.png)<br/><br/>控制台如果输出如上，表示canal已经启动成功
5 | 进入example日志文件查看是否有报错<br/><br/>cd canal-server/logs/example/<br/><br/>cat example.log<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Canal/6.png)

### 3.2 Canal测试

#### 3.2.1 创建测试数据库

操作步骤 | 说明
---|---
1 | 进入sql命令行<br/><br/>docker exec -it mysql bash<br/>mysql -uroot -p123456
2 | 创建mysql数据库<br/><br/>create database if not EXISTS test DEFAULT charset utf8 COLLATE utf8_general_ci;
3 | 切换到test数据库<br/><br/>use test;
4 | 在test数据库创建表<br/><br/>DROP  TABLE  IF  EXISTS  `userinfo`;<br/>CREATE  TABLE  `userinfo` (<br/>`id`  int(11)  NOT  NULL  AUTO_INCREMENT,<br/>`name`  varchar(255)  DEFAULT  NULL,<br/>`age`  int(11)  DEFAULT  NULL,<br/>PRIMARY  KEY (`id`)<br/>)  ENGINE=InnoDB  DEFAULT  CHARSET=utf8;

#### 3.2.2 配置canalserver端

> `1 进入canal-server容器`

```
docker exec -it canal-server bash
```

> `2 修改 canal/conf目录中的 canal.properties 文件`

```
vi /home/admin/canal-server/conf/canal.properties
```

```
#################################################
#########               common argument         ############# 
#################################################
canal.id = 1
canal.ip =
canal.port = 11111  #canal-server监听的端口（TCP模式下，非TCP模式不监听1111端口）
canal.metrics.pull.port = 11112 #canal-server metrics.pull监听的端口
canal.zkServers =   #集群模式下要配置zookeeper进行协调配置，单机模式可以不用配置
# flush data to zk
canal.zookeeper.flush.period = 1000
canal.withoutNetty = false
# tcp, kafka, RocketMQ
canal.serverMode = kafka #canal-server运行的模式，TCP模式就是直连客户端，不经过中间件。kafka和mq是消息队列的模式
# flush meta cursor/parse position to file
canal.file.data.dir = ${canal.conf.dir} #存放数据的路径
canal.file.flush.period = 1000
## memory store RingBuffer size, should be Math.pow(2,n)
canal.instance.memory.buffer.size = 16384
## memory store RingBuffer used memory unit size , default 1kb
canal.instance.memory.buffer.memunit = 1024 
## meory store gets mode used MEMSIZE or ITEMSIZE
canal.instance.memory.batch.mode = MEMSIZE
canal.instance.memory.rawEntry = true

## detecing config　　#这里是心跳检查的配置，做HA时会用到
canal.instance.detecting.enable = false
#canal.instance.detecting.sql = insert into retl.xdual values(1,now()) on duplicate key update x=now()
canal.instance.detecting.sql = select 1
canal.instance.detecting.interval.time = 3
canal.instance.detecting.retry.threshold = 3
canal.instance.detecting.heartbeatHaEnable = false

# support maximum transaction size, more than the size of the transaction will be cut into multiple transactions delivery
canal.instance.transaction.size =  1024
# mysql fallback connected to new master should fallback times
canal.instance.fallbackIntervalInSeconds = 60

# network config
canal.instance.network.receiveBufferSize = 16384
canal.instance.network.sendBufferSize = 16384
canal.instance.network.soTimeout = 30

# binlog filter config　　
#binlog过滤的配置，指定过滤那些SQL
canal.instance.filter.druid.ddl = true
canal.instance.filter.query.dcl = false
canal.instance.filter.query.dml = false
canal.instance.filter.query.ddl = false
canal.instance.filter.table.error = false
canal.instance.filter.rows = false
canal.instance.filter.transaction.entry = false

# binlog format/image check　
#binlog格式检测，使用ROW模式，非ROW模式也不会报错，但是同步不到数据
canal.instance.binlog.format = ROW,STATEMENT,MIXED 
canal.instance.binlog.image = FULL,MINIMAL,NOBLOB

# binlog ddl isolation
canal.instance.get.ddl.isolation = false

# parallel parser config
#并行解析配置，如果是单个CPU就把下面这个true改为false
canal.instance.parser.parallel = true
## concurrent thread number, default 60% available processors, suggest not to exceed Runtime.getRuntime().availableProcessors()
#canal.instance.parser.parallelThreadSize = 16
## disruptor ringbuffer size, must be power of 2
canal.instance.parser.parallelBufferSize = 256

# table meta tsdb info
canal.instance.tsdb.enable = true
canal.instance.tsdb.dir = ${canal.file.data.dir:../conf}/${canal.instance.destination:}
canal.instance.tsdb.url = jdbc:h2:${canal.instance.tsdb.dir}/h2;CACHE_SIZE=1000;MODE=MYSQL;
canal.instance.tsdb.dbUsername = canal
canal.instance.tsdb.dbPassword = canal
# dump snapshot interval, default 24 hour
canal.instance.tsdb.snapshot.interval = 24
# purge snapshot expire , default 360 hour(15 days)
canal.instance.tsdb.snapshot.expire = 360

# aliyun ak/sk , support rds/mq
canal.aliyun.accesskey =
canal.aliyun.secretkey =

#################################################
#########               destinations            ############# 
#################################################
#canal-server创建的实例，在这里指定你要创建的实例的名字，比如test1，test2等，逗号隔开
canal.destinations = example
# conf root dir
canal.conf.dir = ../conf
# auto scan instance dir add/remove and start/stop instance
canal.auto.scan = true
canal.auto.scan.interval = 5

canal.instance.tsdb.spring.xml = classpath:spring/tsdb/h2-tsdb.xml
#canal.instance.tsdb.spring.xml = classpath:spring/tsdb/mysql-tsdb.xml

canal.instance.global.mode = spring
canal.instance.global.lazy = false
#canal.instance.global.manager.address = 127.0.0.1:1099
#canal.instance.global.spring.xml = classpath:spring/memory-instance.xml
canal.instance.global.spring.xml = classpath:spring/file-instance.xml
#canal.instance.global.spring.xml = classpath:spring/default-instance.xml

##################################################
#########                    MQ                      #############
##################################################
#kafka为bootstrap.servers rocketMQ中为nameserver列表
canal.mq.servers = 192.168.88.20:9092
#发送失败重试次数
canal.mq.retries = 0
#kafka为ProducerConfig.BATCH_SIZE_CONFIG rocketMQ无意义
canal.mq.batchSize = 16384
#kafka为ProducerConfig.MAX_REQUEST_SIZE_CONFIG rocketMQ无意义
canal.mq.maxRequestSize = 1048576
#kafka为ProducerConfig.LINGER_MS_CONFIG , 如果是flatMessage格式建议将该值调大, 如: 200 rocketMQ无意义
canal.mq.lingerMs = 1
#kafka为ProducerConfig.BUFFER_MEMORY_CONFIG rocketMQ无意义
canal.mq.bufferMemory = 33554432
#获取canal数据的批次大小
canal.mq.canalBatchSize = 50
#获取canal数据的超时时间
canal.mq.canalGetTimeout = 100
#是否为json格式 如果设置为false,对应MQ收到的消息为protobuf格式 需要通过CanalMessageDeserializer进行解码
canal.mq.flatMessage = true
canal.mq.compressionType = none
canal.mq.acks = all
```

> 修改内容如下

```
#默认为TCP，也就是你通过官方的example可以在终端查看数据，我们修改为kafka
canal.serverMode = kafka
canal.destinations = example
#kafka地址
canal.mq.servers = 192.168.88.20:9092
```

> `3 修改 canal/conf/example目录中的 instance.properties  文件 `

```
vi /home/admin/canal-server/conf/example/instance.properties
#################################################
## mysql serverId , v1.0.26+ will autoGen 
# canal.instance.mysql.slaveId=0

# enable gtid use true/false
canal.instance.gtidon=false

# position info
canal.instance.master.address=192.168.88.10:3306
canal.instance.master.journal.name=
canal.instance.master.position=
canal.instance.master.timestamp=
canal.instance.master.gtid=

# rds oss binlog
canal.instance.rds.accesskey=
canal.instance.rds.secretkey=
canal.instance.rds.instanceId=

# table meta tsdb info
canal.instance.tsdb.enable=true
#canal.instance.tsdb.url=jdbc:mysql://127.0.0.1:3306/canal_tsdb
#canal.instance.tsdb.dbUsername=canal
#canal.instance.tsdb.dbPassword=canal

#canal.instance.standby.address =
#canal.instance.standby.journal.name =
#canal.instance.standby.position =
#canal.instance.standby.timestamp =
#canal.instance.standby.gtid=

# username/password
canal.instance.dbUsername=root
canal.instance.dbPassword=123456
canal.instance.connectionCharset = UTF-8
# 连接默认数据库
canal.instance.defaultDatabaseName =test
# enable druid Decrypt database password
canal.instance.enableDruid=false
#canal.instance.pwdPublicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALK4BUxdDltRRE5/zXpVEVPUgunvscYFtEip3pmLlhrWpacX7y7GCMo2/JM6LeHmiiNdH1FWgGCpUfircSwlWKUCAwEAAQ==

# table regex
canal.instance.filter.regex=test\\..*
# table black regex
canal.instance.filter.black.regex=

# mq config
#mq里的topic名
canal.mq.topic=example
canal.mq.partition=0
# hash partition config
#canal.mq.partitionsNum=3
#散列规则定义 库名.表名 : 唯一主键，比如mytest.person: id 1.1.3版本支持新语法
#canal.mq.partitionHash=mytest.person:id,mytest.role:id
#################################################
```

> 修改内容如下

```
# position info
canal.instance.master.address=192.168.88.10:3306
# username/password
canal.instance.dbUsername=root
canal.instance.dbPassword=123456
# 数据库及表过滤，这里我只抽取sourcedb的日志
canal.instance.filter.regex=test\\..*
# mq config
canal.mq.topic=example
```

> `4 重启canal-server`

```
bin/restart.sh
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Canal/7.png)

> `5 启动大数据服务器的kafka集群`

```
bin/kafka-server-start.sh config/server.properties > /dev/null 2>&1 &
```

> `6 启动kafka的消费者命令行`

```
cd /export/services/kafka
./kafka-simple-consumer-shell.sh  --broker-list node2.itcast.cn:9092  --topic example
或者
./kafka-console-consumer.sh --bootstrap-server node2.itcast.cn:9092 --topic example --from-beginning
```

> `7 在mysql数据库中插入一条sql语句`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Canal/8.png)

> `8 观察kafka消费者命令行的输出`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Canal/9.png)


### 3.3 常见错误

> `启动canal-server后，example.log日志错误如下`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Canal/10.png)

> `错误原因`：
>>启动docker时，docker进程会创建一个名为docker0的虚拟网桥，用于宿主机与容器之间的通信。当启动一个docker容器时，docker容器将会附加到虚拟网桥上，容器内的报文通过docker0向外转发。
如果docker容器访问宿主机，那么docker0网桥将报文直接转发到本机，报文的源地址是docker0网段的地址。而如果docker容器访问宿主机以外的机器，docker的SNAT网桥会将报文的源地址转换为宿主机的地址，通过宿主机的网卡向外发送。
因此，当docker容器访问宿主机时，如果宿主机服务端口会被防火墙拦截，从而无法连通宿主机，出现No route to host的错误。
而访问宿主机所在局域网内的其他机器，由于报文的源地址是宿主机ip，因此，不会被目的机器防火墙拦截，所以可以访问。
因此，也可以通过开放相对应的端口即可。

> `解决方式：`

```
firewall-cmd --zone=public --add-port=3306/tcp --permanent    
firewall-cmd --reload
```

> 确认下防火墙是否关闭，如果没有关闭需要关掉

```
查看防火墙状态：firewall-cmd --state
停止firewall：systemctl stop firewalld.service
禁止firewall开机启动：systemctl disable firewalld.service
```

## 4 Canal采集业务数据到Kafka

### 4.1 配置Canal-Server

> 1进入canal-server容器

```
docker exec -it canal-server bash
```

> 2 修改 canal/conf目录中的 canal.properties 文件

```
vi canal-server/conf/canal.properties
canal.mq.servers = 192.168.88.20:9092
```

> 3 修改 canal/conf/example目录中的 instance.properties 文件

```
canal.instance.master.address=192.168.88.20:3306
canal.instance.dbUsername=root
canal.instance.dbPassword=123456
canal.instance.defaultDatabaseName =itcast_crm
canal.instance.filter.regex=itcast_crm\\..*
```
> 4 重启canal-server

```
/home/admin/canal-server/bin/restart.sh
```

### 4.2 导入业务数据到Mysql

> 导入数据后查看kafka有没有摄取到数据

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
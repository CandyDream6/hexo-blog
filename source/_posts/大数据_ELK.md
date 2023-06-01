---
title: ELK 实时数据收集，存储，索引，检索
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(53).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(53).jpg
tags:
  - 大数据
  - ElasticSearch
category:
  - - 编程
    - 大数据
 
date: 2020-02-04 22:27:06
---

ELK 包含三款产品，分别是 `Elasticsearch`, `Logstash`, `Kibana`

是一套实时数据收集，存储，索引，检索，统计分析及可视化的解决方案。

最新版本已经改名为Elastic Stack，并新增了Beats项目。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 ElasticSearch简介

https://www.elastic.co/cn/elasticsearch

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/1.png)

### 1.1 简介

#### 1.1.1 介绍

> Elasticsearch是一个基于Lucene的搜索服务器、

> 提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口

> Elasticsearch是用Java语言开发的，并作为Apache许可条款下的开放源码发布，是一种流行的企业级搜索引擎。Elasticsearch用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。官方客户端在Java、.NET（C#）、PHP、Python、Apache Groovy、Ruby和许多其他语言中都是可用的

> 根据DB-Engines的排名显示，Elasticsearch是最受欢迎的企业搜索引擎，其次是Apache Solr，也是基于Lucene。

#### 1.1.2 创始人

> Shay Banon （谢巴农）

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/2.png)

### 1.2 可以做什么

#### 1.2.1 信息检索

> 电商 / 招聘 / 门户 / 论坛

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/3.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/4.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/5.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/6.png)

#### 1.2.2 企业内部系统搜索

> 关系型数据库使用like进行模糊检索，会导致索引失效，效率低下

> 可以基于Elasticsearch来进行检索，效率杠杠的

> OA / CRM / ERP

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/7.png)

#### 1.2.3 数据分析引擎

> Elasticsearch 聚合可以对数十亿行日志数据进行聚合分析，探索数据的趋势和规律。

### 1.3 特点

#### 1.3.1 海量数据处理

> 大型分布式集群（数百台规模服务器）

> 处理PB级数据

> 小公司也可以进行单机部署

#### 1.3.2 开箱即用

> 简单易用，操作非常简单

> 快速部署生产环境

#### 1.3.3 作为传统数据库的补充

> 传统关系型数据库不擅长全文检索（MySQL自带的全文索引，与ES性能差距非常大）

> 传统关系型数据库无法支持搜索排名、海量数据存储、分析等功能

> Elasticsearch可以作为传统关系数据库的补充，提供RDBM无法提供的功能

### 1.4 使用案例

> 2013年初，GitHub抛弃了Solr，采取ElasticSearch 来做PB级的搜索。 “GitHub使用ElasticSearch搜索20TB的数据，包括13亿文件和1300亿行代码”

> 维基百科：启动以elasticsearch为基础的核心搜索架构

> SoundCloud：“SoundCloud使用ElasticSearch为1.8亿用户提供即时而精准的音乐搜索服务”

> 百度：百度目前广泛使用ElasticSearch作为文本数据分析，采集百度所有服务器上的各类指标数据及用户自定义数据，通过对各种数据进行多维分析展示，辅助定位分析实例异常或业务层面异常。目前覆盖百度内部20多个业务线（包括casio、云分析、网盟、预测、文库、直达号、钱包、风控等），单集群最大100台机器，200个ES节点，每天导入30TB+数据

> 新浪使用ES 分析处理32亿条实时日志

> 阿里使用ES 构建挖财自己的日志采集和分析体系

### 1.5 ElasticSearch对比Solr

> Solr 利用 Zookeeper 进行分布式管理，而 Elasticsearch 自身带有分布式协调管理功能;

> Solr 支持更多格式的数据，而 Elasticsearch 仅支持json文件格式；

> Solr 官方提供的功能更多，而 Elasticsearch 本身更注重于核心功能，高级功能多有第三方插件提供；

> Solr 在传统的搜索应用中表现好于 Elasticsearch，但在处理实时搜索应用时效率明显低于 Elasticsearch

### 1.6 发展历史

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/8.png)

> 2004年，发布第一个版本名为Compass的搜索引擎，创建搜索引擎的目的主要是为了搜索食谱

> 2010年，发布第二个版本更名为Elasticsearch，基于Apache Lucene开发并开源

> 2012年，创办Elasticsearch公司

> 2015年，Elasticsearch公司更名为Elastic，是专门从事与Elasticsearch相关的商业服务，并衍生了Logstash和Kibana两个项目，填补了在数据采集、数据可视化的空白。于是，ELK就诞生了

> 2015年，Elastic公司将开源项目Packetbeat整合到Elasticsearch技术栈中，并更名为Beats，它专门用于数据采集的轻量级组件，可以将网络日志、度量、审计等各种数据作为不同的源头发送到Logstash或者Elasticsearch

> ELK不再包括Elastic公司所有的开源项目，ELK开始更名为Elastic Stack，将来还有更多的软件加入其中，包括数据采集、清洗、传输、存储、检索、分析、可视化等

> 2018年，Elastic公司在纽交所挂牌上市

## 2 Lucene全文检索库

### 2.1 什么是全文检索

#### 2.1.1 结构化数据与非结构化数据

> 结构化数据：指具有固定格式或有限长度的数据，如数据库，元数据等

> 非结构化数据：指不定长或无固定格式的数据，如邮件，word文档等磁盘上的文件

#### 2.1.2 搜索结构化数据和非结构化数据

> 使用SQL语言专门搜索结构化的数据

> 使用ES/Lucene/solr建立倒排索引，根据关键字就可以搜索一些非结构化(文本)的数据

#### 2.1.3 全文检索

> 通过一个程序扫描文本中的每一个单词，针对单词建立索引，并保存该单词在文本中的位置、以及出现的次数

> 用户查询时，通过之前建立好的索引来查询，将索引中单词对应的文本位置、出现的次数返回给用户，因为有了具体文本的位置，所以就可以将具体内容读取出来了

> 类似于通过字典中的检索字表查字的过程

### 2.2 Lucene简介

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/9.png)

> Lucene是一种高性能的全文检索库，在2000年开源，最初由大名鼎鼎的Doug Cutting（道格·卡丁）开发

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/10.png)

> Lucene是Apache的一个顶级开源项目，是一个全文检索引擎工具包。但Lucene不是一个完整的全文检索引擎，它只是提供一个基本的全文检索的架构，还提供了一些基本的文本分词库

> Lucene是一个简单易用的工具包，可以方便的实现全文检索的功能

### 2.3 倒排索引结构

> 倒排索引是一种建立索引的方法。是全文检索系统中常用的数据结构。通过倒排索引，就是根据单词快速获取包含这个单词的文档列表。倒排索引通常由两个部分组成：单词词典、文档。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/11.png)

### 2.4 企业中为什么不直接使用Lucene

#### 2.4.1 Lucene的内建不支持分布式

> Lucene是作为嵌入的类库形式使用的，本身是没有对分布式支持。

#### 2.4.2 区间范围搜索速度非常缓慢

> Lucene的区间范围搜索API是扩展补充的，对于在单个文档中term出现比较多的情况，搜索速度会变得很慢

> Lucene只有在数据生成索引文件之后（Segment），才能被查询到，做不到实时

#### 2.a4.3 可靠性无法保障

> 无法保障Segment索引段的可靠性

## 3 ElasticSearch核心概念

### 3.1 索引 index

> 一个索引就是一个拥有几分相似特征的文档的集合。比如说，可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引

> 一个索引由一个名字来标识（必须全部是小写字母的），并且当我们要对对应于这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字

> 在一个集群中，可以定义任意多的索引。

### 3.2 映射 mapping

> ElasticSearch中的映射（Mapping）用来定义一个文档

> mapping是处理数据的方式和规则方面做一些限制，如某个字段的数据类型、默认值、分析器、是否被索引等等，这些都是映射里面可以设置的

### 3.3 字段 field

> 相当于是数据表的字段，对文档数据根据不同属性进行的分类标识

### 3.4 类型 type

> 每一个字段都应该有一个对应的类型，例如：Text、Keyword、Byte等

### 3.5 文档 document

> 一个文档是一个可被索引的基础信息单元。比如，可以拥有某一个客户的文档，某一个产品的一个文档，当然，也可以拥有某个订单的一个文档。文档以JSON（Javascript Object Notation）格式来表示，而JSON是一个到处存在的互联网数据交互格式

### 3.6 集群 cluster

> 一个集群就是由一个或多个节点组织在一起，它们共同持有整个的数据，并一起提供索引和搜索功能

> 一个集群由一个唯一的名字标识，这个名字默认就是“elasticsearch”

> 这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群

### 3.7 节点 node

> 一个节点是集群中的一个服务器，作为集群的一部分，它存储数据，参与集群的索引和搜索功能

> 一个节点可以通过配置集群名称的方式来加入一个指定的集群。默认情况下，每个节点都会被安排加入到一个叫做“elasticsearch”的集群中

> 这意味着，如果在网络中启动了若干个节点，并假定它们能够相互发现彼此，它们将会自动地形成并加入到一个叫做“elasticsearch”的集群中

> 在一个集群里，可以拥有任意多个节点。而且，如果当前网络中没有运行任何Elasticsearch节点，这时启动一个节点，会默认创建并加入一个叫做“elasticsearch”的集群。

### 3.8 分片和副本 shards&replicas

#### 3.8.1 分片

> 一个索引可以存储超出单个结点硬件限制的大量数据。比如，一个具有10亿文档的索引占据1TB的磁盘空间，而任一节点都没有这样大的磁盘空间；或者单个节点处理搜索请求，响应太慢

> 为了解决这个问题，Elasticsearch提供了将索引划分成多份的能力，这些份就叫做分片

> 当创建一个索引的时候，可以指定你想要的分片的数量

> 每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上

> 分片很重要，主要有两方面的原因
>> 允许水平分割/扩展你的内容容量
>>
>> 允许在分片之上进行分布式的、并行的操作，进而提高性能/吞吐量

> 至于一个分片怎样分布，它的文档怎样聚合回搜索请求，是完全由Elasticsearch管理的，对于作为用户来说，这些都是透明的

#### 3.8.2 副本

> 在一个网络/云的环境里，失败随时都可能发生，在某个分片/节点不知怎么的就处于离线状态，或者由于任何原因消失了，这种情况下，有一个故障转移机制是非常有用并且是强烈推荐的。为此目的，Elasticsearch允许你创建分片的一份或多份拷贝，这些拷贝叫做副本分片，或者直接叫副本

> 副本之所以重要，有两个主要原因
>> 在分片/节点失败的情况下，提供了高可用性。注意到复制分片从不与原/主要（original/primary）分片置于同一节点上是非常重要的
>>
>> 扩展搜索量/吞吐量，因为搜索可以在所有的副本上并行运行

> 每个索引可以被分成多个分片。一个索引有0个或者多个副本

> 一旦设置了副本，每个索引就有了主分片和副本分片，分片和副本的数量可以在索引创建的时候指定

> 在索引创建之后，可以在任何时候动态地改变副本的数量，但是不能改变分片的数量

## 4 ElasticSearch安装

### 4.1 ElasticSearch安装

#### 4.1.1 创建普通用户

> `ES不能使用root用户来启动，必须使用普通用户来安装启动`。这里我们创建一个普通用户以及定义一些常规目录用于存放我们的数据文件以及安装包等。

> 创建一个es专门的用户（`必须`）

> 使用root用户在三台机器执行以下命令

```
useradd itcast
passwd itcast
```

#### 4.1.2 为普通用户itcast添加sudo权限

> 为了让普通用户有更大的操作权限，我们一般都会给普通用户设置sudo权限，方便普通用户的操作

> `三台机器使用root用户执行`visudo命令然后为es用户添加权限

```
visudo
# 第100行
itcast      ALL=(ALL)       ALL
```

#### 4.1.3 上传压缩包并解压

> 以下操作 使用创建目录, 三台虚拟机都需要创建

```
# 在node1.itcast.cn、node2.itcast.cn、node3.itcast.cn创建es文件夹，并修改owner为itcast用户
mkdir -p /export/servers/es
chown -R itcast:itcast /export/servers/es
```

> 将es的安装包下载并上传到node1.itcast.cn服务器的/export/software路径下，然后进行解压

> `使用itcast用户`来执行以下操作，将es安装包上传到node1.itcast.cn服务器，并使用es用户执行以下命令解压。

```
# 解压Elasticsearch
cd ~
tar -zvxf elasticsearch-7.6.1-linux-x86_64.tar.gz -C /export/servers/es/
```

#### 4.1.4 修改配置文件

##### 4.1.4.1 修改elasticsearch.yml

> node1.itcast.cn服务器使用itcast用户来修改配置文件

```
cd /export/servers/es/elasticsearch-7.6.1/config
mkdir -p /export/servers/es/elasticsearch-7.6.1/log
mkdir -p /export/servers/es/elasticsearch-7.6.1/data
rm -rf elasticsearch.yml

vim elasticsearch.yml
cluster.name: itcast-es
node.name: node1.itcast.cn
path.data: /export/servers/es/elasticsearch-7.6.1/data
path.logs: /export/servers/es/elasticsearch-7.6.1/log
network.host: node1.itcast.cn
http.port: 9200
discovery.seed_hosts: ["node1.itcast.cn", "node2.itcast.cn", "node3.itcast.cn"]
cluster.initial_master_nodes: ["node1.itcast.cn", "node2.itcast.cn"]
bootstrap.system_call_filter: false
bootstrap.memory_lock: false
http.cors.enabled: true
http.cors.allow-origin: "*"
```

##### 4.2.4.2 修改jvm.option

> 修改jvm.option配置文件，调整jvm堆内存大小

> `node1.itcast.cn`使用itcast用户执行以下命令调整jvm堆内存大小，每个人根据自己服务器的内存大小来进行调整。

```
cd /export/servers/es/elasticsearch-7.6.1/config
vim jvm.options
-Xms2g
-Xmx2g
```

#### 4.1.5 将安装包分发到其他服务器上面

> node1.itcast.cn使用itcast用户将安装包分发到其他服务器上面去

```
cd /export/servers/es/
scp -r elasticsearch-7.6.1/ node2.itcast.cn:$PWD
scp -r elasticsearch-7.6.1/ node3.itcast.cn:$PWD
```

#### 4.1.6 node2.itcast.cn与node3.itcast.cn修改es配置文件

> node2.itcast.cn与node3.itcast.cn也需要修改es配置文件
> node2.itcast.cn使用itcast用户执行以下命令修改es配置文件

```
cd /export/servers/es/elasticsearch-7.6.1/config
mkdir -p /export/servers/es/elasticsearch-7.6.1/log
mkdir -p /export/servers/es/elasticsearch-7.6.1/data

vim elasticsearch.yml
cluster.name: itcast-es
node.name: node2.itcast.cn
path.data: /export/servers/es/elasticsearch-7.6.1/data
path.logs: /export/servers/es/elasticsearch-7.6.1/log
network.host: node2.itcast.cn
http.port: 9200
discovery.seed_hosts: ["node1.itcast.cn", "node2.itcast.cn", "node3.itcast.cn"]
cluster.initial_master_nodes: ["node1.itcast.cn", "node2.itcast.cn"]
bootstrap.system_call_filter: false
bootstrap.memory_lock: false
http.cors.enabled: true
http.cors.allow-origin: "*"

```

> node3.itcast.cn使用itcast用户执行以下命令修改配置文件

```
cd /export/servers/es/elasticsearch-7.6.1/config
mkdir -p /export/servers/es/elasticsearch-7.6.1/log
mkdir -p /export/servers/es/elasticsearch-7.6.1/data

vim elasticsearch.yml
cluster.name: itcast-es
node.name: node3.itcast.cn
path.data: /export/servers/es/elasticsearch-7.6.1/data
path.logs: /export/servers/es/elasticsearch-7.6.1/log
network.host: node3.itcast.cn
http.port: 9200
discovery.seed_hosts: ["node1.itcast.cn", "node2.itcast.cn", "node3.itcast.cn"]
cluster.initial_master_nodes: ["node1.itcast.cn", "node2.itcast.cn"]
bootstrap.system_call_filter: false
bootstrap.memory_lock: false
http.cors.enabled: true
http.cors.allow-origin: "*"
```

#### 4.1.7 修改系统配置，解决启动时候的问题

> 由于现在使用普通用户来安装es服务，且es服务对服务器的资源要求比较多，包括内存大小，线程数等。所以我们需要给普通用户解开资源的束缚

##### 4.1.7.1 普通用户打开文件的最大数限制

`问题错误信息描述：`

```
max file descriptors [4096] for elasticsearch process likely too low, increase to at least [65536]
```

> ES因为需要大量的创建索引文件，需要大量的打开系统的文件，所以我们需要解除linux系统当中打开文件最大数目的限制，不然ES启动就会抛错

> `三台机器使用itcast用户执行以下命令解除打开文件数据的限制`

```
sudo vi /etc/security/limits.conf
```

> 添加如下内容: 注意*不要去掉了

```
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```

> `此文件修改后需要重新登录用户，才会生效`

##### 4.1.7.2 普通用户启动线程数限制

`问题错误信息描述`

> 修改普通用户可以创建的最大线程数

```
max number of threads [1024] for user [es] likely too low, increase to at least [4096]
```

> 原因：无法创建本地线程问题,用户最大可创建线程数太小

> 解决方案：修改90-nproc.conf 配置文件。

> 三台机器使用itcast用户执行以下命令修改配置文件

```
Centos6
sudo vi /etc/security/limits.d/90-nproc.conf
Centos7
sudo vi /etc/security/limits.d/20-nproc.conf
```

找到如下内容：

```
* soft nproc 1024
```

#修改为

```
* soft nproc 4096
```

##### 4.1.7.3 普通用户调大虚拟内存

`错误信息描述：`

```
max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
```

>调大系统的虚拟内存<br/>
原因：最大虚拟内存太小<br/>
每次启动机器都手动执行下。<br/>

> 三台机器执行以下命令
```
sudo  sysctl -w vm.max_map_count=262144

sudo vim /etc/sysctl.conf
在最后添加一行
vm.max_map_count=262144
```

> `备注：以上三个问题解决完成之后，重新连接secureCRT或者重新连接xshell生效`

#### 4.1.8 启动ES服务

> 三台机器使用itcast用户执行以下命令启动es服务

```
nohup /export/servers/es/elasticsearch-7.6.1/bin/elasticsearch 2>&1 &
```

> 启动成功之后jsp即可看到es的服务进程，并且访问页面

```
http://node1.itcast.cn:9200/?pretty
http://node2.itcast.cn:9200/?pretty
http://node3.itcast.cn:9200/?pretty
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/12.png)

> 能够看到es启动之后的一些信息

> 注意：如果哪一台机器服务启动失败，那么就到哪一台机器的

```
/export/servers/es/elasticsearch-7.6.1/log
```

> 这个路径下面去查看错误日志

### 4.2 ElasticSearch-head插件

> 由于es服务启动之后，访问界面比较丑陋，为了更好的查看索引库当中的信息，我们可以通过安装elasticsearch-head这个插件来实现，这个插件可以更方便快捷的看到es的管理界面

> elasticsearch-head这个插件是es提供的一个用于图形化界面查看的一个插件工具，可以安装上这个插件之后，通过这个插件来实现我们通过浏览器查看es当中的数据

> 安装elasticsearch-head这个插件这里提供两种方式进行安装，第一种方式就是自己下载源码包进行编译，耗时比较长，网络较差的情况下，基本上不可能安装成功。第二种方式就是直接使用我已经编译好的安装包，进行修改配置即可

> 要安装elasticsearch-head插件，需要先安装Node.js

#### 4.2.1 安装nodejs

> Node.js是一个基于 Chrome V8 引擎的 JavaScript 运行环境。

> Node.js是一个Javascript运行环境(runtime environment)，发布于2009年5月，由Ryan Dahl开发，实质是对Chrome V8引擎进行了封装。Node.js 不是一个 JavaScript 框架，不同于CakePHP、Django、Rails。Node.js 更不是浏览器端的库，不能与 jQuery、ExtJS 相提并论。Node.js 是一个让 JavaScript 运行在服务端的开发平台，它让 JavaScript 成为与PHP、Python、Perl、Ruby 等服务端语言平起平坐的脚本语言。

> 安装步骤参考：https://www.cnblogs.com/kevingrace/p/8990169.html

##### 4.2.1.1 下载安装包

> node1.itcast.cn机器执行以下命令下载安装包，然后进行解压

```
cd ~
wget https://npm.taobao.org/mirrors/node/v8.1.0/node-v8.1.0-linux-x64.tar.gz
tar -zxvf node-v8.1.0-linux-x64.tar.gz -C /export/servers/es/
```

##### 4.2.1.2 创建软连接

> node1.itcast.cn执行以下命令创建软连接

```
sudo ln -s /export/servers/es/node-v8.1.0-linux-x64/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm
sudo ln -s /export/servers/es/node-v8.1.0-linux-x64/bin/node /usr/local/bin/node
```

##### 4.2.1.3 修改环境变量

> node1.itcast.cn服务器添加环境变量

```
sudo vim /etc/profile
export NODE_HOME=/export/servers/es/node-v8.1.0-linux-x64
export PATH=:$PATH:$NODE_HOME/bin
```

> 修改完环境变量使用source生效

```
source /etc/profile
```

##### 4.2.1.4 验证安装成功

> node1.itcast.cn执行以下命令验证安装生效

```
node -v
npm -v
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/13.png)

#### 4.2.2 本地安装（推荐）

##### 4.2.2.1 上传压缩包到/export/software路径下去

> 将我们的压缩包  elasticsearch-head-compile-after.tar.gz  上传到node1.itcast.cn机器的/export/software 路径下面去

##### 4.2.2.2 解压安装包

> node1.itcast.cn执行以下命令解压安装包

```
cd ~
tar -zxvf elasticsearch-head-compile-after.tar.gz -C /export/servers/es/
```

##### 4.2.2.3 node1机器修改Gruntfile.js

> 修改Gruntfile.js这个文件

```
cd /export/servers/es/elasticsearch-head
vim Gruntfile.js
```

> 找到代码中的93行：hostname: '192.168.100.100', 修改为：node1.itcast.cn

```
connect: {
    server: {
        options: {
        hostname: 'node1.itcast.cn',
        port: 9100,
        base: '.',
        keepalive: true
        }
    }
}
```

##### 4.2.2.4 node1机器修改app.js

> 第一台机器修改app.js

```
cd /export/servers/es/elasticsearch-head/_site
vim app.js
```

> 在Vim中输入「:4354」，定位到第4354行，修改 http://localhost:9200为http://node1.itcast.cn:9200。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/14.png)

##### 4.2.2.5 启动head服务

> ode1.itcast.cn启动elasticsearch-head插件

```
cd /export/servers/es/elasticsearch-head/node_modules/grunt/bin/
```

> 进程前台启动命令

```
./grunt server
```

> 进程后台启动命令

```
nohup ./grunt server >/dev/null 2>&1 &

Running "connect:server" (connect) task
Waiting forever...
Started connect web server on http://192.168.52.100:9100
```

> `如何停止`：elasticsearch-head进程

> 执行以下命令找到elasticsearch-head的插件进程，然后使用kill  -9  杀死进程即可

```
netstat -nltp | grep 9100
kill -9 8328
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/15.png)

#### 4.2.3 在线安装（网速慢，不推荐）

> 这里选择node1.itcast.cn进行安装

##### 4.2.3.1 在线安装必须依赖包

> 初始化目录

```
cd /export/serverss/es
```

> 安装GCC

```
sudo yum install -y gcc-c++ make git
```

##### 4.2.3.2 从git上面克隆编译包并进行安装

```
cd /export/serverss/es
git clone https://github.com/mobz/elasticsearch-head.git
# 进入安装目录
cd /export/serverss/es/elasticsearch-head
# intall 才会有 node-modules
npm install
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/16.png)

> 以下进度信息

```
npm WARN notice [SECURITY] lodash has the following vulnerability: 1 low. Go here for more details:
npm WARN notice [SECURITY] debug has the following vulnerability: 1 low. Go here for more details: https://nodesecurity.io/advisories?search=debug&version=0.7.4 - Run `npm i npm@latest -g` to upgrade your npm version, and then `npm audit` to get more info.
npm ERR! Unexpected end of input at 1:2096
npm ERR! 7c1a1bc21c976bb49f3ea","tarball":"https://registry.npmjs.org/safer-bu
npm ERR!                                                                      ^
npm ERR! A complete log of this run can be found in:
npm ERR!     /home/es/.npm/_logs/2018-11-27T14_35_39_453Z-debug.log
以上错误可以不用管。
```

##### 4.2.3.3 node1机器修改Gruntfile.js

> 第一台机器修改Gruntfile.js这个文件

```
cd /export/serverss/es/elasticsearch-head
vim Gruntfile.js
```

找到以下代码：

添加一行： hostname: '192.168.52.100', //192.168.52.100是虚拟机ip


```
connect: {
    server: {
        options: {
        hostname: '192.168.52.100',
        port: 9100,
        base: '.',
        keepalive: travelue
        }
    }
}
```

##### 4.2.3.4 node01机器修改app.js

> 第一台机器修改app.js

```
cd /export/serverss/es/elasticsearch-head/_site
vim app.js
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/17.png)

>`更改前：http://localhost:9200`<br/>
`更改后：http://node01:9200`

#### 4.2.4 访问elasticsearch-head界面

> http://node1.itcast.cn:9100/

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/18.png)

### 4.3 安装IK分词器

> 我们后续也需要使用Elasticsearch来进行中文分词，所以需要单独给Elasticsearch安装IK分词器插件。以下为具体安装步骤：

> 1.下载Elasticsearch IK分词器

```
https://github.com/medcl/elasticsearch-analysis-ik/releases
```

> 2.切换到itcast用户，并在es的安装目录下/plugins创建ik

```
mkdir -p /export/servers/es/elasticsearch-7.6.1/plugins/ik
```

> 3.将下载的ik分词器上传并解压到该目录

```
cd /export/servers/es/elasticsearch-7.6.1/plugins/ik
sudo rz
unzip elasticsearch-analysis-ik-7.6.1.zip
```

> 4.将plugins下的ik目录分发到每一台服务器

```
cd /export/servers/es/elasticsearch-7.6.1/plugins
scp -r ik/ node2.itcast.cn:$PWD
scp -r ik/ node3.itcast.cn:$PWD
```

> 5.重启Elasticsearch

### 4.4 准备VSCode开发环境

> 在VScode中安装Elasticsearch for VScode插件。该插件可以直接与Elasticsearch交互，开发起来非常方便。

> 打开VSCode，在应用商店中搜索elasticsearch，找到Elasticsearch for VSCode,安装

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/19.png)

### 4.5 测试分词器

> 1.打开VSCode

> 2.新建一个文件，命名为 0.IK分词器测试.es

> 3.右键点击 命令面板 菜单

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/20.png)

> 4.选择ES:Elastic: Set Host，然后输入Elasticsearch的机器名和端口号。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/21.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/22.png)

> 5.将以下内容复制到ES中，并测试。

`Standard标准分词器：`

```
post _analyze
{
"analyzer":"standard",
"text":"我爱你中国"
}
```

> 能看出来Standard标准分词器，是一个个将文字切分。并不是我们想要的结果。

`IK分词器：`

```
post _analyze
{
"analyzer":"ik_max_word",
"text":"我爱你中国"
}
```

> IK分词器，切分为了“我爱你”、“爱你”、“中国”，这是我们想要的效果。

`注意：`
`analyzer中的单词一定要写对，不能带有多余的空格，否则会报错：找不到对应名字的解析器。`

## 5 ElasticSearch基础使用

### 5.1 创建索引 

> 为了能够搜索职位数据，我们需要提前在Elasticsearch中创建索引，然后才能进行关键字的检索。这里先回顾下，我们在MySQL中创建表的过程。在MySQL中，如果我们要创建一个表，我们需要指定表的名字，指定表中有哪些列、列的类型是什么。同样，在Elasticsearch中，也可以使用类似的方式来定义索引。

#### 5.1.1 创建带有映射的索引

> Elasticsearch中，我们可以使用RESTful API（http请求）来进行索引的各种操作。创建MySQL表的时候，我们使用DDL来描述表结构、字段、字段类型、约束等。在Elasticsearch中，我们使用Elasticsearch的DSL来定义——使用JSON来描述。例如：

```
PUT /my-index
{
    "mapping": {
        "properties": {
            "employee-id": {
            "type": "keyword",
            "index": false
            }
        }
    }
}
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/23.png)

#### 5.1.2 字段的类型

> 在Elasticsearch中，每一个字段都有一个类型（type）。以下为Elasticsearch中可以使用的类型：

类型名称 | 说明
---|---
text | 需要进行全文检索的字段，通常使用text类型来对应邮件的正文、产品描述或者短文等非结构化文本数据。分词器先会将文本进行分词转换为词条列表。将来就可以基于词条来进行检索了。文本字段不能用户排序、也很少用户聚合计算。
keyword | 使用keyword来对应结构化的数据，如ID、电子邮件地址、主机名、状态代码、邮政编码或标签。可以使用keyword来进行排序或聚合计算。注意：keyword是不能进行分词的。
date | 保存格式化的日期数据，例如：2015-01-01或者2015/01/01 12:10:30。在Elasticsearch中，日期都将以字符串方式展示。可以给date指定格式：”format”: “yyyy-MM-dd HH:mm:ss”
long/integer/short/byte | 4位整数/32位整数/16位整数/8位整数
double/float/half_float | 64位双精度浮点/32位单精度浮点/16位半进度浮点
boolean | “true”/”false”
ip | IPV4（192.168.1.110）/IPV6（192.168.0.0/16）
JSON分层嵌套类型 | object	用于保存JSON对象
nested | 用于保存JSON数组
geo_point | 用于保存经纬度坐标
geo_shape | 用于保存地图上的多边形坐标

#### 5.1.3 创建索引

> 索引名为 /job_idx<br/>
判断是使用text、还是keyword，主要就看是否需要分词

```
PUT /job_idx
{
    "mappings": {
        "properties" : {
            "area": { "type": "text", "store": true},
            "exp": { "type": "text", "store": true},
            "edu": { "type": "keyword", "store": true},
            "salary": { "type": "keyword", "store": true},
            "job_type": { "type": "keyword", "store": true},
            "cmp": { "type": "text", "store": true},
            "pv": { "type": "keyword", "store": true},
            "title": { "type": "text", "store": true},
            "jd": { "type": "text", "store": true}
        }
    }
}
```

#### 5.1.4 查看索引映射

>  查看索引映射

```
GET /job_idx/_mapping
```

> 使用head插件也可以查看到索引映射信息。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/24.png)

#### 5.1.5 查看ElasticSearch中的所有索引

```
GET _cat/indices
```

#### 5.1.6 删除索引

```
delete /job_idx
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/25.png)

#### 5.1.7 指定使用IK分词器

> 因为存放在索引库中的数据，是以中文的形式存储的。所以，为了有更好地分词效果，我们需要使用IK分词器来进行分词。这样，将来搜索的时候才会更准确。

```
PUT /job_idx
{
    "mappings": {
        "properties" : {
            "area": { "type": "text", "store": true, "analyzer": "ik_max_word"},
            "exp": { "type": "text", "store": true, "analyzer": "ik_max_word"},
            "edu": { "type": "keyword", "store": true},
            "salary": { "type": "keyword", "store": true},
            "job_type": { "type": "keyword", "store": true},
            "cmp": { "type": "text", "store": true, "analyzer": "ik_max_word"},
            "pv": { "type": "keyword", "store": true},
            "title": { "type": "text", "store": true, "analyzer": "ik_max_word"},
            "jd": { "type": "text", "store": true, "analyzer": "ik_max_word"}
        }
    }
}
```

### 5.2 添加一条数据

```
PUT /customer/_doc/1
{
    "name": "John Doe"
}
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/26.png)

> 如果在customer中，不存在ID为1的文档，Elasticsearch会自动创建

```
PUT /job_idx/_doc/29097
{
    "area": "深圳-南山区",
    "exp": "1年经验",
    "edu": "大专以上",
    "salary": "6-8千/月",
    "job_type": "实习",
    "cmp": "乐有家",
    "pv": "61.6万人浏览过  / 14人评价  / 113人正在关注",
    "title": "桃园 深大销售实习 岗前培训",
    "jd": "薪酬待遇】 本科薪酬7500起 大专薪酬6800起 以上无业绩要求，同时享有业绩核算比例55%~80% 人均月收入超1.3万 【岗位职责】 1.爱学习，有耐心： 通过公司系统化培训熟悉房地产基本业务及相关法律、金融知识，不功利服务客户，耐心为客户在房产交易中遇到的各类问题； 2.会聆听，会提问： 详细了解客户的核心诉求，精准匹配合适的产品信息，具备和用户良好的沟通能力，有团队协作意识和服务意识； 3.爱琢磨，善思考: 热衷于用户心理研究，善于从用户数据中提炼用户需求，利用个性化、精细化运营手段，提升用户体验。 【岗位要求】 1.18-26周岁，自考大专以上学历； 2.具有良好的亲和力、理解能力、逻辑协调和沟通能力； 3.积极乐观开朗，为人诚实守信，工作积极主动，注重团队合作； 4.愿意服务于高端客户，并且通过与高端客户面对面沟通有意愿提升自己的综合能力； 5.愿意参加公益活动，具有爱心和感恩之心。 【培养路径】 1.上千堂课程;房产知识、营销知识、交易知识、法律法规、客户维护、目标管理、谈判技巧、心理学、经济学; 2.成长陪伴：一对一的师徒辅导 3.线上自主学习平台：乐有家学院，专业团队制作，每周大咖分享 4.储备及管理课堂： 干部训练营、月度/季度管理培训会 【晋升发展】 营销【精英】发展规划：A1置业顾问-A6资深置业专家 营销【管理】发展规划：（入职次月后就可竞聘） 置业顾问-置业经理-店长-营销副总经理-营销副总裁-营销总裁 内部【竞聘】公司职能岗位：如市场、渠道拓展中心、法务部、按揭经理等都是内部竞聘 【联系人】 黄媚主任15017903212（微信同号）"
}
```

> Elasticsearch响应结果：

```
{
    "_index": "job_idx",
    "_type": "_doc",
    "_id": "29097",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 2,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

> 使用ES-head插件浏览数据：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/27.png)

### 5.3 修改数据

> 需要将原有的薪资6-8千/月，修改为15-20千/月

```
POST /job_idx/_update/29097
{
    "doc": {
        "salary": "15-20千/月"
    }
}
```

### 5.4 删除数据

> ID为29097的职位，已经被取消。所以，我们需要在索引库中也删除该岗位。

```
DELETE /job_idx/_doc/29097
```

### 5.5 批量导入JSON数据

> 为了方便后面的测试，我们需要先提前导入一些测试数据到ES中。在资料文件夹中有一个job_info.json数据文件。我们可以使用Elasticsearch中自带的bulk接口来进行数据导入。

> 1.上传JSON数据文件到Linux<br/>
2.执行导入命令

```
curl -H "Content-Type: application/json" -XPOST "node1.itcast.cn:9200/job_idx/_bulk?pretty&refresh" --data-binary "@job_info.json"
```

> 查看索引状态

```
GET _cat/indices?index=job_idx
```

> 通过执行以上请求，Elasticsearch返回数据如下：

```
[
    {
        "health": "green",
        "status": "open",
        "index": "job_idx",
        "uuid": "Yucc7A-TRPqnrnBg5SCfXw",
        "pri": "1",
        "rep": "1",
        "docs.count": "6765",
        "docs.deleted": "0",
        "store.size": "23.1mb",
        "pri.store.size": "11.5mb"
    }
]
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/28.png)

### 5.6 根据ID检索数据

> 用户提交一个文档ID，Elasticsearch将ID对应的文档直接返回给用户。

```
GET /job_idx/_search
{
    "query": {
        "ids": {
            "values": ["46313"]
        }
    }
}
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/29.png)

### 5.7 根据关键字检索数据

> 搜索职位中带有「销售」关键字的职位

```
GET  /job_idx/_search
{
    "query": {
        "match": {
            "jd": "销售"
        }
    }
}
```

> 除了检索职位描述字段以外，我们还需要检索title中包含销售相关的职位，所以，我们需要进行多字段的组合查询。

```
GET  /job_idx/_search
{
    "query": {
        "multi_match": {
            "query": "销售",
            "fields": ["title", "jd"]
        }
    }
}
```

> 更多地查询：<br/>
官方地址：https://www.elastic.co/cn/webinars/getting-started-elasticsearch?baymax=rtp&elektra=docs&storm=top-video&iesrc=ctr

### 5.8 根据关键字分页搜索

> 在存在大量数据时，一般我们进行查询都需要进行分页查询。例如：我们指定页码、并指定每页显示多少条数据，然后Elasticsearch返回对应页码的数据。

#### 5.8.1 使用from和size来进行分页

> 在执行查询时，可以指定from（从第几条数据开始查起）和size（每页返回多少条）数据，就可以轻松完成分页。

```
from = (page – 1) * size
```

```
GET  /job_idx/_search
{
    "from": 0,
    "size": 5,
    "query": {
        "multi_match": {
            "query": "销售",
            "fields": ["title", "jd"]
        }
    }
}
```

#### 5.8.2 使用scroll方式进行分页

> 前面使用from和size方式，查询在1W-5W条数据以内都是OK的，但如果数据比较多的时候，会出现性能问题。Elasticsearch做了一个限制，不允许查询的是10000条以后的数据。如果要查询1W条以后的数据，需要使用Elasticsearch中提供的scroll游标来查询。

> 在进行大量分页时，每次分页都需要将要查询的数据进行重新排序，这样非常浪费性能。使用scroll是将要用的数据一次性排序好，然后分批取出。性能要比from + size好得多。使用scroll查询后，排序后的数据会保持一定的时间，后续的分页查询都从该快照取数据即可。

## 6 ElasticSearch编程

### 6.1 `pom`

```xml
<dependencies>
        <!-- lucene核心类库 -->
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-core</artifactId>
            <version>8.4.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-analyzers-common</artifactId>
            <version>8.4.0</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>
        <dependency>
            <groupId>com.jianggujin</groupId>
            <artifactId>IKAnalyzer-lucene</artifactId>
            <version>8.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.6.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.11.1</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.62</version>
        </dependency>
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>6.14.3</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

### 6.2 `entity`

```
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.annotation.JSONField;

public class JobDetail {
    // 因为此处无需将id序列化为文档中
    @JSONField(serialize = false)
    private long id;            // 唯一标识
    private String area;        // 职位所在区域
    private String exp;         // 岗位要求的工作经验
    private String edu;         // 学历要求
    private String salary;      // 薪资范围
    private String job_type;    // 职位类型（全职/兼职）
    private String cmp;         // 公司名
    private String pv;          // 浏览量
    private String title;       // 岗位名称
    private String jd;          // 职位描述

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getArea() {
        return area;
    }

    public void setArea(String area) {
        this.area = area;
    }

    public String getExp() {
        return exp;
    }

    public void setExp(String exp) {
        this.exp = exp;
    }

    public String getEdu() {
        return edu;
    }

    public void setEdu(String edu) {
        this.edu = edu;
    }

    public String getSalary() {
        return salary;
    }

    public void setSalary(String salary) {
        this.salary = salary;
    }

    public String getJob_type() {
        return job_type;
    }

    public void setJob_type(String job_type) {
        this.job_type = job_type;
    }

    public String getCmp() {
        return cmp;
    }

    public void setCmp(String cmp) {
        this.cmp = cmp;
    }

    public String getPv() {
        return pv;
    }

    public void setPv(String pv) {
        this.pv = pv;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getJd() {
        return jd;
    }

    public void setJd(String jd) {
        this.jd = jd;
    }

    @Override
    public String toString() {
        return "JobDetail{" +
                "id=" + id +
                ", area='" + area + '\'' +
                ", exp='" + exp + '\'' +
                ", edu='" + edu + '\'' +
                ", salary='" + salary + '\'' +
                ", job_type='" + job_type + '\'' +
                ", cmp='" + cmp + '\'' +
                ", pv='" + pv + '\'' +
                ", title='" + title + '\'' +
                ", jd='" + jd + '\'' +
                '}';
    }


    public static void main(String[] args) {

        JobDetail jobDetail = new JobDetail();
        jobDetail.setId(1);
        jobDetail.setArea("北京-昌平");
        jobDetail.setExp("1年以上");
        jobDetail.setEdu("本科以上");
        jobDetail.setSalary("10k~15k");
        jobDetail.setJob_type("全职");
        jobDetail.setCmp("传智播客");
        jobDetail.setPv("10000pv");
        jobDetail.setTitle("大数据工程师");
        jobDetail.setJd("必须培训过....");

        // 想吧上述对象, 转换为json的字符串:
        String json = JSONObject.toJSONString(jobDetail);
        System.out.println(json);


        JobDetail jobDetail1 = JSONObject.parseObject(json, JobDetail.class);

        System.out.println(jobDetail1);

    }
```

### 6.3 `service`

```
import cn.itcast.elasticsearch.entity.JobDetail;

import java.io.IOException;
import java.util.List;
import java.util.Map;

public interface JobFullTextService {
    // 添加一个职位数据
    void add(JobDetail jobDetail);

    // 根据ID检索指定职位数据
    JobDetail findById(long id) throws IOException;

    // 修改职位薪资
    void update(JobDetail jobDetail) throws IOException;

    // 根据ID删除指定位置数据
    void deleteById(long id) throws IOException;

    // 根据关键字检索数据
    List<JobDetail> searchByKeywords(String keywords) throws IOException;

    // 分页检索
    Map<String, Object> searchByPage(String keywords, int pageNum, int pageSize) throws IOException;

    // scroll分页解决深分页问题
    Map<String, Object> searchByScrollPage(String keywords, String scrollId, int pageSize) throws IOException;
}
```
`
### 6.4 `serviceImpl`

```
import cn.itcast.elasticsearch.entity.JobDetail;
import cn.itcast.elasticsearch.service.JobFullTextService;
import com.alibaba.fastjson.JSONObject;
import org.apache.http.HttpHost;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.search.SearchScrollRequest;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestClientBuilder;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.text.Text;
import org.elasticsearch.common.unit.TimeValue;
import org.elasticsearch.common.xcontent.XContentType;
import org.elasticsearch.index.query.MatchQueryBuilder;
import org.elasticsearch.index.query.MultiMatchQueryBuilder;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightField;

import javax.swing.event.CaretListener;
import java.io.IOException;
import java.util.List;
import java.util.Map;

public class JobFullTextServiceImpl implements JobFullTextService {
    private   RestHighLevelClient esClient ;
    public JobFullTextServiceImpl() {
        RestClientBuilder clientBuilder = RestClient.builder(
                new HttpHost("node1.itcast.cn", 9200),
                new HttpHost("node2.itcast.cn", 9200),
                new HttpHost("node3.itcast.cn", 9200)
        );
        esClient = new RestHighLevelClient(clientBuilder);
    }

    // 添加一个职位数据
    @Override
    public void add(JobDetail jobDetail) {
        try {
            IndexRequest indexRequest = new IndexRequest("job_idx");
            indexRequest.id(jobDetail.getId() + "");  // 设置文档的id

            String json = JSONObject.toJSONString(jobDetail);
            indexRequest.source(json, XContentType.JSON); // 设置文档的数据

            esClient.index(indexRequest, RequestOptions.DEFAULT); //添加索引

            esClient.close();
        }catch (Exception e) {
            e.printStackTrace();
        }

    }
    // 根据ID检索指定职位数据
    @Override
    public JobDetail findById(long id) throws IOException {
        // 1.1: 封装查询条件
        GetRequest getRequest = new GetRequest("job_idx");

        getRequest.id(id+"");

        // 1 发送请求, 获取结果:
        GetResponse response = esClient.get(getRequest, RequestOptions.DEFAULT);

        //2. 处理结果集
        String source = response.getSourceAsString();

        System.out.println(source);

        JobDetail jobDetail = JSONObject.parseObject(source, JobDetail.class);
        jobDetail.setId(id);

        //3. 释放资源
        esClient.close();


        return jobDetail;
    }
    // 修改职位薪资
    @Override
    public void update(JobDetail jobDetail) throws IOException {
        //1.1 封装修改内容
        UpdateRequest updateRequest = new UpdateRequest("job_idx",jobDetail.getId()+"");

        String json = JSONObject.toJSONString(jobDetail);
        updateRequest.doc(json,XContentType.JSON);

        //1. 发送请求执行操作
        esClient.update(updateRequest,RequestOptions.DEFAULT);

    }
    // 根据ID删除指定位置数据
    @Override
    public void deleteById(long id) throws IOException {
        DeleteRequest deleteRequest = new DeleteRequest("job_idx",id+"");

        esClient.delete(deleteRequest,RequestOptions.DEFAULT);

    }
    // 根据关键字检索数据
    @Override
    public List<JobDetail> searchByKeywords(String keywords) throws IOException {
        //1.1: 创建查询的请求对象: 指定要查询那个索引库
        SearchRequest searchRequest = new SearchRequest("job_idx");


        //1.3: 创建 查询 数据的构建器对象
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        //1.4: 封装查询的条件:  MatchQueryBuilder 关键词查询   MultiMatchQueryBuilder 多字段关键词查询
        sourceBuilder.query(new MatchQueryBuilder("title",keywords));
        //1.2: 设置查询数据的构建器对象
        searchRequest.source(sourceBuilder);

        //1. 执行查询操作: 设置查询的请求对象
        SearchResponse response = esClient.search(searchRequest, RequestOptions.DEFAULT);

        //2. 处理结果集 : 在解析json的过程  参考这 在vscode中执行查询的结果集, 进行对照解析
        SearchHits searchHits = response.getHits();
        long totalNum = searchHits.getTotalHits().value;
        System.out.println("共计查询到:"+totalNum);

        SearchHit[] searchHitsHits = searchHits.getHits();
        for (SearchHit searchHitsHit : searchHitsHits) {
            String id = searchHitsHit.getId();
            String source = searchHitsHit.getSourceAsString();

            System.out.println(id+":"+source);

        }

        return null;
    }
    // 分页检索 : size 和 from 实现分页    多字段查询
    @Override
    public Map<String, Object> searchByPage(String keywords, int pageNum, int pageSize) throws IOException {

        //1.1: 创建查询的请求对象:
        SearchRequest searchRequest = new SearchRequest("job_idx");

        //1.2.1: 创建 查询的数据的构建器
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

        //1.2.2: 封装查询的条件
        sourceBuilder.query(new MultiMatchQueryBuilder(keywords,"title","jd"));

        //1.2.3: 封装分页条件
        sourceBuilder.from( (pageNum-1)*pageSize );
        sourceBuilder.size(pageSize);

        //1.2: 封装查询的数据的对象
        searchRequest.source(sourceBuilder);


        //1. 执行查询的操作:
        SearchResponse response = esClient.search(searchRequest, RequestOptions.DEFAULT);


        //2. 解析结果集
        SearchHits searchHits = response.getHits();

        long totalNum = searchHits.getTotalHits().value;
        System.out.println("总查询的条数:"+totalNum);

        SearchHit[] hitsHits = searchHits.getHits();
        for (SearchHit hitsHit : hitsHits) {
            String id = hitsHit.getId();

            String source = hitsHit.getSourceAsString();

            System.out.println(id+":"+source);

        }

        return null;
    }
    // scroll分页解决深分页问题
    @Override
    public Map<String, Object> searchByScrollPage(String keywords, String scrollId, int pageSize) throws IOException {

        String scroll_ID = scrollId;

        if(scroll_ID == null){ // 说明第一次来

            //1.1: 创建查询的请求对象:
            SearchRequest searchRequest = new SearchRequest("job_idx");

            //1.2.1: 创建 查询的数据的构建器
            SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

            //1.2.2: 封装查询的条件
            sourceBuilder.query(new MultiMatchQueryBuilder(keywords,"title","jd"));
            sourceBuilder.size(pageSize);

            //1.2: 封装查询的数据的对象
            searchRequest.source(sourceBuilder);
            searchRequest.scroll(TimeValue.timeValueMinutes(1));

            //1. 执行查询的操作:
            SearchResponse response = esClient.search(searchRequest, RequestOptions.DEFAULT);

            //2. 处理结果集
            parseResponse(response);

            scroll_ID = response.getScrollId();
            System.out.println(scroll_ID);

        }else {
            SearchScrollRequest searchScrollRequest = new SearchScrollRequest();
            searchScrollRequest.scrollId(scroll_ID);
            searchScrollRequest.scroll(TimeValue.timeValueMinutes(1));
            SearchResponse response = esClient.scroll(searchScrollRequest, RequestOptions.DEFAULT);

            parseResponse(response);


        }



        return null;
    }



    private void parseResponse(SearchResponse response) {
        // 处理结果集
        SearchHits searchHits = response.getHits();

        long totalNum = searchHits.getTotalHits().value;
        System.out.println("总查询的条数:"+totalNum);

        SearchHit[] hitsHits = searchHits.getHits();
        for (SearchHit hitsHit : hitsHits) {
            String id = hitsHit.getId();

            String source = hitsHit.getSourceAsString();

            System.out.println(id+":"+source);

        }
    }


    @SuppressWarnings("ALL")
    public Map<String, Object> higGearchByPage(String keywords, int pageNum, int pageSize) throws IOException {

        //1.1: 创建查询的请求对象:
        SearchRequest searchRequest = new SearchRequest("job_idx");

        //1.2.1: 创建 查询的数据的构建器
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

        //1.2.2: 封装查询的条件
        sourceBuilder.query(new MultiMatchQueryBuilder(keywords,"title","jd"));

        //1.2.3: 封装分页条件
        sourceBuilder.from( (pageNum-1)*pageSize );
        sourceBuilder.size(pageSize);


        // 封装高亮的条件:
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.field("title");
        highlightBuilder.field("jd");
        highlightBuilder.preTags("<font color='red'>");
        highlightBuilder.postTags("</font>");
        sourceBuilder.highlighter(highlightBuilder);

        //1.2: 封装查询的数据的对象
        searchRequest.source(sourceBuilder);


        //1. 执行查询的操作:
        SearchResponse response = esClient.search(searchRequest, RequestOptions.DEFAULT);


        //2. 解析结果集
        SearchHits searchHits = response.getHits();

        long totalNum = searchHits.getTotalHits().value;
        System.out.println("总查询的条数:"+totalNum);

        SearchHit[] hitsHits = searchHits.getHits();
        for (SearchHit hitsHit : hitsHits) {

            String id = hitsHit.getId();

            String source = hitsHit.getSourceAsString();
            JobDetail jobDetail = JSONObject.parseObject(source, JobDetail.class);

            jobDetail.setId(Long.parseLong(id));




            // Map<String, HighlightField> :  key表示高亮的字段  value 表示高亮数据
            Map<String, HighlightField> highlightFields = hitsHit.getHighlightFields();
            // 1.1 获取title高亮字段
            HighlightField titleHl = highlightFields.get("title");
            // 1.2 获取jd高亮字段
            HighlightField jdHl = highlightFields.get("jd");

            if(titleHl != null) {
                Text[] fragments = titleHl.getFragments();  // 这个片段就是高亮的数据
                StringBuilder stringBuilder = new StringBuilder();
                for (Text fragment : fragments) {
                    stringBuilder.append(fragment.string());
                }

                jobDetail.setTitle(stringBuilder.toString());

            }

            String jdH = "";
            if(jdHl != null) {
                Text[] fragments = jdHl.getFragments();
                StringBuilder stringBuilder = new StringBuilder();
                for (Text fragment : fragments) {
                    stringBuilder.append(fragment.string());
                }
                jobDetail.setJd(stringBuilder.toString());

            }

            System.out.println(jobDetail);

        }

        return null;
    }
}
```

### 6.5 `test`

```
import cn.itcast.elasticsearch.entity.JobDetail;
import cn.itcast.elasticsearch.service.impl.JobFullTextServiceImpl;
import org.testng.annotations.Test;

public class ESTest {
    private  JobFullTextServiceImpl service = new JobFullTextServiceImpl();
    // 添加数据
    @Test
    public void test01(){


        JobDetail jobDetail = new JobDetail();
        jobDetail.setId(300000);
        jobDetail.setArea("北京-昌平");
        jobDetail.setExp("1年以");
        jobDetail.setEdu("本科以上");
        jobDetail.setSalary("10k~15k");
        jobDetail.setJob_type("全职");
        jobDetail.setCmp("传智播客");
        jobDetail.setPv("10000pv");
        jobDetail.setTitle("大数据工程师");
        jobDetail.setJd("必须培训过....");

        service.add(jobDetail);
    }

    //根据id查询数据
    @Test
    public void test02() throws Exception{

        JobDetail detail = service.findById(200000);
        System.out.println(detail);

    }

    //修改数据
    @Test
    public void test03() throws Exception{
        JobDetail jobDetail = new JobDetail();
        jobDetail.setId(200000);
        jobDetail.setSalary("15k~25k");

        service.update(jobDetail);
    }

    //删除数据
    @Test
    public void test04() throws Exception{
        service.deleteById(200000);
    }

    // 根据关键词查询数据
    @Test
    public void test05() throws Exception{
        service.searchByKeywords("大数据工程师 ETL工程师 数据分析工程师");
    }

    // 分页查询数据: 浅分页
    @Test
    public void test06() throws Exception{
        service.searchByPage("大数据工程师 ETL工程师 数据分析工程师",3,3);
    }
    @Test
    public void test07() throws Exception{
        service.searchByScrollPage("大数据工程师 ETL工程师 数据分析工程师","DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAA0WN2dleVp3OXBSMHlsTFFLOTJzOU5xdw==",3);
    }


    @Test
    public void test08() throws Exception{
        service.higGearchByPage("大数据工程师 ETL工程师 数据分析工程师",3,3);
    }
}
```

## 7 ElasticSearch架构原理

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/30.png)

### 7.1 节点类型

> 在Elasticsearch有两类节点，一类是Master，一类是DataNode。

#### 7.1.1 Master节点

> 在Elasticsearch启动时，会选举出来一个Master节点。当某个节点启动后，然后使用Zen Discovery机制找到集群中的其他节点，并建立连接。

```
discovery.seed_hosts: ["node1.itcast.cn", "node2.itcast.cn", "node3.itcast.cn"]
```

> 并从候选主节点中选举出一个主节点。

```
cluster.initial_master_nodes: ["node1.itcast.cn", "node2.itcast.cn"]
```

> Master节点主要负责：
>> 管理索引（创建索引、删除索引）、分配分片
>> 维护元数据
>> 管理集群节点状态
>>不负责数据写入和查询，比较轻量级

> 一个Elasticsearch集群中，只有一个Master节点。在生产环境中，内存可以相对小一点，但机器要稳定。

#### 7.1.2 DataNode节点

>在Elasticsearch集群中，会有N个DataNode节点。DataNode节点主要负责：
>>数据写入、数据检索，大部分Elasticsearch的压力都在DataNode节点上
>>在生产环境中，内存最好配置大一些

### 7.2 分片和副本机制

#### 7.2.1 分片（shard）

> Elasticsearch是一个分布式的搜索引擎，索引的数据也是分成若干部分，分布在不同的服务器节点中

> 分布在不同服务器节点中的索引数据，就是分片（Shard）。Elasticsearch会自动管理分片，如果发现分片分布不均衡，就会自动迁移

> 一个索引（index）由多个shard（分片）组成，而分片是分布在不同的服务器上的

#### 7.2.2 副本

> 为了对Elasticsearch的分片进行容错，假设某个节点不可用，会导致整个索引库都将不可用。所以，需要对分片进行副本容错。每一个分片都会有对应的副本。在Elasticsearch中，默认创建的索引为1个分片、每个分片有1个主分片和1个副本分片。
>>每个分片都会有一个Primary Shard（主分片），也会有若干个Replica Shard（副本分片）
>>Primary Shard和Replica Shard不在同一个节点上

#### 7.2.3 指定分片、副本数量

>  创建指定分片数量、副本数量的索引

```
PUT /job_idx_shard
{
    "mappings": {
        "properties": {
            "id": { "type": "long", "store": true },
            "area": { "type": "keyword", "store": true },
            "exp": { "type": "keyword", "store": true },
            "edu": { "type": "keyword", "store": true },
            "salary": { "type": "keyword", "store": true },
            "job_type": { "type": "keyword", "store": true },
            "cmp": { "type": "keyword", "store": true },
            "pv": { "type": "keyword", "store": true },
            "title": { "type": "text", "store": true },
            "jd": { "type": "text"}
        }
    },
    "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 2
    }
}
```

>  查看分片、主分片、副本分片

```
GET /_cat/indices?v
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/31.png)

### 7.3 重要工作流程

#### 7.3.1 文档写入原理

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/32.png)

> 1.选择任意一个DataNode发送请求，例如：node2.itcast.cn。此时，node2.itcast.cn就成为一个		coordinating node（协调节点）

> 2.计算得到文档要写入的分片
>> `shard = hash(routing) % number_of_primary_shards`
>> routing 是一个可变值，默认是文档的 _id

> 3.coordinating node会进行路由，将请求转发给对应的primary shard所在的DataNode（假设primary shard在node1.itcast.cn、replica shard在node2.itcast.cn）

> 4.node1.itcast.cn节点上的Primary Shard处理请求，写入数据到索引库中，并将数据同步到			Replica shard

> 5.Primary Shard和Replica Shard都保存好了文档，返回client

#### 7.3.2 检索原理

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/33.png)

> client发起查询请求，某个DataNode接收到请求，该DataNode就会成为协调节点（Coordinating Node）

> 协调节点（Coordinating Node）将查询请求广播到每一个数据节点，这些数据节点的分片会处理该查询请求。协调节点会轮询所有的分片来自动进行负载均衡

> 每个分片进行数据查询，将符合条件的数据放在一个优先队列中，并将这些数据的文档ID、节点信息、分片信息返回给协调节点

> 协调节点将所有的结果进行汇总，并进行全局排序

> 协调节点向包含这些文档ID的分片发送get请求，对应的分片将文档数据返回给协调节点，最后协调节点将数据返回给客户端

### 7.4 准实时索引实现

#### 7.4.1 溢写倒文件系统缓存

> 当数据写入到ES分片时，会首先写入到内存中，然后通过内存的buffer生成一个segment，并刷到文件系统缓存中，数据可以被检索（注意不是直接刷到磁盘）

> ES中默认1秒，refresh一次

#### 7.4.2 写translog保障容错

> 在写入到内存中的同时，也会记录translog日志，在refresh期间出现异常，会根据translog来进行数据恢复

> 等到文件系统缓存中的segment数据都刷到磁盘中，清空translog文件

#### 7.4.3 flush倒磁盘

> ES默认每隔30分钟会将文件系统缓存的数据刷入到磁盘

#### 7.4.4 segment合并

> Segment太多时，ES定期会将多个segment合并成为大的segment，减少索引查询时IO开销，此阶段ES会真正的物理删除（之前执行过的delete的数据）

## 8 ElasticSearch SQL

> Elasticsearch SQL允许执行类SQL的查询，可以使用REST接口、命令行或者是JDBC，都可以使用SQL来进行数据的检索和数据的聚合。

> Elasticsearch SQL特点：
>> 本地集成
>>> Elasticsearch SQL是专门为Elasticsearch构建的。每个SQL查询都根据底层存储对相关节点有效执行。
>
>> 没有额外的要求
>>> 不依赖其他的硬件、进程、运行时库，Elasticsearch SQL可以直接运行在Elasticsearch集群上
>
>> 轻量且高效
>>> 像SQL那样简洁、高效地完成查询

### 8.1 SQL与Elasticsearch对应关系


SQL | Elasticsearch
---|---
column（列） | field（字段）
row（行） | document（文档）
table（表） | index（索引）
schema（模式） | N/A
database（数据库） | Elasticsearch集群实例

### 8.2 Elasticsearch SQL语法

```
SELECT select_expr [, ...]
[ FROM table_name ]
[ WHERE condition ]
[ GROUP BY grouping_element [, ...] ]
[ HAVING condition]
[ ORDER BY expression [ ASC | DESC ] [, ...] ]
[ LIMIT [ count ] ]
[ PIVOT ( aggregation_expr FOR column IN ( value [ [ AS ] alias ] [, ...] ) ) ]
```

> 目前FROM只支持一个表

### 8.3 职位查询案例

#### 8.3.1 查询职位索引库中的一条数据

> format：表示指定返回的数据类型

```
// 1. 查询职位信息
GET /_sql?format=txt
{
"query": "SELECT * FROM job_idx limit 1"
}
```

> 除了txt类型，Elasticsearch SQL还支持以下类型，

格式 | 描述
---|---
csv | 逗号分隔符
json | JSON格式
tsv | 制表符分隔符
txt | 类cli表示
yaml | YAML人类可读的格式

#### 8.3.2 将SQL转换为DSL

```
GET /_sql/translate
{
"query": "SELECT * FROM job_idx limit 1"
}
```

#### 8.3.3 职位scroll分页查询

> 第一次查询
>> fetch_size表示每页显示多少数据，而且当我们指定format为Json格式时，会返回一个cursor ID。

```
GET /_sql?format=json
{
    "query": "SELECT * FROM job_idx",
    "fetch_size": 10
}
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/34.png)

> 第二次查询

```
GET /_sql?format=json
{
    "cursor": "5/WuAwFaAXNARFhGMVpYSjVRVzVrUm1WMFkyZ0JBQUFBQUFBQUFJZ1dUM054VUZaMk9YVlJWalowYkVJeFowUkdVak10ZHc9Pf////8PCgFmBGFyZWEBBGFyZWEBB2tleXdvcmQBAAABZgNjbXABA2NtcAEHa2V5d29yZAEAAAFmA2VkdQEDZWR1AQdrZXl3b3JkAQAAAWYDZXhwAQNleHABB2tleXdvcmQBAAABZgJpZAECaWQBBGxvbmcAAAABZgJqZAECamQBBHRleHQAAAABZghqb2JfdHlwZQEIam9iX3R5cGUBB2tleXdvcmQBAAABZgJwdgECcHYBB2tleXdvcmQBAAABZgZzYWxhcnkBBnNhbGFyeQEHa2V5d29yZAEAAAFmBXRpdGxlAQV0aXRsZQEEdGV4dAAAAAL/Aw=="
}
```

> 清除游标

```
POST /_sql/close
{
    "cursor": "5/WuAwFaAXNARFhGMVpYSjVRVzVrUm1WMFkyZ0JBQUFBQUFBQUFJZ1dUM054VUZaMk9YVlJWalowYkVJeFowUkdVak10ZHc9Pf////8PCgFmBGFyZWEBBGFyZWEBB2tleXdvcmQBAAABZgNjbXABA2NtcAEHa2V5d29yZAEAAAFmA2VkdQEDZWR1AQdrZXl3b3JkAQAAAWYDZXhwAQNleHABB2tleXdvcmQBAAABZgJpZAECaWQBBGxvbmcAAAABZgJqZAECamQBBHRleHQAAAABZghqb2JfdHlwZQEIam9iX3R5cGUBB2tleXdvcmQBAAABZgJwdgECcHYBB2tleXdvcmQBAAABZgZzYWxhcnkBBnNhbGFyeQEHa2V5d29yZAEAAAFmBXRpdGxlAQV0aXRsZQEEdGV4dAAAAAL/Aw=="
}
```

#### 8.3.4 职位全文检索

> 1.需求：检索title和jd中包含hadoop的职位。

> 2.MATCH函数
>>在执行全文检索时，需要使用到MATCH函数。

```
MATCH(
field_exp,   
constant_exp
[, options])
```

> field_exp：匹配字段<br/> 
constant_exp：匹配常量表达式

> 3.实现

```
GET /_sql?format=txt
{
    "query": "select * from job_idx where MATCH(title, 'hadoop') or MATCH(jd, 'hadoop') limit 10"
}
```

### 8.4 订单统计分析案例

#### 8.4.1 案例介绍

> 有以下数据集：

订单ID | 订单状态 | 支付金额 | 支付方式ID | 用户ID | 操作时间 | 商品分类
---|---|---|---|---|---|---
id | status | pay_money | payway | userid | operation_date | category
1 | 已提交 | 4070 | 1 | 4944191 | 2020-04-25 12:09:16 | 手机;
2 | 已完成 | 4350 | 1 | 1625615 | 2020-04-25 12:09:37 | 家用电器;;电脑;
3 | 已提交 | 6370 | 3 | 3919700 | 2020-04-25 12:09:39 | 男装;男鞋;
4 | 已付款 | 6370 | 3 | 3919700 | 2020-04-25 12:09:44 | 男装;男鞋;

> 我们需要基于按数据，使用Elasticsearch中的聚合统计功能，实现一些指标统计。

#### 8.4.2 创建索引

```
PUT /order_idx/
{
    "mappings": {
        "properties": {
            "id": {
                "type": "keyword",
                "store": true
            },
            "status": {
                "type": "keyword",
                "store": true
            },
            "pay_money": {
                "type": "double",
                "store": true
            },
            "payway": {
                "type": "byte",
                "store": true
            },
            "userid": {
                "type": "keyword",
                "store": true
            },
            "operation_date": {
                "type": "date",
                "format": "yyyy-MM-dd HH:mm:ss",
                "store": true
            },
            "category": {
                "type": "keyword",
                "store": true
            }
        }
    }
}
```

#### 8.4.3 导入测试数据

> 1.上传资料中的order_data.json数据文件到Linux

> 2.使用bulk进行批量导入命令

```
curl -H "Content-Type: application/json" -XPOST "node1.itcast.cn:9200/order_idx/_bulk?pretty&refresh" --data-binary "@order_data.json"
```

#### 8.4.4 统计不同支付方式的的订单数量

##### 8.4.4.1 使用JSON DSL的方式来实现

> 这种方式就是用Elasticsearch原生支持的基于JSON的DSL方式来实现聚合统计。

```
GET /order_idx/_search
{
    "size": 0,
    "aggs": {
        "group_by_state": {
            "terms": {
                "field": "payway"
            }
        }
    }
}
```

> 统计结果：

```
"aggregations": {
    "group_by_state": {
        "doc_count_error_upper_bound": 0,
        "sum_other_doc_count": 0,
        "buckets": [
            {
                "key": 2,
                "doc_count": 1496
            },
            {
                "key": 1,
                "doc_count": 1438
            },
            {
                "key": 3,
                "doc_count": 1183
            },
            {
                "key": 0,
                "doc_count": 883
            }
        ]
    }
}
```

> 这种方式分析起来比较麻烦，如果将来我们都是写这种方式来分析数据，简直是无法忍受。所以，Elasticsearch想要进军OLAP领域，是一定要支持SQL，能够使用SQL方式来进行统计和分析的。

##### 8.4.4.2 基于Elasticsearch SQL方式实现

```
GET /_sql?format=txt
{
"query": "select payway, count(*) as order_cnt from order_idx group by payway"
}
```

> 这种方式要更加直观、简洁。

#### 8.4.5 基于JDBC方式统计不同方式的订单数量

> Elasticsearch中还提供了基于JDBC的方式来访问数据。我们可以像操作MySQL一样操作Elasticsearch。使用步骤如下：

> 1.在pom.xml中添加以下镜像仓库

```xml
<repositories>
    <repository>
        <id>elastic.co</id>
        <url>https://artifacts.elastic.co/maven</url>
    </repository>
</repositories>
```

> 2.导入Elasticsearch JDBC驱动Maven依赖

```xml
<dependency>
    <groupId>org.elasticsearch.plugin</groupId>
    <artifactId>x-pack-sql-jdbc</artifactId>
    <version>7.6.1</version>
</dependency>
```
> 3.驱动

```
org.elasticsearch.xpack.sql.jdbc.EsDriver
```

> 4.JDBC URL

```
jdbc:es:// http:// host:port
```

> 5.开启X-pack高阶功能试用，如果不开启试用，会报如下错误

```
current license is non-compliant for [jdbc]
```

> 在node1.itcast.cn节点上执行：

```
curl http://node1.itcast.cn:9200/_license/start_trial?acknowledge=true -X POST
{"acknowledged":true,"trial_was_started":true,"type":"trial"}
```

> 试用期为30天。

> 参考代码：

```
/**
* 基于JDBC访问Elasticsearch
  */
  public class ElasticJdbc {

      public static void main(String[] args) throws Exception {
          Class.forName("org.elasticsearch.xpack.sql.jdbc.EsDriver");
        
          Connection connection = DriverManager.getConnection("jdbc:es://http://node1.itcast.cn:9200");
          PreparedStatement ps = connection.prepareStatement("select payway, count(*) as order_cnt from order_idx group by payway");
          ResultSet resultSet = ps.executeQuery();
    
          while(resultSet.next()) {
               int payway = resultSet.getInt("payway");
               int order_cnt = resultSet.getInt("order_cnt");
               System.out.println("支付方式: " + payway + " 订单数量: " + order_cnt);
          }
           resultSet.close();
           ps.close();
           connection.close();
      }
  }
```

> 注意：如果在IDEA中无法下载依赖，请参考以下操作：

```
在Idea的File -->settings中，设置Maven的importing和Runner参数，忽略证书检查即可。(Eclipse下解决原理类似，设置maven运行时参数)，并尝试手动执行Maven compile执行编译。
具体参数：
-Dmaven.multiModuleProjectDirectory=$MAVEN_HOME 
-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true 
-Dmaven.wagon.http.ssl.ignore.validity.dates=true
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/35.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/36.png)

#### 8.4.6 统计不同支付方式订单数，并按照订单数量倒序排序

```
GET /_sql?format=txt
{
    "query": "select payway, count(*) as order_cnt from order_idx group by payway order by order_cnt desc"
}
```

#### 8.4.7 只统计「已付款」状态的不同支付方式的订单数量

```
GET /_sql?format=txt
{
    "query": "select payway, count(*) as order_cnt from order_idx where status = '已付款' group by payway order by order_cnt desc"
}
```

### 8.5 统计不同状态的订单总额、不同支付方式最高、最低订单金额

> 统计不同状态的订单总额、不同支付方式最高、最低订单金额

```
GET /_sql?format=txt
{
"query": "select userid, count(1) as cnt, sum(pay_money) as total_money from order_idx group by userid"
}
```

### 8.6 Elasticsearch SQL目前的一些限制

> 目前Elasticsearch SQL还存在一些限制。例如：不支持JOIN、不支持较复杂的子查询。所以，有一些相对复杂一些的功能，还得借助于DSL方式来实现。

## 9 Beats

> Beats是一个开放源代码的数据发送器。我们可以把Beats作为一种代理安装在我们的服务器上，这样就可以比较方便地将数据发送到Elasticsearch或者Logstash中。Elastic Stack提供了多种类型的Beats组件。

数据类型 | 组件
---|---
审计数据 | AuditBeat
日志文件 | FileBeat
云数据 | FunctionBeat
可用性数据 | HeartBeat
系统日志 | JournalBeat
指标数据 | MetricBeat
网络流量数据 | PacketBeat
Windows事件日志 | Winlogbeat

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/37.png)

> Beats可以直接将数据发送到Elasticsearch或者发送到Logstash，基于Logstash可以进一步地对数据进行处理，然后将处理后的数据存入到Elasticsearch，最后使用Kibana进行数据可视化。

### 9.1 FileBeat简介

> FileBeat专门用于转发和收集日志数据的轻量级采集工具。它可以为作为代理安装在服务器上，FileBeat监视指定路径的日志文件，收集日志数据，并将收集到的日志转发到Elasticsearch或者Logstash。

### 9.2 FileBeat工作原理

> 启动FileBeat时，会启动一个或者多个输入（Input），这些Input监控指定的日志数据位置。FileBeat会针对每一个文件启动一个Harvester（收割机）。Harvester读取每一个文件的日志，将新的日志发送到libbeat，libbeat将数据收集到一起，并将数据发送给输出（Output）。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/38.png)
 
### 9.3 FileBeat安装

> 安装FileBeat只需要将FileBeat Linux安装包上传到Linux系统，并将压缩包解压到系统就可以了。FileBeat官方下载地址：https://www.elastic.co/cn/downloads/past-releases/filebeat-7-6-1

> 上传FileBeat安装到Linux，并解压。

```
tar -xvzf filebeat-7.6.1-linux-x86_64.tar.gz
```

### 9.4 使用FileBeat采集Kafka日志到Elasticsearch

#### 9.4.1 需求分析

> 在资料中有一个kafka_server.log.tar.gz压缩包，里面包含了很多的Kafka服务器日志，现在我们为了通过在Elasticsearch中快速查询这些日志，定位问题。我们需要用FileBeats将日志数据上传到Elasticsearch中。

> 问题：
>> 首先，我们要指定FileBeat采集哪些Kafka日志，因为FileBeats中必须知道采集存放在哪儿的日志，才能进行采集。
>> 其次，采集到这些数据后，还需要指定FileBeats将采集到的日志输出到Elasticsearch，那么Elasticsearch的地址也必须指定。

#### 9.4.2 配置FileBeats

>FileBeats配置文件主要分为两个部分。

```
1.inputs
2.output
```

> 从名字就能看出来，一个是用来输入数据的，一个是用来输出数据的。

##### 9.4.2.1 input配置

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log
  #- c:\programdata\elasticsearch\logs\*
 ```

> 在FileBeats中，可以读取一个或多个数据源。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/39.png)

##### 9.4.2.2 output配置

> 默认FileBeat会将日志数据放入到名称为：filebeat-%filebeat版本号%-yyyy.MM.dd 的索引中。

> PS：
>> FileBeats中的filebeat.reference.yml包含了FileBeats所有支持的配置选项。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/40.png)

#### 9.4.3 配置文件

> 1.创建配置文件

```
cd /export/servers/es/filebeat-7.6.1-linux-x86_64
vim filebeat_kafka_log.yml
```

> 2.复制一下到配置文件中

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /export/servers/es/data/kafka/servers.log.*

output.elasticsearch:
hosts: ["node1.itcast.cn:9200", "node2.itcast.cn:9200", "node3.itcast.cn:9200"]
```

#### 9.4.4 运行FileBeats

> 1.运行FileBeat

```
./filebeat -c filebeat_kafka_log.yml -e
```

> 2.将日志数据上传到/var/kafka/log，并解压
```
mkdir -p /export/servers/es/data/kafka/
tar -xvzf kafka_server.log.tar.gz
```

> 注意: 文件权限的报错
>> 如果在启动fileBeat的时候, 报了一个配置文件权限的错误, 请修改其权限为 -rw-r--r--

#### 9.4.5 查询数据

> 1.查看索引信息

```
GET /_cat/indices?v
{
    "health": "green",
    "status": "open",
    "index": "filebeat-7.6.1-2021.01.26-000001",
    "uuid": "dplqB_hTQq2XeSk6S4tccQ",
    "pri": "1",
    "rep": "1",
    "docs.count": "213780",
    "docs.deleted": "0",
    "store.size": "71.9mb",
    "pri.store.size": "35.8mb"
}
```
> 2.查询索引库中的数据

```
GET /filebeat-7.6.1-2021.01.26-000001/_search
{
    "_index": "filebeat-7.6.1-2021.01.26-000001",
    "_type": "_doc",
    "_id": "-72pX3IBjTeClvZff0CB",
    "_score": 1,
    "_source": {
        "@timestamp": "2020-05-29T09:00:40.041Z",
        "log": {
            "offset": 55433,
            "file": {
                "path": "/var/kafka/log/servers.log.2020-05-02-16"
            }
        },
        "message": "[2020-05-02 16:01:30,682] INFO Socket connection established, initiating session, client: /192.168.88.100:46762, server: node1.itcast.cn/192.168.88.100:2181 (org.apache.zookeeper.ClientCnxn)",
        "input": {
            "type": "log"
        },
        "ecs": {
            "version": "1.4.0"
        },
        "host": {
            "name": "node1.itcast.cn"
        },
        "agent": {
            "id": "b4c5c4dc-03c3-4ba4-9400-dc6afcb36d64",
            "version": "7.6.1",
            "type": "filebeat",
            "ephemeral_id": "b8fbf7ab-bc37-46dd-86c7-fa7d74d36f63",
            "hostname": "node1.itcast.cn"
        }
    }
}

```

> FileBeat自动给我们添加了一些关于日志、采集类型、Host各种字段。

#### 9.4.6 解决一个日志涉及到多行问题

> 我们在日常日志的处理中，经常会碰到日志中出现异常的情况。类似下面的情况：

```
[2020-04-30 14:00:05,725] WARN [ReplicaFetcher replicaId=0, leaderId=1, fetcherId=0] Error when sending leader epoch request for Map(test_10m-2 -> (currentLeaderEpoch=Optional[161], leaderEpoch=158)) (kafka.server.ReplicaFetcherThread)
java.io.IOException: Connection to node2.itcast.cn:9092 (id: 1 rack: null) failed.
at org.apache.kafka.clients.NetworkClientUtils.awaitReady(NetworkClientUtils.java:71)
at kafka.server.ReplicaFetcherBlockingSend.sendRequest(ReplicaFetcherBlockingSend.scala:102)
at kafka.server.ReplicaFetcherThread.fetchEpochEndOffsets(ReplicaFetcherThread.scala:310)
at kafka.server.AbstractFetcherThread.truncateToEpochEndOffsets(AbstractFetcherThread.scala:208)
at kafka.server.AbstractFetcherThread.maybeTruncate(AbstractFetcherThread.scala:173)
at kafka.server.AbstractFetcherThread.doWork(AbstractFetcherThread.scala:113)
at kafka.utils.ShutdownableThread.run(ShutdownableThread.scala:96)
[2020-04-30 14:00:05,725] INFO [ReplicaFetcher replicaId=0, leaderId=1, fetcherId=0] Retrying leaderEpoch request for partition test_10m-2 as the leader reported an error: UNKNOWN_SERVER_ERROR (kafka.server.ReplicaFetcherThread)
[2020-04-30 14:00:08,731] WARN [ReplicaFetcher replicaId=0, leaderId=1, fetcherId=0] Connection to node 1 (node2.itcast.cn/192.168.88.101:9092) could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)
在FileBeat中，Harvest是逐行读取日志文件的。但上述的日志会出现一条日志，跨多行的情况。有异常信息时，肯定会出现多行。我们先来看一下，如果默认不处理这种情况会出现什么问题。
```

##### 9.4.6.1 导入错误日志

> 1.在/export/servers/es/data/kafka/中创建名为server.log.2020-09-10的日志文件

> 2.将资料中的err.txt日志文本贴入到该文件中

> 观察FileBeat，发现FileBeat已经针对该日志文件启动了Harvester，并读取到数据数据。

```
2020-05-29T19:11:01.236+0800    INFO    log/harvester.go:297    Harvester started for file: /var/kafka/log/servers.log.2020-09-10
```

> 3.在Elasticsearch检索该文件。

```
GET /filebeat-7.6.1-2021.01.26-000001/_search
{
    "query": {
        "match": {
            "log.file.path": "/export/servers/es/data/kafka/servers.log.2020-09-10"
        }
    }
}
```

> 我们发现，原本是一条日志中的异常信息，都被作为一条单独的消息来处理了~

> 这明显是不符合我们的预期的，我们想要的是将所有的异常消息合并到一条日志中。那针对这种情况该如何处理呢？

##### 9.4.6.2 问题分析

> 每条日志都是有统一格式的开头的，就拿Kafka的日志消息来说，[2020-04-30 14:00:05,725]这是一个统一的格式，如果不是以这样的形式开头，说明这一行肯定是属于某一条日志，而不是独立的一条日志。所以，我们可以通过日志的开头来判断某一行是否为新的一条日志。

##### 9.4.6.3 FileBeat多行配置选项

> 在FileBeat的配置中，专门有一个解决一条日志跨多行问题的配置。主要为以下三个配置：

```
multiline.pattern: ^\[
multiline.negate: false
multiline.match: after
```

```
multiline.pattern表示能够匹配一条日志的模式，默认配置的是以[开头的才认为是一条新的日志。
multiline.negate:配置该模式是否生效，默认为false。
multiline.match:表示是否将未匹配到的行追加到上一日志，还是追加到下一个日志。
```

##### 9.4.6.4 重新配置FileBeat

> 1.修改filebeat.yml，并添加以下内容

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /export/servers/es/data/kafka/servers.log.*
      multiline.pattern: '^\['
      multiline.negate: true
      multiline.match: after

output.elasticsearch:
hosts: ["node1.itcast.cn:9200", "node2.itcast.cn:9200", "node3.itcast.cn:9200"]
```

> 2.修改「注册表」/data.json，将server.log.2020-09-10对应的offset设置为0

```
cd /export/servers/es/filebeat-7.6.1-linux-x86_64/data/registry/filebeat
vim data.json
```

> 3.删除之前创建的文档

```
// 删除指定文件的文档
POST /filebeat-7.6.1-2021.01.26-000001/_delete_by_query
{
    "query": {
        "match": {
            "log.file.path": "/var/kafka/log/servers.log.2020-09-10"
        }
    }
}
```

> 4.重新启动FileBeat

```
./filebeat -e
```

### 9.5 FileBeat是如何工作的

> FileBeat主要由input和harvesters（收割机）组成。这两个组件协同工作，并将数据发送到指定的输出。

#### 9.5.1 input和harvester

> inputs（输入）
>> input是负责管理Harvesters和查找所有要读取的文件的组件。如果输入类型是 log，input组件会查找磁盘上与路径描述的所有文件，并为每个文件启动一个Harvester，每个输入都独立地运行。

> Harvesters（收割机）
>>Harvesters负责读取单个文件的内容，它负责打开/关闭文件，并逐行读取每个文件的内容，并将读取到的内容发送给输出，每个文件都会启动一个Harvester。但Harvester运行时，文件将处于打开状态。如果文件在读取时，被移除或者重命名，FileBeat将继续读取该文件。

#### 9.5.2 FileBeats如何保持文件状态

> FileBeat保存每个文件的状态，并定时将状态信息保存在磁盘的「注册表」文件中，该状态记录Harvester读取的最后一次偏移量，并确保发送所有的日志数据。如果输出（Elasticsearch或者Logstash）无法访问，FileBeat会记录成功发送的最后一行，并在输出（Elasticsearch或者Logstash）可用时，继续读取文件发送数据。在运行FileBeat时，每个input的状态信息也会保存在内存中，重新启动FileBeat时，会从「注册表」文件中读取数据来重新构建状态。

> 在/export/servers/es/filebeat-7.6.1-linux-x86_64/data目录中有一个Registry文件夹，里面有一个data.json，该文件中记录了Harvester读取日志的offset。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/41.png)

## 10 Logstash

### 10.1 简介

> Logstash是一个开源的数据采集引擎。它可以动态地将不同来源的数据统一采集，并按照指定的数据格式进行处理后，将数据加载到其他的目的地。最开始，Logstash主要是针对日志采集，但后来Logstash开发了大量丰富的插件，所以，它可以做更多的海量数据的采集。

> 它可以处理各种类型的日志数据，例如：Apache的web log、Java的log4j日志数据，或者是系统、网络、防火墙的日志等等。它也可以很容易的和Elastic Stack的Beats组件整合，也可以很方便的和关系型数据库、NoSQL数据库、Kafka、RabbitMQ等整合。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/42.png)

#### 10.1.1 经典架构

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/43.png)

#### 10.1.2 对比Flume

> 1.Apache Flume是一个通用型的数据采集平台，它通过配置source、channel、sink来实现数据的采集，支持的平台也非常多。而Logstash结合Elastic Stack的其他组件配合使用，开发、应用都会简单很多

> 2.Logstash比较关注数据的预处理，而Flume跟偏重数据的传输，几乎没有太多的数据解析预处理，仅仅是数据的产生，封装成Event然后传输。

#### 10.1.3 对比FileBeat

> logstash是jvm跑的，资源消耗比较大

> 而FileBeat是基于golang编写的，功能较少但资源消耗也比较小，更轻量级

> logstash 和filebeat都具有日志收集功能，Filebeat更轻量，占用资源更少

> logstash 具有filter功能，能过滤分析日志

> 一般结构都是filebeat采集日志，然后发送到消息队列，redis，kafka中然后logstash去获取，利用filter功能过滤分析，然后存储到elasticsearch中

### 10.2 安装Logstash

> 1.下载Logstash

```
https://www.elastic.co/cn/downloads/past-releases/logstash-7-6-1
```
> 2.解压Logstash到指定目录

> 3.运行测试

```
cd /export/servers/es/logstash-7.6.1/
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

> 等待一会，让Logstash启动完毕。

```
Sending Logstash logs to /export/servers/es/logstash-7.6.1/logs which is now configured via log4j2.properties
[2020-05-28T16:31:44,159][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
[2020-05-28T16:31:44,264][INFO ][logstash.runner          ] Starting Logstash {"logstash.version"=>"7.6.1"}
[2020-05-28T16:31:45,631][INFO ][org.reflections.Reflections] Reflections took 37 ms to scan 1 urls, producing 20 keys and 40 values
[2020-05-28T16:31:46,532][WARN ][org.logstash.instrument.metrics.gauge.LazyDelegatingGauge][main] A gauge metric of an unknown type (org.jruby.RubyArray) has been create for key: cluster_uuids. This may result in invalid serialization.  It is recommended to log an issue to the responsible developer/development team.
[2020-05-28T16:31:46,560][INFO ][logstash.javapipeline    ][main] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>2, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50, "pipeline.max_inflight"=>250, "pipeline.sources"=>["config string"], :thread=>"#<Thread:0x3ccbc15b run>"}
[2020-05-28T16:31:47,268][INFO ][logstash.javapipeline    ][main] Pipeline started {"pipeline.id"=>"main"}
The stdin plugin is now waiting for input:
[2020-05-28T16:31:47,348][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
[2020-05-28T16:31:47,550][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
```

> 然后，随便在控制台中输入内容，等待Logstash的输出。

```
{
"host" => "node1.itcast.cn",
"message" => "hello world",
"@version" => "1",
"@timestamp" => 2020-05-28T08:32:31.007Z
}
```

> ps：
>> -e选项表示，直接把配置放在命令中，这样可以有效快速进行测试

### 10.3 采集Apache Web服务器日志

#### 10.3.1 需求

> Apache的Web Server会产生大量日志，当我们想要对这些日志检索分析。就需要先把这些日志导入到Elasticsearch中。此处，我们就可以使用Logstash来实现日志的采集。

> 打开这个文件，如下图所示。我们发现，是一个纯文本格式的日志。如下所示：

```
90.224.57.84 - - [15/Apr/2020:00:27:19 +0800] "POST /report HTTP/1.1" 404 21 "www.baidu.com" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900"
61.26.110.244 - - [16/Apr/2020:00:27:20 +0801] "POST /itcast.cn/index.html HTTP/1.1" 200 45 "www.jd.com/search" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900"
50.76.240.38 - - [17/Apr/2020:00:27:19 +0801] "GET /itcast.cn/index1.html HTTP/1.1" 200 44 "www.jd.com" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900"
54.199.92.133 - - [18/Apr/2020:00:27:19 +0801] "GET /itcast.cn/index2.html HTTP/1.1" 200 39 "www.zhihu.com" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900"
24.114.52.236 - - [19/Apr/2020:00:27:20 +0802] "POST /itcast.cn/course/12355.html HTTP/1.1" 200 230 "www.douyin.com" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900"
......
```

> 这个日志其实由一个个的字段拼接而成，参考以下表格。

字段名 | 说明
---|---
client IP | 浏览器端IP
timestamp | 请求的时间戳
method | 请求方式（GET/POST）
uri | 请求的链接地址
status | 服务器端响应状态
length | 响应的数据长度
reference | 从哪个URL跳转而来
browser | 浏览器

> 因为最终我们需要将这些日志数据存储在Elasticsearch中，而Elasticsearch是有模式（schema）的，而不是一个大文本存储所有的消息，而是需要将字段一个个的保存在Elasticsearch中。所以，我们需要在Logstash中，提前将数据解析好，将日志文本行解析成一个个的字段，然后再将字段保存到Elasticsearch中。

#### 10.3.2 准备日志数据

> 将Apache服务器日志上传到 /export/servers/es/data/apache/ 目录

```
mkdir -p /export/servers/es/data/apache/
```

#### 10.3.3 使用FileBeats将日志发送到Logstash

> 在使用Logstash进行数据解析之前，我们需要使用FileBeat将采集到的数据发送到Logstash。之前，我们使用的FileBeat是通过FileBeat的Harvester组件监控日志文件，然后将日志以一定的格式保存到Elasticsearch中，而现在我们需要配置FileBeats将数据发送到Logstash。FileBeat这一端配置以下即可：

```
#----------------------------- Logstash output ---------------------------------
#output.logstash:
# Boolean flag to enable or disable the output module.
#enabled: true

# The Logstash hosts
#hosts: ["localhost:5044"]
```

> hosts配置的是Logstash监听的IP地址/机器名以及端口号。
>> `114.113.220.255`

> `准备FileBeat配置文件`

```
cd /export/servers/es/filebeat-7.6.1-linux-x86_64
vim filebeat-logstash.yml
```

> 因为Apache的web log日志都是以IP地址开头的，所以我们需要修改下匹配字段。

 ```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /export/servers/es/data/apache/access.*
      multiline.pattern: '^\d+\.\d+\.\d+\.\d+ '
      multiline.negate: true
      multiline.match: after

output.logstash:
enabled: true
hosts: ["node1.itcast.cn:5044"]
```

> 启动FileBeat，并指定使用新的配置文件

```
./filebeat -e -c filebeat-logstash.yml
```

> FileBeat将尝试建立与Logstash监听的IP和端口号进行连接。但此时，我们并没有开启并配置Logstash，所以FileBeat是无法连接到Logstash的。

```
2020-06-01T11:28:47.585+0800    ERROR   pipeline/output.go:100  Failed to connect to backoff(async(tcp://node1.itcast.cn:5044)): dial tcp 192.168.88.100:5044: connect: connection refused
```

#### 10.3.4 配置Logstash接收FileBeat数据并打印

> Logstash的配置文件和FileBeat类似，它也需要有一个input、和output。基本格式如下：
```
# #号表示添加注释
# input表示要接收的数据
input {
}

# file表示对接收到的数据进行过滤处理
filter {

}

# output表示将数据输出到其他位置
output {
}
```

> 配置从FileBeat接收数据

```
cd /export/servers/es/logstash-7.6.1/config
vim filebeat-print.conf
```

```
input {
    beats {
        port => 5044
    }
}

output {
    stdout {
        codec => rubydebug
    }
}
```

> 测试logstash配置是否正确

```
bin/logstash -f config/filebeat-print.conf --config.test_and_exit
```

```
[2020-06-01T11:46:33,940][INFO ][logstash.runner          ] Using config.test_and_exit mode. Config Validation Result: OK. Exiting Logstash
```

> 启动logstash

```
bin/logstash -f config/filebeat-print.conf --config.reload.automatic
```

```
reload.automatic：修改配置文件时自动重新加载
```

> 测试
>> 创建一个access.log.1文件，使用cat test >> access.log.1往日志文件中追加内容。
>> test文件中只保存一条日志：

```
[root@node1 log]# cat test
235.9.200.242 - - [15/Apr/2015:00:27:19 +0849] "POST /itcast.cn/bigdata.html 200 45 "www.baidu.com" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900 144.180.122.249
```

> 当我们启动Logstash之后，就可以发现Logstash会打印出来从FileBeat接收到的数据：

```
{
    "log" => {
        "file" => {
            "path" => "/var/apache/log/access.log.1"
        },
        "offset" => 825
    },
    "input" => {
        "type" => "log"
    },
    "agent" => {
        "ephemeral_id" => "d4c3b652-4533-4ebf-81f9-a0b78c0d4b05",
        "version" => "7.6.1",
        "type" => "filebeat",
        "id" => "b4c5c4dc-03c3-4ba4-9400-dc6afcb36d64",
        "hostname" => "node1.itcast.cn"
    },
    "@timestamp" => 2020-06-01T09:07:55.236Z,
    "ecs" => {
        "version" => "1.4.0"
        在使用Logstash进行数据解析之前，我们需要使用FileBeat将采集到的数据发送到Logstash。之前，我们使用的FileBeat是通过FileBeat的Harvester组件监控日志文件，然后将日志以一定的格式保存到Elasticsearch中，而现在我们需要配置FileBeats将数据发送到Logstash。FileBeat这一端配置以下即可：
        #----------------------------- Logstash output ---------------------------------
        #output.logstash:
        # Boolean flag to enable or disable the output module.
        #enabled: true
        
        # The Logstash hosts
        #hosts: ["localhost:5044"]
        
        hosts配置的是Logstash监听的IP地址/机器名以及端口号。
        114.113.220.255
        准备FileBeat配置文件
        cd /export/servers/es/filebeat-7.6.1-linux-x86_64
        vim filebeat-logstash.yml
        因为Apache的web log日志都是以IP地址开头的，所以我们需要修改下匹配字段。
        filebeat.inputs:
        - type: log
          enabled: true
          paths:
            - /export/servers/es/data/apache/access.*
              multiline.pattern: '^\d+\.\d+\.\d+\.\d+ '
              multiline.negate: true
              multiline.match: after
        
        output.logstash:
        enabled: true
        hosts: ["node1.itcast.cn:5044"]
        
        启动FileBeat，并指定使用新的配置文件
        ./filebeat -e -c filebeat-logstash.yml
        
        FileBeat将尝试建立与Logstash监听的IP和端口号进行连接。但此时，我们并没有开启并配置Logstash，所以FileBeat是无法连接到Logstash的。
        2020-06-01T11:28:47.585+0800    ERROR   pipeline/output.go:100  Failed to connect to backoff(async(tcp://node1.itcast.cn:5044)): dial tcp 192.168.88.100:5044: connect: connection refused    
    },
    "host" => {
        "name" => "node1.itcast.cn"
    },
    "tags" => [
        [0] "beats_input_codec_plain_applied"
    ],
    "message" => "235.9.200.242 - - [15/Apr/2015:00:27:19 +0849] \"POST /itcast.cn/bigdata.html 200 45 \"www.baidu.com\" \"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900 144.180.122.249",
    "@version" => "1"
}
```

#### 10.3.5 Logstash输出数据到ElasticaSearch 

> 过控制台，我们发现Logstash input接收到的数据没有经过任何处理就发送给了output组件。而其实我们需要将数据输出到Elasticsearch。所以，我们修改Logstash的output配置。配置输出Elasticsearch只需要配置以下就可以了：

```
output {
elasticsearch {
hosts => [ "localhost:9200" ]
}}
```

> 操作步骤：

> 1.重新拷贝一份配置文件

```
cp filebeat-print.conf filebeat-es.conf
```

> 2.将output修改为Elasticsearch

```
input {
    beats {
        port => 5044
    }
}

output {
    elasticsearch {
        hosts => [ "node1.itcast.cn:9200","node2.itcast.cn:9200","node3.itcast.cn:9200"]
    }
}
```

> 3.重新启动Logstash

```
bin/logstash -f config/filebeat-es.conf --config.reload.automatic
```

> 4.追加一条日志到监控的文件中，并查看Elasticsearch中的索引、文档

```
 cat test >> access.log.1
```

> 查看索引数据

```
GET /_cat/indices?v
```

> 我们在Elasticsearch中发现一个以logstash开头的索引。

```
{
    "health": "green",
    "status": "open",
    "index": "logstash-2020.06.01-000001",
    "uuid": "147Uwl1LRb-HMFERUyNEBw",
    "pri": "1",
    "rep": "1",
    "docs.count": "2",
    "docs.deleted": "0",
    "store.size": "44.8kb",
    "pri.store.size": "22.4kb"
}
```

> 查看索引库的数据

```
GET /logstash-2020.06.01-000001/_search?format=txt
{
"from": 0,
"size": 1
}
```

> 我们可以获取到以下数据：

```
"@timestamp": "2020-06-01T09:38:00.402Z",
"tags": [
    "beats_input_codec_plain_applied"
],
"host": {
    "name": "node1.itcast.cn"
},
"@version": "1",
"log": {
    "file": {
        "path": "/var/apache/log/access.log.1"
    },
    "offset": 1343
},
"agent": {
    "version": "7.6.1",
    "ephemeral_id": "d4c3b652-4533-4ebf-81f9-a0b78c0d4b05",
    "id": "b4c5c4dc-03c3-4ba4-9400-dc6afcb36d64",
    "hostname": "node1.itcast.cn",
    "type": "filebeat"
},
"input": {
    "type": "log"
},
"message": "235.9.200.242 - - [15/Apr/2015:00:27:19 +0849] \"POST /itcast.cn/bigdata.html 200 45 \"www.baidu.com\" \"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900 144.180.122.249",
"ecs": {
    "version": "1.4.0"
}
```

> 从输出返回结果，我们可以看到，日志确实已经保存到了Elasticsearch中，而且我们看到消息数据是封装在名为message中的，其他的数据也封装在一个个的字段中。我们其实更想要把消息解析成一个个的字段。例如：IP字段、时间、请求方式、请求URL、响应结果，这样。

#### 10.3.6 Logstash过滤器

> 在Logstash中可以配置过滤器Filter对采集到的数据进行中间处理，在Logstash中，有大量的插件供我们使用。参考官网：

> https://www.elastic.co/guide/en/logstash/7.6/filter-plugins.html

##### 10.3.6.1 查看Logstash已经安装的插件

```
bin/logstash-plugin list
```

##### 10.3.6.2 Grok插件

> Grok是一种将非结构化日志解析为结构化的插件。这个工具非常适合用来解析系统日志、Web服务器日志、MySQL或者是任意其他的日志格式。

> Grok官网：https://www.elastic.co/guide/en/logstash/7.6/plugins-filters-grok.html

##### 10.3.6.3 Grok语法

> Grok是通过模式匹配的方式来识别日志中的数据,可以把Grok插件简单理解为升级版本的正则表达式。它拥有更多的模式，默认，Logstash拥有120个模式。如果这些模式不满足我们解析日志的需求，我们可以直接使用正则表达式来进行匹配。

> 官网：https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/grok-patterns

> grok模式的语法是：%{SYNTAX:SEMANTIC}<br/>
SYNTAX指的是Grok模式名称，SEMANTIC是给模式匹配到的文本字段名。例如：

```
%{NUMBER:duration} %{IP:client}
duration表示：匹配一个数字，client表示匹配一个IP地址。
```

> 默认在Grok中，所有匹配到的的数据类型都是字符串，如果要转换成int类型（目前只支持int和float），可以这样：%{NUMBER:duration:int} %{IP:client}

> 以下是常用的Grok模式：

类型 | 解释
---|---
NUMBER | 匹配数字（包含：小数）
INT | 匹配整形数字
POSINT | 匹配正整数
WORD | 匹配单词
DATA | 匹配所有字符
IP | 匹配IP地址
PATH | 匹配路径

##### 10.3.6.4 用法

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/44.png)

```
filter {
  grok {
    match => { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" }
  }
}
```

#### 10.3.7 匹配日志中的IP、日期并打印

```
235.9.200.242 - - [15/Apr/2015:00:27:19 +0849] "POST /itcast.cn/bigdata.html 200 45 "www.baidu.com" "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900 144.180.122.249
```

> 我们使用IP就可以把前面的IP字段匹配出来，使用HTTPDATE可以将后面的日期匹配出来。

> `配置Grok过滤插件`

> 1.配置Logstash

```
input {
    beats {
        port => 5044
    }
}

filter {
    grok {
        match => {
            "message" => "%{IP:ip} - - \[%{HTTPDATE:date}\]"
        }
    }   
}

output {
    stdout {
        codec => rubydebug
    }
}
```

> 2.启动Logstash

```
bin/logstash -f config/filebeat-filter-print.conf --config.reload.automatic
```

```
{
    "log" => {
        "offset" => 1861,
        "file" => {
            "path" => "/var/apache/log/access.log.1"
        }
    },
    "input" => {
        "type" => "log"
    },
    "tags" => [
        [0] "beats_input_codec_plain_applied"
    ],
    "date" => "15/Apr/2015:00:27:19 +0849",
    "ecs" => {
        "version" => "1.4.0"
    },
    "@timestamp" => 2020-06-01T11:02:05.809Z,
    "message" => "235.9.200.242 - - [15/Apr/2015:00:27:19 +0849] \"POST /itcast.cn/bigdata.html 200 45 \"www.baidu.com\" \"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900 144.180.122.249",
    "host" => {
        "name" => "node1.itcast.cn"
    },
    "ip" => "235.9.200.242",
    "agent" => {
        "hostname" => "node1.itcast.cn",
        "version" => "7.6.1",
        "ephemeral_id" => "d4c3b652-4533-4ebf-81f9-a0b78c0d4b05",
        "id" => "b4c5c4dc-03c3-4ba4-9400-dc6afcb36d64",
        "type" => "filebeat"
    },
    "@version" => "1"
}
```

> 我们看到，经过Grok过滤器插件处理之后，我们已经获取到了ip和date两个字段。接下来，我们就可以继续解析其他的字段。

#### 10.3.8 解析所有字段

> 将日志解析成以下字段：

字段名 | 说明
---|---
client IP | 浏览器端IP
timestamp | 请求的时间戳
method | 请求方式（GET/POST）
uri | 请求的链接地址
status | 服务器端响应状态
length | 响应的数据长度
reference | 从哪个URL跳转而来
browser | 浏览器

> 1.修改Logstash配置文件

```
input {
    beats {
        port => 5044
    }
}

filter {
    grok {
        match => {
            "message" => "%{IP:ip} - - \[%{HTTPDATE:date}\] \"%{WORD:method} %{PATH:uri} %{DATA}\" %{INT:status} %{INT:length} \"%{DATA:reference}\" \"%{DATA:browser}\""
        }
    }   
}

output {
    stdout {
        codec => rubydebug
    }
}
```

> 2.测试并启动Logstash
>> 我们可以看到，8个字段都已经成功解析。

```
{
    "reference" => "www.baidu.com",
    "@version" => "1",
    "ecs" => {
    "version" => "1.4.0"
},
"@timestamp" => 2020-06-02T03:30:10.048Z,
"ip" => "235.9.200.241",
"method" => "POST",
"uri" => "/itcast.cn/bigdata.html",
"agent" => {
    "id" => "b4c5c4dc-03c3-4ba4-9400-dc6afcb36d64",
    "ephemeral_id" => "734ae9d8-bcdc-4be6-8f97-34387fcde972",
    "version" => "7.6.1",
    "hostname" => "node1.itcast.cn",
    "type" => "filebeat"
},
"length" => "45",
"status" => "200",
"log" => {
    "file" => {
        "path" => "/var/apache/log/access.log"
    },
    "offset" => 1
},
"input" => {
    "type" => "log"
},
"host" => {
    "name" => "node1.itcast.cn"
},
"tags" => [
    [0] "beats_input_codec_plain_applied"
],
"browser" => "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900",
"date" => "15/Apr/2015:00:27:19 +0849",
"message" => "235.9.200.241 - - [15/Apr/2015:00:27:19 +0849] \"POST /itcast.cn/bigdata.html HTTP/1.1\" 200 45 \"www.baidu.com\" \"Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900\""
}
```

#### 10.3.9 将数据输出到ElasticSearch

> 到目前为止，我们已经通过了Grok Filter可以将日志消息解析成一个一个的字段，那现在我们需要将这些字段保存到Elasticsearch中。我们看到了Logstash的输出中，有大量的字段，但如果我们只需要保存我们需要的8个，该如何处理呢？而且，如果我们需要将日期的格式进行转换，我们又该如何处理呢？

##### 10.3.9.1 过滤出来需要的字段

> 要过滤出来我们需要的字段。我们需要使用mutate插件。mutate插件主要是作用在字段上，例如：它可以对字段进行重命名、删除、替换或者修改结构。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/46.png)

> 官方文档：https://www.elastic.co/guide/en/logstash/7.6/plugins-filters-mutate.html

> 例如，mutate插件可以支持以下常用操作

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/47.png)

> 配置：
>>注意：此处为了方便进行类型的处理，将status、length指定为int类型。

```
input {
    beats {
        port => 5044
    }
}

filter {
    grok {
        match => {
            "message" => "%{IP:ip} - - \[%{HTTPDATE:date}\] \"%{WORD:method} %{PATH:uri} %{DATA}\" %{INT:status:int} %{INT:length:int} \"%{DATA:reference}\" \"%{DATA:browser}\""
        }
    }
    mutate {
        enable_metric => "false"
        remove_field => ["message", "log", "tags", "@timestamp", "input", "agent", "host", "ecs", "@version"]
    }
}

output {
    stdout {
        codec => rubydebug
    }
}
```

##### 10.3.9.2 转换日期格式

> 要将日期格式进行转换，我们可以使用Date插件来实现。该插件专门用来解析字段中的日期，官方说明文档：https://www.elastic.co/guide/en/logstash/7.6/plugins-filters-date.html

> 用法如下：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/47.png)

> 将date字段转换为「年月日 时分秒」格式。默认字段经过date插件处理后，会输出到@timestamp字段，所以，我们可以通过修改target属性来重新定义输出字段。

> Logstash配置修改为如下：

```
input {
    beats {
        port => 5044
    }
}

filter {
    grok {
        match => {
            "message" => "%{IP:ip} - - \[%{HTTPDATE:date}\] \"%{WORD:method} %{PATH:uri} %{DATA}\" %{INT:status:int} %{INT:length:int} \"%{DATA:reference}\" \"%{DATA:browser}\""
        }
    }
    mutate {
        enable_metric => "false"
        remove_field => ["message", "log", "tags", "@timestamp", "input", "agent", "host", "ecs", "@version"]
    }
    date {
        match => ["date","dd/MMM/yyyy:HH:mm:ss Z","yyyy-MM-dd HH:mm:ss"]
        target => "date"
    }
}

output {
    stdout {
        codec => rubydebug
    }
}
```

> 启动Logstash：

```
bin/logstash -f config/filebeat-filter-print.conf --config.reload.automatic
```
```
{
"status" => "200",
"reference" => "www.baidu.com",
"method" => "POST",
"browser" => "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.104 Safari/537.36 Core/1.53.4549.400 QQBrowser/9.7.12900",
"ip" => "235.9.200.241",
"length" => "45",
"uri" => "/itcast.cn/bigdata.html",
"date" => 2015-04-14T15:38:19.000Z
}
```

##### 10.3.9.3 输出到ElasticSearch

> 我们可以通过

```
elasticsearch {
hosts => ["node1.itcast.cn:9200" ,"node2.itcast.cn:9200" ,"node3.itcast.cn:9200"]
index => "xxx"
}
```

> index来指定索引名称，默认输出的index名称为：logstash-%{+yyyy.MM.dd}。但注意，要在index中使用时间格式化，filter的输出必须包含 @timestamp字段，否则将无法解析日期。

```
input {
    beats {
        port => 5044
    }
}

filter {
    grok {
        match => {
            "message" => "%{IP:ip} - - \[%{HTTPDATE:date}\] \"%{WORD:method} %{PATH:uri} %{DATA}\" %{INT:status:int} %{INT:length:int} \"%{DATA:reference}\" \"%{DATA:browser}\""
        }
    }
    mutate {
        enable_metric => "false"
        remove_field => ["message", "log", "tags", "input", "agent", "host", "ecs", "@version"]
    }
    date {
        match => ["date","dd/MMM/yyyy:HH:mm:ss Z","yyyy-MM-dd HH:mm:ss"]
        target => "date"
    }
}

output {
    stdout {
        codec => rubydebug
    }
    elasticsearch {
        hosts => ["node1.itcast.cn:9200" ,"node2.itcast.cn:9200" ,"node3.itcast.cn:9200"]
        index => "apache_web_log_%{+YYYY-MM}"
    }
}
```

> 启动Logstash

```
bin/logstash -f config/filebeat-apache-weblog.conf --config.reload.automatic
```

> 注意：<br/>
`index名称中，不能出现大写字符`

## 11 Kibana

### 11.1 简介

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/48.png)

> 通过上面的这张图就可以看到，Kibana可以用来展示丰富的图表。
>> Kibana是一个开源的数据分析和可视化平台，使用Kibana可以用来搜索Elasticsearch中的数据，构建漂亮的可视化图形、以及制作一些好看的仪表盘
>> Kibana是用来管理Elastic stack组件的可视化平台。例如：使用Kibana可以进行一些安全设置、用户角色设置、对Elasticsearch进行快照等等
>> Kibana提供统一的访问入口，不管是日志分析、还是查找文档，Kibana提供了一个使用这些功能的统一访问入口
>> Kibana使用的是Elasticsearch数据源，Elasticsearch是存储和处理数据的引擎，而Kibana就是基于Elasticsearch之上的可视化平台

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/49.png)

> 主要功能：

- 探索和查询Elasticsearch中的数据

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/50.png)

- 可视化与分析

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/51.png)

### 11.2 Kibana安装

> 在Linux下安装Kibana，可以使用Elastic stack提供 tar.gz压缩包。官方下载地址：

```
https://www.elastic.co/cn/downloads/past-releases/kibana-7-6-1
```

> 1.解压Kibana gz压缩包

```
tar -xzf kibana-7.6.1-linux-x86_64.tar.gz
```

> 2.进入到Kibana目录

```
cd kibana-7.6.1-linux-x86_64/
```

> 3.配置Kibana

```
#本机host
server.host: "node1.itcast.cn"
# The Kibana server's name.  This is used for display purposes.，kibana服务名称
server.name: "itcast-kibana"

# The URLs of the Elasticsearch instances to use for all your queries.
elasticsearch.hosts:      ["http://node1.itcast.cn:9200","http://node2.itcast.cn:9200","http://node3.itcast.cn:9200"]
#设置中文
i18n.locale: "zh-CN"
```

> 4.运行Kibana

```
./bin/kibana
```

#### 11.2.1 查看Kibana状态

> 输入以下网址，可以查看到Kibana的运行状态：

```
http://node1.itcast.cn:5601/status
```

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/52.png)

#### 11.2.2 查看ElasticSearch的状态

> 点击齿轮设置按钮，再点击 「Index Management」，可以查看到Elasticsearch集群中的索引状态。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/53.png)

> 点击索引的名字，可以进一步查看索引更多的信息。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/54.png)

> 点击「Manage」按钮，还可以用来管理索引。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/55.png)

### 11.3 添加Elasticsearch数据源

#### 11.3.1 Kibana索引模式

> 在开始使用Kibana之前，我们需要指定想要对哪些Elasticsearch索引进行处理、分析。在Kibana中，可以通过定义索引模式（Index Patterns）来对应匹配Elasticsearch索引。在第一次访问Kibana的时候，系统会提示我们定义一个索引模式。或者我们可以通过点击按钮，再点击Kibana下方的Index Patterns，来创建索引模式。参考下图：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/56.png)

> 1.定义索引模式，用于匹配哪些Elasticsearch中的索引。点击「Next step」

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/57.png)

> 2.选择用于进行时间过滤的字段

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/58.png)

> 3.点击「Create Index Pattern」按钮，创建索引模式。创建索引模式成功后，可以看到显示了该索引模式对应的字段。里面描述了哪些可以用于搜索、哪些可以用来进行聚合计算等。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/59.png)

### 11.4 探索数据（Discovery）

> 通过Kibana中的Discovery组件，我们可以快速地进行数据的检索、查询。

#### 11.4.1 使用探索数据功能

> 点击指南针按钮可以打开Discovery页面。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/60.png)

> 我们发现没有展示任何的数据。但我们之前已经把数据导入到Elasticsearch中了。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/61.png)

> Kibana提示，让我们扩大我们的查询的时间范围。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/62.png)

> 默认Kibana是展示最近15分钟的数据。我们把时间范围调得更长一些，就可以看到数据了。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/63.png)

> 将时间范围选择为1年范围内的，我们就可以查看到Elasticsearch中的数据了。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/64.png)

#### 11.4.2 导入更多的Apache Web日志数据

> 1.将资料中的 access.log 文件上传到Linux

> 2.将access.log移动到/var/apache/log，并重命名为access.log.2

```
mv access.log /var/apache/log/access.log.2
```

> 3.启动FileBeat

```
./filebeat -e -c filebeat-logstash.yml
```

> 4.启动Logstash

```
bin/logstash -f config/filebeat-es.conf --config.reload.automatic
```

#### 11.4.3 基于时间过滤查询

##### 11.4.3.1 选择时间范围

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/65.png)

##### 11.4.3.2 指定查询某天的数据

> 查询2020年5月6日的所有日志数据。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/66.png)

##### 11.4.3.3 从直方图上选择日期更细粒度范围

> 如果要选择查看某一天的日志，上面这种方式会有一些麻烦，我们有更快更容易的方式。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/67.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/68.png)

#### 11.4.4 使用Kibana索索数据

> 在Kibana的Discovery组件中，可以在查询栏中输入搜索条件。默认情况下，可以使用Kibana内置的标准查询语言，来进行快速查询。还有一种是遗留的基于Lucene的查询语法目前暂时可用，这种查询语法也可以使用基于JSON的Elasticsearch DSL也是可用的。当我们在Discovery搜索数据时，对应的直方图、文档列表都会随即更新。默认情况下，优先展示最新的文档，按照时间倒序排序的。

##### 11.4.4.1 Kibabna查询语言（KQL）

> 在7.0中，Kibana上线了新的查询语言。这种语言简洁、易用，有利于快速查询。

> 查询语法：

```
「字段:值」，如果值是字符串，可以用双引号括起来。
```

> 查询包含zhihu的请求
```
*zhihu*
```

> 查询页面不存在的请求

```
status : 404
```
> 查询请求成功和不存在的请求

```
status: (404 or 200)
```

> 查询方式为POST请求，并请求成功的日志

```
status: 200 and method: post
```

> 查询方式为GET成功的请求，并且响应数据大于512的日志

```
status: 200 and method: get and length > 512
```

> 查询请求成功的且URL为「/itcast.cn」开头的日志

```
uri: "\/itcast.cn\/*"
```

> 注意：因为/为特殊字符，需要使用反斜杠进行转义

##### 11.4.4.2 过滤字段

> Kibana的Discovery组件提供各种各样的筛选器，这样可以筛选出来我们关注的数据上。例如：我们只想查询404的请求URI。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/69.png)

> 指定过滤出来404以及请求的URI、从哪儿跳转来的日志

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/70.png)

> 将查询保存下来，方便下次直接查看

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/71.png)

> 下次直接点击Open就可以直接打开之前保存的日志了

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/72.png)

### 11.5 数据可视化（Visualize）

> Kibana中的Visualize可以基于Elasticsearch中的索引进行数据可视化，然后将这些可视化图表添加到仪表盘中。

#### 11.5.1 数据可视化的类型

> Lens

- 通过简单地拖拽数据字段，快速构建基本的可视化
> 常用的可视化对象

- 线形图（Line）、面积图（Area）、条形图（Bar）：可以用这些带X/Y坐标的图形来进行不同分类的比较
- 饼图（Pie）：可以用饼图来展示占比
- 数据表（Data Table）：以数据表格的形式展示
- 指标（Metrics）：以数字的方式展示
- 目标和进度：显示带有进度指标的数字
-标签云/文字云（Tag Cloud）：以文字云方式展示标签，文字的大小与其重要性相关

> Timelion

- 从多个时间序列数据集来展示数据

> 地图

- 展示地理位置数据

> 热图

- 在矩阵的单元格展示数据

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/73.png)

> 仪表盘工具

- Markdown部件：显示一些MD格式的说明
- 控件：在仪表盘中添加一些可以用来交互的组件

> Vega

#### 11.5.2 以饼图展示404与200的占比

> 效果图：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/74.png)

> 操作步骤：

> 1.创建可视化

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/75.png)

> 2.选择要进行可视化图形类型，此处我们选择Pie（饼图类型）

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/76.png)

> 3.选择数据源

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/77.png)

> 4.添加分桶、分片（还记得吗？我们在Elasticsearch进行分组聚合都是以分桶方式进行的，可以把它理解为分组）

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/78.png)

> 5.配置分桶以及指标计算方式

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/79.png)

> 6.点击蓝色播放按钮执行。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/80.png)

> 7.保存图形（取名为：apache_log@404_200）

#### 11.5.3 以条形图方式展示2020年5月每日请求数

> 效果如下：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/81.png)

> 开发步骤：

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/82.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/83.png)

> 我们还可以修改图形的样式，例如：以曲线、面积图的方式展示。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/84.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/85.png)

#### 11.5.4 以TSVB可视化不同访问来源的数据

> TSVB是一个基于时间序列的数据可视化工具，它可以使用Elasticsearch聚合的所有功能。使用TSVB，我们可以轻松地完成任意聚合方式来展示复杂的数据。它可以让我们快速制作效果的图表：

> 1.基于时间序列的图形展示

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/86.png)

> 2.展示指标数据

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/87.png)

> 3.TopN

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/88.png)

> 4.类似油量表的展示

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/89.png)

> 5.Markdown自定义数据展示

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/90.png)

> 6.以表格方式展示数据

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/91.png)

> 操作步骤：

> 1.创建TSVB可视化对象

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/92.png)

> 2.配置Time Series数据源分组条件

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/93.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/94.png)

> 3.配置Metric

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/95.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/96.png)

> 4.TopN

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/97.png)

#### 11.5.5 制作用户选择请求方式、响应字节大小控制组件

##### 11.5.5.1 制作控件

> 在Kibana中，我们可以使用控件来控制图表的展示。例如：提供一个下列列表，供查看图表的用户只展示比较关注的数据。我们可以添加两个类型的控制组件：

> 1.选项列表

- 根据一个或多个指定选项来筛选内容。例如：我们先筛选某个城市的数据，就可以通过选项列表来选择该城市
> 2.范围选择滑块

- 筛选出来指定范围的数据。例如：我们筛选某个价格区间的商品等。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/98.png)

##### 11.5.5.2 Kibana开发

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/99.png)

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/100.png)

### 11.6 制作Dashboard

> 接下来，我们把前面的几个图形放到一个看板中。这样，我们就可以在一个看板中，浏览各类数据了。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/101.png)

> 1.点击第三个组件图标，并创建一个新的Dashboard。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/102.png)

> 2.点击Edit编辑Dashboard。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/103.png)

> 3.依次添加我们之前制作好的图表。

 ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_ElasticSearch/104.png)

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

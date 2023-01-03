---
title: CDH集群目录汇总
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(88).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(88).jpg
tags:
  - 大数据
  - CDH
category:
  - - 编程
    - 大数据
  - - 编程
    - CDH
date: 2021-08-04 10:40:52
---

最近需要改`CDH中组件的一些配置文件`，在`Cloudera可视化界面`就可以改。

但是`Java通过Phoenix访问Hbase`的时候，需要把`Hbase的配置文件`拿出来放到项目中。

所以这里把`CDH中的一些目录`记录一下，不然太多了根本记不住。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 日志目录

路径 | 说明
---|---
/var/log/cloudera-scm-agent | CDH客户端日志
/var/log/cloudera-scm-server | CDH服务端日志
/var/log/hadoop-hdfs | hadoop日志目录
/var/log/hadoop-mapreduce | mapreduce任务目录
/var/log/hadoop-yarn | yarn目录
/var/log/zookeeper | zookeeper目录
/var/log/kafka | kafka目录
/var/log/hbase | hbase日志目录
/var/log/hive | hive日志目录

### 1.1 solr日志位置

```
tail -f /var/log/solr/solr-cmf-solr-SOLR_SERVER-...(hostname).log.out
```

## 2 配置文件目录

路径 | 说明
---|---
/etc/cloudera-scm-agent/config.ini | cm agent的配置目录
/etc/cloudera-scm-server/ | cm server的配置目录
/etc/hadoop/conf | Hadoop各个组件的配置
/etc/hive/conf | hive配置文件目录
/etc/hbase/conf | hbase配置文件目录
/opt/cloudera/parcels/CDH/etc | hadoop集群以及组件的配置文件文件夹

## 3 安装各个组件位置

路径 | 说明
---|---
/opt/cloudera/parcels/CDH | 各组件安装目录

## 4 jar包目录

路径 | 说明
---|---
/opt/cloudera/parcels/CDH/jars | 所有jar包所在目录
/opt/cloudera/parcels/CDH/lib/ | 各个服务组件对应的jar包

## 5 Parcels包目录

路径 | 说明
---|---
/opt/cloudera/parcel-repo/ | 服务软件包数据(parcels)
/opt/cloudera/parcel-cache/ | 服务软件包缓存数据

## 6 服务运行时所有组件的配置文件目录

路径 | 说明
---|---
/var/run/cloudera-scm-agent/process/ | 运行时组件配置文件目录

> 包含文件内容

```shell
107-cluster-host-inspector
117-cluster-host-inspector
124-cluster-host-inspector
12-host-perf-inspector
131-cluster-host-inspector
138-cluster-host-inspector
145-cluster-host-inspector
152-cluster-host-inspector
159-cluster-host-inspector
166-cluster-host-inspector
173-cluster-host-inspector
180-cluster-host-inspector
187-cluster-host-inspector
194-cluster-host-inspector
19-cluster-host-inspector
201-cluster-host-inspector
208-cluster-host-inspector
215-cluster-host-inspector
222-cluster-host-inspector
229-cluster-host-inspector
236-cluster-host-inspector
243-cluster-host-inspector
250-cluster-host-inspector
257-cluster-host-inspector
264-cluster-host-inspector
26-cluster-host-inspector
271-cluster-host-inspector
278-cluster-host-inspector
285-cluster-host-inspector
292-cluster-host-inspector
299-cluster-host-inspector
306-cluster-host-inspector
313-cluster-host-inspector
320-cluster-host-inspector
327-cluster-host-inspector
334-cluster-host-inspector
33-host-perf-inspector
341-cluster-host-inspector
348-cluster-host-inspector
355-cluster-host-inspector
362-cluster-host-inspector
369-cluster-host-inspector
376-cluster-host-inspector
383-cluster-host-inspector
390-cluster-host-inspector
397-cluster-host-inspector
404-cluster-host-inspector
40-host-perf-inspector
411-cluster-host-inspector
418-cluster-host-inspector
425-cluster-host-inspector
432-cluster-host-inspector
439-cluster-host-inspector
446-cluster-host-inspector
453-cluster-host-inspector
460-cluster-host-inspector
476-cluster-host-inspector
479-cluster-host-inspector
47-host-perf-inspector
484-cluster-host-inspector
491-cluster-host-inspector
499-cluster-host-inspector
500-cluster-host-inspector
520-cluster-host-inspector
521-cluster-host-inspector
523-cluster-host-inspector
526-cluster-host-inspector
537-cluster-host-inspector
540-hdfs-NAMENODE-format
541-cloudera-mgmt-SERVICEMONITOR
542-cloudera-mgmt-EVENTSERVER
543-cloudera-mgmt-ALERTPUBLISHER
544-cloudera-mgmt-HOSTMONITOR
545-hdfs-NAMENODE-format
54-cluster-host-inspector
553-hdfs-NAMENODE
554-hdfs-NAMENODE-nnRpcWait
555-hdfs-NAMENODE-createtmp
556-dfs-create-dir
557-dfs-create-dir
558-dfs-create-dir
559-yarn-RESOURCE-MANAGER-upload-mr-framework-jars
562-yarn-RESOURCEMANAGER
568-cloudera-mgmt-SERVICEMONITOR
569-cloudera-mgmt-EVENTSERVER
570-cloudera-mgmt-ALERTPUBLISHER
571-cloudera-mgmt-HOSTMONITOR
579-hdfs-NAMENODE
580-hdfs-NAMENODE-nnRpcWait
584-yarn-RESOURCEMANAGER
596-hdfs-NAMENODE
597-hdfs-NAMENODE-nnRpcWait
601-yarn-RESOURCEMANAGER
613-hdfs-NAMENODE
614-hdfs-NAMENODE-nnRpcWait
618-yarn-RESOURCEMANAGER
61-cluster-host-inspector
623-cloudera-mgmt-SERVICEMONITOR
624-cloudera-mgmt-EVENTSERVER
625-cloudera-mgmt-ALERTPUBLISHER
626-cloudera-mgmt-HOSTMONITOR
627-cloudera-mgmt-SERVICEMONITOR
628-cloudera-mgmt-EVENTSERVER
629-cloudera-mgmt-ALERTPUBLISHER
630-cloudera-mgmt-HOSTMONITOR
666-cloudera-mgmt-SERVICEMONITOR
667-cloudera-mgmt-EVENTSERVER
668-cloudera-mgmt-ALERTPUBLISHER
669-cloudera-mgmt-HOSTMONITOR
673-collect-host-statistics
682-host-inspector
6-host-inspector
85-cluster-host-inspector
ccdeploy_hadoop-conf_etchadoopconf.cloudera.hdfs_5818835745948542271
ccdeploy_hadoop-conf_etchadoopconf.cloudera.hdfs_6752681924895224861
ccdeploy_hadoop-conf_etchadoopconf.cloudera.yarn_6904562266639177485
ccdeploy_hadoop-conf_etchadoopconf.cloudera.yarn_8922238058878386299
```

## 7 CDH集群相常用命令

### 7.1 操作CM

命令 | 说明
---|---
systemctl start cloudera-scm-server | 启动cm
systemctl restart cloudera-scm-server | 重启cm
systemctl stop cloudera-scm-server | 停止CM

### 7.2 查看日志

```shell
tail -f /var/log/kafka/kafka-broker-cdh1.log
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```

### 7.3 开启应用目录

```shell
/opt/cloudera/parcels/CDH/bin
```

### 7.4 查看组件版本号

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CDH集群目录汇总/1.png)

## 8 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
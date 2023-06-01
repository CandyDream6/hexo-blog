---
title: 记一次CDH磁盘不足排查 
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(93).jpg 
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(93).jpg 
tags:
  - 大数据
  - CDH 
category:
  - - 编程
    - 大数据
  - - 编程
    - CDH 
date: 2021-08-26 19:26:54
---

最近用公司`CDH`的时候，动不动就`万紫千红`的，搞得人`头大`，主节点只有`100G磁盘`，因此这次排查主要看是否`空间问题引起`。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 检查系统磁盘占用

### 1.1 查看磁盘空间占用情况

```shell
[root@node1 /]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                  16G     0   16G    0% /dev
tmpfs                     16G     0   16G    0% /dev/shm
tmpfs                     16G  9.0M   16G    1% /run
tmpfs                     16G     0   16G    0% /sys/fs/cgroup
/dev/mapper/centos-root   37G   27G   11G   73% /
/dev/vda2               1014M  211M  804M   21% /boot
/dev/vda1                200M   12M  189M    6% /boot/efi
/dev/sda1                197G   61M  187G    1% /data
cm_processes              16G  139M   16G    1% /run/cloudera-scm-agent/process
overlay                   37G   27G   11G   73% /var/lib/docker/overlay2/da4ac312ee41ce6d9f1bb353f0805acbb42f438c3a1c8b50b897372816232d15/merged
shm                       64M     0   64M    0% /var/lib/docker/containers/795835e333fb377858ddf928d3cf97555c5679567521b84a3420cba96484dc33/shm
tmpfs                    3.2G     0  3.2G    0% /run/user/0
```

> 发现磁盘使用了`73%`

### 1.2 查看那个目录占用空间大

```shell
[root@node1 /]# du -s /* | sort -nr          //也可以在-s附近加个-h，清晰一点：du -h -s /* | sort -nr
15058040	/var
8459224	/opt
3358884	/usr
741116	/root
415324	/dfs
194188	/boot
151480	/run
76752	/tmp
37532	/etc
13868	/bigdata
36	/data
24	/group_conf.java
16	/student.java
16	/OWHDC_STUDENT.java
16	/OWHDC_student.java
0	/sys
0	/srv
0	/sbin
0	/proc
0	/mnt
0	/media
0	/lib64
0	/lib
0	/impala
0	/home
0	/dev
0	/derby.log
0	/bin
```

> 发现var目录占用了`15个G`

```shell
du -s /var/* | sort -nr
```

> 然后那个目录占用多 再通过`du -s /var/* | sort -nr` 一层层排查，找到占用文件多的地方。

### 1.3 排查结果

> 这次排查主要发现三个`占用空间大的集群相关目录`

```shell
/var/log                //CDH各组件的日志一般放在这里
/var/lib                //监控服务的日志放在这里
/tmp                    //这个目录里会有很大的*.hprof文件,这是由于Service Monitor的内存溢出后不停往/tmp写文件导致
```

## 2 释放空间

> 既然找到了这文件占用大的地方，那么我们`清理一下`就好了，这里写个`脚本`清理一下

```shell
#!/bin/bash

echo "清除监控内存溢出后的溢写文件,是/tmp/*.hprof"
rm -rf /tmp/*.hprof

echo "清除监控服务产生的日志,主要在/var/lib下"
rm /var/lib/cloudera-host-monitor/ts/*/partition*/* -rf
rm /var/lib/cloudera-service-monitor/ts/*/partition*/* -rf

echo "清除部分CDH组件日志，这些日志都在/var/log下，这里我先只清除cm、hdfs、yarn、kudu、hive、zookeeper的日志"
rm -rf /var/log/cloudera-scm-eventserver/*.out.*
rm -rf /var/log/cloudera-scm-firehose/*.out.*
rm -rf /var/log/cloudera-scm-agent/*.log.*
rm -rf /var/log/cloudera-scm-agent/*.out.*
rm -rf /var/log/cloudera-scm-server/*.out.*
rm -rf /var/log/cloudera-scm-server/*.log.*
rm -rf /var/log/hadoop-hdfs/*.out.*
rm -rf /var/log/hadoop-httpfs/*.out.*
rm -rf /var/log/hadoop-kms/*.out.*
rm -rf /var/log/hadoop-mapreduce/*.out.*
rm -rf /var/log/hadoop-yarn/*.out.*
rm -rf /var/log/hadoop-kudu/*.out.*
rm -rf /var/log/hadoop-hive/*.out.*
rm -rf /var/log/zookeeper/*.log.*

echo "释放空间成功"
echo "请手动去CDH界面重启一下Cloudera Management Service,不然监控服务不生效"
```

## 3 手动去CDH界面重启一下Cloudera Management Service

> 务必`重启`一下`Cloudera Management Service`,不然`监控服务``不生效`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/记一次CDH磁盘不足排查/1.png)

## 4 头顶一片青青草原真好看

> `这该死`的`绿帽子`真`惹人爱`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/记一次CDH磁盘不足排查/2.png)

## 5 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路

![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

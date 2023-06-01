---
title: HDFS 高可用集群搭建
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(38).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(38).jpg
tags:
  - Cenos7
  - 大数据
  - Hadoop
  - HDFS
category:
  - - 编程
    - 大数据
 
date: 2019-08-01 21:39:58
---

一般的hdfs集群存在单点故障问题，

那么我们就需要搭建高可用的hdfs集群，

今天就记录一下这个过程，

日后需要的时候自己再来看。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 集群规划

> 2.x单节点架构

![2.x单节点架构](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HDFS高可用集群搭建/1.png)

> 2.x高可用架构

![2.x高可用架构](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HDFS高可用集群搭建/2.png)

> 完全分布式，实现namenode高可用，ResourceManager的高可用

> 集群运行服务规划

![集群运行服务规划](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HDFS高可用集群搭建/3.png)


## 2 安装包解压

和之前普通搭建一样，下载包解压就可以

## 3 配置文件修改

**`以下操作都在node1机器上进行`**

修改下面三个文件，其他文件参照{% post_link 大数据_Hadoop集群搭建 hadoop集群搭建%}去配置

### 3.1 修改core-site.xml

<details>
<summary>core-site.xml配置</summary>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
	<!-- 指定NameNode的HA高可用的zk地址  -->
	 <property>
		   <name>ha.zookeeper.quorum</name>
		   <value>node1:2181,node2:2181,node3:2181</value>
	 </property>
<!-- 用于设置Hadoop的文件系统，由URI指定 -->
	 <property>
		    <name>fs.defaultFS</name>
		    <value>hdfs://ns</value>
	 </property>
<!-- 配置Hadoop存储数据目录,默认/tmp/hadoop-${user.name} -->
	 <property>
		   <name>hadoop.tmp.dir</name>
		   <value>/export/server/hadoop-2.7.5/hadoopDatas/tempDatas</value>
	</property>

	<!--  缓冲区大小，实际工作中根据服务器性能动态调整-->
	 <property>
		   <name>io.file.buffer.size</name>
		   <value>4096</value>
	 </property>

	<!--  开启hdfs的垃圾桶机制，删除掉的数据可以从垃圾桶中回收，单位分钟 -->
	 <property>
		   <name>fs.trash.interval</name>
		   <value>10080</value>
	 </property>


</configuration>
```
</details>

### 3.2 修改hdfs-site.xml

<details>
<summary>hdfs-site.xml配置</summary>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
	<!--  指定命名空间  -->
	<property>
		<name>dfs.nameservices</name>
		<value>ns</value>
	</property>
	
	<!--  指定该命名空间下的两个机器作为我们的NameNode  -->
	<property>
		<name>dfs.ha.namenodes.ns</name>
		<value>nn1,nn2</value>
	</property>
	<!-- 配置第一台服务器的namenode通信地址  -->
	<property>
		<name>dfs.namenode.rpc-address.ns.nn1</name>
		<value>node1:8020</value>
	</property>
	
	<!--  配置第二台服务器的namenode通信地址  -->
	<property>
		<name>dfs.namenode.rpc-address.ns.nn2</name>
		<value>node2:8020</value>
	</property>
	
	<!-- 所有从节点之间相互通信端口地址 -->
	<property>
		<name>dfs.namenode.servicerpc-address.ns.nn1</name>
		<value>node1:8022</value>
	</property>
	
	<!-- 所有从节点之间相互通信端口地址 -->
	<property>
		<name>dfs.namenode.servicerpc-address.ns.nn2</name>
		<value>node2:8022</value>
	</property>
	
	
	<!-- 第一台服务器namenode的web访问地址  -->
	<property>
		<name>dfs.namenode.http-address.ns.nn1</name>
		<value>node1:50070</value>
	</property>
	<!-- 第二台服务器namenode的web访问地址  -->
	<property>
		<name>dfs.namenode.http-address.ns.nn2</name>
		<value>node2:50070</value>
	</property>
	
	<!-- journalNode的访问地址，注意这个地址一定要配置 -->
	<property>
		<name>dfs.namenode.shared.edits.dir</name>
		<value>qjournal://node1:8485;node2:8485;node3:8485/ns1</value>
	</property>
	
	
	<!--  指定故障自动恢复使用的哪个java类 -->
	<property>
		<name>dfs.client.failover.proxy.provider.ns</name>
		<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	</property>
	
		<!-- 故障转移使用的哪种通信机制 -->
	<property>
		<name>dfs.ha.fencing.methods</name>
		<value>sshfence</value>
	</property>
	
	<!-- 指定通信使用的公钥  -->
	<property>
		<name>dfs.ha.fencing.ssh.private-key-files</name>
		<value>/root/.ssh/id_rsa</value>
	</property>
	
	
	<!-- journalNode数据存放地址  -->
	<property>
		<name>dfs.journalnode.edits.dir</name>
		<value>/export/servers/hadoop-2.7.5/hadoopDatas/dfs/jn</value>
	</property>
	
	<!-- 启用自动故障恢复功能 -->
	<property>
		<name>dfs.ha.automatic-failover.enabled</name>
		<value>true</value>
	</property>
	
	
	
	
	<!-- 指定namenode元数据的存放位置 -->
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:///export/servers/hadoop-2.7.5/hadoopDatas/namenodeDatas</value>
	</property>
	<!--  定义datanode数据存储的节点位置 -->
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:///export/servers/hadoop-2.7.5/hadoopDatas/datanodeDatas</value>
	</property>	
	<!-- 定义namenode的edits文件存放路径 -->
	<property>
		<name>dfs.namenode.edits.dir</name>
		<value>file:///export/servers/hadoop-2.7.5/hadoopDatas/nn/edits</value>
	</property>

	
	<!-- 文件切片的副本个数-->
	<property>
		<name>dfs.replication</name>
		<value>3</value>
	</property>

	<!-- 设置HDFS的文件权限-->
	<property>
		<name>dfs.permissions</name>
		<value>false</value>
	</property>
	<!-- 设置一个文件切片的大小：128M-->
	<property>
		<name>dfs.blocksize</name>
		<value>134217728</value>
	</property>
	<!-- 指定DataNode的节点配置文件 -->
	<property>
		 <name> dfs.hosts </name>
		 <value>/export/servers/hadoop-2.7.5/etc/hadoop/slaves</value>
	</property>

</configuration>
```
</details>

### 3.3 修改yarn-site.xml，注意node3与node2配置不同

> **yarn.resourcemanager.ha.id这个配置在node1上不配置，在node2上配置rm1，在node3上配置rm2**

<details>
<summary>yarn-site.xml配置</summary>

```xml
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>
	
	<!--开启resource manager HA,默认为false--> 
	<property>
			<name>yarn.resourcemanager.ha.enabled</name>
			<value>true</value>
	</property>
	
	<!-- 集群的Id，使用该值确保RM不会做为其它集群的active -->
	<property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>mycluster</value>
	</property>
	
	<!--配置resource manager  命名-->
	<property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
	</property>
	<!-- 配置第一台机器的resourceManager -->
	<property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>node2</value>
	</property>
	
	<!-- 配置第二台机器的resourceManager -->
	<property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>node3</value>
	</property>
	
	<!-- 配置第一台机器的resourceManager通信地址 -->
	<property>
        <name>yarn.resourcemanager.address.rm1</name>
        <value>node2:8032</value>
	</property>
	
	<property>
        <name>yarn.resourcemanager.scheduler.address.rm1</name>
        <value>node2:8030</value>
	</property>
	
	<property>
        <name>yarn.resourcemanager.resource-tracker.address.rm1</name>
        <value>node2:8031</value>
	</property>
	
	<property>
        <name>yarn.resourcemanager.admin.address.rm1</name>
        <value>node2:8033</value>
	</property>
	
	<property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>node2:8088</value>
	</property>
	
	<!-- 配置第二台机器的resourceManager通信地址 -->
	<property>
        <name>yarn.resourcemanager.address.rm2</name>
        <value>node3:8032</value>
	</property>
	<property>
        <name>yarn.resourcemanager.scheduler.address.rm2</name>
        <value>node3:8030</value>
	</property>
	<property>
        <name>yarn.resourcemanager.resource-tracker.address.rm2</name>
        <value>node3:8031</value>
	</property>
	<property>
        <name>yarn.resourcemanager.admin.address.rm2</name>
        <value>node3:8033</value>
	</property>
	<property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>node3:8088</value>
	</property>
	
	<!--开启resourcemanager自动恢复功能-->
	<property>
        <name>yarn.resourcemanager.recovery.enabled</name>
        <value>true</value>
	</property>
	
	<!--在node2上配置rm1,在node3上配置rm2,注意：一般都喜欢把配置好的文件远程复制到其它机器上，但这个在YARN的另一个机器上一定要修改，其他机器上不配置此项-->
	<property>       
		<name>yarn.resourcemanager.ha.id</name>
		<value>rm1</value>
       <description>If we want to launch more than one RM in single node, we need this configuration</description>
	</property>
	
	  <!--用于持久存储的类。尝试开启-->
	<property>
        <name>yarn.resourcemanager.store.class</name>
        <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
	</property>
	
	<property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>node1:2181,node2:2181,node3:2181</value>
        <description>For multiple zk services, separate them with comma</description>
	</property>
	
	<!--开启resourcemanager故障自动切换，指定机器--> 
	<property>
        <name>yarn.resourcemanager.ha.automatic-failover.enabled</name>
        <value>true</value>
        <description>Enable automatic failover; By default, it is enabled only when HA is enabled.</description>
	</property>
	
	<property>
        <name>yarn.client.failover-proxy-provider</name>
        <value>org.apache.hadoop.yarn.client.ConfiguredRMFailoverProxyProvider</value>
	</property>
	
	
	<!--rm失联后重新链接的时间--> 
	<property>
        <name>yarn.resourcemanager.connect.retry-interval.ms</name>
        <value>2000</value>
	</property>
	
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	
	<!-- 开启日志聚合功能 -->
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
	</property>
	<!-- 设置聚合日志在hdfs上的保存时间 -->
	<property>
		<name>yarn.log-aggregation.retain-seconds</name>
		<value>604800</value>
	</property>
	<!-- 设置yarn集群的内存分配方案 -->
	<property>    
		<name>yarn.nodemanager.resource.memory-mb</name>    
		<value>20480</value>
	</property>
	<property>  
        	 <name>yarn.scheduler.minimum-allocation-mb</name>
         	<value>2048</value>
	</property>
	<property>
		<name>yarn.nodemanager.vmem-pmem-ratio</name>
		<value>2.1</value>
	</property>
</configuration>

```

</details>

## 4 集群启动

### 4.1 启动zookeeper集群

> 三台机器都执行

```
cd /export/servers/zookeeper-3.4.6/bin
zkServer.sh start
```

### 4.2 启动HDFS

```
cd /export/servers/hadoop-2.7.5
bin/hdfs zkfc -formatZK  (这时候zookeeper里会注册一个hadoop-ha的Znode,如下图)
sbin/hadoop-daemons.sh start journalnode
bin/hdfs namenode -initializeSharedEdits -force
sbin/start-dfs.sh
```

![启动HDFS](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HDFS高可用集群搭建/4.png)

启动后的进程为

![启动HDFS后的进程](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_HDFS高可用集群搭建/5.png)

> 在node2上执行

```
cd /export/servers/hadoop-2.7.5
bin/hdfs namenode -bootstrapStandby
sbin/hadoop-daemon.sh start namenode
```

### 4.3 启动yarn

> 在node2和node3上执行

```
cd   /export/servers/hadoop-2.7.5
sbin/start-yarn.sh
```

### 4.4 查看resourceManager状态

> 在node2执行

```
cd   /export/servers/hadoop-2.7.5
bin/yarn rmadmin -getServiceState rm1
```

> 在node3执行

```
cd   /export/servers/hadoop-2.7.5
bin/yarn rmadmin -getServiceState rm2
```

### 4.5 node3启动jobHistory

> 在node3执行

```
cd /export/servers/hadoop-2.7.5
sbin/mr-jobhistory-daemon.sh start historyserver
```

### 4.6 hdfs状态查看

> node1机器查看hdfs状态

http://192.168.88.161:50070/dfshealth.html#tab-overview

>node2机器查看hdfs状态

http://192.168.88.162:50070/dfshealth.html#tab-overview

### 4.7 yarn集群访问查看

http://192.168.88.163:8088/cluster

### 4.8 历史任务浏览界面

http://192.168.88.163:19888/jobhistory

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
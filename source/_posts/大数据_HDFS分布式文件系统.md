---
title: HDFS 分布式文件系统
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(35).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(35).jpg
tags:
  - Cenos7
  - 大数据
  - Hadoop
  - HDFS
category:
  - - 编程
    - 大数据
 
date: 2019-07-17 21:39:01
---

HDFS（Hadoop Distributed File System）是 Apache Hadoop 项目的一个子项目。

 Hadoop 非常适于存储大型数据 (比如 TB 和 PB), 其就是使用 HDFS 作为存储系统.。
 
 HDFS 使用多台计算机存储文件, 并且提供统一的访问接口, 像是访问一个普通文件系统一样使用分布式文件系统. 

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 架构

> HDFS的四个基本组件:**HDFS Client**、**NameNode**、**DataNode** 和 **Secondary NameNode**。

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/1.png)

> 1、Client：就是客户端。

```
文件切分。文件上传 HDFS 的时候，Client 将文件切分成 一个一个的Block，然后进行存储

与 NameNode 交互，获取文件的位置信息。

与 DataNode 交互，读取或者写入数据。

Client 提供一些命令来管理 和访问HDFS，比如启动或者关闭HDFS。
```

> 2、NameNode：就是 master，它是一个主管、管理者。

```
管理 HDFS 的名称空间

管理数据块（Block）映射信息

配置副本策略

处理客户端读写请求。
```
> 3、DataNode：就是Slave。NameNode 下达命令，DataNode 执行实际的操作。

```
存储实际的数据块。

执行数据块的读/写操作。
```
> 4、Secondary NameNode：并非 NameNode 的热备。当NameNode 挂掉的时候，它并不能马上替换 NameNode 并提供服务。

```
辅助 NameNode，分担其工作量。

定期合并 fsimage和fsedits，并推送给NameNode。

在紧急情况下，可辅助恢复 NameNode。
```

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/2.png)

## 2 副本机制

### 2.1 文件副本机制

> HDFS被设计成能够在一个大集群中跨机器可靠地存储超大文件。**它将每个文件存储成一系列的数据块，这个数据块被称为block**，除了最后一个，所有的数据块都是同样大小的。

> 为了容错，文件的所有block都会有副本。每个文件的数据块大小和副本系数都是可配置的。

> 所有的文件都是以 block 块的方式存放在 HDFS 文件系统当中,作用如下

1. 一个文件有可能大于集群中任意一个磁盘，引入块机制,可以很好的解决这个问题
2. 使用块作为文件存储的逻辑单位可以简化存储子系统
3. 块非常适合用于数据备份进而提供数据容错能力
4. 副本优点是安全，缺点是占空间

> 在 Hadoop1 当中, 文件的 block 块默认大小是 64M, hadoop2 当中, 文件的 block 块大小默认是 128M（134217728字节）。假设文件大小是100GB，从字节位置0开始，每128MB字节划分为一个block，依此类推，可以划分出很多的block。每个block就是128MB大小。

> block块的大小可以通过 hdfs-site.xml 当中的配置文件进行指定，Hadoop默认的副本数为3,也就是每个block会存三份。


```
<property>
   <name>dfs.block.size</name>
   <value>块大小以字节为单位</value>
</property>\
```

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/3.png)

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/4.png)

**注意当一个文件的大小不足128M时，比如文件大小为2M，那么这个文件也占用一个block，但是这个block实际只占2M的空间，所以从某种意义上来讲，block只是一个逻辑单位。**

### 2.2 副本放置策略（机架感知）

> HDFS分布式文件系统的内部有一个副本存放策略，默认副本数为3,在这里以副本数=3为例：

> 第一副本：优先放置到离写入客户端最近的DataNode节点，如果上传节点就是DataNode，则直接上传到该节点，如果是集群外提交，则随机挑选一台磁盘不太慢，CPU不太忙的节点。

> 第二个副本：放置在与第一个同机架的不同机器中

> 第三个副本：放置在另一个机架中, 某一个服务器中

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/5.png)

## 3 Shell命令行使用

[官网手册](https://hadoop.apache.org/docs/r2.7.5/hadoop-project-dist/hadoop-common/FileSystemShell.html#appendToFile)

## 4 高级使用命令

### 4.1 HDFS的安全模式

安全模式是hadoop的一种保护机制，用于保证集群中的数据块的安全性。当集群启动的时候，会首先进入安全模式。当系统处于安全模式时会检查数据块的完整性。

假设我们设置的副本数（即参数dfs.replication）是3，那么在datanode上就应该有3个副本存在，假设只存在2个副本，那么比例就是2/3=0.666。hdfs默认的副本率0.999。我们的副本率0.666明显小于0.999，因此系统会自动的复制副本到其他dataNode，使得副本率不小于0.999。如果系统中有5个副本，超过我们设定的3个副本，那么系统也会删除多于的2个副本。

> **在安全模式状态下，文件系统只接受读数据请求，而不接受删除、修改等变更请求**。<br/>在当整个系统达到安全标准时，HDFS自动离开安全模式。

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/6.png)

> 安全模式操作命令

```
hdfs  dfsadmin -safemode get #查看安全模式状态
hdfs  dfsadmin -safemode  enter #进入安全模式
hdfs  dfsadmin -safemode  leave #离开安全模式
```

### 4.2 HDFS基准测试

> 实际生产环境当中，hadoop的环境搭建完成之后，第一件事情就是进行压力测试，测试我们的集群的读取和写入速度，测试我们的网络带宽是否足够等一些基准测试

#### 8.2.1 测试写入速度

> 向HDFS文件系统中写入数据,10个文件,每个文件10MB,文件存放到/benchmarks/TestDFSIO中

```
hadoop jar /export/servers/hadoop-2.7.5/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.5.jar TestDFSIO -write -nrFiles 10 -fileSize 10MB
```

> 完成之后查看写入速度结果

```
hadoop fs -text /benchmarks/TestDFSIO/io_write/part-00000
```

#### 4.2.2 测试读取速度

> 测试hdfs的读取文件性能
<br/>在HDFS文件系统中读入10个文件,每个文件10M

```
hadoop jar /export/servers/hadoop-2.7.5/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.5.jar  TestDFSIO -read -nrFiles 10 -fileSize 10MB
```

> 查看读取果

```
hadoop fs -text /benchmarks/TestDFSIO/io_read/part-00000
```

#### 4.2.3 清除测试数据

```
 hadoop jar /export/servers/hadoop-2.7.5/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.5.jar TestDFSIO -clean
```

## 5 基本原理

### 5.1 NameNode

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/7.png)

#### 5.1.1 概念

NameNode在内存中保存着整个文件系统的名称空间和文件数据块的地址映射

整个HDFS可存储的文件数受限于NameNode的内存大小

> 1 NameNode元数据信息 

文件名，文件目录结构，文件属性(生成时间，副本数，权限)每个文件的块列表。 以及列表中的块与块所在的DataNode之间的地址映射关系 在内存中加载文件系统中每个文件和每个数据块的引用关系(文件、block、datanode之间的映射信息) 数据会定期保存到本地磁盘（fsImage文件和edits文件）

> 2 NameNode文件操作

NameNode负责文件元数据的操作 DataNode负责处理文件内容的读写请求，数据流不经过NameNode，会询问它跟那个DataNode联系
> 3 DataNode副本

文件数据块到底存放到哪些DataNode上，是由NameNode决定的，NameNode根据全局情况做出放置副本的决定 

> 4 NameNode心跳机制

全权管理数据块的复制，周期性的接受心跳和块的状态报告信息（包含该DataNode上所有数据块的列表） 若接受到心跳信息，NameNode认为DataNode工作正常，如果在10分钟后还接受到不到DataNode的心跳，那么NameNode认为DataNode已经宕机 ,这时候NN准备要把DN上的数据块进行重新的复制。 块的状态报告包含了一个DataNode上所有数据块的列表，blocks report 每个6小时发送一次.（默认6小时，去官网看就好了）

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/8.png)

#### 5.1.2 作用

> 1 NameNode是HDFS的核心。

> 2 NameNode也称为Master。

> 3 NameNode仅存储HDFS的元数据：文件系统中所有文件的目录树，并跟踪整个集群中的文件。

> 4 NameNode不存储实际数据或数据集。数据本身实际存储在DataNodes中。

> 5 NameNode知道HDFS中任何给定文件的块列表及其位置。使用此信息NameNode知道如何从块中构建文件。

> 6 NameNode并不持久化存储每个文件中各个块所在的DataNode的位置信息，这些信息会在系统启动时从数据节点重建。

> 7 NameNode对于HDFS至关重要，当NameNode关闭时，HDFS / Hadoop集群无法访问。

> 8 NameNode是Hadoop集群中的单点故障。

> 9 NameNode所在机器通常会配置有大量内存（RAM）。

### 5.2 DataNode

> 1 Data Node以数据块的形式存储HDFS文件

> 2 DataNode也称为Slave。

> 3 NameNode和DataNode会保持不断通信。

> 4 DataNode启动时，它将自己发布到NameNode并汇报自己负责持有的块列表。

> 5 datanode 每隔6个小时, 会向namenode报告一次完整的块信息

> 6 当某个DataNode关闭时，它不会影响数据或群集的可用性。NameNode将安排由其他DataNode管理的块进行副本复制。

> 7 DataNode所在机器通常配置有大量的硬盘空间。因为实际数据存储在DataNode中。

> 8 DataNode会定期（dfs.heartbeat.interval配置项配置，默认是3秒）向NameNode发送心跳，如果NameNode长时间没有接受到DataNode发送的心跳， NameNode就会认为该DataNode失效。

`timeout = 2 * heartbeat.recheck.interval + 10 * dfs.heartbeat.interval`<br/>

而默认的heartbeat.recheck.interval大小为5分钟(单位毫秒)，<br/>dfs.heartbeat.interval默认大小为3秒.

**所以namenode如果在10分钟+30秒后，仍然没有收到datanode的心跳，就认为datanode已经宕机，并标记为dead。**

> 9 datanode block汇报时间间隔取参数dfs.blockreport.intervalMsec,参数未配置的话默认为6小时.

## 6 HDFS的读写工作机制

### 6.1 HDFS写数据流程

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/9.png)

### 6.2 HDFS读数据流程

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/10.png)

## 7 元数据辅助管理

当 Hadoop 的集群当中, NameNode的所有元数据信息都保存在了 FsImage 与 Eidts 文件当中, 这两个文件就记录了所有的数据的元数据信息, 元数据信息的保存目录配置在了 **hdfs-site.xml** 当中

```
<property>
   <name>dfs.namenode.name.dir</name>    
   <value>
       file:///export/serverss/hadoop2.7.5/hadoopDatas/namenodeDatas</value>
</property>
<property>
    <name>dfs.namenode.edits.dir</name>
    <value>file:///export/serverss/hadoop-2.7.5/hadoopDatas/nn/edits</value>
</property>>
```

### 7.1 FsImage和Edits

> edits

```
edits 是在NameNode启动时对整个文件系统的快照存放了客户端最近一段时间的操作日志

客户端对 HDFS 进行写文件时会首先被记录在 edits 文件中

edits 修改时元数据也会更新
```

> FsImage

```
fsimage是在NameNode启动时对整个文件系统的快照

NameNode 中关于元数据的镜像, 一般称为检查点, fsimage 存放了一份比较完整的元数据信息

因为 fsimage 是 NameNode 的完整的镜像, 如果每次都加载到内存生成树状拓扑结构，这是非常耗内存和CPU, 所以一般开始时对 NameNode 的操作都放在 edits 中

fsimage 内容包含了 NameNode 管理下的所有 DataNode 文件及文件 block 及 block 所在的 DataNode 的元数据信息.

随着edits 内容增大, 就需要在一定时间点和 fsimage 合并
```

### 7.2 SecondaryNameNode的作用

```
SecondaryNameNode的作用是合并fsimage和edits文件。
NameNode的存储目录树的信息，而目录树的信息则存放在fsimage文件中，当NameNode启动的时候会首先读取整个fsimage文件，将信息装载到内存中。
Edits文件存储日志信息，在NameNode上所有对目录的操作，增加，删除，修改等都会保存到edits文件中，并不会同步到fsimage中，当NameNode关闭的时候，也不会将fsimage和edits进行合并。
所以当NameNode启动的时候，首先装载fsimage文件，然后按照edits中的记录执行一遍所有记录的操作，最后把信息的目录树写入fsimage中，并删掉edits文件，重新启用新的edits文件。
```

### 7.3 SecondaryNameNode出现的原因

```
但是如果NameNode执行了很多操作的话，就会导致edits文件会很大，那么在下一次启动的过程中，就会导致NameNode的启动速度很慢，慢到几个小时也不是不可能，所以出现了SecondNameNode。
```

### 7.4 SecondaryNameNode唤醒合并的规则

```
SecondaryNameNode 会按照一定的规则被唤醒，进行fsimage和edits的合并，防止文件过大。
合并的过程是，将NameNode的fsimage和edits下载到SecondryNameNode 所在的节点的数据目录，然后合并到fsimage文件，最后上传到NameNode节点。合并的过程中不影响NameNode节点的操作
SecondaryNameNode被唤醒的条件可以在hdfs-site.xml中配置：

dfs.namenode.checkpoint.period：单位秒，默认值3600，检查点的间隔时间，当距离上次检查点执行超过该时间后启动检查点，就是edits和fsimage的合并
dfs.namenode.checkpoint.txns：事务操作次数，默认值1000000，当edits文件事务操作超过这个次数，就进行edits和fsimage的合并
dfs.namenode.checkpoint.check.period：单位秒，默认值60。1分钟检查一次操作次数

```

**[core-site.xml]**

```
<!-- 多久进行edits和fsimage的合并 -->
<property>
	<name>dfs.namenode.checkpoint.period</name>
	<value>3600</value>
</property>
<!-- 多少次事务操作之后进行edits和fsimage的合并 -->
<property>
	<name>dfs.namenode.checkpoint.txns</name>
	<value>1000000</value>
</property>
<!-- 1分钟检查一次操作次数 -->
<property>
	<name>dfs.namenode.checkpoint.check.period</name>
	<value>60</value>
</property>
```

SecondaryNameNode一般处于休眠状态，当两个检查点满足一个，即唤醒SecondaryNameNode执行合并过程。

### 7.5 SecondaryNameNode工作过程

```
第一步：将hdfs更新记录写入一个新的文件——edits.new。
第二步：将fsimage和editlog通过http协议发送至secondary namenode。
第三步：将fsimage与editlog合并，生成一个新的文件——fsimage.ckpt。这步之所以要在secondary namenode中进行，是因为比较耗时，如果在namenode中进行，或导致整个系统卡顿。
第四步：将生成的fsimage.ckpt通过http协议发送至namenode。
第五步：重命名fsimage.ckpt为fsimage，edits.new为edits。
第六步：等待下一次checkpoint触发SecondaryNameNode进行工作，一直这样循环操作。
```

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/11.png)

注意:SecondaryNameNode 在合并 edits 和 fsimage 时需要消耗的内存和 NameNode 差不多, 所以一般

> SNN服务器内存一般要>=NameNode内存<br/>
SNN和NameNode放在不同的机器上

### 7.6 FsImage中的文件信息查看

使用命令

```
hdfs oiv
```

```
cd /export/server/hadoop-2.7.5/hadoopDatas/namenodeDatas/
ll
cd current/
cp fsimage_0000000000000000150 /export/data/
cd /export/data && ll
hdfs oiv -i fsimage_0000000000000000150 -p XML -o hello.xml
```

### 7.7 edits中的文件信息查看

使用命令

```
hdfs oev
```

```
cd /export/server/hadoop-2.7.5/hadoopDatas/nn/edits
cd /current && ll
cp edits_0000000000000000016-0000000000000000148 /export/data/
cd /export/data/
hdfs oev -i   edits_0000000000000000016-0000000000000000148 -p XML -o myedit.xml
```

### 7.8 NameNode元数据恢复

当NameNode发生故障时,我们可以通过将SecondaryNameNode中数据拷贝到NameNode存储数据的目录的方式来恢复NameNode的数据
操作步骤:

> 1 杀死NameNode进程

```
kill -9 NameNode进程号
```

> 2 删除NameNode存储的数据

```
rm -rf /export/server/hadoop-2.7.5/hadoopDatas/namenodeDatas/*
rm -rf /export/server/hadoop-2.7.5/hadoopDatas/nn/edits/*
```

> 3 拷贝SecondaryNameNode中数据到原NameNode存储数据目录

```
cd /export/server/hadoop-2.7.5/hadoopDatas/namenodeDatas/
scp -r node2:/export/server/hadoop-2.7.5/hadoopDatas/snn/name/* ./
```

```
cd /export/server/hadoop-2.7.5/hadoopDatas/nn/edits
scp -r node2:/export/server/hadoop-2.7.5/hadoopDatas/dfs/snn/edits/* ./
```

> 4 重新启动NameNode

```
hadoop-daemon.sh start namenode
```

## 8 API操作

### 8.1 配置Windows下Hadoop环境

在windows上做HDFS客户端应用开发，需要设置Hadoop环境,而且要求是windows平台编译的Hadoop

> 1 将已经编译好的Windows版本Hadoop解压到到一个没有中文没有空格的路径下面

```
链接：https://pan.baidu.com/s/1AxiSFAFNw88o7XVHxwebGA 
提取码：6666 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V3的分享
```

> 2 在windows上面配置hadoop的环境变量： HADOOP_HOME，并将%HADOOP_HOME%\bin添加到path中

![环境变量配置](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/16.png)

![环境变量配置](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/17.png)

> 3 把hadoop2.7.5文件夹中bin目录下的hadoop.dll文件放到系统盘:  C:\Windows\System32 目录下

> 4 关闭windows重启

### 8.2 导入maven依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.7.5</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>2.7.5</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>2.7.5</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-core</artifactId>
        <version>2.7.5</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13</version>
    </dependency>
</dependencies>
```

### 8.3 java操作hadoop

<details>
<summary>代码示例</summary>

```java
package cn.itcast.hdfs;

import org.apache.commons.io.IOUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.*;
import org.junit.Test;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

/**
 * Author xiaoma
 * Date 2020/8/20 9:45
 * Desc 这个类实现 对 hdfs的上传，下载，创建文件夹，读取文件，移动文件，赋权等操作
 */
public class  HdfsTest {
    //1.如何获取 HDFS的客户端 FileSystem
    @Test
    public void test01() throws IOException, URISyntaxException, InterruptedException {
        /**
         * 1.获取FileSystem的对象
         * 2.通过3种方式获取FileSystem
         */
        Configuration conf = new Configuration();
        FileSystem fileSystem = FileSystem.get(conf);
        //System.out.println(fileSystem);
        //org.apache.hadoop.fs.LocalFileSystem@6989da5e
        //本地的文件系统，windows 文件系统
        System.out.println(fileSystem);

        //读取 node1集群的 hdfs文件系统
        conf = new Configuration();
        conf.set("fs.defaultFS", "hdfs://node1:8020");
        FileSystem fileSystem1 = FileSystem.get(conf);
        //System.out.println(fileSystem1);

        //通过uri和伪装用户来创建 FileSystem
        URI uri = new URI("hdfs://node1:8020");
        FileSystem fileSystem2 = FileSystem.get(uri, conf, "root");
        System.out.println(fileSystem2);
    }

    //2.获取 HDFS 上某个路径下所有的文件
    @Test
    public void test02() throws URISyntaxException, IOException, InterruptedException {
        //1.获取 hdfs的客户端对象 FileSystem
        FileSystem filesystem = FileSystem.get(new URI("hdfs://node1:8020"), new Configuration(), "root");
        //2.执行查询路径下的所有文件
        RemoteIterator<LocatedFileStatus> listFiles = filesystem.listFiles(new Path("/user"), true);

        while (listFiles.hasNext()) {
            LocatedFileStatus fileStatus = listFiles.next();

            Path path = fileStatus.getPath();
            String fileName = path.getName();
            System.out.println(fileName);
        }

        //3.关闭释放资源
        filesystem.close();
    }

    //3.在HDFS上创建一个文件夹
    @Test
    public void makeDir() throws URISyntaxException, IOException, InterruptedException {
        //1.创建hdfs的客户端
        URI uri = new URI("hdfs://node1:8020");
        Configuration conf = new Configuration();
        FileSystem fileSystem = FileSystem.newInstance(uri, conf, "root");
        //2.创建文件夹
        fileSystem.mkdirs(new Path("/app/config"));
        //3.关闭释放资源
        fileSystem.close();
    }
    //4.在HDFS上创建一个文件
    @Test
    public void makeFile() throws URISyntaxException, IOException, InterruptedException {
        //1.创建hdfs的客户端
        URI uri = new URI("hdfs://node1:8020");
        Configuration conf = new Configuration();
        FileSystem fileSystem = FileSystem.newInstance(uri, conf, "root");
        //2.创建文件夹
        FSDataOutputStream outputStream = fileSystem.create(new Path("/app/config/c3p0.xml"));
        outputStream.write("username=zhangsan\r\npassword=123456".getBytes());
        //刷新数据
        outputStream.flush();
        //3.关闭释放资源
        fileSystem.close();
    }
    //5.完成文件下载的操作
    @Test
    public void download() throws URISyntaxException, IOException, InterruptedException {
        //1.创建hdfs的客户端
        URI uri = new URI("hdfs://node1:8020");
        Configuration conf = new Configuration();
        FileSystem fileSystem = FileSystem.newInstance(uri, conf, "root");
        //2.下载文件
        fileSystem.copyToLocalFile(new Path("/app/config/c3p0.xml"),
                new Path("F:\\_workspace_java\\hadoop_hdfs\\"));

        //3.释放资源
        fileSystem.close();
    }
    //6.完成文件上传的操作
    @Test
    public void upload() throws URISyntaxException, IOException, InterruptedException {
        //1.创建hdfs的客户端
        URI uri = new URI("hdfs://node1:8020");
        Configuration conf = new Configuration();
        FileSystem fileSystem = FileSystem.newInstance(uri, conf, "root");
        //2.下载文件
        fileSystem.copyFromLocalFile(new Path("F:\\_workspace_java\\hadoop_hdfs\\c3p0.xml"),new Path("/"));
        //3.释放资源
        fileSystem.close();
    }
    //7本地有多个小文件，上传到hdfs中，因为hdfs中不推荐使用小文件，将这些小文件进行合并操作
    //合并成一个大文件统一上传。
    @Test
    public void mergeUpload() throws Exception{
        //1.创建hdfs的客户端
        URI uri = new URI("hdfs://node1:8020");
        Configuration conf = new Configuration();
        FileSystem fileSystem = FileSystem.newInstance(uri, conf, "root");
        //2.执行创建一个文件，生成输出流
        FSDataOutputStream outputStream = fileSystem.create(new Path("/merge.txt"));
        //3.获取本地文件系统
        LocalFileSystem localFileSystem = FileSystem.getLocal(new Configuration());
        //4.获取本地小文件
        RemoteIterator<LocatedFileStatus> localListFiles = localFileSystem.listFiles(new Path("F:\\_workspace_java\\hadoop_hdfs\\"), false);
        
        while(localListFiles.hasNext()){
            LocatedFileStatus fileStatus = localListFiles.next();
            //获取本地文件的路径
            Path path = fileStatus.getPath();
            FSDataInputStream inputStream = localFileSystem.open(path);
            IOUtils.copy(inputStream,outputStream);
            IOUtils.closeQuietly(inputStream);
        }
        outputStream.close();
    }
    //hdfs 访问权限设置
    @Test
    public void acl() throws Exception{
        //1.创建hdfs的客户端
        URI uri = new URI("hdfs://node1:8020");
        Configuration conf = new Configuration();
        FileSystem fileSystem = FileSystem.newInstance(uri, conf,"root");
        //2.将core-site.xml 保存到本地系统
        fileSystem.copyToLocalFile(new Path("/config2/core-site.xml"),new Path("F:\\_workspace_java\\hadoop_hdfs\\core-site.xml"));
        //3.释放资源
        fileSystem.close();
    }
}

```
</details>

## 9 拷贝、归档、快照、回收站

### 9.1 跨集群数据拷贝

DistCp（distributed copy）是一款被用于大型集群间/集群内的复制工具,该命令的内部原理是MapReduce。

```shell
cd /export/serverss/hadoop-2.7.5/
bin/hadoop distcp hdfs://node1:8020/jdk-8u241-linux-x64.tar.gz  hdfs://cluster2:8020/
```

### 9.2 归档 archive

> 功能：对存储的小文件进行合并成为大文件。

> 目的：减少hdfs中小文件的数量。

> 如何进行小文件的归档操作呢：

- 使用 HDFS JAVA API 实现小文件的合并
  - 场景：初始文件一般都必须在本地，而且文件的类型都要一致
  - 缺点：如果多种类型的文件，那么没办法归档成一个文件；如果合并之后的文件，想要拆分，只能自己写代码还是先将归档的文件解压到指定目录。
- 使用 hadoop 归档操作命令，使用 hadoop archive shell命令实现归档文件操作。
  - 场景：这些文件已经存在于HDFS 中，对HDFS中某个目录下文件进行归档
> 什么是归档文件
  - 归档文件，可以理解为 将多个文件压到一起，类似于 Linux 中 tar cvf archive.tar a.txt b.txt 这个命令只是将多个小文件进行了压成一个文件，而并没有缩小存储。
  - 归档文件注意：
    - 归档文件的后缀名 .har 
    - 启动归档，底层会运行 MapReduce程序，必须启动yarn集群
    - 创建归档文件之后，源文件不会被删除或者修改，归档的文档文件一旦创建就不能修改了。
> 如何使用归档文件，如何创建？

      #格式
      hadoop archive -archiveName name -p <parent> <src>* <dest>
      #示例.将config2 下的所有内容归档保存到 outputdir 文件夹下
      hadoop archive -archiveName config2.har -p /config2 /outputdir
      
> 如何查看归档文件的内容？

```shell
#格式
hadoop archive -archiveName name -p <parent> <src>* <dest>
#示例.将config2 下的所有内容归档保存到 outputdir 文件夹下
hadoop archive -archiveName config2.har -p /config2 /outputdir
```

> 如何查看归档文件的内容？

```shell
#1.直接在web browser 50070端口web查看
http://node1:50070/explorer.html#/outputdir/config2.har

#2.通过shell 命令进行查看
hadoop fs -ls /outputdir/config2.har

#3.查看 har 文件中原有的文件列表
#格式
#har://scheme-hostname:port/archivepath/fileinarchive   
hadoop fs -ls har://hdfs-node1:8020/outputdir/config2.har

#4.查看 har 文件中某一个文件的内容
hadoop fs -cat har://hdfs-node1:8020/outputdir/config2.har/core-site.xml

#5.将归档的文件解压出来

#5.1 创建一个文件夹 config3
hdfs dfs -mkdir -p /config3

#5.2 将归档的文件解压到config3中
hadoop fs -cp har://hdfs-node1:8020/outputdir/config2.har/* /config3
```

### 9.3 快照

> 快照就是snapshot （几乎不用）

> hdfs 的快照什么场景上使用：
  - 数据的备份
  - 放置用户操作不当出现错误的操作
  - 试验、测试
  - 灾备恢复
  
> hdfs 的快照是什么呢？
  - 相当于对HDFS中的某一个文件夹进行 拍照，保持当前这个文件夹的一个状态信息（差异化快照）
  - 差异化快照：拍完快照，快照文件只是对源文件的映射关系匹配。
  
>  dfs 的快照主要是针对文件夹。

> 如何进行快照的使用

```
#开启快照
hdfs dfsadmin -allowSnapshot /config3

#创建快照
hdfs dfs -createSnapshot /config3 backup_config3_20200820_1521

#修改一下文件在创建一个快照
hdfs dfs -rm -r /config3/core-site.xml
hdfs dfs -createSnapshot /config3 backup_config3_20200820_1523

#查看快照
hdfs dfs -ls /config3/.snapshot

#重命名快照
hdfs dfs renameSnapshot /config3 backup_config3_20200820_1523 backup_config3_20200820_1525

#删除快照
hdfs dfs -deleteSnapshot /config3 backup_config3_20200820_1521

#禁用快照
hdfs dfsadmin -disallowSnapshot /config3

#列出当前用户可快照的目录
hdfs lsSnapshottableDir
```

总结：HDFS的快照功能虽然能够保证数据的安全性，但是一般不建议大家使用，快照功能会占用非常大的磁盘空间。HDFS本身是带3备份，不能放置数据丢失，这个时候就开启快照功能。

### 9.4 Trash回收站

> Trash回收机制应用场景

- 放置用户手一抖彻底删除数据，当放置到Trash回收站里，还可以再次恢复数据。

> Trash回收站原理

- 当用户默认删除数据的时候，并不是直接从物理磁盘删掉，而只是将文件移动到指定的文件夹下，如果一致不恢复数据（根据默认时间7天等相关参数），Trash数据将从磁盘中抹掉。(`这个默认是关闭的，也就是fs.trash.interval=0`)

```xml
<property>  
    <name>fs.trash.interval</name>  
    <value>10080</value>  
    <description>Number of minutes after which the checkpoint gets deleted. If zero, the trash feature is disabled.</description>  
</property>  

<property>  
    <name>fs.trash.checkpoint.interval</name>  
    <value>0</value>  
    <description>Number of minutes between trash checkpoints. Should be smaller or equal to fs.trash.interval. If zero, the value is set to the value of fs.trash.interval.</description>  
</property>
```


属性 | 说明
---|---
fs.trash.interval | 分钟数，回收站文件的存活时间, 当超过这个分钟数后文件会被删除。如果为零，回收站功能将被禁用。
fs.trash.checkpoint.interval | 检查点创建的时间间隔(单位为分钟)。其值应该小于或等于fs.trash.interval。如果为零，则将该值设置为fs.trash.interval的值。



- 如果使用java API 来删除数据的话，直接将文件从磁盘抹掉，不会移动到Trash回收站中，shell才会。

- 如果不想放到回收站

```
#直接删除数据，不放在回收站中
hdfs dfs -rm -r -skipTrash /config3/yarn-site.xml 

#放在回收站中
hdfs dfs -rm -r  /config3/yarn-site.xml 
```

> 恢复数据

```
#将Trash回收站中的指定的数据恢复到指定文件夹中
hdfs dfs -mv hdfs://node1:8020/user/root/.Trash/200820154000/config3/yarn-site.xml /config3/
```

> 清空Trash回收站

```
hdfs dfs -expunge
```

总结，Trash回收机制为了保证操作数据，删除数据的时候，防止误删除，所以建议生产环境，在删除数据的时候，不要 skipTrash 跳过Trash回收机制。

## 10 HDFS的高可用机制

### 10.1 介绍

在Hadoop 中，NameNode 所处的位置是非常重要的，整个HDFS文件系统的元数据信息都由NameNode 来管理，NameNode的可用性直接决定了Hadoop 的可用性，一旦NameNode进程不能工作了，就会影响整个集群的正常使用。 

在典型的HA集群中，两台独立的机器被配置为NameNode。在工作集群中，NameNode机器中的一个处于Active状态，另一个处于Standby状态。Active NameNode负责群集中的所有客户端操作，而Standby充当从服务器。Standby机器保持足够的状态以提供快速故障切换（如果需要）。

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/12.png)

### 10.2 组件介绍

> ZKFailoverController

是基于Zookeeper的故障转移控制器，它负责控制NameNode的主备切换，ZKFailoverController会监测NameNode的健康状态，当发现Active NameNode出现异常时会通过Zookeeper进行一次新的选举，完成Active和Standby状态的切换

> HealthMonitor

周期性调用NameNode的HAServiceProtocol RPC接口（monitorHealth 和 getServiceStatus），监控NameNode的健康状态并向ZKFailoverController反馈

> ActiveStandbyElector

接收ZKFC的选举请求，通过Zookeeper自动完成主备选举，选举完成后回调ZKFailoverController的主备切换方法对NameNode进行Active和Standby状态的切换.

> DataNode

NameNode包含了HDFS的元数据信息和数据块信息（blockmap），其中数据块信息通过DataNode主动向Active NameNode和Standby NameNode上报

> 共享存储系统

共享存储系统负责存储HDFS的元数据（EditsLog），Active NameNode（写入）和 Standby NameNode（读取）通过共享存储系统实现元数据同步，在主备切换过程中，新的Active NameNode必须确保元数据同步完成才能对外提供服务

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/13.png)

### 10.3 高可用集群搭建

{% post_link 大数据_HDFS高可用集群搭建 高可用HDFS集群搭建教程 %}。

## 11 Hadoop的联邦机制(Federation)

### 11.1 背景概述

单NameNode的架构使得HDFS在集群扩展性和性能上都有潜在的问题，当集群大到一定程度后，NameNode进程使用的内存可能会达到上百G，NameNode成为了性能的瓶颈。因而提出了namenode水平扩展方案-- Federation。

Federation中文意思为联邦,联盟，是NameNode的Federation,也就是会有多个NameNode。多个NameNode的情况意味着有多个namespace(命名空间)，区别于HA模式下的多NameNode，它们是拥有着同一个namespace。既然说到了NameNode的命名空间的概念,这里就看一下现有的HDFS数据管理架构,如下图所示:

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/14.png)

从上图中,我们可以很明显地看出现有的HDFS数据管理,数据存储2层分层的结构.也就是说,所有关于存储数据的信息和管理是放在NameNode这边,而真实数据的存储则是在各个DataNode下.而这些隶属于同一个NameNode所管理的数据都是在同一个命名空间下的.而一个namespace对应一个block pool。Block Pool是同一个namespace下的block的集合.当然这是我们最常见的单个namespace的情况,也就是一个NameNode管理集群中所有元数据信息的时候.如果我们遇到了之前提到的NameNode内存使用过高的问题,这时候怎么办?元数据空间依然还是在不断增大,一味调高NameNode的jvm大小绝对不是一个持久的办法.这时候就诞生了HDFS Federation的机制.

### 11.2 Federation架构设计

> HDFS Federation是解决namenode内存瓶颈问题的水平横向扩展方案。

Federation意味着在集群中将会有多个namenode/namespace。这些namenode之间是联合的，也就是说，他们之间相互独立且不需要互相协调，各自分工，管理自己的区域。分布式的datanode被用作通用的数据块存储存储设备。每个datanode要向集群中所有的namenode注册，且周期性地向所有namenode发送心跳和块报告，并执行来自所有namenode的命令。

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/HDFS分布式文件系统/15.png)

Federation一个典型的例子就是上面提到的NameNode内存过高问题,我们完全可以将上面部分大的文件目录移到另外一个NameNode上做管理.更重要的一点在于,这些NameNode是共享集群中所有的DataNode的,它们还是在同一个集群内的。
这时候在DataNode上就不仅仅存储一个Block Pool下的数据了,而是多个(在DataNode的datadir所在目录里面查看BP-xx.xx.xx.xx打头的目录)。

`概括起来：`

多个NN共用一个集群里的存储资源，每个NN都可以单独对外提供服务。
每个NN都会定义一个存储池，有单独的id，每个DN都为所有存储池提供存储。
DN会按照存储池id向其对应的NN汇报块信息，同时，DN会向所有NN汇报本地存储可用资源情况。

### 11.3 HDFS Federation不足

HDFS Federation并没有完全解决单点故障问题。虽然namenode/namespace存在多个，但是从单个namenode/namespace看，仍然存在单点故障：如果某个namenode挂掉了，其管理的相应的文件便不可以访问。Federation中每个namenode仍然像之前HDFS上实现一样，配有一个secondary namenode，以便主namenode挂掉一下，用于还原元数据信息。
所以一般集群规模真的很大的时候，会采用HA+Federation的部署方案。也就是每个联邦的namenodes都是ha的。

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
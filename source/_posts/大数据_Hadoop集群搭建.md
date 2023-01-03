---
title: Hadoop 集群搭建
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(34).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(34).jpg
tags:
  - Cenos7
  - 大数据
  - Hadoop
category:
  - - 编程
    - 大数据
 
date: 2019-06-05 15:09:34
---

Hadoop是Apache旗下的一个用java语言实现开源软件框架，是一个开发和运行处理大规模数据的软件平台。

允许使用简单的编程模型在大量计算机集群上对大型数据集进行分布式处理。

今天先简单记录一下集群的搭建过程，以后再对各个比较重要的概念进行展开分析。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
[Hadoop官网](https://hadoop.apache.org/)

## 1 上传hadoop包

> 由于appache给出的hadoop的安装包没有提供带C程序访问的接口，所以我们在使用本地库（本地库可以用来做压缩，以及支持C程序等等）的时候就会出问题,需要对Hadoop源码包进行重新编译

> 具体怎么编译以后出一篇吧，今天有点忙，先记录下如何搭建集群吧，本文用的包是编译好的，可以从下面链接下载。

```
链接：https://pan.baidu.com/s/1oUyRaKbrhbfi2f6JHBKUIA 
提取码：6666 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V3的分享
```

### 1.1 上传hadoop包

```
cd /export/software/
rz
```

### 1.2 解压

```
tar -zxvf hadoop-2.7.5.tar.gz -C /export/servers/
```

> 目录如下

![Hadoop目录](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/1.png)

### 1.3 检查包是否编译成功

```
./bin/hadoop checknative
```

> 官方未编译版本如下，会发现, snappy 和 bzip2 都为false, 表示不支持

![Hadoop检查包](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/2.png)

> 我们编译过的版本如下，会发现, snappy 和 bzip2 都为true, 表示支持

![Hadoop检查包](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/3.png)

> OpenSSL依然为false，安装一下就好

```
yum -y install openssl-devel
```

## 2 Hadoop配置文件修改

> 在node1做如下修改

### 2.1 hadoop-env.sh

> 文件中设置的是Hadoop运行时需要的环境变量。JAVA_HOME是必须设置的，即使我们当前的系统中设置了JAVA_HOME，它也是不认识的，因为Hadoop即使是在本机上执行，它也是把当前的执行环境当成远程服务器。

```
cd  /export/servers/hadoop-2.7.5/etc/hadoop
vim  hadoop-env.sh
```

添加以下内容:

```
export JAVA_HOME=/export/servers/jdk1.8.0_241
```

### 2.2 core-site.xml

> hadoop的核心配置文件，有默认的配置项core-default.xml。<br/>

> core-default.xml与core-site.xml的功能是一样的，如果在core-site.xml里没有配置的属性，则会自动会获取core-default.xml里的相同属性的值。

在该文件中的<configuration>标签中添加以下配置,
<br/><br/>
<configuration><br/>
  在这里添加配置<br/>
</configuration>

```
cd  /export/servers/hadoop-2.7.5/etc/hadoop
vim  core-site.xml
```

配置内容如下:

```
<configuration>
    <!-- 用于设置Hadoop的文件系统，由URI指定 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://node1:8020</value>
    </property>
    <!-- 配置Hadoop存储数据目录,默认/tmp/hadoop-${user.name} -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/export/servers/hadoop-2.7.5/hadoopDatas/tempDatas</value>
    </property>

    <!--  缓冲区大小，实际工作中根据服务器性能动态调整: 根据自己的虚拟机的内存大小进行配置即可, 不要小于1GB, 最高配置为 4gb  -->
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

### 2.3 hdfs-site.xml

> HDFS的核心配置文件，主要配置HDFS相关参数，有默认的配置项hdfs-default.xml。

> hdfs-default.xml与hdfs-site.xml的功能是一样的，如果在hdfs-site.xml里没有配置的属性，则会自动会获取hdfs-default.xml里的相同属性的值。

在该文件中的<configuration>标签中添加以下配置,

<configuration><br/>
  在这里添加配置<br/>
</configuration>

```
cd  /export/servers/hadoop-2.7.5/etc/hadoop
vim  hdfs-site.xml
```

配置一下内容

```
<configuration>
    <!-- 指定SecondaryNameNode的主机和端口 -->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node2:50090</value>
    </property>
    <!-- 指定namenode的页面访问地址和端口 -->
    <property>
        <name>dfs.namenode.http-address</name>
        <value>node1:50070</value>
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

    <!-- 配置检查点目录 -->
    <property>
        <name>dfs.namenode.checkpoint.dir</name>
        <value>file:///export/servers/hadoop-2.7.5/hadoopDatas/snn/name</value>
    </property>

    <property>
        <name>dfs.namenode.checkpoint.edits.dir</name>
        <value>file:///export/servers/hadoop-2.7.5/hadoopDatas/dfs/snn/edits</value>
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
        <name>dfs.hosts</name>
        <value>/export/servers/hadoop-2.7.5/etc/hadoop/slaves</value>
    </property>
</configuration>
```

### 2.4 mapred-site.xml

> MapReduce的核心配置文件，Hadoop默认只有个模板文件mapred-site.xml.template,需要使用该文件复制出来一份mapred-site.xml文件

```
cd  /export/servers/hadoop-2.7.5/etc/hadoop
cp mapred-site.xml.template mapred-site.xml
```

在mapred-site.xml文件中的<configuration>标签中添加以下配置,

<configuration><br/>
  在这里添加配置<br/>
</configuration>

```
vim  mapred-site.xml
```

配置一下内容:

```
<configuration>
    <!-- 指定分布式计算使用的框架是yarn -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

    <!-- 开启MapReduce小任务模式 -->
    <property>
        <name>mapreduce.job.ubertask.enable</name>
        <value>true</value>
    </property>

    <!-- 设置历史任务的主机和端口 -->
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>node1:10020</value>
    </property>

    <!-- 设置网页访问历史任务的主机和端口 -->
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>node1:19888</value>
    </property>
</configuration>
```

### 2.5 mapred-env.sh

> 在该文件中需要指定JAVA_HOME,将原文件的JAVA_HOME配置前边的注释去掉，然后按照以下方式修改:

```
cd  /export/servers/hadoop-2.7.5/etc/hadoop
vim  mapred-env.sh
```

```
export JAVA_HOME=/export/servers/jdk1.8.0_241
```

### 2.6 yarn-site.xml

> ARN的核心配置文件,在该文件中的<configuration>标签中添加以下配置,

<configuration><br/>
  在这里添加配置<br/>
</configuration>

```
cd  /export/servers/hadoop-2.7.5/etc/hadoop
vim  yarn-site.xml
```

添加以下配置：

```
<configuration>

    <!-- 配置yarn主节点的位置 -->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>node1</value>
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

### 2.7 slaves

> slaves文件里面记录的是集群主机名。一般有以下两种作用：

> 一是：配合一键启动脚本如start-dfs.sh、stop-yarn.sh用来进行集群启动。这时候slaves文件里面的主机标记的就是从节点角色所在的机器。

> 二是：可以配合hdfs-site.xml里面dfs.hosts属性形成一种白名单机制。

> dfs.hosts指定一个文件，其中包含允许连接到NameNode的主机列表。必须指定文件的完整路径名,那么所有在slaves中的主机才可以加入的集群中。如果值为空，则允许所有主机。

```
cd  /export/servers/hadoop-2.7.5/etc/hadoop
vim  slaves
```

删除slaves中的localhost，然后添加以下内容:

```
node1
node2
node3
```

## 3 数据目录创建和文件分发

> 注意,以下所有操作都在node1主机进行

### 3.1 目录创建

创建Hadoop所需目录

```
mkdir -p /export/servers/hadoop-2.7.5/hadoopDatas/tempDatas
mkdir -p /export/servers/hadoop-2.7.5/hadoopDatas/namenodeDatas
mkdir -p /export/servers/hadoop-2.7.5/hadoopDatas/datanodeDatas
mkdir -p /export/servers/hadoop-2.7.5/hadoopDatas/nn/edits
mkdir -p /export/servers/hadoop-2.7.5/hadoopDatas/snn/name
mkdir -p /export/servers/hadoop-2.7.5/hadoopDatas/dfs/snn/edits
```
### 3.2 文件分发

将配置好的Hadoop目录分发到node2和node3主机。

```
 scp -r /export/servers/hadoop-2.7.5/ node2:/export/servers/
 scp -r /export/servers/hadoop-2.7.5/ node3:/export/servers/
```

## 4 配置Hadoop的环境变量

> 注意，三台机器都需要执行以下命令

```
vim  /etc/profile
```

> 添加以下内容:

```
export HADOOP_HOME=/export/server/hadoop-2.7.5
export PATH=:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

> 配置完成之后生效

```
source /etc/profile
```

> 测试

```
hadoop version
```

## 5 启动集群（下面命令都在sbin下）

### 5.1 启动前需要格式化

> 要启动Hadoop集群，需要启动HDFS和YARN两个集群。

> **注意：首次启动HDFS时，必须对其进行格式化操作**。本质上是一些清理和准备工作，因为此时的HDFS在物理上还是不存在的。

> 在node1上执行格式化指令

```
hadoop namenode -format
```

### 5.2 单节点逐个启动

> 在node1主机上使用以下命令启动HDFS NameNode：

```
hadoop-daemon.sh start namenode
```

> 在node1、node2、node3三台主机上，分别使用以下命令启动HDFS DataNode：

```
hadoop-daemon.sh start datanode
```

> 在node1主机上使用以下命令启动YARN ResourceManager：

```
yarn-daemon.sh  start resourcemanager
```

> 在node1、node2、node3三台主机上使用以下命令启动YARN nodemanager：

```
yarn-daemon.sh start nodemanager
```

> 在node2上启动 secondarynamenode

```
hadoop-daemon.sh start secondarynamenode
```

> 以上脚本位于/export/server/hadoop-2.7.5/sbin目录下。如果想要停止某个节点上某个角色，只需要把命令中的start改为stop即可。

### 5.3 脚本一键启动

> 启动HDFS

```
start-dfs.sh
```

> 启动Yarn

```
start-yarn.sh
```

> 启动历史任务服务进程

```
mr-jobhistory-daemon.sh start historyserver
```

> 启动之后，使用jps命令查看相关服务是否启动，jps是显示Java相关的进程命令。

node1：

![Hadoop进程查看1](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/4.png)

node2：

![Hadoop进程查看2](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/5.png)

node3：

![Hadoop进程查看3](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/6.png)

停止集群
```
stop-dfs.sh
stop-yarn.sh
mr-jobhistory-daemon.sh stop historyserver
```
> 注意:如果在启动之后，有些服务没有启动成功，则需要查看启动日志，Hadoop的启动日志在每台主机的/export/server/hadoop-2.7.5/logs/目录，需要根据哪台主机的哪个服务启动情况去对应的主机上查看相应的日志，以下是node1主机的日志目录.

![Hadoop日志](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/7.png)

### 5.4 一键启动(平时用这个就好了)

```
start-all.sh
mr-jobhistory-daemon.sh start historyserver
```

## 6 集群的页面访问

### 6.1 IP访问

> 一旦Hadoop集群启动并运行，可以通过web-ui进行集群查看，如下所述：

> 查看NameNode页面地址:

```
http://192.168.88.161:50070/
```
![NameNode页面](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/8.png)

> 查看Yarn集群页面地址:

```
http://192.168.88.161:8088/cluster 
```
![Yarn页面](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/9.png)

> 查看MapReduce历史任务页面地址:

```
http://192.168.88.161:19888/jobhistory
```

![MapReduce历史任务页面](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/10.png)

### 6.2 主机名访问

> 请注意，以上的访问地址只能使用IP地址，如果想要使用主机名，则对Windows进行配置。
配置方式:

1. 打开Windows的C:\Windows\System32\drivers\etc目录下hosts文件
2. 在hosts文件中添加以下域名映射

```
192.168.88.161  node1  node1.itcast.cn
192.168.88.162  node2  node2.itcast.cn
192.168.88.163  node3  node3.itcast.cn
```

> 配置完之后，可以将以上地址中的IP替换为主机名即可访问，如果还不能访问，则需要重启Windows电脑，比如访问NameNode，可以使用http://node1:50070/ 。

## 7 Hadoop初体验

### 7.1 HDFS使用

![HDFS使用](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/11.png)

1. 从Linux本地上传一个文本文件到hdfs的/目录下

```
#在/export/data/目录中创建a.txt文件，并写入数据
cd /export/data/
touch a.txt
echo "hello" > a.txt 

#将a.txt上传到HDFS的根目录
hadoop fs -put a.txt  /
```
2. 通过页面查看

> 通过NameNode页面.进入HDFS：http://node1:50070/

![通过页面查看](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/12.png)


查看文件是否创建成功.

![查看文件是否创建成功](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/bigDataHadoop/13.png)

### 7.2 运行mapreduce程序

> 在Hadoop安装包的share/hadoop/mapreduce下有官方自带的mapreduce程序。我们可以使用如下的命令进行运行测试。

示例程序jar:<br/>
 hadoop-mapreduce-examples-2.7.5.jar<br/>
计算圆周率

```
hadoop jar 

/export/servers/hadoop-2.7.5/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.5.jar 

pi 2 50
```


关于圆周率的估算，感兴趣的可以查询资料蒙特卡洛方法来计算Pi值，计算命令中2表示计算的线程数，50表示投点数，该值越大，则计算的pi值越准确。





## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
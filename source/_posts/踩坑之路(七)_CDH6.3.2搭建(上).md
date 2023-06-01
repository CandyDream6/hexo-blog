---
title: '踩坑之路(七) CDH6.3.2搭建(上)'
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(83).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(83).jpg
tags:
  - 大数据
  - 踩坑之路
category:
  - - 编程
    - 大数据
    
date: 2021-05-22 10:10:57
---

`CDH`，全称`Cloudera's Distribution, including Apache Hadoop`。

是`Hadoop`众多分支中对应中的一种，由Cloudera维护，基于稳定版本的`Apache Hadoop`构建，提供了Hadoop的核心（`可扩展存储`、`分布式计算`），最为重要的是提供`基于web的用户界面`。

今年2月份开始Cloudera全面`下架了免费的安装包下载渠道`，在网上找了很久，都不是很全，可`把人一遍一遍坑惨了`，最后琢磨出下面这套流程!

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

> 整篇文章使用的文件链接

```shell
链接：https://pan.baidu.com/s/1mAQ4M262NzYv3D8Vx9FSyQ 
提取码：8888 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/7_CDH6.3.2搭建(上)/2.png)

## 1 基础配置

> `先保证基础基础已经搭建好`{% post_link Linux_集群搭建 %}

### 1.1 禁用透明大页(所有节点)

> 查看
>>  查看透明大页的设置和启动状态

```shell
cat /sys/kernel/mm/transparent_hugepage/defrag
cat /sys/kernel/mm/transparent_hugepage/enabled
```

> 临时关闭

```shell
echo never > /sys/kernel/mm/transparent_hugepage/defrag
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```
> 永久关闭

```shell
echo 'echo never > /sys/kernel/mm/transparent_hugepage/defrag' >> /etc/rc.d/rc.local
echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.d/rc.local
chmod +x /etc/rc.d/rc.local
```

> 验证

```shell
cat /etc/rc.d/rc.local
```

### 1.2 修改Linux swappiness参数(所有节点)

> 为了避免服务器使用swap功能而影响服务器性能，一般都会把vm.swappiness修改为0（cloudera建议10以下）

> 查看

```shell
cd /usr/lib/tuned
grep "vm.swappiness" * -R
```

> 操作

```shell
sed -i s/"vm.swappiness = 30"/"vm.swappiness = 10"/g  /usr/lib/tuned/virtual-guest/tuned.conf
```

## 2 搭建本地yum源

> 我这里选择把yum源配置在node1节点上

### 2.1 安装并启动Apache http

> ⚠ `在node1节点完成以下操作`

> 安装Apache http

```shell
yum install -y httpd
```

> 启动Apache http

```shell
systemctl start httpd
```

> 设置自启动Apache http

```shell
systemctl enable httpd
```

### 2.2 上传安装文件

> ⚠ `在node1节点完成以下操作`

> 创建安装文件http根目录

```shell
mkdir -p /var/www/html/cm6
```

> 上传下列文件

```shell
allkeys.asc
cloudera-manager-server-db-2-6.3.1-1466458.el7.x86_64.rpm
cloudera-manager-agent-6.3.1-1466458.el7.x86_64.rpm
enterprise-debuginfo-6.3.1-1466458.el7.x86_64.rpm
cloudera-manager-daemons-6.3.1-1466458.el7.x86_64.rpm
cloudera-manager-server-6.3.1-1466458.el7.x86_64.rpm
```

### 2.3 创建yum仓库

> ⚠ `在node1节点完成以下操作`

```shell
cd /var/www/html/cm6
yum install -y createrepo
createrepo .
```

### 2.4 配置yum仓库文件

> ⚠ `在所有节点完成以下操作`

```shell
vim /etc/yum.repos.d/cloudera-manager.repo
```

```shell
[cloudera-manager]
name=Cloudera Manager 6.3.1
baseurl=http://node1/cm6
gpgcheck=0
enabled=1
autorefresh=0
type=rpm-md
```

### 2.5 更新仓库信息，确认本地yum源已被添加

> ⚠ `在所有节点完成以下操作`

```shell
yum clean all
yum makecache
```

> 执行结果差不多是下面这样的话就好了

```shell
[root@node1 ~]# yum makecache
已加载插件：fastestmirror, langpacks
Determining fastest mirrors
* base: mirrors.163.com
* extras: mirrors.163.com
* updates: mirrors.163.com
base                                              | 3.6 kB     00:00
cloudera-manager                                  | 2.9 kB     00:00
extras                                            | 2.9 kB     00:00
updates                                           | 2.9 kB     00:00
(1/13): base/7/x86_64/group_gz                      | 153 kB   00:01
(2/13): cloudera-manager/filelists_db               | 118 kB   00:01
(3/13): cloudera-manager/other_db                   | 1.0 kB   00:00
(4/13): cloudera-manager/primary_db                 | 8.6 kB   00:01
(5/13): extras/7/x86_64/filelists_db                | 227 kB   00:01
(6/13): extras/7/x86_64/other_db                    | 136 kB   00:01
(7/13): extras/7/x86_64/primary_db                  | 227 kB   00:06
(8/13): updates/7/x86_64/filelists_db               | 3.9 MB   00:06
(9/13): base/7/x86_64/primary_db                    | 6.1 MB   00:16
(10/13): updates/7/x86_64/other_db                  | 516 kB   00:08
(11/13): base/7/x86_64/other_db                     | 2.6 MB   00:24
(12/13): base/7/x86_64/filelists_db                 | 7.2 MB   00:27
(13/13): updates/7/x86_64/primary_db                | 6.5 MB   00:56
元数据缓存已建立
```

## 3 安装MySQL

> `一键安装mysql`{% post_link Linux一键安装MySQL脚本 %}


### 3.1 上传mysql的jdbc驱动包

> ⚠ `在node1节点完成以下操作`

> 重命名驱动包，并移动到httpd目录下，方便其他机器下载

```shell
mv mysql-connector-java-8.0.18.jar /var/www/html/cm6/mysql-connector-java.jar
```

> ⚠ `在所有节点完成以下操作`

> 由于集群中有多个服务都需要使用mysql进行元数据的管理，所以这里我们将mysql的jdbc依赖提前为每一个节点都准备好依赖

```shell
# 递归创建目标目录
mkdir -p /usr/share/java
# 进入目录
cd /usr/share/java
# wget从node00上下载jar
wget http://node1/cm6/mysql-connector-java.jar
```

## 4 安装服务

### 4.1 安装JDK

> `在所有节点完成以下操作`

```shell
yum -y install java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-headless.x86_64 java-1.8.0-openjdk-devel.x86_64
```

### 4.2 安装cm-agent

> ⚠ `在所有节点完成以下操作`

> 在所有节点安装 cloudera-manager-agent

```shell
yum install -y cloudera-manager-agent
```

> 启动/停止/重启/查看状态cloudera-manager-agent

```shell
systemctl start cloudera-scm-agent
systemctl stop cloudera-scm-agent
systemctl restart cloudera-scm-agent
systemctl status cloudera-scm-agent
```

> 查看日志

```shell
tail -f /var/log/cloudera-scm-agent/cloudera-scm-agent.log
```

### 4.3 管理节点安装cm-server

> ⚠ `在node1节点完成以下操作`

```shell
yum install -y cloudera-manager-server
```

> `cloudera-manager-server`安装完毕后，会自动创建`/opt/cloudera/parcel-repo`目录

> 将之前下载好的CDH安装包`CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel`上传到`/opt/cloudera/parcel-repo`目录

> 上传完成后计算校验和

```shell
cd /opt/cloudera/parcel-repo
sha1sum CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel | awk '{ print $1 }' > CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel.sha
```

> 好了之后会在同级会生产一个.sha文件，就好了。

## 5 初始化cloudera-manager-server

### 5.1 在mysql中为CMServer创建数据库

> ⚠ `在node1节点完成以下操作`

```shell
mysql -uroot -p123456
```

```shell
create database cmserver character set 'utf8';
show databases;
exit
```

### 5.2 执行CM初始化脚本

> ⚠ `在node1节点完成以下操作`

```shell
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql cmserver root 123456
```

```shell
[root@node1 parcel-repo]# /opt/cloudera/cm/schema/scm_prepare_database.sh mysql cmserver root 123456
JAVA_HOME=/usr/java/jdk1.8.0_181-cloudera
Verifying that we can write to /etc/cloudera-scm-server
Creating SCM configuration file in /etc/cloudera-scm-server
Executing:  /usr/java/jdk1.8.0_181-cloudera/bin/java -cp /usr/share/java/mysql-connector-java.jar:/usr/share/java/oracle-connector-java.jar:/usr/share/java/postgresql-connector-java.jar:/opt/cloudera/cm/schema/../lib/* com.cloudera.enterprise.dbutil.DbCommandExecutor /etc/cloudera-scm-server/db.properties com.cloudera.cmf.db.
Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
[                          main] DbCommandExecutor              INFO  Successfully connected to database.
All done, your SCM database is configured correctly!
```

### 5.3 启动cm-server服务

> CDH用到的端口太多太多了，咱也不乱来，照着官网一通开

```shell
https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_ig_ports.html#concept_k5z_vwy_4j
```

> ⚠ `在node1节点完成以下操作`

> 启动/停止/重启/查看状态cloudera-manager-server

```shell
systemctl start cloudera-scm-server.service
systemctl stop cloudera-scm-server.service
systemctl restart cloudera-scm-server.service
systemctl status cloudera-scm-server.service
```

> 查看server启动日志

```shell
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```

> cm-server默认使用7180端口进行WebUI访问

```shell
netstat -anp| grep 7180
```

> 查看到端口已经正常服务，打开浏览器进入ip:7180

```
账号：admin
密码：admin
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/踩坑之路/7_CDH6.3.2搭建(上)/1.png)


`可算是有这么一点点进展了，这两天踩坑把我踩麻了，给各路神仙请个安，希望接下来CDH的安装能够顺利一点！`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

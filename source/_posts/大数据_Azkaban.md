---
title: Azkaban 开源的工作流管理器
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(67).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(67).jpg
tags:
  - 大数据
  - Azkaban
category:
  - - 编程
    - 大数据
date: 2020-08-04 11:28:55
---

Azkaban是LinkedIn为运行Hadoop作业而创建的`一个批处理工作流作业调度程序`。

Azkaban`通过作业依赖关系来解析排序`，并提供一个易于使用的web用户界面来维护和跟踪您的工作流程。

Azkaban的设计`主要考虑了可用性`。它已经在LinkedIn运行了好几年，并驱动了他们的许多Hadoop和数据仓库流程。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 工作流调度 

### 1.1 工作流产生背景

> `工作流（Workflow）`，指“`业务过程的部分或整体在计算机应用环境下的自动化`”。是对工作流程及其各操作步骤之间业务规则的抽象、概括描述。工作流解决的主要问题是：为了实现某个业务目标，利用计算机软件在多个参与者之间按某种预定规则自动传递文档、信息或者任务。

> 一个完整的数据分析系统通常都是由多个前后依赖的模块组合构成的：数据采集、数据预处理、数据分析、数据展示等。各个模块单元之间存在时间先后`依赖关系`，且存在着`周期性重复`。

> 为了很好地组织起这样的复杂执行计划，需要一个工作流调度系统来调度执行。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/1.png)

### 1.2 工作流调度实现方式

> `简单的任务调度`：直接使用linux的`crontab`来定义,但是缺点也是比较明显，无法设置依赖。

> `复杂的任务调度`：自主开发调度平台，使用开源调度系统，比如`azkaban`、`Apache Oozie`、`Cascading`、`Hamake`等。

> 其中知名度比较高的是`Apache Oozie`，但是其配置工作流的过程是编写大量的XML配置，而且代码复杂度比较高，不易于二次开发。

#### 1.2.1 简单的任务调度

> `crontab`
>> 使用linux的crontab来定义调度,但是缺点比较明显，无法设置依赖复杂任务调度。且需要编写相关shell脚本。

#### 1.2.2 复杂的任务调度

> 当下企业两种选择，
>> 自主开发工作流调度系统
> 
>> 使用开源调度系统，比如azkaban、Apache Oozie、Cascading、Hamake等。

> 知名度比较高的是`Apache Oozie`，但是其配置工作流的过程是编写大量的XML配置，而且代码复杂度比较高，不易于二次开发。

### 1.3 工作流调度工具之间对比

> 下面的表格对四种hadoop工作流调度器的关键特性进行了比较，尽管这些工作流调度器能够解决的需求场景基本一致，但在`设计理念`，`目标用户`，`应用场景`等方面还是存在显著的区别，在做技术选型的时候，可以提供参考。

特性 | Hamake | Oozie | Azkaban | Cascading
---|---|---|---|---
工作流描述语言 | XML | XML (xPDL based) | text file with key/value pairs | Java API
依赖机制 | data-driven | explicit | explicit | explicit
是否要web容器 | No | Yes | Yes | No
进度跟踪 | console/log messages | web page | web page | Java API
Hadoop job调度支持 | no | yes | yes | yes
运行模式 | command line utility | daemon | daemon | API
Pig支持 | yes | yes | yes | yes
事件通知 | no | no | no | yes
需要安装 | no | yes | yes | no
支持的hadoop版本 | 0.18+ | 0.20+ | currently unknown | 0.18+
重试支持 | no | workflownode evel | yes | yes
运行任意命令 | yes | yes | yes | yes
Amazon EMR支持 | yes | no | currently unknown | yes

## 2 Azkaban调度器

### 2.1 Azkaban介绍

> Azkaban是由linkedin（领英）公司推出的一个批量工作流任务调度器，用于在一个工作流内以一个特定的顺序运行一组工作和流程。

> `Azkaban使用job配置文件建立任务之间的依赖关系，并提供一个易于使用的web用户界面维护和跟踪你的工作流。`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/2.png)

> Azkaban功能特点：
>> 提供功能清晰，简单易用的Web UI界面
> 
>> 提供job配置文件快速建立任务和任务之间的依赖关系
> 
>> 提供模块化和可插拔的插件机制，原生支持command、Java、Hive、Pig、Hadoop
> 
>> 基于Java开发，代码结构清晰，易于二次开发

### 2.2 Azkaban原理架构

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/3.png)

> `mysql服务器`: 存储元数据，如项目名称、项目描述、项目权限、任务状态、SLA规则等

> `AzkabanWebServer`:对外提供web服务，使用户可以通过web页面管理。职责包括项目管理、权限授权、任务调度、监控executor。

> `AzkabanExecutorServer`:负责具体的工作流的提交、执行。

### 2.3 Azkaban三种部署模式

#### 2.3.1 solo server mode

> 该模式中webServer和executorServer运行在同一个进程中，进程名是AzkabanSingleServer。使用自带的H2数据库。这种模式包含Azkaban的所有特性，但一般用来学习和测试。

#### 2.3.2 two-server mode

> 该模式使用MySQL数据库， Web Server和Executor Server运行在不同的进程中。

#### 2.3.3 multiple-executor mode

> 该模式使用MySQL数据库， Web Server和Executor Server运行在不同的机器中。且有多个Executor Server。该模式适用于大规模应用。

## 3 Azkaban安装

### 3.1 下载Azkaban源码并编译 

> 1 `下载Azkaban-3.71.0`

```
cd /export/softwares/
curl -L -O https://github.com/azkaban/azkaban/archive/3.71.0.tar.gz
```
> 2 `解压`

```
tar -zxf azkaban-3.71.0.tar.gz -C /export/services/
```

> 3 `创建软连接`

```
ln -s /export/services/azkaban-3.71.0 /export/services/azkaban
```

> 4 `安装Git`

```
yum install -y git
```

> 5 `修改Azkaban依赖的node版本`
>> Azkaban默认使用NodeJs-8.10.0版本，但是前面已经安装过最新的NodeJs-12.16.1版本，所以直接使用即可

```
vim /export/services/azkaban/azkaban-web-server/build.gradle
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/4.png)

> 6 `编译`

```
cd /export/services/azkaban
```

> 跳过测试

```
./gradlew build installDist -x test
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/5.png)

> 7 `查看编译后的二进制软件包`

```
ll azkaban-solo-server/build/distributions/
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/6.png)

### 3.2 安装Azkaban

> 1 `解压缩`

```
tar -zxf /export/services/azkaban/azkaban-solo-server/build/distributions/azkaban-solo-server-0.1.0-SNAPSHOT.tar.gz -C /export/services/
```

>  重命名

```
mv azkaban-solo-server-0.1.0-SNAPSHOT azkaban-3.71.0-bin
```

> 2 `进入/export/services目录`

```
cd /export/services/
```

> 3 `创建软连接`

```
ln -s /export/services/azkaban-3.71.0-bin /export/services/azkaban
```

> 4`添加环境变量`

```
vim /etc/profile
export AZKABAN_HOME=/export/services/azkaban
PATH=$AZKABAN_HOME/bin:$PATH
source /etc/profile
```

> 5 `配置azkaban.properties`

```
cd /export/services/azkaban
vim conf/azkaban.properties
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/7.png)

> 6 `配置commonprivate.properties`

```
vim plugins/jobtypes/commonprivate.properties
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/8.png)

> 7 `在mysql中添加azkaban数据库`

```
mysql -uroot -pAbcd1234.
create database azkaban default character set utf8 collate utf8_general_ci;
quit
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/9.png)

> 8 `启动Azkaban`

```
cd /export/services/azkaban
./bin/start-solo.sh
```

> 注意：这个start-solo.sh脚本使用的是相对路径，必须进入到/export/services/azkaban路径下执行./bin/start-solo.sh，否则mysql数据库会初始化失败

> 9 `打开Azkaban的WebUI`

```
在浏览器中输入http://node2.xiaoma.cn:8081
账户：azkaban
密码：azkaban
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/10.png)

> 登录成功后的样子如图：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/11.png)

## 4 Azkaban使用实战

### 4.1 shell command调度

> 创建job描述文件

```
vi command.job
#command.job
type=command                                                    
command=echo 'hello'
```

> 将job资源文件打包成zip文件

```
zip command.job
```

> 通过azkaban的web管理平台创建project并上传job压缩包
>> 首先创建Project
>> ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/12.png) 
> 
>> 上传zip包
>> ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/13.png)

> 启动执行该job

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/14.png)

### 4.2 job依赖调度

> 创建有依赖关系的多个job描述

> 第一个job：foo.job

```
# foo.job
type=command
command=echo foo
```

> 第二个job：bar.job依赖foo.job

```
# bar.job
type=command
dependencies=foo
command=echo bar
```

> 将所有job资源文件打到一个zip包中

> 在azkaban的web管理界面创建工程并上传zip包

> 启动工作流flow

### 4.3 HDFS任务调度

> 创建job描述文件

```
# fs.job
type=command
command=hadoop fs -mkdir /azaz
```

> 将job资源文件打包成zip文件

> 通过azkaban的web管理平台创建project并上传job压缩包

> 启动执行该job

### 4.4 定时任务调度

> 除了手动立即执行工作流任务外，azkaban也支持配置定时任务调度。开启方式如下：

> 首页选择待处理的project

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/15.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/16.png)

> 上述图片中，`选择左边schedule表示配置定时调度信息，选择右边execute表示立即执行工作流任务`。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/17.png)

## 5 Azkaban进行主题及指标业务调度开发

### 5.1 数据同步调度

> `shell脚本（scheduler.sh）`

```
#!/bin/sh

cls=$1
flag=0
clsDwd=cn.xiaoma.logistics.offline.dwd.${cls}DWD
clsDws=cn.xiaoma.logistics.offline.dws.${cls}DWS
baseDir=/export/services/logistics/lib/

if [[ $cls = "Customer" || $cls = "ExpressBill" || $cls = "TransportTool" || $cls = "Warehouse" || $cls = "Waybill" ]]; then
echo -e "\e[32m==== MainClass is: "$clsDwd" and "$clsDws"\e[0m"
flag=1
else
echo -e "\e[31mUsage : \n\tExpressBill\n\tCustomer\n\tTransportTool\n\tWarehouse\n\tWaybill\e[0m"
fi

if [[ $flag = 1 ]]; then
echo -e "\e[32m==== builder spark commands ====\e[0m"
cmd1="spark-submit --packages org.apache.kudu:kudu-spark2_2.11:1.9.0-cdh6.2.1 --class ${clsDwd} --master yarn --deploy-mode cluster --driver-memory 512m --executor-cores 1 --executor-memory 512m --queue default --verbose ${baseDir}logistics-etl.jar"
cmd2="spark-submit --packages org.apache.kudu:kudu-spark2_2.11:1.9.0-cdh6.2.1 --class ${clsDws} --master yarn --deploy-mode cluster --driver-memory 512m --executor-cores 1 --executor-memory 512m --queue default --verbose ${baseDir}logistics-etl.jar"
echo -e "\e[32m==== CMD1 is: $cmd1 ====\e[0m"
echo -e "\e[32m==== CMD2 is: $cmd2 ====\e[0m"
fi

if [[ $flag = 1 && `ls -A $baseDir|wc -w` = 1 ]]; then
echo -e "\e[32m==== start execute ${clsDwd} ====\e[0m"
sh $cmd1
echo -e "\e[32m==== start execute ${clsDws} ====\e[0m"
sh $cmd2
else
echo -e "\e[31m==== The jar package in $baseDir directory does not exist! ====\e[0m"
echo -e "\e[31m==== Plase upload logistics-common.jar,logistics-etl.jar,logistics-generate.jar ====\e[0m"
fi
```

### 5.2 job编写

> 快递单主题Job（schedulerExpressBill.job）

```
#command
type=command
command=sh /export/services/logistics/bin/scheduler.sh ExpressBill
```

> 运单主题Job（schedulerWaybill.job）

```
#command
type=command
command=sh /export/services/logistics/bin/scheduler.sh Waybill
```

> 仓库主题Job（schedulerWarehouse.job）

```
#command
type=command
command=sh /export/services/logistics/bin/scheduler.sh Warehouse
```

> 车辆主题Job（schedulerTransportTool.job）

```
#command
type=command
command=sh /export/services/logistics/bin/scheduler.sh TransportTool
```

> 客户主题Job（schedulerCustomer.job）

```
#command
type=command
command=sh /export/services/logistics/bin/scheduler.sh Customer
```

### 5.3 操作步骤

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Azkaban/18.png)

> 在Azkaban的WebUI创建一个Project（例如Waybill运单ETL）

> 点击“Waybill运单ETL”项目进去，然后再点击右上角上传按钮上传schedulerWaybill.zip（将schedulerWaybill打进压缩包，压缩包名为schedulerWaybill.zip）

> 设置该调度程序的触发周期为：0 1 0  * * *（每天0点01分执行）

> 提交运行，Azkaban会等时间到了就开始运行

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

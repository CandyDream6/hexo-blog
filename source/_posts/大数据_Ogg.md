---
title: GoldenGate 基于日志的结构化数据复制软件
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(64).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(64).jpg
tags:
  - 大数据
  - Ogg
category:
  - - 编程
    - 大数据
 
date: 2020-06-15 11:47:25
---

OGG是一种基于日志的结构化数据复制软件，它通过解析源数据库在线日志或归档日志获得数据的增删改变化（数据量只有日志的四分之一左右）

OGG能够实现大量交易数据的`实时捕捉，变换和投递`，实现源数据库与目标数据库的数据同步，保持最少10ms的数据延迟。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 OGG介绍

### 1.1 OGG简介

> OGG 是一种基于日志的结构化数据复制软件，它通过解析源数据库在线日志或归档日志获得数据的增删改变化（数据量只有日志的四分之一左右）

> OGG 能够实现大量交易数据的`实时捕捉，变换和投递`，实现源数据库与目标数据库的数据同步，保持最少10ms的数据延迟。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/1.png)

### 1.2 应用场景

> 高可用容灾

> 数据库迁移、升级（支持跨版本、异构数据库、零宕机时间、亚秒级恢复）

> 实时数据集成（支持异构数据库、多源数据库）

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/2.png)

### 1.3 基本原理

> 基于日志捕获技术的实时增量数据集成

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/3.png)

> Oracle GoldenGate 数据复制过程如下：
>> 利用抽取进程(Extract Process)在源端数据库中读取Online Redo Log或者Archive Log，然后进行解析，只提取其中数据的变化信息，比如DML操作——增、删、改操作
> 
>> 将抽取的信息转换为GoldenGate自定义的中间格式存放在队列文件(trail file)中
> 
>> 再利用传输进程将队列文件(trail file)通过TCP/IP传送到目标系统。
> 
>> 目标端有一个进程叫Server ipCollector，这个进程接受了从源端传输过来的数据变化信息
> 
>> 把信息缓存到GoldenGate 队列文件(trail file)当中，等待目标端的复制进程读取数据。
> 
>> GoldenGate 复制进程(replicat process)从队列文件(trail file)中读取数据变化信息，并创建对应的SQL语句，通过数据库的本地接口执行，提交到目标端数据库，提交成功后更新自己的检查点，记录已经完成复制的位置，数据的复制过程最终完成。

### 1.4 基本架构

> Oracle GoldenGate主要由如下组件组成

组件 | 说明
---|---
Manager | 不管是源端还是目标端必须并且只能有一个Manager进程，可以启动、关闭、监控其他进程的健康状态，报告错误事件、分配数据存储空间，发布阀值报告等，其作用：<br/><br/>1：监控与启动 GoldenGate 的其它进程<br/><br/>2：管理 trail 文件及 Reporting
Extract | Extract 进程运行在数据库源端上，它是Golden Gate的捕获机制，可以配置Extract 进程来做如下工作：<br/><br/>1：初始数据装载：对于初始数据装载，Extract 进程直接从源对象中提取数据<br/><br/>2：同步变化捕获：保持源数据与其它数据集的同步。初始数据同步完成后，Extract 进程捕获源数据的变化；如DML变化、 DDL变化等
Replicat | Replicat 进程是运行在目标端系统的一个进程，负责读取 Extract 进程提取到的数据（变更的事务或 DDL 变化）并应用到目标数据库，就像 Extract 进程一样，也可以配置 Replicat 进程来完成如下工作：<br/><br/>1：初始化数据装载：对于初始化数据装载，Replicat 进程应用数据到目标对象或者路由它们到一个高速的 Bulk-load 工具上；<br/><br/>2：数据同步，将 Extract 进程捕获到的提交了的事务应用到目标数据库中；
Collector | Collector 是运行在目标端的一个后台进程，接收从 TCP/IP 网络传输过来的数据库变化，并写到 Trail 文件里
Trails | 为了持续地提取与复制数据库变化，GoldenGate 将捕获到的数据变化临时存放在磁盘上的一系列文件中，这些文件就叫做 Trail 文件
Data Pumps | Data Pump 是一个配置在源端的辅助的 Extract 机制，`Data Pump 是一个可选组件`，如果不配置 Data Pump，那么由 Extract 主进程将数据发送到目标端的 Remote Trail 文件中；如果配置了 Data Pump，会由 Data Pump将Extract 主进程写好的本地 Trail 文件通过网络发送到目标端的 Remote Trail 文件中

### 1.5 常用的拓扑结构

> 单向复制：由一个源数据库复制到一个目的数据库，一般用于高可用性和容灾，为生产机保持一个活动的备份数据库，从而在发生灾难的时候迅速切换，减少数据丢失和系统宕机时间；

> 双向复制：利用GoldenGate TDM可以实现两个数据库之间数据的双向复制，任何一方的数据变化都会被传递到另一端，可以利用此模式开展双业务中心；

> 广播复制：由一个数据库向多个数据库复制，利用GoldenGate TDM的数据过滤功能可以实现数据的有选择分发；

> 集中复制：由多个数据库向一个数据库复制，可以将分布的、跨平台或异构的多个数据库集中到一个数据库。此种模式广泛应用于n+1模式的容灾，通过将多个系统数据库集中到一起，可以充分利用备份中心的设施，大幅减少投资；另外也用于跨平台多系统的数据集成，为这些提供系统提供一个统一视图便于查询和统计数据。

> 多层复制：由A数据库向B复制，同时又由B向C复制，可以在以上几种模式基础上无限制扩展。

> 由此可见，GoldenGate TDM的复制模式非常灵活，用户可以根据自己的需求选择特定的复制方式，并根据系统扩展对复制进行扩展。

### 1.6 支持的环境

> 源和目标的操作系统和数据库可以进行任意的组合
> 
![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/5.png)

## 2 OGG安装部署

### 2.1 配置Oracle11gR2数据库

#### 2.1.1 OraclegR2打开归档模式

> 需要切换到oracle用户操作：

```
su - oracle
```

> 因为配置数据库需要在sqlplus中执行，所以使用sysdba用户登录：

```
sqlplus / as sysdba
```

> `验证数据库是否开启自动归档`

> 执行归档查询命令：

```
archive log list
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/6.png)

> `Automatic archival是Disabled状态，因为Oracle默认是不开启自动归档的`

> `开启自动归档`

> 以DBA的身份连接数据库，执行命令：

```
conn /as sysdba
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/7.png)

> 关闭数据库，执行命令：

```
shutdown immediate
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/8.png)

> 启动并装载数据库，但没有打开数据文件，该命令常用来修改数据库运行模式或恢复数据库。执行命令：

```
startup mount
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/9.png)

> 执行开启归档命令：

```
alter database archivelog;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/10.png)

> 执行打开数据库命令：

```
alter database open;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/11.png)

> 执行自动归档命令：

```
alter system archive log start;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/12.png)

> `验证是否开启自动归档成功`

```
执行归档查询命令：archive log list
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/13.png)

> Automatic archival变成了Enabled状态，表示已经开启自动归档成功

#### 2.1.2 Oracle开启辅助日志和补充日志

> `验证数据库是否开启辅助日志和补充日志`

```
执行SQL语句验证：select force_logging,supplemental_log_data_min from v$database;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/14.png)

> 当显示NO的时候表示没有开启，需要调整

> `开启数据库的辅助日志和补充日志`

> 开启强制日志后数据库会记录除临时表空间或临时回滚段外所有的操作，命令：

```
alter database force logging;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/15.png)

> 开启辅助日志命令：

```
alter database add supplemental log data;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/16.png)

> 开启主键附加日志命令：

``` 
alter database add supplemental log data (primary key) columns;
```

> 开启全列附加日志命令：

```
alter database add supplemental log data (all) columns;
```
> `检查数据库是否成功开启辅助日志和补充日志`

```
执行SQL语句验证：select force_logging,supplemental_log_data_min from v$database;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/17.png)

> 当显示为YES的时候表示开启成功。

### 2.2 安装OGG源端

#### 2.2.1 解压和安装OGG源端软件包

操作步骤 | 说明
---|---
1 | 创建OGG源端的目录<br/><br/>使用root用户创建：mkdir /u01/app/ogg/src
2 | 添加OGG源端的目录到oracle用户的环境变量中（`需要切换到oracle用户操作`）<br/><br/>su - oracle<br/><br/>vim ~/.bash_profile<br/><br/>export OGG_SRC_HOME=/u01/app/ogg/src<br/><br/>export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/usr/lib<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/18.png)<br/><br/>source ~/.bash_profile<br/><br/>退出oracle用户shell命令：exit
3 | 解压OGG源端软件<br/><br/>OGG源端的软件包是V34339-01.zip，存放在/export/softwares/oracle/ogg目录下。需要使用root用户解压<br/><br/>cd /export/softwares/oracle/ogg<br/><br/>创建src文件夹是用来存放解压后的OGG源端软件<br/><br/>mkdir /export/softwares/oracle/ogg/src/<br/><br/>解压OGG源端软件到src文件夹下<br/><br/>unzip /export/softwares/oracle/ogg/V34339-01.zip -d /export/softwares/oracle/ogg/src/<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/19.png)<br/><br/>cd /export/softwares/oracle/ogg/src/<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/20.png)<br/><br/>fbo_ggs_Linux_x64_ora11g_64bit.tar文件才是OGG源端的软件包，解压该文件到/u01/app/ogg/src目录下，执行命令：<br/><br/>tar -xf fbo_ggs_Linux_x64_ora11g_64bit.tar -C /u01/app/ogg/src
4 | 配置/u01/app/ogg/src目录及其所有文件的权限<br/><br/>使用root用户执行授权命令：<br/><br/>chown -R oracle:oinstall /u01/app/ogg/src<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/21.png)<br/><br/>可以看到/u01/app/ogg/目录下的src属于oracle用户和oinstall组<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/22.png)<br/><br/>可以看到/u01/app/ogg/src目录下的所有文件都属于oracle用户和oinstall组

#### 2.2.2 在Oracle中创建OGG相关的用户和表空间

操作步骤 | 说明
---|---
1 | 创建表空间在磁盘中的物理路径（`需要到root用户操作`）<br/><br/>mkdir -p /u01/app/oracle/oggdata/orcl/<br/><br/>chown -R oracle:oinstall /u01/app/oracle/oggdata/orcl
2 | 进入sqlplus<br/><br/>切换到oracle用户：su - oracle<br/><br/>登录sqlplus：sqlplus "/as sysdba"
3 | 创建oggtbs表空间<br/><br/>create tablespace oggtbs datafile '/u01/app/oracle/oggdata/orcl/oggtbs.dbf' size 500M autoextend on;<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/23.png)
4 | 创建ogg用户(用户名和密码都是ogg)<br/><br/>create user ogg identified by ogg default tablespace oggtbs;<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/24.png)
5 | 赋予ogg用户dba权限<br/><br/>grant dba to ogg;<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/25.png)

#### 2.2.3 OGG源端初始化

操作步骤 | 说明
---|---
1 | 使用oracle用户登录源端OGG的命令行中<br/><br/>su – oracle<br/><br/>cd $OGG_SRC_HOME<br/><br/>./ggsci<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/26.png)
2 | 初始化源端OGG目录<br/><br/>注意：如果不在OGG_SRC_HOME下，初始化OGG目录时会报错<br/><br/>create subdirs<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/27.png)<br/><br/>退出OGG命令行客户端：exit![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/28.png)
3 | 检查源端OGG初始化后的目录<br/><br/>初始化完成后，可以查询在$OGG_SRC_HOME下是否存在dirchk、dirdat、dirdef、dirjar、dirout、dirpcs、dirprm、dirrpt、dirsql、dirtmp共11个目录。<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/29.png)

### 2.3 配置OGG源端

#### 2.3.1 Oracle创建测试表

> 切换到oracle用户：

```
su – oracle
```

> 登录sqlplus：

```
sqlplus "/as sysdba"
```

> 在oracle中创建test_ogg用户：

```
create user test_ogg identified by test_ogg default tablespace users;
```

> 为test_ogg用户授权：

```
grant dba to test_ogg;
```

> 使用test_ogg用户登录：

```
conn test_ogg/test_ogg;
```

> 创建test_ogg表：

```
create table test_ogg(id int ,name varchar(20),primary key(id));
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/30.png)

#### 2.3.2 配置OGG的全局变量

> `使用oracle用户进入OGG_SRC_HOME目录下`

> 切换到oracle用户下：

```
su – oracle
```

> 打印源端OGG_SRC_HOME：

```
echo $OGG_SRC_HOME
```

> 进入OGG_SRC_HOME：

```
cd $OGG_SRC_HOME
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/31.png)

> 进入源端OGG命令行

```
./ggsci
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/32.png)

> 使用oracle中的ogg用户登录

```
dblogin userid ogg password ogg
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/33.png)

> 配置全局变量

```
edit param ./globals
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/34.png)

```
oggschema ogg
```

> 然后跟使用vi一样，在新窗口中添加`oggschema ogg`后保存退出编辑

#### 2.3.3 配置管理器MGR进程

> 进入源端OGG命令行

```
./ggsci
```

> 创建mgr进程：

```
edit param mgr
```

```
PORT 7809
DYNAMICPORTLIST 7810-7909
AUTORESTART EXTRACT *,RETRIES 5,WAITMINUTES 3
PURGEOLDEXTRACTS ./dirdat/*,usecheckpoints, minkeepdays 3
```

参数名称 | 参数说明
---|---
PORT | mgr的默认监听端口
DYNAMICPORTLIST | 当指定的mgr端口不可用时，会在这个端口列表中选择一个，最大指定范围为256个
AUTORESTART EXTRACT *,RETRIES 5,WAITMINUTES 3 | 重启EXTRACT进程的参数，最多5次，每次间隔3分钟
PURGEOLDEXTRACTS | TRAIL文件的定期清理

#### 2.3.4 添加复制表

> 进入源端OGG命令行

```
./ggsci
```

```
add trandata test_ogg.test_ogg
```

```
info trandata test_ogg.test_ogg
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/35.png)

#### 2.3.5 配置extract进程

> 配置Extract进程：

```
edit param extkafka
```

> 新增内容：

```
extract extkafka
dynamicresolution
SETENV (ORACLE_SID = "orcl")
SETENV (NLS_LANG = "american_america.AL32UTF8")
userid ogg,password ogg
exttrail /u01/app/ogg/src/dirdat/to
table test_ogg.test_ogg;
```

参数名称 | 参数说明
---|---
extract extkafka | 定义extract进程名称
dynamicresolution | 启用动态解析
SETENV (ORACLE_SID = "orcl") | 设置Oracle数据库
SETENV (NLS_LANG = "american_america.AL32UTF8") | 设置字符集
userid ogg,password ogg | OGG连接Oracle数据库的帐号密码
exttrail /u01/app/ogg/src/dirdat/to | 定义trail文件的保存位置以及文件名，文件字母最多2个，否则会报错
table test_ogg.test_ogg; | 复制表的表名，支持*通配，必须以;结尾


> 添加Extract进程：

```
add extract extkafka,tranlog,begin now
```

> 将trail文件配置与extract进程绑定：

```
add exttrail /u01/app/ogg/src/dirdat/to,extract extkafka
```

#### 2.3.6 配置pump进程

> 配置Pump进程：

```
edit param pukafka
```

> 新增内容：

```
extract pukafka
passthru
dynamicresolution
userid ogg,password ogg
rmthost localhost mgrport 7809
rmttrail /u01/app/ogg/tgr/dirdat/to
table test_ogg.test_ogg;
```

> extract进程名称；passthru即禁止OGG与Oracle交互，我们这里使用pump逻辑传输，故禁止即可；dynamicresolution动态解析；userid ogg,password ogg即OGG连接Oracle数据库的帐号密码rmthost和mgrhost即目标端(kafka)OGG的mgr服务的地址以及监听端口；rmttrail即目标端trail文件存储位置以及名称。

参数名称 | 参数说明
---|---
extract pukafka | 定义pump进程名称
passthru | 因使用了pump逻辑传输，所以禁止OGG与Oracle交互
dynamicresolution | 配置动态解析
userid ogg,password ogg | OGG连接Oracle数据库的帐号密码
rmthost localhost mgrport 7809 | 目标端OGG的mgr服务的地址以及监听端口
rmttrail /u01/app/ogg/tgr/dirdat/to | 目标端OGG的trail文件存储位置以及名称
table test_ogg.test_ogg; | 要采集的表,必须使用;结尾

> 将源端trail文件绑定到Extract进程：

```
add extract pukafka,exttrailsource /u01/app/ogg/src/dirdat/to
```

> 将目标端trail文件绑定到Extract进程：

```
add rmttrail /u01/app/ogg/tgr/dirdat/to,extract pukafka
```

#### 2.3.7 配置define文件

> `注意：该文件用来在异构数据源之间传输时，需明确知道表之间的映射关系，比如：`
>> `Oracle与MySQL，Hadoop集群（HDFS，Hive，kafka等）等之间数据传输可以定义为异构数据类型的传输，故需要定义表之间的关系映射，在OGG命令行执行：`

> 配置define文件：

```
edit param test_ogg
```

```
defsfile /u01/app/ogg/src/dirdef/test_ogg.test_ogg
userid ogg,password ogg
table test_ogg.test_ogg;
```

> 生成表schema文件：（`在OGG_SRC_HOME目录下执行(oracle用户)）`

```
./defgen paramfile dirprm/test_ogg.prm
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/36.png)

> 将生成的/u01/app/ogg/src/dirdef/test_ogg.test_ogg发送的目标端ogg目录下的dirdef里：

```
scp -r /u01/app/ogg/src/dirdef/test_ogg.test_ogg   /u01/app/ogg/tgr/dirdef/
```

> `因为目标端目录还没有创建，因此发送文件可能会失败，所以执行完目标端配置后发送即可`

### 2.4 配置OGG目标端

#### 2.4.1 解压和安装OGG目标端软件包

> `创建OGG目标端的目录`

> 使用root用户创建：

```
mkdir /u01/app/ogg/tgr
```
> `添加OGG目标端的目录到oracle用户的环境变量中`

> 从root用户切换到oracle用户：

```
su oracle
```

> `注意【非常重要】：在这里，需要调整oracle用户的.bash_profile，主要是更新LD_LIBRARY_PATH的值，来确保源端和目标端的ggsci命令都可以正常使用。如果不更新的话，目标端ggsci命令可以使用，但源端的ggsci命令无法使用，会报错./ggsci: error while loading shared libraries: libclntsh.so.11.1: cannot open shared object file: No such file or directory`

```
vim ~/.bash_profile
```

```
export OGG_TGR_HOME=/u01/app/ogg/tgr
export LD_LIBRARY_PATH=$JAVA_HOME/jre/lib/amd64:$JAVA_HOME/jre/lib/amd64/server:$JAVA_HOME/jre/lib/amd64/libjsig.so:$JAVA_HOME/jre/lib/amd64/server/libjvm.so:$ORACLE_HOME/lib:/usr/lib
```

```
source  ~/.bash_profile
```

> 退出oracle用户shell命令：exit

> `解压OGG目标端软件`

> OGG源端的软件包是V971332-01.zip，存放在/export/softwares/oracle/ogg目录下。需要使用root用户解压

```
cd /export/softwares/oracle/ogg
```

> 创建tgr文件夹是用来存放解压后的OGG目标端软件

```
mkdir -p /export/softwares/oracle/ogg/tgr/
```

> 解压OGG目标端软件到tgr文件夹下

```
unzip /export/softwares/oracle/ogg/V971332-01.zip -d /export/softwares/oracle/ogg/tgr/
```

```
cd /export/softwares/oracle/ogg/tgr/
```

> ggs_Adapters_Linux_x64.tar文件是真正的OGG目标端软件包，解压该文件到/u01/app/ogg/tgr目录下，执行命令：

```
tar -xf ggs_Adapters_Linux_x64.tar -C /u01/app/ogg/tgr/
```
> `配置/u01/app/ogg/tgr目录及其所有文件的权限`

> 使用root用户执行授权命令：

```
chown -R oracle:oinstall /u01/app/ogg/tgr
```

> 可以看到/u01/app/ogg/目录下的tgr属于oracle用户和oinstall组。

```
ll /u01/app/ogg/tgr
```

> 可以看到/u01/app/ogg/tgr目录下的所有文件都属于oracle用户和oinstall组。

#### 2.4.2 OGG目标端初始化

> `使用oracle用户登录目标端OGG的命令行中`

> 可以看到/u01/app/ogg/目录下的tgr属于oracle用户和oinstall组。

```
su oracle
```

> 切换oracle用户时需要重新加载环境变量：

```
source ~/.bash_profile
```

```
cd $OGG_TGR_HOME
```

```
./ggsci
```

> `初始化目标端OGG目录`

> `注意：如果不在OGG_TGR_HOME下，初始化目标端OGG目录时会报错`

```
create subdirs
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/37.png)

> 退出OGG命令行客户端：exit

> `检查目标端OGG初始化后的目录`

> 初始化完成后，可以查询在$OGG_TGR_HOME下是否存在dirchk、dircrd、dirdat、dirdef、dirdmp、diretc、dirout、dirpcs、dirprm、dirrpt、dirsql、dirtmp、dirwlt、dirwww共14个目录。

#### 2.4.3 拷贝源端的define文件到目标端

> 将生成的/u01/app/ogg/src/dirdef/test_ogg.test_ogg发送的目标端ogg目录下的dirdef里：

```
scp -r $OGG_SRC_HOME/dirdef/test_ogg.test_ogg $OGG_TGR_HOME/dirdef/
```

#### 2.4.4 安装zookeeper和Kafka

> `安装ZooKeeper(使用root用户操作)`

> 解压：

```
tar -zxf /export/softwares/zookeeper-3.4.14.tar.gz -C /export/services/
```

> 创建软连接：

```
ln -s /export/services/zookeeper-3.4.14 /export/services/zookeeper
```

> 创建zoo.cfg：

```
cp /export/services/zookeeper/conf/zoo_sample.cfg /export/services/zookeeper/conf/zoo.cfg
```

> 配置zoo.cfg：

```
vim /export/services/zookeeper/conf/zoo.cfg
```

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/export/datas/zookeeper/data
dataLogDir=/export/datas/zookeeper/log
clientPort=2181
```

> 创建ZooKeeper的数据路径：

```
mkdir -p /export/datas/zookeeper/data
mkdir -p /export/datas/zookeeper/log
```

> 添加到环境变量：

```
vim /etc/profile
```

```
export ZOOKEEPER_HOME=/export/services/zookeeper
export PATH=.:$ZOOKEEPER_HOME/bin:$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
```

```
source /etc/profile
```

> 启动ZooKeeper：

```
zkServer.sh start
zkServer.sh status
```

> `安装kafka(使用root用户操作)`

> 解压：

```
tar -zxf /export/softwares/kafka_2.11-2.2.0.tgz  -C /export/services/
```

> 创建软连接：

```
ln -s /export/services/kafka_2.11-2.2.0 /export/services/kafka
```

> 配置server.prperties：

``` 
vim /export/services/kafka/config/server.properties
```

```
listeners=PLAINTEXT://server01:9092
broker.id=0
zookeeper.connect=server01:2181
```

> 添加环境变量：vim /etc/profile

```
export KAFKA_HOME=/export/services/kafka
export PATH=.:$KAFKA_HOME/bin:$ZOOKEEPER_HOME/bin:$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
```

```
source /etc/profile
```

> 启动Kafka：

```
kafka-server-start.sh -daemon /export/services/kafka/config/server.properties
```

> 创建主题：

```
kafka-topics.sh --create --zookeeper server01:2181 --replication-factor 1 --partitions 1 --topic test_ogg
```

> 查看主题：

```
kafka-topics.sh --list --zookeeper server01:2181
```

#### 2.4.5 配置管理器MRG进程

操作步骤 | 说明
---|---
1 | 使用oracle用户进入OGG_SRC_HOME目录下<br/><br/>切换到oracle用户下：su – oracle<br/><br/>打印目标端OGG_TGR_HOME：echo $OGG_TGR_HOME<br/><br/>进入OGG_TGR_HOME：cd $OGG_TGR_HOME<br/><br/>启动ggsci：./ggsci
2 | 配置目标端MRG进程<br/><br/>配置MGR进程：edit param mgr<br/><br/>新增内容：<br/><br/>PORT 7810gg<br/>DYNAMICPORTLIST 7810-7909<br/>AUTORESTART EXTRACT *,RETRIES 5,WAITMINUTES 3g<br/>PURGEOLDEXTRACTS ./dirdat/*,usecheckpoints, minkeepdays 3g


#### 2.4.6 配置checkpoint

```
edit param ./GLOBALS
```

> 新增内容：

```
CHECKPOINTTABLE test_ogg.checkpoint
```

#### 2.4.7 配置Replicate进程

操作步骤 | 说明
---|---
1 | 配置目标端Replicate进程<br/><br/>配置replicate进程：edit param rekafka<br/>REPLICAT rekafka<br/>sourcedefs /u01/app/ogg/tgr/dirdef/test_ogg.test_ogg<br/>TARGETDB LIBFILE libggjava.so SET property=dirprm/kafka.props<br/>REPORTCOUNT EVERY 1 MINUTES, RATE<br/>GROUPTRANSOPS 10000<br/>MAP test_ogg.test_ogg, TARGET test_ogg.test_ogg;

#### 2.4.8 添加trail文件到replicate进程

操作步骤 | 说明
---|---
1 | 添加trail文件到Replicate进程<br/><br/>add replicat rekafka exttrail /u01/app/ogg/tgr/dirdat/to,checkpointtable test_ogg.checkpoint

#### 2.4.9 配置kafka.props

> 配置kafka.props

```
cd $OGG_TGR_HOME
```

```
vim dirprm/kafka.props
```

> 新增内容：

```
gg.handlerlist=kafkahandler
gg.handler.kafkahandler.type=kafka
gg.handler.kafkahandler.KafkaProducerConfigFile=custom_kafka_producer.properties
gg.handler.kafkahandler.topicMappingTemplate=test_ogg
gg.handler.kafkahandler.format=json
gg.handler.kafkahandler.mode=op
gg.classpath=dirprm/:/export/services/kafka/libs/*:/u01/app/ogg/tgr/:/u01/app/ogg/tgr/lib/*
```

> 配置custom_kafka_producer.properties

```
cd $OGG_TGR_HOME
```

```
vim dirprm/custom_kafka_producer.properties
```

> 新增内容：

```
bootstrap.servers=server01:9092
acks=1
compression.type=gzip
reconnect.backoff.ms=1000
value.serializer=org.apache.kafka.common.serialization.ByteArraySerializer
key.serializer=org.apache.kafka.common.serialization.ByteArraySerializer
batch.size=102400
linger.ms=10000
```

#### 2.4.10 最后确认所有的进程

> 在目标端，主要做了4个操作，共包括2个进程，分别是MANAGER和REPLICAT。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/38.png)

### 2.5 OGG测试

#### 2.5.1 启动Docker myoracle容器

> 注意：使用root用户执行

##### 2.5.1.1 启动myoracle容器

```
docker container start myoracle
```

##### 2.5.1.2 进入myoracle容器中

```
docker exec -it myoracle /bin/bash
```

##### 2.5.1.3 切换到oracle用户

```
su - oracle
```

##### 2.5.1.4 启动oracle服务

###### 2.5.1.4.1 启动oracle监听服务

```
source ~/.bash_profile
lsnrctl start
```

###### 2.5.1.4.2 启动oracle数据库

```
sqlplus "/as sysdba"
startup
```

> ·用客户端连一下试试·

##### 2.5.1.5 启动OracleGoldenGate服务

> 注意：必须使用oracle用户执行，需要打开两个Shell会话窗口（分别用于启动源端和目标端）

###### 2.5.1.5.1 在第一个Shell窗口执行（源端）

```
docker exec -it myoracle /bin/bash
su - oracle
cd $OGG_SRC_HOME
./ggsci
```

###### 2.5.1.5.2 在第二个Shell窗口执行（目标端）

```
docker exec -it myoracle /bin/bash
su - oracle
cd $OGG_TGR_HOME
./ggsci
```

###### 2.5.1.5.3 启动源端管理进程

```
start mgr
```

###### 2.5.1.5.4 启动目标端管理进程

```
start mgr
```

###### 2.5.1.5.5 启动源端extract进程

```
start extkafka
```

###### 2.5.1.5.6 启动源端pump进程

```
start pukafka
```

###### 2.5.1.5.7 启动目标端replicat进程

```
start rekafka
```


## 3 测试OGG

### 3.1 源端进程列表

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/39.png)

### 3.2 目标端进程列表

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/40.png)

### 3.3 目标端的配置

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Ogg/41.png)

### 3.4 测试

> 这时候去操作oracle里的数据，应该就可以看到kafka里同步了。

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

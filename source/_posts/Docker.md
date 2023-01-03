---
title: Docker 开源的应用容器引擎
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(63).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(63).jpg
tags:
  - 大数据
  - Docker
category:
  - - 编程
    - 大数据
 
date: 2020-06-05 22:39:21
---

Docker 是一个`开源的应用容器引擎`，基于 Go 语言 并遵从 Apache2.0 协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个`轻量级、可移植的容器中`，然后发布到任何流行的 Linux 机器上，也可以实现`虚拟化`。

`容器`是完全使用`沙箱机制`，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是`容器性能开销极低`。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 大数据项目为什么使用Docker

> 随着大数据平台型产品方向的深入应用实践和`Docker开源社区`的逐渐成熟，业界有不少的大数据研发团队开始使用Docker。简单来说，`Docker会让大数据平台部署更加简单快捷、让研发和测试团队集成交付更加敏捷高效、让产线环境的运维更加有质量保障`。

### 1.1 场景一

> 在大数据平台型产品的开发过程中，经常要跟许多模块打交道，包括`Hadoop`、`HBase`、`Hive`、`Spark`、`Sqoop`、`Zookeeper`……等多达几十个开源组件，为了不影响团队成员间的工作任务协同，开发人员其实非常需要自己有一套独立的集群环境，以便反复测试自己负责的模块，可真实的企业开发环境往往只有一两个大的虚拟集群，这可怎么办？

> 难道要给每个开发人员都配几台独立的物理机器？

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/1.png)

### 1.2 场景二

> 针对每一次新版本的发布，`产品测试组`都需要反复的重装整个平台以便发现问题，而正如本文前面所阐述的那样，大数据平台所依赖的组件繁多，不同组件模块依赖的底层库也不尽相同，经常会出现各种依赖冲突问题，而一旦安装完成，就很难再让Linux系统恢复到一个非常干净的状态，通过`Remove`、`UnInstall`、`rpm -e`等手动方式卸载，往往需要花费很长的时间，那如何才能快速地恢复大数据平台集群的系统环境?

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/2.png)

### 1.3 场景三

> 当测试人员在测试大数据平台过程中发现了一个Bug，需要保存现场，这里面包括相关的大数据组件配置、进程状态、运行日志、还有一些中间数据，可是，平台集群服务器节点数量很多，针对每个进程的配置目录和日志文件，都相对较独立，一般都需要专业的开发工程师或者运维工程师进入相关服务器节点，按照不同组件的个性化配置信息，手工方式收集所需的各个条目信息，然后打包汇集到日志中心服务器进行统一分析，而目前业界并没有一款能够自动分布式收集故障相关的日志系统，但测试工作还要继续，怎么办?

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/3.png)

> `传统解决方案的缺陷`

> 想要解决这些问题，第一个想到的方案当然是用虚拟机，但这种方式并不能完美的解决以上问题，比如：
>> 虽然虚拟机也可以完成系统环境的迁移，但这并不是它所擅长的，不够灵活，很笨重。
> 
>> 虚拟机的快照可以保存当前的状态，但要恢复回去，就得把当前正在运行的虚拟机关闭，所以并不适合频繁保存当前状态的业务场景。
> 
>> 虽然可以给每个人都分配几个虚拟机用，但它是一个完整的系统，本身需要较多的资源，底层物理机的资源很快就被用完了，所以我们需要寻找其它方式来弥补这些不足。

## 2 Docker介绍

### 2.1 什么是虚拟化

> 在计算机中，`虚拟化`（英语：Virtualization）是`一种资源管理技术`，是将计算机的各种实体资源，如：服务器、网络、内存、存储等等，予以抽象、转换后呈现出来，打破实体结构间的不可切割的障碍，使用户可以比原来的组态更好的方式来应用这些资源，这些资源的核心虚拟部分是不受现有资源的架设方式，低于或者物理组态所限制，一般所指的虚拟化资源包括计算能力和资料存储。

> 在实际的生产过程中，虚拟化技术主要是用来解决高性能的物理硬件产能过剩和老的硬件产能过低的重用重组，透明化底层物理硬件，从而最大化的利用物理硬件，对资源充分利用

> `虚拟化技术种类`很多，例如：`软件虚拟化`、`硬件虚拟化`、`内存虚拟化`、`网络虚拟化（vip）`，`桌面虚拟化`、`服务虚拟化`、`虚拟机`等等。

> `虚拟化简单讲，就是把一台物理计算机虚拟成多台逻辑计算机，每个逻辑计算机里面可以运行不同的操作系统，相互不受影响，这样就可以充分利用硬件资源`

### 2.2 初始Docker

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/4.png)

> Docker是一个开源的`应用容器引擎`

> `诞生于2013年初`，基于Go语言实现，dotCloud公司出品（后改名为Docker Inc）

> Docker可以让开发者打包他们的应用以及依赖包到一个`轻量级，可移植的容器中`，然后发布到任何流行的linux服务器上

> 容器是完全使用沙箱机制，相互隔离

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/5.png)

> 容器性能开销极低

> Docker从17.3版本之后分为CE（Community Edition社区版）和EE（Enterprise Edition：企业版）

> `小结`：
>> `Docker是一种容器技术，解决软件跨环境迁移的问题`

### 2.3 容器与虚拟机的比较

> `什么是虚拟机`
>> 虚拟机是一个计算机系统的仿真，简单来说，虚拟机可以实现在一台物理计算机上模拟多台计算机运行任务。
> 
>> 操作系统和应用共享一台或多台主机(集群)的硬件资源，每台VM有自己的OS，硬件资源是虚拟化的。管理程序(hypervisor)负责创建和运行VM，它连接了硬件资源和虚拟机，完成server的虚拟化。
> 
>> 由于虚拟化技术和云服务的出现，IT部门通过部署VM可以可减少cost提高效率。
> 
>> ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/6.png)
> 
>> VMs也消耗大量系统资源，每个VM不仅运行一个OS的完整copy并且需要所有硬件的虚拟化copy，这消耗大量RAM和CPU。相比单独计算机，VM是比较经济的，但对于一些应用VM是过度浪费的，需要容器。

> `什么是容器`
>> 容器是将操作系统虚拟化，这与VM虚拟化一个完整的计算机有所不同。
> 
>> 容器是在操作系统之上，每个容器共享OS内核，执行文件和库等。共享的组件是`只读`的，通过共享OS资源能够减少复现OS的代码，意味着一台server仅安装一个OS可以运行多个任务。容器是非常轻量的，仅仅`MB`水平并且几秒即可启动。相比容器，VM需要几分钟启动，并且大小也大很多。
> 
>> 与VM相比，容器仅需OS、支撑程序和库文件便可运行应用，这意味你可以在同一个server上相比VM运行2-3倍多的应用，并且，容器能帮助创建一个可移植的，一致的开发测试部署环境。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/7.png)

> 小结：

特性 | 虚拟机 | 容器
---|---|---
隔离级别 | 操作系统级 | 进程级
隔离策略 | 运行于Hypervisor上 | 直接运行在宿主机内核中
系统资源 | 5-15% | 0-5%
启动速度 | 慢，分钟级 | 快，秒级
占用磁盘空间 | 非常大，GB-TB级 | 小，KB-MB甚至KB级
并发性 | 一台宿主机十几个，最多几十个 | 上百个，甚至上百上千个
高可用策略 | 备份、容灾、迁移 | 弹性、负载、动态

> `结论`：
>> `与传统的虚拟化相比，Docker优势体现在启动速度快，占用体积小`

## 3 Docker与虚拟机的形象比喻

### 3.1 什么是物理机 

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/8.png)

### 3.2 什么是虚拟机

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/9.png)

### 3.3 什么是docker

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/10.png)

## 4 Docker组件

### 4.1 Docker服务端和客户端

> `Docker是一个客户端-服务端（C/S）架构程序`，Docker客户端只需要向Docker服务端或者守护进程发出请求，服务端或者守护进程完成所有工作返回结果，Docker提供了一个命令行工具Docker以及一整套的`Restful API`，可以在同一台宿主机器上运行Docker守护进程或者客户端，也可以从本地的Docker客户端连接到运行在另一台宿主机上的远程Docker守护进程

> docker引擎是一个c/s结构的应用，主要组件见下图：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/11.png)

> Server是一个常驻进程

> REST API 实现了client和server间的交互协议

> CLI 实现容器和镜像的管理，为用户提供统一的操作界面

### 4.2 Docker架构

> Docker使用C/S架构，Client 通过接口与Server进程通信实现容器的构建，运行和发布。client和server可以运行在同一台集群，也可以通过跨主机实现远程通信。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/12.png)

#### 4.2.1 Docker镜像 

> Docker 镜像（Image）就是一个只读的模板。例如：一个镜像可以包含一个`完整的操作系统环境`，里面仅安装了 Apache 或用户需要的其它`应用程序`。镜像可以用来创建 Docker 容器，一个镜像可以创建很多容器。`Docker 提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用`。

> 镜像（Image）就是一堆只读层（read-only layer）的统一视角，也许这个定义有些难以理解，看看下面这张图：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/13.png)

> 右边我们看到了多个`只读层`，它们重叠在一起。除了最下面一层，其它层都会有一个指针指向下一层。这些层是Docker内部的实现细节，并且能够在docker宿主机的文件系统上访问到。统一文件系统（Union File System）技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统。

#### 4.2.2 Docker容器

> Docker 利用容器（Container）来运行应用。容器是从镜像创建的`运行实例`。它可以被`启动、开始、停止、删除`。每个容器都是相互隔离的、保证安全的平台。

> `可以把容器看做是一个简易版的 Linux 环境`（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

> 创建Container首先要有Image，也就是说Container是通过image创建的。

> Container是在原先的Image之上新加的一层，称作`Container layer`，这一层是可读可写的（Image是只读的）。

> 在面向对象的编程语言中，有类跟对象的概念。类是抽象的，对象是类的具体实现。Image跟Container可以类比面向对象中的类跟对象，Image就相当于抽象的类，Container就相当于具体实例化的对象。

> Image跟Container的职责区别：`Image负责APP的存储和分发，Container负责运行APP`。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/14.png)

> `结论：`
>> `容器 = 镜像 + 读写层。并且容器的定义并没有提及是否要运行容器`

#### 4.2.3 Registy（注册中心）

> 仓库（Repository）是集中存放`镜像文件`的场所。有时候会把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。

> 仓库分为公开仓库（Public）和私有仓库（Private）两种形式。最大的公开仓库是 Docker Hub，存放了数量庞大的镜像供用户下载。国内的公开仓库包括 时速云 、网易云 等，可以提供大陆用户更稳定快速的访问。当然，用户也可以在本地网络内创建一个私有仓库。

> 当用户创建了自己的镜像之后就可以使用 push 命令将它上传到公有或者私有仓库，这样下次在另外一台机器上使用这个镜像时候，只需要从仓库上 pull 下来就可以了。

> Docker 仓库的概念跟 Git 类似，注册服务器可以理解为 GitHub 这样的托管服务。

## 5 Docker总结

### 5.1 什么是Docker

> 使用最广泛的开源容器引擎

> 一种操作系统的虚拟化技术linux内核

> 依赖于linux内核特性：`NameSpace和Cgroups`

> 一个简单的应用程序打包工具

### 5.2 作用和目的

> 提供简单的应用程序打包工具

> 开发人员和运维人员职责逻辑分离

> 多环境保持一致，消除环境的差异

### 5.3 Docker的应用场景

> 应用程序的打包和发布

> 应用程序的隔离

> 持续集成

> 部署微服务

> 快速搭建测试环境

> 提供PaaS平台级别产品

### 5.4 容器带来的好处有哪些

> 秒级的交付和部署

> 保证环境一致性

> 高效的资源利用

> 弹性的伸缩

> 动态调度迁移成本低

### 5.5 需要注意的内容

> 大家需要注意，`Docker本身并不是容器，它是创建容器的工具，是应用容器引擎`。想要搞懂Docker，其实看它的两句口号就行。

> 第一句，是“`Build, Ship and Run`”。也就是，“`搭建、发送、运行`”，三板斧。

> 第二句口号就是：“`Build once，Run anywhere`（`搭建一次，到处能用`）”。

> Docker技术的三大核心概念，分别是：
>> `镜像（Image）`
> 
>> `容器（Container）`
> 
>> `仓库（Repostitory）`

> 负责对Docker镜像进行管理的，是Docker Registry服务（类似仓库管理员）,不是任何人建的任何镜像都是合法的，万一有人盖了一个有问题的房子呢？

> 所以，Docker Registry服务队镜像的管理是非常严格的

> 最常使用的Registry公开服务，是官方的`Docker Hub`，这也是默认的Registry，并拥有大量的高质量的官方镜像

> `官方地址`：https://hub.docker.com/

## 6 Docker的安装和启动

### 6.1 安装Docker

> Docker官方建议在`Ubuntu`中安装，因为Docker是基于Unbantu发布的，而且一般Docker出现的问题Ubuntu是最先更新或者打补丁的，在很多版本的Centos中是不支持更新最新的一些补丁包的。

> 这里将Docker安装到`Centos`上，注意：建议安装在`Centos7.x`以上的版本，在Centos6.x的版本中，安装前需要安装其他很多的环境，而且Docker很多补丁不支持更新。

#### 6.1.1 验证Linux内核版本

> Docker要求Linux的`Kernel`版本`必须大于3.8`，推荐使用`3.10及更高`，所以需要先验证CentOS的内核是否大于3.8。

```
uname -r
```

#### 6.1.2 卸载已安装的Docker

> 如果已经安装过Docker，请`先卸载，再重新安装`，来确保整体的环境是一致的。由于这个虚拟机是新创建的，所以并没有安装过Docker，但如果同学们使用已有的虚拟机，则需要按照下面的命令卸载。

```
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```

#### 6.1.3 安装yum工具包和存储驱动

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

#### 6.1.4 安装Docker的yum源

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

> 如果连接超时，可以使用alibaba源

```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

#### 6.1.5 安装Docker

```
yum install docker-ce docker-ce-cli containerd.io
```

#### 6.1.6 启动Docker

```
sudo service docker start
```

#### 6.1.7 设置开机启动

```
systemctl enable docker
```

#### 6.1.8 将指定用户添加到用户组

```
usermod -aG docker root
```

> `退出`，然后`重新登录`，以便让权限生效。

#### 6.1.9 安装后查看Docker版本

```
docker version
```

### 6.2 Docker的启动和停止

操作 | 指令
---|---
启动docker | systemctl start docker
停止docker | systemctl stop docker
重启docker | systemctl restart docker
查看docker状态 | systemctl status docker
开机启动 | systemctl enable docker
查看docker概要信息 | docker info
查看docker帮助文档 | docker --help

### 6.3 配置阿里云镜像加速

操作步骤 | 说明
---|---
1 | 鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，我们可以需要配置加速器来解决<br/><br/>https://help.aliyun.com/product/60716.html
2 | 注册一个属于自己的阿里云账户(可复用淘宝账号)<br/><br/>登陆阿里云开发者平台<br/><br/>https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors<br/><br/>获取加速器地址<br/><br/> ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/15.png)
3| 配置本机Docker运行镜像加速器<br/><br/>阿里云的本人自己账号的镜像地址(需要自己注册有一个属于你自己的)：   https://xxxx.mirror.aliyuncs.com<br/><br/>vim /etc/docker/daemon.json<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/16.png)
4| 重启daemon服务<br/><br/>systemctl daemon-reload
5| 重启Docker服务<br/><br/>systemctl restart docker
6| Linux 系统下配置完加速器需要检查是否生效<br/><br/>systemctl status docker<br/><br/>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/7.png)

## 7 Docker常用命令

### 7.1 帮助命令

操作 | 指令
---|---
查看Docker版本 | docker version
查看docker概要信息 | docker info
查看docker帮助文档 | docker --help

### 7.2 镜像命令

#### 7.2.1 搜索镜像

> 网站：https://hub.docker.com

> 如果需要在网络中查找需要的镜像，可以通过以下命令搜索

```
docker search 某个XXX镜像名字
```

> 例如：要下载centos镜像

```
docker search centos
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/18.png)

```
NAME：镜像名称
DESCRIPTION：镜像描述
STARS：用户评价，反应一个镜像的受欢迎程度
OFFICIAL：是否官方
AUTOMATED：自动构建，表示该镜像由Docker Hub自动构建流程创建的
```

> OPTIONS说明：

```
--no-trunc : 显示完整的镜像描述
-s : 列出收藏数不小于指定值的镜像。
--automated : 只列出 automated build类型的镜像；
```

#### 7.2.2 拉取镜像

> 拉取镜像就是从中央仓库中下载镜像到本地，命令：

```
docker pull 镜像名字
```

> 例如，要下载centos7镜像：

```
docker pull centos:7
```

#### 7.2.3 查看镜像

> 命令：

```
docker images
```
```
REPOSITORY：镜像名称
TAG：镜像标签
IMAGE ID：镜像id
CREATED：镜像的创建日期（不是获取该镜像的日期）
SIZE：镜像大小
```

> OPTIONS说明：

```
-a :列出本地所有的镜像（含中间映像层）
-q :只显示镜像ID。
--digests :显示镜像的摘要信息
--no-trunc :显示完整的镜像信息
```

#### 7.2.4 删除镜像

> 删除单个镜像

```
docker rmi  -f 镜像ID
```

> 删除多个镜像

```
docker rmi -f 镜像名1:TAG 镜像名2:TAG
```

> 删除全部

```
docker rmi -f $(docker images -qa)
```

### 7.3 容器命令

> 有镜像才能创建容器，这是根本前提

#### 7.3.1 查看容器

> 查看正在运行的容器

```
docker ps
```

> 查看所有容器

```
docker ps -a
```

> 查看最后一次运行的容器

```
docker ps -l
```

> 查看停止的容器

```
docker ps -f status=exited
```

#### 7.3.2 创建与启动容器

> 创建容器常用的参数说明

> 创建容器的命令：

```
docker run
```

> OPTIONS说明（常用）：有些是一个减号，有些是两个减号

```
--name="容器新名字": 为容器指定一个名称；
-d: 在run后面加上参数-d，会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加-i-t两个参数，创建后会自动进入容器），并返回容器ID，也即启动守护式容器；
-i：以交互模式运行容器，通常与 -t 同时使用；
-t：表示容器启动后会进入其命令行，为容器重新分配一个伪输入终端，通常与 -i 同时使用；
-p: 表示端口映射
有以下四种格式
ip:hostPort:containerPort
ip::containerPort
hostPort:containerPort
containerPort
前者表示宿主机端口，后者是容器内的映射端口，可以使用多个-p做多个端口映射
```

> 启动交互式容器

```
docker run -it --name=centos 镜像名称:标签 /bin/bash
拉取centos：docker pull centos
docker run -it centos /bin/bash

这时通过ps命令查看，发现可以看到启动的容器，状态为启动状态
也可以这样写
docker run -it --name=mycentos centos:latest /bin/bash

/bin/bash的作用是因为docker后台必须运行一个进程，否则容器就会退出，在这里表示启动容器后启动bash。
```
> 退出当前容器

```
exit
```

> 守护式方式创建容器：

```
docker run -di --name=容器名称 镜像名称:标签
docker run -di --name=mycentos2 centos:latest
```

> 登录守护式容器方式：

```
docker exec -it 容器名称(或者容器id) /bin/bash
```

> `什么是守护式容器`：
>> 能够长期运行
> 
>> 没有交互式会话
> 
>> 适合运行应用程序和服务

#### 7.3.3 停止与启动容器

> 停止容器

```
docker stop 容器名称（或者容器id）
docker stop mycentos2
```

> 启动容器

```
docker start 容器名称（或者容器id）
docker start mycentos2
```

> 重启容器

```
docker restart 容器名称（或者容器id）
docker restart mycentos2
```

> 强制停止容器

```
docker kill 容器名称（或者容器id）
docker kill mycentos2
```

#### 7.3.4 文件拷贝

> 如果需要将文件拷贝到容器内可以使用cp命令

```
docker cp 需要拷贝的文件或者目录 容器名称:容器目录
docker cp  /tmp/anaconda.log  mycentos2:/tmp
docker exec -it mycentos2 /bin/bash
```

> 也可以将文件从容器内拷贝出来

```
docker cp 容器名称:容器目录 需要拷贝的文件或者目录
docker cp  mycentos2:/tmp /export/
```

#### 7.3.5 目录挂载

> 可以在`创建容器的时候`，将宿主机的目录和容器内的目录进行映射，这样就可以通过修改宿主机的某个目录的文件从而去影响容器

> 创建容器添加-v参数，后边为宿主机目录：容器目录，例如：

```
docker run -di -v /usr/local/myhtml:/usr/local/myhtml --name=mycentos3 centos:latest
docker exec -it mycentos3 /bin/bash
ls /usr/local/myhtml/
cp /export/tmp/anaconda.log /usr/local/myhtml/
docker exec -it mycentos3 /bin/bash
docker run -di --privileged=true -v /usr/local/myhtml:/usr/local/myhtml --name=mycentos3 centos:latest
```

> `如果共享的是多级的目录，可能会出现权限不足的提示。`

> `我们需要添加参数--privileged=true来解决挂载的目录没有权限的问题？`

#### 7.3.6 查看容器ip地址

> 可以通过以下命令查看容器运行的各种数据

```
docker inspect 容器名称（容器id）
docker inspect mycentos3
```

> 也可以直接执行下面的命令直接输出IP地址

```
docker inspect --format=’{{.NetworkSettings.IPAddress}}’ 容器名称（容器id）
docker inspect --format=’{{.NetworkSettings.IPAddress}}’ mycentos3
```

#### 7.3.7 删除容器

> 删除指定的容器：

```
docker rm 容器名称（容器ID）
docker stop mycentos3
docker rm mycentos3
```

## 8 Docker应用部署

### 8.1 MySQL部署

> Docker hub上查找mysql镜像

```
docker search mysql
```

> 从docker hub上(阿里云加速器)拉取mysql镜像到本地标签为5.7

```
docker pull centos/mysql-57-centos7
```

> 创建容器

```
docker run -di --name=tensquare_mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 centos/mysql-57-centos7

-p 代表端口映射，格式为宿主机映射端口:容器运行端口
-e 代表添加环境变量，MYSQL_ROOT_PASSWORD是root用户的登录密码
```

> 使用mysql客户端连接

### 8.2 Nginx部署

> Docker hub上查找nginx镜像

```
docker search nginx
```

> 从docker hub上(阿里云加速器)拉取nginx镜像到本地

```
docker pull nginx
```

> 创建nginx容器

```
docker run -di --name=mynginx -p 80:80 nginx

-p 代表端口映射，格式为宿主机映射端口:容器运行端口
```

> 打开浏览器访问:http://ip/

### 8.3 Redis部署

> Docker hub上查找redis镜像

```
docker search redis:4.0
```

> 从docker hub上(阿里云加速器)拉取redis镜像到本地

```
docker pull redis:4.0
```

> 创建redis容器

```
docker run -di --name=myredis -p 6379:6379 redis:4.0

-p 代表端口映射，格式为宿主机映射端口:容器运行端口
```

> 使用redis-cli连接

```
docker exec -it myredis redis-cli
```

> 打开redis客户端连接redis服务器

> 测试持久化文件生成（挂载目录的方式）
>> 在宿主机创建目录

```
mkdir -p /export/docker/myredis/data/
mkdir -p /export/docker/myredis/conf/
```

> 在conf创建配置文件redis.conf

> 启动容器

```
docker run -p 6379:6379 -v /export/docker/myredis/data/:/data -v /export/docker/myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf -d redis:4.0 redis-server /usr/local/etc/redis/redis.conf --appendonly yes
```

> 查看持久化文件是否生成

```
ls /export/docker/myredis/data/
```

## 9 Docker的迁移与备份

### 9.1 容器保存为镜像

> 可以通过以下命令将容器保存为镜像

```
docker commit mynginx mynginx_image
```

> 基于新创建的镜像创建容器

```
docker run -di  --name=mynginx2 -p 81:80 mynginx_image
```
> 访问81端口,http://ip:81/

### 9.2 镜像备份

> 可以通过以下命令将镜像保存为tar文件

```
docker save -o mynginx.tar mynginx_image
-o：表示output，输出的意思
```

### 9.3 镜像恢复与迁移

> 先删除掉mynginx_image镜像，然后执行此命令进行恢复

```
docker rmi mynginx_image
```

> 因为该镜像存在容器，所以先删除容器，在删除镜像

> 使用此命令进行恢复镜像

```
docker load -i mynginx.tar

-i：表示输入的文件，执行后再次查看镜像，可以看到镜像已经恢复
```

## 10 Docker镜像

### 10.1 Docker镜像是什么

> 镜像是一种轻量级、可执行的独立软件包，`用来打包软件运行环境和基于运行环境开发的软件`，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

#### 10.1.1 UnionFS(联合文件系统)

> `UnionFS（联合文件系统）`：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它`支持对文件系统的修改作为一次提交来一层层的叠加`，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。

> Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/19.png)

> 特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

#### 10.1.2 Docker镜像加载原理

> docker的镜像实际上由一层一层的文件系统组成，`这种层级的文件系统UnionFS`。
>> `bootfs(boot file system)`主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。
> 
>> `rootfs (root file system)` ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/20.png)

> `平时我们安装进虚拟机的CentOS都是好几个G，为什么docker这里才200M？？`

> 对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别, 因此不同的发行版可以公用bootfs。

#### 10.1.3 分层的镜像

> 以我们的pull为例，在下载的过程中我们可以看到docker的镜像好像是在一层一层的在下载

#### 10.1.4 为什么Docker镜像要采用这种分层结构

> `最大的一个好处就是 - 共享资源`

> 比如：有多个镜像都从相同的 base 镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像，同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

### 10.2 Docker镜像的特点

> Docker镜像都是只读的

> 当容器启动时，一个新的可写层被加载到镜像的顶部

> 这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”

## 11 Dockerfile

### 11.1 什么是DockerFile

#### 11.1.1 介绍

> `Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像`
>> 对于开发人员：可以为开发团队提供一个完全一致的开发环境
> 
>> 对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作
> 
>> 对于运维人员：在部署时，可以实现应用的无缝移植

#### 11.1.2 DockerFile构建步骤

```
编写Dockerfile文件
docker build
docker run
```

#### 11.1.3 DockerFile文件内容

> 以熟悉的centos为例：https://hub.docker.com/_/centos/

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/22.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/23.png)

### 11.2 DockerFile构建过程分析

#### 11.2.1 DockerFile内容基础知识

> 每条保留字指令都必须为大写字母且后面要跟随至少一个参数

> 指令按照从上到下，顺序执行

> 井号表示注释

> 每条指令都会创建一个新的镜像层，并对镜像进行提交

#### 11.2.2 Docker执行DockerFile的大致流程

> docker从基础镜像运行一个容器

> 执行一条指令并对容器作出修改

> 执行类似docker commit的操作提交一个新的镜像层

> docker再基于刚提交的镜像运行一个新容器

> 执行dockerfile中的下一条指令直到所有指令都执行完成

#### 11.2.3 总结

> 从应用软件的角度来看，`Dockerfile、Docker镜像与Docker容器`分别代表软件的三个不同阶段，
>> Dockerfile是软件的原材料
> 
>> Docker镜像是软件的交付品
> 
>> Docker容器则可以认为是软件的运行状态。

> `Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/24.png)

> `Dockerfile`，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等;

> `Docker镜像`，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时，会真正开始提供服务;

> `Docker容器`，容器是直接提供服务的。

### 11.3 常用命令

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/25.png)

命令 | 作用
---|---
FROM image_name:tag | 定义了使用哪个基础镜像启动构建流程
MAINTAINER user_name | 声明镜像的创建者，创建者的用户名和邮箱地址
ENV key value | 设置环境变量（可以写多条）
RUN command | 是Dockerfile的核心部分（可以写多条）
ADD source_dir/file dest_dir/file | 将宿主机的文件复制到容器内，如果是一个压缩文件，将会在复制后自动解压
COPY source_dir/file dest_dir/file | 和ADD相似，但是如果有压缩文件并不能解压
WORKDIR path_dir | 设置工作目录

#### 11.3.1 FROM

> 指明构建的新镜像是来自于哪个基础镜像，例如：

```
FROM centos: latest
```
#### 11.3.2 MAINTAINER

> 指明镜像维护着及其联系方式（一般是邮箱地址），例如：

```
MAINTAINER JC Zhang <zhangsan@163.com>
```

> 不过，MAINTAINER并不推荐使用，更推荐使用LABEL来指定镜像作者，例如：

```
LABEL maintainer="zhangsan.cn"
```

#### 11.3.3 RUN

> 构建镜像时运行的Shell命令，例如：

```
RUN ["yum", "install", "httpd"]
RUN yum install httpd
```

#### 11.3.4 CMD

> 启动容器时执行的Shell命令，例如：

```
CMD ["-C", "/start.sh"]
CMD ["/usr/sbin/sshd", "-D"]
CMD /usr/sbin/sshd -D
```

#### 11.3.5 EXPOSE

> 声明容器运行的服务端口，例如：

```
EXPOSE 80 443
```

#### 11.3.6 ENV

> 设置环境内环境变量，例如：

```
ENV MYSQL_ROOT_PASSWORD 123456
ENV JAVA_HOME /usr/local/jdk1.8.0_45
```

#### 11.3.7 ADD

> 拷贝文件或目录到镜像中，例如：

```
ADD <src>...<dest>
ADD html.tar.gz /var/www/html
ADD https://xxx.com/html.tar.gz /var/www/html
```

> PS：如果是URL或压缩包，会自动下载或自动解压。

#### 11.3.8 COPY

> 拷贝文件或目录到镜像中，用法同ADD，只是不支持自动下载和解压，例如：

```
COPY ./start.sh /start.sh
```

#### 11.3.9 ENTRYPOINT

> 启动容器时执行的Shell命令，同CMD类似，只是由ENTRYPOINT启动的程序不会被docker run命令行指定的参数所覆盖，而且，这些命令行参数会被当作参数传递给ENTRYPOINT指定指定的程序，例如：

```
ENTRYPOINT ["/bin/bash", "-C", "/start.sh"]
ENTRYPOINT /bin/bash -C '/start.sh'
```

> `PS：Dockerfile文件中也可以存在多个ENTRYPOINT指令，但仅有最后一个会生效。`

#### 11.3.10 VOLUME

> 指定容器挂载点到宿主机自动生成的目录或其他容器，例如：

```
VOLUME ["/var/lib/mysql"]
```

> `PS：一般不会在Dockerfile中用到，更常见的还是在docker run的时候指定-v数据卷。`

#### 11.3.11 WORKDIR

> 为RUN、CMD、ENTRYPOINT以及COPY和AND设置工作目录，例如：

```
WORKDIR /data
```

### 11.4 使用脚本创建镜像

#### 11.4.1 编写DockerFile

> 创建目录

```
mkdir -p /export/docker/jdk8
```

> 将jdk-8u241-linux-x64.tar.gz上传到服务器（虚拟机）中的/export/docker/jdk8目录

> 创建文件Dockerfile

```
cd /export/docker/jdk8/
vi Dockerfile
```

```
#依赖镜像名称和id
FROM centos:latest
#指定镜像创建者信息
MAINTAINER XIAOMA
#切换工作目录
WORKDIR /usr
RUN mkdir /usr/local/java
#ADD 是相对路径jar，把java添加到容器中
ADD jdk-8u241-linux-x64.tar.gz /usr/local/java

#配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_241
ENV JAR_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH
```

#### 11.4.2 构建镜像

```
docker build -t='jdk1.8' .
```

> `注意后面的空格和点，不要省略，点表示当前目录`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/26.png)

#### 11.4.3 查看镜像是否构建完成

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/27.png)

### 11.5 自定义镜像MyCentos

#### 11.5.1 编写DockerFile

> Hub默认centos镜像

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/28.png)

> 准备编写DockerFile文件

```
vim  Dockerfile
FROM centos:latest
MAINTAINER xiaoma<xiaoma@163.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash
```

#### 11.5.2 构建镜像

```
docker build -t mycentos:1.1 .
```

#### 11.5.3 运行镜像

```
docker run -it mycentos:1.1
```

## 12 Docker私有仓库

### 12.1 registry的搭建

#### 12.1.1 搭建

> Docker 官方提供了一个搭建私有仓库的镜像 registry ，只需把镜像下载下来，运行容器并暴露5000端口，就可以使用了。

```
docker pull registry:2
```

```
docker run -di -v /opt/registry:/var/lib/registry -p 5000:5000 --name myregistry registry:2
```

> Registry服务默认会将上传的镜像保存在容器的/var/lib/registry，我们将主机的/opt/registry目录挂载到该目录，即可实现将镜像保存到主机的/opt/registry目录了。

> 浏览器访问http://ip:5000/v2/_catalog，出现下面情况说明registry运行正常。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/29.png)

#### 12.1.2 验证

> 现在通过push镜像到registry来验证一下。

> 查看本地镜像：

```
docker images
```

> 要通过docker tag将该镜像标志为要推送到私有仓库：

```
docker tag nginx:latest localhost:5000/nginx:latest
```

> 通过 docker push 命令将 nginx 镜像 push到私有仓库中：

```
docker push localhost:5000/nginx:latest
```

> 访问 http://ip:5000/v2/_catalog 查看私有仓库目录，可以看到刚上传的镜像了：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/30.png)

> 下载私有仓库的镜像，使用如下命令：

```
docker pull localhost:5000/镜像名:版本号
# 例如
docker pull localhost:5000/nginx:latest
```
### 12.2 docker镜像推送到阿里云镜像仓库

#### 12.2.1 容器镜像服务控制台概览

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/31.png)

#### 12.2.2 创建镜像仓库

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/32.png)

#### 12.2.3 选择本地仓库

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/33.png)

#### 12.2.4 点击管理

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/34.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/35.png)

#### 12.2.5 Docker登录阿里云镜像仓库

```
$ sudo docker login --username=xxx@aliyun.com registry.cn-hangzhou.aliyuncs.com
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/36.png)

#### 12.2.6 推送创建的orcale11g_centos7镜像到阿里云镜像仓库

> 参考如下的命令

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker/37.png)

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)

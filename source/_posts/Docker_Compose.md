---
title: Docker_Compose安装和使用
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(113).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(113).jpg
tags:
  - 大数据
  - 数据库
category:
  - - 编程
    - 大数据
  - - 编程
    - 数据库
date: 2022-01-15 22:40:09
---

以前没怎么看`Docker Compose`，最近在看很多`官方demo`的时候，`环境准备`都是用`Docker Compose`做的，不得不看一下了。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 docker compose 简介

> `Compose` 项目是 `Docker` 官方的开源项目，负责实现对 Docker 容器集群的快速编排。从功能上看，跟 OpenStack 中的 Heat 十分类似。

> Compose 中有两个重要的概念：
>> `服务 (service)`：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
> 
>> `项目 (project)`：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

> Compose 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

> Compose 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 Compose 来进行编排管理。

## 2 docker compose 安装与卸载

### 2.1 Linux安装

```
vim install_docker.sh
```

```shell
#!/bin/bash

echo "===================================开始安装docker====================================================="
# 如果以前安装过Docker，那么为了保证环境的一致性，还是先卸载了再安装比较好
echo "开始卸载旧docker"
yum -y remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine docker-selinux docker-engine-selinux
rpm -qa |grep docker*
# 卸载docker-ce
yum -y remove docker-ce
# 清理docker-ce数据
rm -rf /var/lib/docker

echo "安装依赖包"
# yum-utils提供yum-config-manager和utility工具
# lvm2和device-mapper-persistent-data提供devicemapper的存储驱动
yum -y install yum-utils lvm2 device-mapper-persistent-data -y

echo "配置stable库"
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

echo "禁用edge和test库"
# 如果启用edge和test库，yum安装时会安装最新版的docker，一般为test测试版，如果要安装最新的稳定版需要禁用该选项 
yum-config-manager --disable docker-ce-edge docker-ce-test
# yum-config-manager --enable docker-ce-edge docker-ce-test

echo "开始安装docker"
# docker-ce是最新版docker，如果用yum -y install docker ，那么只能安装到1.13版本，很多特性是没法使用的
yum -y install docker-ce
# 升级：yum -y upgrade
# 指定版本：yum -y install docker-ce-18.06.1.ce-3.el7

echo "设置开机启动"
systemctl enable docker

echo "自定义网桥"
# 是因为docker的网桥网段有可能与主机冲突，我就是碰到了这个问题
mkdir -p /etc/docker
echo '{  
  "debug" : true,
  "default-address-pools" : [
    {
      "base" : "172.31.0.0/16",
      "size" : 24
    }
  ]
}' > /etc/docker/daemon.json

echo "重启daemon服务" 
systemctl daemon-reload

echo "重启docker"
systemctl restart docker

echo "docker安装成功，版本信息如下"
docker -v
echo "===================================docker安装完成====================================================="
echo -e '\n\n\n\n'
echo "===================================开始安装docker-compose====================================================="
echo "开始卸载旧docker-compose"
rm /usr/local/bin/docker-compose

echo "开始安装docker-compose"
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

echo "脚本授予执行权限"
chmod +x /usr/local/bin/docker-compose

echo "bash命令补全"
curl -L https://raw.githubusercontent.com/docker/compose/1.25.5/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose

echo "docker-compose安装成功，版本信息如下"
docker-compose -v
echo "===================================docker-compose安装完成====================================================="
```

## 3 docker compose 使用

### 3.1 入门demo_tomcat

> 创建`docker-compose.yml`文件

```shell
version: "3.0"

services:
  tomcat: #服务名称(唯一) 
    image: tomcat:8.0-jre8 # 创建当前这个服务使用的景象名称
    ports:
      - 8080:8080
  tomcat1: #服务名称(唯一) 
    image: tomcat:8.0-jre8 # 创建当前这个服务使用的景象名称
    ports:
      - 8081:8080
```

> 启动:`当前yml目录执行`

```shell
docker-compose up
```

> `访问`

```shell
http://192.168.88.58:8080
http://192.168.88.58:8081
```

### 3.2 docker-compose 模板文件

> `官网地址`：https://docs.docker.com/compose/compose-file/compose-file-v3/

> 模板文件是使用 `Compose` 的核心，涉及到的`指令关键字`也比较多。但大家不用担心，这里面大部分指令跟 `docker run` 相关参数的含义都是类似的。

> 默认的模板文件名称为 `docker-compose.yml`，格式为 YAML 格式。

### 3.3 docker-compose 常用命令

> `官网地址`：https://docs.docker.com/compose/reference/

```shell
# 启动(会创建容器)
docker-compose up
# 当前目录下不止一个yml时，需要指定文件名称，或者用绝对路径也可以
docker-compose -f ***.yml up 
# 后台启动
docker-compose up -d
# 停止(会删除容器)
docker-compose down
# 启动停止(无关容器)
docker-compose start
docker-compose stop
```

## 4 docker可视化工具

> `我个人还是比较喜欢命令行操作，所以没怎么用过这个东西`

### 4.1 安装Portainer

> `官网安装说明链接`：https://www.portainer.io/installation

```
echo "===================================开始安装docker Portainer====================================================="
echo "创建Portainer Server用来存储数据库的卷"
docker volume create portainer_data

echo "下载并安装Portainer Server容器"
docker run  -d -p 8000:8000 -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    cr.portainer.io/portainer/portainer-ce:2.9.3
echo "docker Portainer安装完成"
echo "===================================docker Portainer安装完成====================================================="
echo -e '\n\n\n\n' 
echo "请访问 https://ip:9443 查看portainer可视化界面"
echo -e '\n\n\n\n'
```

### 4.2 登录Portainer

> 用浏览器访问：`https://ip:9443`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Docker_Compose/1.png)

## 5 常用docker-compose.yml

### 5.1 mysql

```
vim docker_compose_mysql.yml
```

```
version: '3'
services:
    docker_compose_mysql_8.0.17_3306:
        image: mysql:8.0.17
        container_name: docker_compose_mysql_8.0.17_3306
        privileged: true
        environment:
            - MYSQL_ROOT_PASSWORD=root
        restart: always
        ports:
            - "3306:3306"
        #volumes:
        #    - /data/database/mysql/docker_compose_mysql_8.0.17_3306/log/:/var/log/mysql/
        #    - /data/database/mysql/docker_compose_mysql_8.0.17_3306/lib/:/var/lib/mysql/
        #    - /data/database/mysql/docker_compose_mysql_8.0.17_3306/etc/:/etc/mysql/
        #    - /data/database/mysql/docker_compose_mysql_8.0.17_3306/mysql-files/:/var/lib/mysql-files/
```

```
docker-compose -f docker_compose_mysql.yml up -d
```

> `不挂载目录的话`上面步骤就可以了，`挂载的话`接着走下面的

#### 5.1.1 挂载目录

> `创建文件夹`并`拷贝挂载目录`

```
mkdir -p /data/database/mysql/docker_compose_mysql_8.0.17_3306/log/ 
mkdir -p /data/database/mysql/docker_compose_mysql_8.0.17_3306/lib/ 
mkdir -p /data/database/mysql/docker_compose_mysql_8.0.17_3306/etc/ 
mkdir -p /data/database/mysql/docker_compose_mysql_8.0.17_3306/mysql-files/
docker cp docker_compose_mysql_8.0.17_3306:/var/log/mysql/ /data/database/mysql/docker_compose_mysql_8.0.17_3306/log/ 
docker cp docker_compose_mysql_8.0.17_3306:/var/lib/mysql/ /data/database/mysql/docker_compose_mysql_8.0.17_3306/lib/ 
docker cp docker_compose_mysql_8.0.17_3306:/etc/mysql/ /data/database/mysql/docker_compose_mysql_8.0.17_3306/etc/ 
docker cp docker_compose_mysql_8.0.17_3306:/var/lib/mysql-files/ /data/database/mysql/docker_compose_mysql_8.0.17_3306/mysql-files/
```

> `关闭容器`

```
docker-compose -f docker_compose_mysql.yml down
```

> 打开上面`yml`中关于`挂载的配置`

```
version: '3'
services:
    docker_compose_mysql_8.0.17_3306:
        image: mysql:8.0.17
        container_name: docker_compose_mysql_8.0.17_3306
        privileged: true
        environment:
            - MYSQL_ROOT_PASSWORD=root
        restart: always
        ports:
            - "3306:3306"
        volumes:
            - /data/database/mysql/docker_compose_mysql_8.0.17_3306/log/:/var/log/mysql/
            - /data/database/mysql/docker_compose_mysql_8.0.17_3306/lib/:/var/lib/mysql/
            - /data/database/mysql/docker_compose_mysql_8.0.17_3306/etc/:/etc/mysql/
            - /data/database/mysql/docker_compose_mysql_8.0.17_3306/mysql-files/:/var/lib/mysql-files/
```

> `启动容器`

```
docker-compose -f docker_compose_mysql.yml up -d
```

> 可以去目标目录`/data/database/mysql/docker_compose_mysql_8.0.17_3306/`下看，容器的内容`已经挂载出来了`

> `初始连接信息如下`

```
Host:ip
Port:3306
User:root
Password:root
```

### 5.2 oracle

#### 5.2.1 oracle 11g

```
vim docker_compose_oracle.yml
```

```
version: '3'
services:
    oracle11g1521:
        image: oracleinanutshell/oracle-xe-11g
        container_name: docker_compose_oracle_11g_1521
        privileged: true
        environment:
            - TZ=Asia/Shanghai
        restart: always
        ports:
            - "1521:1521"
        #volumes:
        #    - /data/database/oracle/oracle11g1521/:/u01/app/oracle/
```

```
docker-compose -f docker_compose_oracle.yml up -d
```

> `不挂载目录的话`上面步骤就可以了，`挂载的话`接着走下面的

#### 5.2.1 挂载目录

> `创建文件夹`并`拷贝挂载目录`

```
mkdir -p /data/database/oracle/oracle11g1521/
docker cp docker_compose_oracle_11g_1521:/u01/app/oracle/ /data/database/oracle/oracle11g1521/
```

> 关闭容器

```
docker-compose -f docker_compose_oracle.yml down
```

> 打开上面`yml`中关于`挂载的配置`

```
version: '3'
services:
    oracle11g1521:
        image: oracleinanutshell/oracle-xe-11g
        container_name: docker_compose_oracle_11g_1521
        privileged: true
        environment:
            - TZ=Asia/Shanghai
        restart: always
        ports:
            - "1521:1521"
        volumes:
            - /data/database/oracle/oracle11g1521/:/u01/app/oracle/
```

> `启动容器`

```
docker-compose -f docker_compose_oracle.yml up -d 
```

> 可以去目标目录`/data/database/oracle/oracle11g1521/`下看，容器的内容已经挂载出来了

> `初始连接信息如下`

```
Host:ip
Port:1521
SID:XE
User:system
Password:oracle
```

#### 5.2.1 oracle 12c

```
version: '3'
services:
    oracle12c1521:
        image: truevoly/oracle-12c
        container_name: docker_compose_oracle_12c_1521
        privileged: true
        environment:
            - TZ=Asia/Shanghai
        restart: always
        ports:
            - "1521:1521"
            - "8080:8080"
        #volumes:
        #    - /data/database/oracle/oracle12c1521/:/u01/app/oracle12c/
```

> `初始连接信息如下`

```
Host:ip
Port:1521
SID:XE
User:system
Password:oracle
```

> `注意这个URL`,注意这个里面的@//ip:port/SID

```
jdbc:oracle:thin:@//192.168.88.55:1521/XE
```

### 5.3 redis

## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
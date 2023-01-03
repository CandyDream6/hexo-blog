---
title: '踩坑之路(五) MySQL安装'
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(81).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(81).jpg
tags:
  - 大数据
  - 踩坑之路
category:
  - - 编程
    - 大数据
    
date: 2021-05-10 14:10:57
---

由于我需要把`MySQL`的数据通过`Canal``实时摄取`到`Kafka`，所以先在服务器上安装个MySQL吧。

这里我选择用`docker`的方式安装。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 安装Docker

> 创建初始化脚本 

```shell
mkdir "/lankr/script/docker"
vim /lankr/script/docker/init_docker.sh
```

> 输入以下内容

```shell
#!/bin/bash

# 如果以前安装过Docker，那么为了保证环境的一致性，还是先卸载了再安装比较好
echo "开始卸载旧docker"
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine

echo "开始安装docker"
yum -y install docker

echo "设置开机启动"
systemctl enable docker.service

echo "设置阿里云镜像加速"
echo '{
  "registry-mirrors": ["https://moln2pbd.mirror.aliyuncs.com"]
}' > /etc/docker/daemon.json

echo "重启daemon服务"
systemctl daemon-reload

echo "重启docker"
systemctl restart docker
```

> 执行初始化脚本

```shell
chmod +x /lankr/script/docker/init_docker.sh
/lankr/script/docker/init_docker.sh
```

## 2 安装MySQL

### 2.1 创建初始化脚本

```shell
vim /lankr/script/mysql/init_mysqlCanal.sh
```

> 输入以下内容

```shell
#!/bin/bash

echo "拉取mysql镜像"
docker pull mysql:8.0.17

echo "创建mysql容器并启动"
docker run -p 3306:3306 --name mysqlCanal \
-v /lankr/application/mysqlCanal/log:/var/log/mysql:rw \
-v /lankr/application/mysqlCanal/data:/var/lib/mysql:rw \
-v /lankr/application/mysqlCanal/conf:/etc/mysql:rw \
-v /lankr/application/mysqlCanal/mysql-files:/var/lib/mysql-files/:rw \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:8.0.17

echo "创建临时mysql容器，用于复制配置文件"
docker run -p 3307:3306 --name mysqlCanalTemporary -e MYSQL_ROOT_PASSWORD=root -d mysql:8.0.17

echo "拷贝临时容器的配置文件到宿主机"
docker cp mysqlCanalTemporary:/var/log/mysql/. /lankr/application/mysqlCanal/log/
docker cp mysqlCanalTemporary:/var/lib/mysql/. /lankr/application/mysqlCanal/data/
docker cp mysqlCanalTemporary:/etc/mysql/. /lankr/application/mysqlCanal/conf/
docker cp mysqlCanalTemporary:/var/lib/mysql-files/. /lankr/application/mysqlCanal/mysql-files/

echo "删除临时容器"
docker rm -f mysqlCanalTemporary

echo "设置容器开机启动"
docker update --restart=always mysqlCanal
```

### 2.2 执行初始化脚本

```shell
chmod +x /lankr/script/mysql/init_mysqlCanal.sh
/lankr/script/mysql/init_mysqlCanal.sh
```

### 2.3 配置密码过期及加密方式

```shell
# 进入容器
docker exec -it mysqlCanal /bin/bash

# 登录mysql
mysql -uroot -p 

# 修改密码过期时间以及加密方式并刷新权限
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root' PASSWORD EXPIRE NEVER;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
ALTER USER 'root'@'%' IDENTIFIED BY 'root' PASSWORD EXPIRE NEVER;
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
FLUSH PRIVILEGES;

# 退出mysql登录
exit;

# 退出mysql容器
exit;
```

### 3 踩坑

> 今天启动容器的时候，`死活就起不来`，容器起不来，没法查看容器日志，浪费了好长时间，最后发现是服务器的`SELINUX`没有关闭。虽然前面服务器设置了`SELINUX=disabled`，但是没有重启服务器，所以没有生效。那么就执行下下面命令

```shell
setenforce 0
```

> 这样就可以让SELINUX=disabled临时生效，不用重启服务器。

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
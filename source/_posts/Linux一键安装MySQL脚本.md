---
title: Linux一键安装MySQL脚本
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(102).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(102).jpg
tags:
  - 数据库
  - MySQL
category:
  - - 数据库
    - MySQL
date: 2021-11-22 10:08:20
---

经常需要本地虚拟机安装`MySQL服务`进行学习或者开发测试什么的。

今天记录一下自己写的`Linux一键安装MySQL的脚本`。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 安装Docker

> 因为我是用`Docker`安装的`MySQL`，所以第一步安装一下`Docker`

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

echo "docker安装成功，版本信息如下"
docker -v
```

## 2 安装MySQL

> 脚本需要`3个参数`
>> 1:`mysql镜像名称`
>
>> 2:`端口`
>
>> 3:`mysql密码`

```shell
sh install_mysql.sh mysql:8.0.17 3306 myPassword
```

```shell
#!/bin/bash

# 使用示例:  sh install_mysql.sh mysql:8.0.17 3306 myPassword

# 判断如果不是一个参数的话就报错
if [ "$#" != 3 ]; then
    echo "Script needs 3 param,that is mysql version and port and password,like sh install_mysql.sh mysql:8.0.17 3306 myPassword,please check."
    exit 1
fi

# 创建初始文件夹
base_dir="/usr/local/share/applications/$1_$2"
base_dir_replace=${base_dir//':'/'_'}
dirs=("log" "data" "conf" "mysql-files")
for dir in ${dirs[@]}
do
    currentDir="$base_dir_replace/$dir"
    if [ ! -d $currentDir ]; then
        echo "目录不存在，准备创建:$currentDir"
        mkdir -p $currentDir
        if [ -d $currentDir ]
        then
            echo "目录创建成功:$currentDir"
        else
            echo "目录创建失败:$currentDir"
        fi
    else
        echo "目录已存在，不创建:$currentDir"
    fi
    echo
done

# 开始安装mysql

echo "拉取$1镜像"
docker pull $1

my_mysql_contain_name=my_$1_$2
my_mysql_contain_name_repalce=${my_mysql_contain_name//':'/'_'}
echo "创建mysql容器并启动"
docker run -p $2:3306 --name $my_mysql_contain_name_repalce \
-v $base_dir_replace/log:/var/log/mysql:rw \
-v $base_dir_replace/data:/var/lib/mysql:rw \
-v $base_dir_replace/conf:/etc/mysql:rw \
-v $base_dir_replace/mysql-files:/var/lib/mysql-files/:rw \
-e MYSQL_ROOT_PASSWORD=$3 \
-d $1

echo "创建临时mysql容器，用于复制配置文件"
my_temporary_mysql_contain_name=myTemporary_$1_$2
my_temporary_mysql_contain_name_replace=${my_temporary_mysql_contain_name//':'/'_'}
docker run -p $[$2+100]:3306 --name $my_temporary_mysql_contain_name_replace -e MYSQL_ROOT_PASSWORD=root -d $1

echo "拷贝临时容器的配置文件到宿主机"
docker cp $my_temporary_mysql_contain_name_replace:/var/lib/mysql/. $base_dir_replace/data/
docker cp $my_temporary_mysql_contain_name_replace:/etc/mysql/. $base_dir_replace/conf/

echo "删除临时容器"
docker rm -f $my_temporary_mysql_contain_name_replace

echo "设置容器开机启动"
docker update --restart=always $my_mysql_contain_name_repalce
```

## 8 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

---
title: Python离线安装
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(111).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(111).jpg
tags:
  - 大数据
  - Python
category:
  - - 编程
    - Python
date: 2021-12-20 10:28:56
---

> 我们产品卖出去之后，需要在客户现场`做POC`，但是客户给的服务器`基础环境什么的都没有`。

> 所以用到的`Python需要离线安装`，这里记录一下。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 安装基础依赖

### 1.1 下载基础依赖

```shell
yum install -y --downloadonly --downloaddir=/home/downloadRpm zlib-devel bzip2-devel openssl-devel ncurses-devel  epel-release gcc gcc-c++ xz-devel readline-devel gdbm-devel sqlite-devel tk-devel db4-devel libpcap-devel libffi-devel
yum reinstall -y --downloadonly --downloaddir=/home/downloadRpm zlib-devel bzip2-devel openssl-devel ncurses-devel  epel-release gcc gcc-c++ xz-devel readline-devel gdbm-devel sqlite-devel tk-devel db4-devel libpcap-devel libffi-devel
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Python离线安装/1.png)

> 参数说明

```shell
--downloadonly：表明只是下载，不进行安装
--downloaddir ：指定下载位置
```

> `注意：如果该服务器已经安装了需要下载的软件包，那么使用 install下载就不行，可以使用reinstall下载`。所以建议先install一下，再reinstall一下。

### 1.2 安装基础依赖

```shell
cd /home/downloadRpm                #将依赖包copy到离线服务器，进入目录
rpm -Uvh ./*.rpm --nodeps --force   #安装所有rpm依赖包
```

## 2 安装python

### 2.1 下载Python

```shell
https://www.python.org/downloads/
```

### 2.2 解压编译安装

```shell
tar -zxvf Python-3.7.1.tgz	                  # 解压python3安装包
cd Python-3.7.1 	                          # 进入python3安装包目录
./configure --prefix=/usr/local/bin/python3   # 将python3安装在这个目录
make                                  # 编译：有的环境执行make && make install会报错，所以就分开来
make install	                      # 安装
```

### 2.3 创建软连接

```shell
ln -s /usr/local/bin/python3/bin/python3 /usr/bin/python3	 # 创建python3软连接
ln -s /usr/local/bin/python3/bin/pip3 /usr/bin/pip3	         # 创建pip3的软连接
```
## 3 更新pip

### 3.1 下载pip轮子

> 下载*.whl文件

```shell
https://pypi.org/project/pip/#files
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Python离线安装/2.png)

### 3.2 更新pip

```shell
python3 -m pip install --upgrade ./pip-21.3.1-py3-none-any.whl
```

## 4 安装第三方模块

### 4.1 下载第三方模块

> 语法

```shell
pip3 download xxx  -d /home/downloadPip3Package
pip3 download paramiko==2.4.2  -d /home/downloadPip3Package   # 指定版本
```

> 示例

```shell
pip3 download paramiko requests jpype1 fs gnupg wheel pycryptodome -d /home/downloadPip3Package 
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Python离线安装/3.png)
### 4.2 安装第三方模块

```shell
pip3 install /home/downloadPip3Package/*
```

### 4.3 查看已安装的模块

```shell
pip3 list
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Python离线安装/4.png)

## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
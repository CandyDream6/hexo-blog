---
title: '踩坑之路(二) Linux_JDK安装'
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(78).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(78).jpg
tags:
  - 大数据
  - 踩坑之路
category:
  - - 编程
    - 大数据
    
date: 2021-05-07 14:10:57
---

我们使用的绝大多数组件都是`依赖JDK`的。

因为`java语言生态的完善以及强大`，很多产品`底层都是使用java实现`，没用java实现的，也基本多多少少有用到java的类库。

所以服务器上配置好java环境是必不可少的。

`干干干！`

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 下载上传JDK安装包

https://kafka.apache.org/downloads

> `jdk-8u291-linux-x64.tar.gz`，把他上传到/lankr/software

## 2 解压配置JDK

> 创建脚本init_jdk.sh

```shell
touch /lankr/script/init/init_jdk.sh
chmod +x /lankr/script/init/init_jdk.sh
vim /lankr/script/init/init_jdk.sh
```

> 输入下面内容

```shell
#!/bin/bash

echo "开始解压JDK安装包"
tar -zxvf /lankr/software/jdk-8u291-linux-x64.tar.gz -C /lankr/application
echo "完成解压JDK安装包"

echo

# 配置环境变量(先备份)
echo "备份/etc/profile到/etc/profile_back"
cp -r /etc/profile /etc/profile_init_back

echo "判断java环境变量是否已经配置"
FIND_FILE="/etc/profile"
FIND_STR_JDK='JAVA_HOME=/lankr/application/jdk1.8.0_291 
CLASSPATH=.:$JAVA_HOME/lib 
PATH=$JAVA_HOME/bin:$PATH 
export JAVA_HOME CLASSPATH PATH'

if [ `grep -c "$FIND_STR_JDK" $FIND_FILE` -ne '0' ]; then
    echo "环境变量KAFKA_HOME已经配置，不再进行配置"
else
    echo "环境变量KAFKA_HOME还未配置，现在进行配置"
    echo '
# 配置JDK环境
JAVA_HOME=/lankr/application/jdk1.8.0_291 
CLASSPATH=.:$JAVA_HOME/lib 
PATH=$JAVA_HOME/bin:$PATH 
export JAVA_HOME CLASSPATH PATH' >> /etc/profile

fi
source /etc/profile
echo "完成配置JDK环境"

# 分发jdk到各个服务器
for i in {2..7}
do
    scp -r /lankr/application/jdk1.8.0_291 node$i:/lankr/application
    scp /etc/profile node$i:/etc
    scp /etc/profile_init_back node$i:/etc
    ssh node$i "source /etc/profile"
done
echo "完成把jdk往各个服务器发送"
```

> 执行脚本安装

```shell
/lankr/script/init/init_jdk.sh
```

## 3 做软连接

> 所有节点执行

```shell
mkdir -p /usr/java
ln -s /lankr/application/jdk1.8.0_291/ /usr/java/default

```

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
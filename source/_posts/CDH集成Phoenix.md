---
title: CDH集成Phoenix
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(87).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(87).jpg
tags:
  - 大数据
  - Phoenix
category:
  - - 编程
    - 大数据
  - - 编程
    - CDH
  - - 编程
    - Phoenix
date: 2021-08-03 16:33:50
---

最近刚开始操作`HBase`，HBase官方提供了`语言访问的API`，但是公司业务里有需要进行`执行SQL`的操作。

所以API的方式不能满足了，故采用了`嫁接Phoenix`的方式来进行操作。

这里记录一下`CDH安装自定义Parcel-Phoenix`的过程。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 下载Parcel包

> 由于`Cloudera官网`关闭了`Cloudera-Phoenix`的下载，所以这里只能自己从网上去找资源，我们CDH集群用的是6.3.2，所以采用跟下面这个包(是我们公司大佬给我发的)，它`支持CDH6.2及其以上版本`。

```text
链接：https://pan.baidu.com/s/1pLGklj0AUG84iDHMshIz6g 
提取码：6666 
--来自百度网盘超级会员V4的分享
```

## 2 本地Parcel包安装

### 2.1 安装并启动Apache http

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

### 2.2 上传Parcel包

#### 2.2.1 初始化文件夹

```
cd  /var/www/html
mkdir -p phoenix/PHOENIX-5.0.0-cdh6.2.0/parcels/
cd /var/www/html/phoenix/PHOENIX-5.0.0-cdh6.2.0/parcels
```

#### 2.2.2 上传包

> 将下面三个文件一起上传 `PHOENIX-5.0.0-cdh6.2.0.p0.1308267-el7.parcel`,`PHOENIX-5.0.0-cdh6.2.0.p0.1308267-el7.parcel.sha`,`manifest.json`

#### 2.2.3 检验是否成功

> 访问`http://你的服务器IP/phoenix/PHOENIX-5.0.0-cdh6.2.0/parcels/`，看是否如下。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CDH集成Phoenix/1.png)

## 3 CDH添加Phoenix

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CDH集成Phoenix/2.png)

> 弄好之后，应该如下图，出现了一个Phoenix Parcel的选项，后面按钮应该是`下载`->`分配`->`激活`，挨个点一下就可以。
>
> `注意`:下载的时候，有可能会出现没有权限的问题，我直接给这个parcel777了，然后就好了，没所谓。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CDH集成Phoenix/3.png)

## 4 HBase配置phoenix开启命名空间和二级索引

```shell
<property>
<name>phoenix.schema.isNamespaceMappingEnabled</name>
<value>true</value>
</property>
<property>
<name>hbase.regionserver.wal.codec</name>
<value>org.apache.hadoop.hbase.regionserver.wal.IndexedWALEditCodec</value>
</property>
<property>
<name>phoenix.schema.isNamespaceMappingEnabled</name>
<value>true</value>
</property>
```

> 上面最后一个配置是我后面加的，下面图里没截，后续我发现不开这个的话，`外部JDBC没法访问`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CDH集成Phoenix/4.png)

> 然后又发现配置了这个还是`没法通过JDBC访问`，心态小炸，请谷大哥出马后得出下面解决方案。
>> 需要把Hbase的配置文件`hbase-site.xml`复制到项目的`resource目录`下，就可以了
> 
>> hbase配置文件目录`/etc/hbase/conf/hbase-site.xml`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CDH集成Phoenix/7.png)

## 5 重启HBase

> 这时候会提示HBase有`过期的配置`，重启一下就好了。

## 6 审判时刻

```shell
cd /opt/cloudera/parcels/PHOENIX-5.0.0-cdh6.2.0.p0.1308267/bin/
./phoenix-sqlline 127.0.0.1:2181
```

> 下图这样表示`成功`！

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CDH集成Phoenix/5.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/CDH集成Phoenix/6.png)

## 7 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

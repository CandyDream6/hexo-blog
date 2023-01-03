---
title: JDBC连接工具
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(119).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(119).jpg
tags:
  - 大数据
  - JDBC
category:
  - - 编程
    - 数据库
  - - 编程
    - JDBC
date: 2022-05-20 18:10:17
---

因为客户环境的千变万化，有时候连一个`连接数据库的工具`都没有，有时候用的数据库直接没听过。
所以操作起来就一直比较困难，所以一直想写一个通用的`JDBC连接工具`，今天先写了一个用用。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 Github地址

https://github.com/xiaoma55/JDBC_TOOl

## 2 项目背景

- 个人经常需要在客户环境操作各种各样的数据库，而客户那边的话工具很少，有时候连个navicat都没有，而且不允许装，免安装版本的也不允许执行(提示禁止)。
- 客户使用的数据库千奇百怪，比如sybase等等，很多都没有听过。
- 所以想的是写一个连接支持JDBC的数据源的通用的工具。

## 3 功能支持

- 支持任意类型的支持JDBC数据源的测试连接。
- 支持任意类型的支持JDBC数据源的任意sql语句执行。
- 支持windows和linux平台。

## 4 目录说明

    ├── config                         # 配置文件：jdbc连接的driver，url，username，password
    │   ├── application.yml
    │   ├── application-mysql.yml
    │   ├── application-oracle.yml
    │   ├── application-sqlserver.yml
    │   ├── application-db2.yml
    │   ├── application-sybase.yml
    ├── lib                             # 驱动：jdbc连接所需要的驱动
    │   ├── jconn2.jar
    │   ├── mysql-connector-java-8.0.21.jar
    │   ├── ojdbc8-19.3.0.0.jar
    JDBC_TOOL_GOD_VERSION.jar

## 5 使用说明

1. 前往release页面下载JDBC_TOOL.zip
2. 修改config文件夹下数据库的相关信息。
3. lib目录下传入自己需要的驱动jar包。
4. 启动jar包

```
java -Dloader.path=lib -jar
```

## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
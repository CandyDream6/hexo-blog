---
title: Linux 四个查找命令
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(27).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(27).jpg
tags:
  - Cenos7
  - Linux
category:
  - - 编程
    - Linux
 
date: 2019-02-18 12:11:36
---

Linux中查找文件主要有四个指令：find，which，whereis，locate。今天稍微整理一下他们的用法。可能有理解错误的地方，以后慢慢成长，慢慢修改吧。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## find

> find - search for files in a directory hierarchy

> find指令将从指定目录向下递归地遍历其各个子目录，将满足条件的文件或者目录显示在终端

```
find [搜索范围] [选项] 

常用选项
    -name<查找方式> ：按照指定的文件名查找模式查找文件
    -user<用户名>   ：查找属于指定用户名的所有文件
    -size<文件大小> ：按照指定的文件大小查找文件
```

```
find . -amin -10 # 查找在系统中最后10分钟访问的文件
find . -atime -2 # 查找在系统中最后48小时访问的文件
find . -[empty()](http://www.xfcodes.com/php/hanshu/32808.htm) # 查找在系统中为空的文件或者文件夹
find . -group cat # 查找在系统中属于 groupcat的文件
find . -mmin -5 # 查找在系统中最后5分钟里修改过的文件
find . -mtime -1 #查找在系统中最后24小时里修改过的文件
find . -nouser #查找在系统中属于作废用户的文件
find . -user fred #查找在系统中属于FRED这个用户的文件

find /home -name hello.txt #根据名称查找/home目录下的hello.txt文件
find /opt -user nobody #查找/opt目录下，用户名为nobody的文件
find / -size +5M #查找整个linux系统大于5m的文件(+n 大于，-n 小于，n 等于)


find . -type f | xargs grep 'your_string'  #在当前目录搜索文件内容含有某字符串（大小写敏感）的文件
find . -type f -name '*.sh' | xargs grep 'your_string'  #在当前目录搜索文件内容含有某字符串（大小写敏感）的特定文件
find . -type f -name '*.sh' | xargs grep -i 'your_string'  在当前目录搜索文件内容含有某字符串（忽略大小写）的特定文件
```

## which

> which - shows the full path of (shell) commands.

> 用于查找可直接执行的命令,只能查找可执行文件

```
[root@node1 ~]# which mysql
/usr/bin/mysql
[root@node1 ~]# which pwd
/usr/bin/pwd
```


## whereis

> whereis - locate the binary, source, and manual page files for a command

> 用于查可直接执行的命令、源文件和man文件


```
[root@node1 ~]# whereis mysql
mysql: /usr/bin/mysql /usr/lib64/mysql /usr/include/mysql /usr/share/mysql /usr/share/man/man1/mysql.1.gz
[root@node1 /]# whereis pwd
pwd: /usr/bin/pwd /usr/share/man/man1/pwd.1.gz
```

## locate

>locate指令可以快速定位文件路径。<br/>
locate指令利用事先建立的系统中所有文件名称和路径的locate数据库实现快速定位给定的文件。<br/>
locate指令无需遍历整个文件系统，查询速度较快。<br/>
为了保证查询结果的准确度，管理员必须定期更新locate时刻。
updatedb

> 这个很少用应该

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
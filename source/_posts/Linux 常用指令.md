---
title: Linux 常用指令
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(30).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(30).jpg
tags:
  - Cenos7
  - Linux
category:
  - - 编程
    - Linux
 
date: 2019-04-03 21:22:12
---

最近用linux指令比较多，总结一些常用的放在这里，自己以后用的话就可以查看了。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 **man**
> 查看命令手册

```
man [命令]
例如：man mkdir
```

## 2 pwd
> 显示当前路径

## 3 cd
> 目录切换，change directory

```
cd ..    返回上一级
cd /     返回根目录
cd -     返回上一个操作的目录(这个还有点意思)
```

## 4 ls ll
> 展示目录

```
ls [选项] 目录
   -a 显示隐藏的文件/文件夹
   -l 等于ll，显示文件详细信息
   -h 人性化展示文件大小
   ls /root 查看root下的目录
```

## 5 tree

> 以树的形式展示目录结构

```
yum install tree
tree
```

## 6 touch
> 创建文件

```
touch 文件名称
touch 文件名称 文件名称  (创建多个文件)
```

## 7 mkdir
> 创建目录

```
mkdir [选项] 目录名称
常用选项
    -p 递归创建，就是如果目录名称是多级，比如/aa/bb/cc，加入没有aa，他就会创建。不写-p的话，就创建不了
```

## 8 mv
> 移动文件与目录或重命名

```
mv oldNameFile newNameFile 重命名
mv source dest 移动文件
```

## 9 cp
> 拷贝文件

```
cp [选项] sorce dest

常用选项
    -r 递归复制整个文件夹
    \cp 表示强制覆盖：如果某个文件或者文件夹里的文件在目标位置存在，那么系统会提示是否覆盖，使用\cp就强制复制，没有提示
```

## 9.1 scp
> 跨服务器拷贝文件

```
scp [选项] file_source file_target 

常用选项
    -r 递归复制整个文件夹
    \cp 表示强制覆盖：如果某个文件或者文件夹里的文件在目标位置存在，那么系统会提示是否覆盖，使用\cp就强制复制，没有提示
```

### 9.1.1 从本地复制到远程

```shell
scp -r local_folder remote_username@remote_ip:remote_folder 
scp -r local_folder remote_ip:remote_folder 

scp -r /home/music/ root@www.runoob.com:/home
scp -r /home/music/ www.runoob.com:/home
```

### 9.1.2 从远程复制到本地

```shell
scp root@www.runoob.com:/home/music /home/music/1.mp3 
scp -r www.runoob.com:/home/others/ /home/music/
```

## 10 rm
> 删除文件或文件夹

```
rm [选项] 要删除的文件或者文件夹

常用选项
    -r 递归删除整个文件夹
    -f 强制删除不提示
```

## 11 cat
> 查看文件内容

```
cat [选项] 要查看的文件

常用选项
    -n :显示行号
    | more :(cat -n 1.txt | more)分页显示，加在指令最后，回车下一行，空格下一页
```

## 12 more
> 查看文件内容

```
more [选项] 要查看的文件

和cat一样，只不过里面内置了滚屏换行等指令，和vim里的一样
```

## 13 less
> 查看文件内容

```
less [选项] 要查看的文件

和less不同的是，less是只加载当前内容，而more是一次性加载所有内容，对于大文件的查看这个就有用
```
## 14 echo
> 输出内容到控制台

```
echo [选项] [输出内容]

echo $PATH :输出当前环境变量的路径

echo $PWD  :当前路径
```

## 15 重定向 > >>
> 覆盖和追加

```
ls -l > 文件 ：列表的内容写入文件a.txt中(覆盖写)
ls -al >> 文件 ：列表的内容追加到文件aa.txt的末尾
cat 文件1 > 文件2 ：文件1的内容覆盖到文件2
echo "内容" >> 文件 ：“内容”追到到文件aaa.txt的末尾


文件不存在，则创建
```

## 16 head
> 显示文件的开头部分，默认显示10行

```
head 文件 ：查看文件头10行内容
head -n 文件 ：-5 查看文件头5行内容
```
## 17 tail
> 用于输出文件中尾部的内容，默认显示后10行内容

```
tail 文件 ：查看文件后10行内容
tail -n 文件 ：-5 查看文件后5行内容
tail -f 文件 ：实时追踪该文档的所有更新
tail -1000f  :动态查看后1000行内容，一般查看日志用
```
## 18 ln
> 软链接也叫符号链接，类似于windows里的快捷方式，主要存放了链接其他文件的路径

```
ln -s [原文件或目录] [软链接名] ：给原文件创建一个软链接
rm -rf [软链接名] ：删除软链接

当我们只用pwd指令查看目录时，仍然看到的是软链接所在目录
motherFucker特别注意：注意删除软链接的时候链接名后面不要加/，比如我上图软链接指向root，指向 rm -rf linkToRoot/ ,导致我的root下所有东西全部删掉了
```
![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing1.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing2.png)

## 19 find
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

## 20 which
> 查看可执行文件的位置

```
which pwd  #查找pwd命令的路径
which ls
which java
```

## 21 whereis
> 用于查可直接执行的命令、源文件和man文件

```
[root@node1 ~]# whereis mysql
mysql: /usr/bin/mysql /usr/lib64/mysql /usr/include/mysql /usr/share/mysql /usr/share/man/man1/mysql.1.gz
[root@node1 /]# whereis pwd
pwd: /usr/bin/pwd /usr/share/man/man1/pwd.1.gz
```

## 22 locate

> locate指令可以快速定位文件路径。<br><br>locate指令利用事先建立的系统中所有文件名称和路径的locate数据库实现快速定位给定的文件。<br><br>
locate指令无需遍历整个文件系统，查询速度较快。<br><br>
为了保证查询结果的准确度，管理员必须定期更新locate时刻。

```
updatedb
locate 搜索文件

由于locate指令基于数据库进行查询，所以第一次运行前，必须使用updatedb指令创建locate数据库
```
## 23 grep和管道 |
> grep过滤查找，管道符“|”表示将前一个命令的处理结果输出传递给后面的命令处理

```
grep [选项] 查找内容 源文件

常用选项
-n ：显示匹配行及行号
-i ：忽略字母大小写
-c : 输出结果有多少行
--color ：加在语句最后，可以高亮显示搜索的内容(xshell好像不用这个，默认就标亮了)
```
1. 请在hello.txt文件中，查找“yes”所在行，并且显示行号

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing3.png)

## 24 压缩和解压缩tar
> tar指令是打包指令，最后打包的文件是tar.gz的文件

```
tar [选项] XXX.tar.gz 打包的内容
tar -zcvf a.tar.gz a1.txt a2.txt ：压缩(痔疮危房)
tar -zxvf myhome.tar.gz -C /home/   ：解压(止血危房)  ，不写目标目录，默认当前目录

常用选项
    -z ：打包同时压缩
    -x ：解包.tar文件
    -v ：显示详细信息
    -f ：指定压缩后的文件名
    -c ：产生.tar打包文件
```
## 25 上传下载
```
yum install -y lrzsz

rz          ：上传
sz 文件路径 ：下载
```

## 26 hostname

> 查看主机名

```
hostname

hostnamectl
```

> 修改主机名(不用重启)

```
hostnamectl --static set-hostname xiaoMaZhuJiMing

# 可以用上面命令查看改好了没，也可以看下 /etc/hostname 这个文件，应该也改了
```

```
/etc/sysconfig/network
这个目录下如果配置了HOSTNAME，删掉这个配置吧
```


## 27 history
> 查看已经执行过的历史命令，也可以执行历史命令

```
history   ：显示历史所有指令
history 10：显示最近使用的10个指令
!198      :执行编号为198的指令
```

## 28 wget下载
```
yum install wget
wget 下载链接

wget http://download.redis.io/releases/redis-4.0.2.tar.gz
```
## 29 关机

```
reboot           : 重新启动
shutdown -h now : 马上关机(断电关机)
halt            : 立刻关机(centos) (不断电关机)
```

## 30 清屏

```
clear

ctrl l
```
## 31 时间日期
> 显示时间

```
date ：显示当前时间信息
date "+%Y %m %d %H %M %S" ：显示年月日时分秒，+号必须有，其他中间分隔符用什么都可以
```
> 设置时间

```
date -s 字符串时间 ：(date -s "1995-07-14 00:00:00")
```
> 显示日历

```
cal ：显示这个月的日历
cal 2020 ：显示2020年的日历
```

## 32 cut

> **cut -c N-M 文件名**          ：N-M表示段区间，比如 1- ， 2-4 ， -5等

> **cut -d "切割符" -f N,M 文件名**  :N,M表示第几段，索引从1开始，不是0

> 文件内容查看命令，cut命令可以从一个文本文件或者文本流中提取文本列

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing4.png)

练习

> 如有一个学生报表信息文件stu.txt，包含id、name、age、score.

```
id name age score
01 tom 18 78
02 jack 20 85
03 bill 16 90
04 mary 24 77
05 anna 21 85
```

> 1)使用-d和-f显示文件中的指定的列

```
#显示id列
[root@node1 shell]# cut -d " " -f 1 stu.txt 
id
01
02
03
04
05

#显示name和age列
[root@node1 shell]# cut -d " " -f 2,3 stu.txt
name age
tom 18
jack 20
bill 16
mary 24
anna 21


解释:
 -d “ ” ：用来指定文件字段之间的分隔符，如果文件的分隔符是制表符则不需要指定该参数
   -f  数字 ：用来指定哪一列
```

> 2)--complement 选项提取指定字段之外的列（打印除了第二列之外的列，就是反选）

```
[root@node1 shell]# cut -d " " -f 2 --complement stu.txt
id age score
01 18 78
02 20 85
03 16 90
04 24 77
05 21 85
```

> 3)指定字段的字符或者字节范围

```
cut命令可以将一串字符作为列来显示，字符字段的记法：
  N-：从第N个字节、字符、字段到结尾；
  N-M：从第N个字节、字符、字段到第M个（包括M在内）字节、字符、字段；
  -M：从第1个字节、字符、字段到第M个（包括M在内）字节、字符、字段。
```

```
#打印每行第1个到第3个字符：
[root@node1 shell]# cut -c 1-4 stu.txt
id n
01 t
02 j
03 b
04 m
05 a

#打印每行前2个字符：
[root@node1 shell]# cut -c -2 stu.txt
id
01
02
03
04
05

#打印从第5个字符开始到结尾
[root@node1 shell]# cut -c 5- stu.txt
ame age score
om 18 78
ack 20 85
ill 16 90
ary 24 77
nna 21 85
```

## 33 wc

> **wc -lwc 文件名**         &nbsp;&nbsp;&nbsp;:不写参数就是按lwc走

> 统计行数 单词数 字节数

在默认的情况下，wc将计算指定文件的行数、字数以及字节数。
命令使用格式为：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing5.png)

>  有个文件test_wc.txt,内容如下:

```
1	11
222 bbb
333 aaa bbb 
444 aaa bbb ccc
555 aaa bbb ccc ddd
666 aaa bbb ccc ddd eee
```

> 1)统计指定文件行数、字数、字节数

```
[root@node1 shell]# wc test_wc.txt 
 6 21 85 test_wc.txt
#01.txt文件: 行数为6, 单词数为21, 字节数为85
```

> 2)查看根目录下有多少个文件

```
root@node1 shell]# ls / | wc -w
24
```

## 34 awk

**awk  -F ':'   '{OFS="|"} /cc/ {print NF} /aaa|ddd/ {print NR ":" $0}' $2<$4&&$3="ccc" {print $2} 文件名**

**awk -F ':' 'BEGING{}{total=total+$2}END{print total}' 文件名**

> BEGIN{读取文件前的操作}{读取文件后的操作} END {操作完文件之后的操作}

> -F 指定文件的切割分隔符

> OFS 用来指定输出记录分隔符

> NF	一条记录的字段的数目<br/>
NR	已经读出的记录数，就是行号，从1开始

> $0 代表整个文本行；<br/>
$1 代表文本行中的第 1 个数据字段；<br/>
$2<代表文本行中的第 2 个数据字段；<br/>
$n 代表文本行中的第 n 个数据字段。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing6.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing7.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing8.png)


案例一：test_awk.txt

```
aaa 111 333
bbb 444 555
ccc 666 777 888
ddd 999 222 999
```
> 1. 打印test_awk.txt 某一段 或者 某几段

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing9.png)

> 2. 打印test_awk.txt 某一段 或者 某几段 重新制定结果的分隔分号

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing10.png)

案例二：test_awk2.txt

```
aaa:111:333
bbb:444:555
ccc:666:777:888
ddd:999:222:999:cccc
```

> 1. 打印出test_awk2.txt的第1段 

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing11.png)

> 2. awk正则匹配

> 2.1 匹配test_awk2.txt中包含cc的内容

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing12.png)

> 2.2  匹配test_awk2.txt中第1段包含cc的内

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing13.png)

> 2.3 匹配test_awk2.txt中第1段包含至少连续两个c的内容

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing14.png)

> 2.4 在test_awk2.txt中如果匹配到abc就打印第1,3段，如果匹配到ccc,就打印 第1,3,4段

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing15.png)

> 2.5 在test_awk2.txt中如果匹配到aaa或者ddd,就打印全部内容

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing16.png)

> 3 awk判断

```
aaa:111:333
bbb:444:555
ccc:666:777:888
ddd:999:222:999:cccc
```

> 3.1 在test_awk2.txt中如果第3段等于222就打印所有内容

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing17.png)

> 3.2 在test_awk2.txt中如果第3段等于333就打印第一段

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing18.png)

> 3.3 在test_awk2.txt中如果第3段大于等于300就打印第一段

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing19.png)

> 3.4 在test_awk2.txt中如果第5段不等于cccc就打印全部

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing20.png)

> 3.5 在test_awk2.txt中如果第1段等于ccc，并且第2段大于300就打印全部

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing21.png)

> 3.6 在test_awk2.txt中如果第1段等于ccc，并且第2段匹配666就打印全部

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing22.png)
 
> 3.7 在test_awk2.txt中如果第3段小于第4段就打印全部

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing23.png)

> 3.8 在test_awk2.txt中如果第2段等于第4段就打印全部

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing24.png)

> 4 两个关键字

> NF	一条记录的字段的数目<br/>
NR	已经读出的记录数，就是行号，从1开始


```
aaa:111:333
bbb:444:555
ccc:666:777:888
ddd:999:222:999:cccc
```

> 4.1 打印test_awk2.txt全部内容显示行号

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing25.png)

> 4.2 打印test_awk2.txt全部内容显示段数

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing26.png)

> 4.3 打印test_awk2.txt前2行，并显示行号 （用二种不同的方式实现）

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing27.png)

> 4.4 打印test_awk2.txt前2行，并且第1段匹配 aa或者eee，打印全部，打印行号 （用两种方式）

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing28.png)

> 4.5 从test_awk2.txt的前3行中匹配出第2段等于 666，并显示行号

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing29.png)

> 4.6 从test_awk2.txt前3行，把第1段内容替换为itheima，指定分隔符为|，显示行号

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing30.png)

> 5 分段求和

> 5.1 对test_awk2.txt中的第2段求和

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing31.png)

综合案例

> 1 对统计当前目录下(/export/data/shell), 所有文本文件的大小

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing32.png)

> 2 打印99乘法表

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing33.png)

> 3 求总成绩

```
Marry    2143 78 84 77
Jack     2321 66 78 45
Tom     2122 48 77 71
Mike     2537 87 97 95
Bob      2415 40 57 62
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/linuxZhiLing/linuxZhiLing34.png)

## 35 date

### 35.1 获取今天的日期

```
date            # 2021年 01月 07日 星期四 16:00:16 CST
date +%Y%m%d    # 20210107
date +%Y-%m-%d  # 2021-01-07
```

### 35.2 指定日期获取内容 `-d`或`--date=`

> 获取指定日期的年月日格式输出

```
date -d "1997-07-14" +%Y%m%d  # 19970714
```

> 获取指定日期的星期（周几）格式输出

```
date --date="1997-07-14" +%w  # 1
```

### 35.3 日期加减 `-d`或`--date==`

> 获取上周日期（day,month,year,hour）

```
date -d "-1 week" +%Y-%m-%d  # 2020-12-31
```

> 获取昨天日期

```
date -d '-1 day' "+%Y-%m-%d"      # 2021-01-06
date --date="-24 hour" +%Y-%m-%d  # 2021-01-06
```

### 35.4 日期格式化规则

```
%a 当前域的星期缩写 (Sun..Sat)
%A 当前域的星期全写 (Sunday..Saturday)
%b 当前域的月份缩写(Jan..Dec)
%B 当前域的月份全称 (January..December)
%d 两位的天 (01..31)
%D 短时间格式 (mm/dd/yy)
%e 短格式天 ( 1..31)
%F 文件时间格式 same as %Y-%m-%d
%h same as %b
%H 24小时制的小时 (00..23)
%I 12小时制的小时 (01..12)
%j 一年中的第几天 (001..366)
%k 短格式24小时制的小时 ( 0..23)
%m 双位月份 (01..12)
%M 双位分钟 (00..59)
%r 12小时制的时间表示（时:分:秒,双位） time, 12-hour (hh:mm:ss [AP]M)
%R 24小时制的时间表示 （时:分,双位）time, 24-hour (hh:mm)
%s 自基础时间 1970-01-01 00:00:00 到当前时刻的秒数(a GNU extension)
%T 24小时制时间表示(hh:mm:ss)
%u 数字表示的星期（从星期一开始 1-7）
%x 本地日期格式 (mm/dd/yy)
%X 本地时间格式 (%H:%M:%S)
%y 两位的年(00..99)
%Y 年 (1970…)
```

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
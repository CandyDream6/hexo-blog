---
title: Shell_入门2 条件|循环|数组|函数
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(29).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(29).jpg
tags:
  - Cenos7
  - Shell
category:
  - - 编程
    - Shell
 
date: 2019-03-04 19:46:11
---

老年人学习shell第二天，赶紧记笔记，赶紧的，时光不接催人老！

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 条件判断

> Shell提供了丰富的语句判断方式，包括数字，字符串和文件。

### 1.1 数字

![条件判断-数字](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/shellBasic2/shellBasic2_1.png)

### 1.2 字符串

![条件判断-字符串](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/shellBasic2/shellBasic2_2.png)

### 1.3 文件

![条件判断-文件](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/shellBasic2/shellBasic2_3.png)

```
#!/bin/bash
#判断字符串是否相等
if [ "ok" == "ok" ]
then
        echo "equal"
fi
#判断23是否大于22
if [ 23 -gt 22 ]
then
        echo "大于"
fi
#判断文件/home/aaa.txt是否存在
if [ -e /home/aaa.txt ]
then
        echo "存在"
fi


equal
大于
存在
```

```
#!/bin/bash
if [ $1 -ge 60 ]
then
        echo "及格了"
elif [ $1 -lt 60 ]
then
        echo "不及格"
else
        echo "搞个鸡儿"
fi
```

## 2 流程控制

### 2.1 for

![流程控制-for](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/shellBasic2/shellBasic2_4.png)

```
#!/bin/bash
for i in "$*"
do
        echo "the num is $i"
done
echo "================"
for i in "$@"
do
        echo "the num is $i"
done


the num is 1 2 3
================
the num is 1
the num is 2
the num is 3
```

```
for i in {1..100}
do
        echo $i
done
```

```
#!/bin/bash
SUM=0
for (( i=1;i<=100;i++))
do
        SUM=$[SUM+$i]
done
echo $SUM


5050
```

```
for file in $(ls /root)
do 
        echo $file  
done
```


### 2.2 while

![流程控制-while](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/shellBasic2/shellBasic2_5.png)

```
#!/bin/bash
SUM=0
i=1
while [ $i -le $1 ]
do
        SUM=$[$SUM+$i]
        i=$[$i+1]
done
echo $SUM


./testWhile.sh 100
5050
```

```
#!/bin/bash
i=1
sum=0
while [ $i -le $1 ]
do
  let sum=sum+$i
  let i++
done
echo $sum
```

### 2.3 case

![流程控制-case](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/shellBasic2/shellBasic2_6.png)

```
#!/bin/bash
case $1 in
"1")
echo "周一"
;;
"2")
echo "周二"
;;
*)
echo "other"
;;
esac
```

### 2.4 break、continue

> 没啥，就是纯粹的break和continue

## 3 数组

### 3.1 定义数组

```
array_name=(value1 ... value)
```

### 3.2 使用数组

#### 3.2.1 获取数组单个元素

```
${array_name[index]}
```

#### 3.2.2 获取数组所有元素

```
${my_array[*]}

${my_array[@]}
```

#### 3.2.3 获取数组长度

```
${#my_array[*]

${#my_array[@]}
```


### 3.3 遍历数组

#### 3.3.1 方式一

```
#!/bin/bash

my_arr=(AA BB CC)

for var in ${my_arr[*]}
do
  echo $var
done
```

#### 3.3.2 方式二

```
#!/bin/bash

my_arr=(AA BB CC)
my_arr_num=${#my_arr[*]}
for((i=0;i<my_arr_num;i++));
do
  echo "-----------------------------"
  echo ${my_arr[$i]}
done
```

## 4 函数

### 4.1 系统函数

![流程控制-case](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/shellBasic2/shellBasic2_7.png)

```
basename /home/aaa/test.txt
basename /home/aaa/test.txt .txt
dirname /home/aaa/test.txt


test.txt
test
/home/aaa
```

### 4.2 自定义函数

```
#!/bin/bash
function getSum(){
        SUM=$[$n1+$n2]
        echo "和是$SUM"
}
read -p "请输入第一个数n1" n1
read -p "请输入第二个数n2" n2
getSum $n1 $n2


请输入第一个数n1100
请输入第二个数n2200
和是300
```

## 5 串行与并行

### 5.1 并行

> shell脚本`默认是按顺序串行`执行的，使用`&`可以将一个命令放在`后台运行`，从而使shell脚本能够继续往后执行：

```
sleep 5 &
echo "done"
```

> 上面的脚本执行后会立即打印出"done"，sleep命令被扔给后台执行，不会阻塞脚本执行。

### 5.2 强制串行

> 如果想要在进入下个循环前，必须`等待上个后台命令执行完毕`，可以使用wait命令：

```
sleep 5 &
wait
echo "done"
```

> 这样，需要等待5s后才能在屏幕上看到"done"。

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
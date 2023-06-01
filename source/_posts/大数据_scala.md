---
title: Scala 一种简洁的高级语言
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(61).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(61).jpg
tags:
  - 大数据
  - Scala
category:
  - - 编程
    - 大数据
 
date: 2020-05-06 17:38:08
---

Scala将`面向对象`和`函数式编程`结合成一种简洁的高级语言。

Scala的`静态类型`有助于避免复杂应用程序中的错误，它的JVM和JavaScript运行时让你可以轻松地访问庞大的库生态系统来`构建高性能系统`。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 环境搭建

### 1.1 安装scala

下载地址：https://www.scala-lang.org/download/

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/1.png)

> 一步一步安装就好

### 1.2 创建scala项目

#### 1.2.1 配置Scala SDK环境

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/7.png)

#### 1.2.2 安装IDEA Scala插件

> 插件提供辅助功能，如`高亮`、`补全`、`格式化`、`重构`等

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/8.png)

#### 1.2.3 创建maven工程

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/2.png)

#### 1.2.4 引入Scala pom

https://mvnrepository.com/artifact/org.scala-lang/scala-library

> 选择自己的scala版本，我是2.13.4

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/3.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/4.png)

#### 1.2.5 创建Scala目录

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/5.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/6.png)

#### 1.2.6 编写代码测试

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/9.png)

## 2 变量

### 2.1 普通变量

```
val/var 变量标识:变量类型 = 初始值

var x:String = "Girl"
```
> `val`定义的是不可重新赋值的变量<br/>
> `var`定义的是可重新赋值的变量

### 2.2 类型推断

> 不指定类型，scala会自己识别

```
val name = "tom"
```

### 2.3 惰性赋值

> 当有一些变量保存的数据较大时，但是不需要马上加载到JVM内存。可以使用惰性赋值来提高效率。`只有val类型变量才行，而var不可以`。

```
lazy val 变量名 = 表达式
```

```
//下面的【对象.方法()】很耗时，比如计算全世界人有多少根头发。使用lazy的话有个好处，仅在有必要的时候（比如flag==true时）才计算。
val flag = false
lazy val result = 对象.方法()
if(flag==true){
    println(result)
}else{
    println("无需计算")
}
```

## 3 字符串

### 3.1 双引号

```
val/var 变量名 = “字符串”
```

```
var name = "小女孩"
println(name + name.length)
```

### 3.2 插值表达式

```
val/var 变量名 = s"${变量/表达式}字符串"
```

```
val name = "zhangsan"
val age = 30
val sex = "male"
val info = s"name=${name}, age=${age}, sex=${sex}"
println(info)
```

> `注意：`
>> 1.在定义字符串之前添加s
>> 2.在字符串中，可以使用${}来引用变量或者编写表达式

### 3.3 三引号

> 如果有大段的文本需要保存，就可以使用三引号来定义字符串

```
val sql = """select
      *
    from
      t_user
    where
      name = "zhangsan""""
      
println(sql)
```

## 4 数据类型与操作符

### 4.1 数据类型

基础类型 | 类型说明
---|---
`Byte` | 8位带符号整数
`Short` | 16位带符号整数
`Int` | 32位带符号整数
`Long` | 64位带符号整数
`Char` | 16位无符号Unicode字符
`String` | Char类型的序列（字符串）
`Float` | 32位单精度浮点数
`Double` | 64位双精度浮点数
`Boolean` | true或false

> `注意下,scala类型与Java的区别`
>> 1 scala中所有的类型都使用`大写字母开头`
>> 2 整形使用`Int而不是Integer`
>> 3 scala中定义变量可以不写类型，让scala编译器自动推断

### 4.2 运算符

类别 | 操作符
---|---
`算术运算符` | +、-、*、/  %
`关系运算符` | >、<、==、!=、>=、<=
`逻辑运算符` | &&、||、!
`位运算符` | &、&#124;&#124;、^、<<、>>

> `scala中没有 ++、--运算符`

> `与Java不一样，在scala中，可以直接使用==、!=进行比较，它们与equals方法表示一致。而比较两个对象的引用值，使用eq。（跟java相反。）`

```
val str1 = "abc"
val str2 = str1 + ""
println(str1 == str2)  //true 
println(str1.eq(str2)) //false
```

### 4.3 scala类型层次结构

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/10.png)

类型 | 说明
---|---
`Any` | 所有类型的父类，它有两个子类AnyRef与AnyVal
`AnyVal` | 所有数值类型的父类
`AnyRef` | 所有对象类型（引用类型）的父类
`Unit` | 表示空，Unit是AnyVal的子类，它只有一个的实例(),它类似于Java中的void，但scala要比Java更加面向对象
`Null` | Null是AnyRef的子类，也就是说它是所有引用类型的子类。它的实例是null 。可以将null赋值给任何对象类型
`Nothing` | 所有类型的子类<br />不能直接创建该类型实例，某个方法抛出异常时，返回的就是Nothing类型，因为Nothing是所有类的子类，那么它可以赋值为任何类型。我们程序员不使用Nothing，这个是系统自行维护的。。

```
以下代码是否有问题？

val b:Int = null

scala会解释报错：
Null类型并不能转换为Int类型，说明Null类型并不是Int类型的子类
```

## 5 条件表达式

### 5.1 有返回值的if

> 与Java不一样的是，
>> `在scala中，条件表达式也是有返回值的`
>> `在scala中，没有三元表达式，可以使用if表达式替代三元表达式`

```
val name = "Girl"
var result = if (name == "Girl") 1 else 0
println(result) //1
```

### 5.2 块表达式

> `scala中，使用{}表示一个块表达式`<br/>
> 和if表达式一样，`块表达式也是有值的`<br/>
> `值就是最后一行代码表达式的值`

```
val a = {
   println("1 + 1")
   1 + 1
}
println(a) // 2
```

## 6 循环

### 6.1 for

```
for(i <- 表达式/数组/集合) {
    // 表达式
}
```

#### 6.1.1 简单循环

```
//生成1-10的数字,使用for表达式遍历，打印每个数字
val nums = 1.to(10)
for(i <- nums) println(i)

//或者
for(i <- 1 to 10) println(i)
```

#### 6.1.2 嵌套循环

```
//使用for表达式打印3行，5列星星，每打印5个星星，换行
for (i <- 1 to 3; j <- 1 to 5) {
  print("*");
  if (j == 5)
    println("")
}
```

#### 6.1.3 守卫

```
//for表达式中，可以添加if判断语句，这个if判断就称之为守卫。我们可以使用守卫让for表达式更简洁。

for(i <- 表达式/数组/集合 if 表达式) {
  // 表达式
}
```

```
// 添加守卫，打印能够整除3的数字
for(i <- 1 to 10 if i % 3 == 0)
  println(i)
```

#### 6.1.4 for推导式

> 将来可以使用for推导式生成一个新的集合（一组数据）

> 在for循环体中，可以使用yield表达式构建出一个集合，我们把使用yield的for表达式称之为`推导式`

```
// for推导式：for表达式中以yield开始，该for表达式会构建出一个集合
// 生成一个10、20、30...100的集合
val v = for(i <- 1 to 10)
  yield i * 10
println(v)
```

### 6.2 while

> scala中while循环和Java中是一致的

```
//打印1-10的数字
var i = 1
while(i <= 10) {
  println(i)
  i = i+1
}
```

### 6.3 break和continue

> 在scala中，类似Java和C++的break/continue关键字被移除了

> 如果一定要使用break/continue，就需要使用scala.util.control包的Breaks类的breable和break方法。

#### 6.3.1 break

> 导入Breaks类import `scala.util.control.Breaks._`

> 使用breakable将for表达式包起来

> for表达式中需要退出循环的地方，添加break()方法调用

```
//使用for表达式打印1-10的数字，如果数字到达5，退出for表达式
import scala.util.control.Breaks._

breakable{
   for(i <- 1 to 10) {
     if(i >= 5) break()
     else println(i)
   }
}
```

#### 6.3.2 continue

> continue的实现与break类似，但有一点不同：

> 实现break是用breakable{}将整个for表达式包起来，而实现continue是在for表达式的循环体中将内容用breakable{}包含起来就可以了

```
//使用for表达式打印1-10的数字，如果数字到达5，退出for表达式
import scala.util.control.Breaks._

for(i <- 1 to 10 ) {
   breakable{
     if(i % 3 == 0)
       break()
     else
       println(i)
   }
}
```

## 7 方法

### 7.1 定义

```
def methodName (参数名:参数类型, 参数名:参数类型) : [return type] = {
    // 方法体：一系列的代码
}
```

> 参数列表的参数类型不能省略<br/>
> 返回值类型可以省略，由scala编译器自动推断<br/>
> 返回值可以不写return，默认就是{}块表达式的值

```
def add(a:Int, b:Int) = a + b
  println(add(1,2))
```

### 7.2 返回值类型推断

> scala定义方法可以省略返回值，由scala自动推断返回值类型。这样方法定义后更加简洁。

> `定义递归方法，不能省略返回值类型`

```
//定义递归方法（求阶乘）10 * 9 * 8 * 7 * 6 * ... * 1
def m2(x:Int) = {
  if(x<=1) 1
  else m2(x-1) * x
}

//会报错
recursive method m2 needs result type
else m2(x-1) * x
```

### 7.3 方法参数

#### 7.3.1 默认参数

```
// x，y带有默认值，调用可以不带参数
def add(x:Int = 1, y:Int = 2) = x + y
add()
```

#### 7.3.2 带名参数

```
def add(x:Int = 0, y:Int = 0) = x + y
add(x=1)
```

#### 7.3.3 变长参数

```
def add(num:Int*) = num.sum
add(1,2,3,4,5)
```

### 7.4 方法调用

#### 7.4.1 后缀调用法

```
// 对象名.方法名(参数)
Math.abs(-1)
```

#### 7.4.2 中缀调用法

```
// 对象名 方法名 参数
Math abs -1
```

#### 7.4.3 花括号调用法

```
Math.abs{
  // 表达式1
  // 表达式2
}
```

```
// 方法只有一个参数，才能使用花括号调用法
Math.abs{-10}
```

#### 7.4.4 无括号调用法

```
// 如果方法没有参数，可以省略方法名后面的括号
def m3()=println("hello")
m3
```

## 8 函数

### 8.1 定义函数

> 函数是一个对象（变量）<br/>
> 类似于方法，函数也有输入参数和返回值<br/>
> 函数定义不需要使用def定义<br/>
> 无需指定返回值类型

```
val 函数变量名 = (参数名:参数类型, 参数名:参数类型....) => {函数体}
```

```
// 定义一个两个数值相加的函数
val add = (x:Int, y:Int) => x + y
add(1,2)
```

### 8.2 方法和函数的区别

> 方法是隶属于类或者对象的，在运行时，它是加载到JVM的方法区中<br/>
> 可以将函数对象赋值给一个变量，在运行时，它是加载到JVM的堆内存中<br/>
> 函数是一个对象，继承自FunctionN，函数对象有apply，curried，toString，tupled这些方法。方法则没有<br/>
> 函数是对象，对象包含方法

```
// 方法无法赋值给变量
def add(x:Int,y:Int)=x+y
val a = add

//报错
missing argument list for method add
  Unapplied methods are only converted to functions when a function type is expected.
  You can make this conversion explicit by writing `add _` or `add(_,_)` instead of `add`.
val a = add
```

### 8.3 方法转换为函数

> 有时候需要将方法转换为函数，作为变量传递，就需要将方法转换为函数<br/>
> 使用 _ 即可将方法转换为函数

```
// 定义一个方法用来进行两个数相加，将该方法转换为一个函数，赋值给变量
def add(x:Int,y:Int)=x+y
val a = add _
println(a(1,2))
```

## 9 数组

### 9.1 定长数组

```
// 通过指定长度定义数组
val/var 变量名 = new Array[元素类型](数组长度)
// 用元素直接初始化数组
val/var 变量名 = Array(元素1, 元素2, 元素3...)
```

```
val a = new Array[Int](100)
println(a(0))

val b = Array("java", "scala", "python")
println(b.length)
```

### 9.2 变长数组

#### 9.2.1 定义变长数组

> 创建变长数组，需要提前导入ArrayBuffer类`import scala.collection.mutable.ArrayBuffer`

```
//创建空的ArrayBuffer变长数组
val/var a = ArrayBuffer[元素类型]()
//创建带有初始元素的ArrayBuffer
val/var a = ArrayBuffer(元素1，元素2，元素3....)
```

```
val a = ArrayBuffer[Int]()
val b = ArrayBuffer("hadoop", "storm", "spark")
```

#### 9.2.2 添加/修改/删除元素

```
val a = ArrayBuffer("hadoop", "spark", "flink")

//使用+=添加元素
a += "flume"

//使用-=删除元素
a -= "hadoop"

//使用++=追加一个数组到变长数组
a ++= Array("hive", "sqoop")
```

### 9.3 遍历数组

```
val a = Array(1,2,3,4,5)

//使用for遍历
for(i<-a) println(i)

//使用索引遍历
for(i <- 0 to a.length - 1) println(a(i))
for(i <- 0 until a.length) println(a(i))
```

### 9.4 数组常用算法

```
val a = Array(1,3,2,5,4)

//求和，最大，最小
a.sum
a.max
a.min

//sorted方法，可以对数组进行升序排序
a.sorted

//reverse方法，可以将数组进行反转，从而实现降序排序
a.sorted.reverse
```

## 10 元组

> 元组可以用来包含一组不同类型的值。例如：姓名，年龄，性别，出生年月。元组的元素是不可变的。

### 10.1 定义元组

```
val/var 元组 = (元素1, 元素2, 元素3....)

//下面箭头方式只支持元组只有两个元素时
val/var 元组 = 元素1->元素2
val/var 元组 = (元素1 ->元素2)
```

```
val a = "zhangsan" -> 20
val b = ("zhangsan" -> 20)
```

### 10.2 访问元组

> 使用_1、_2、_3....来访问元组中的元素，_1表示访问第一个元素，依次类推

```
val a = "zhangsan" -> "male"

// 获取第一个元素
a._1

// 获取第二个元素
a._2
```

## 11 列表

> `可以保存重复的值`

> `有先后顺序`

### 11.1 不可变列表

> 不可变列表就是列表的元素、长度都是不可变的。

```
//使用List(元素1, 元素2, 元素3, ...)来创建一个不可变列表，语法格式：
val/var 变量名 = List(元素1, 元素2, 元素3...)

//使用Nil创建一个不可变的空列表
val/var 变量名 = Nil

//使用::方法创建一个不可变列表
val/var 变量名 = 元素1 :: 元素2 :: Nil
```

> 使用::拼接方式来创建列表，必须在最后添加一个Nil

```
//创建一个不可变列表，存放以下几个元素（1,2,3,4）
val a = List(1,2,3,4)

//使用Nil创建一个不可变的空列表
val b = Nil

//使用::方法创建列表，包含-2、-1两个元素
val c = -2 :: -1 :: Nil
```

### 11.2 可变列表

> `可变列表就是列表的元素、长度都是可变的。`

> 要使用可变列表，先要导入`import scala.collection.mutable.ListBuffer`

> `可变集合都在mutable包中`

> `不可变集合都在immutable包中（默认导入）`

```
//使用ListBuffer[元素类型]()创建空的可变列表，语法结构：
val/var 变量名 = ListBuffer[Int]()

//使用ListBuffer(元素1, 元素2, 元素3...)创建可变列表，语法结构：
val/var 变量名 = ListBuffer(元素1，元素2，元素3...)
```

```
//创建空的整形可变列表
val a = ListBuffer[Int]()

//创建一个可变列表，包含以下元素：1,2,3,4
val b = ListBuffer(1,2,3,4)
```

### 11.3 可变列表操作

```
//获取元素（使用括号访问(索引值)）
//添加元素（+=）
//追加一个列表（++=）
//更改元素（使用括号获取元素，然后进行赋值）
//删除元素（-=）
//转换为List（toList）
//转换为Array（toArray）

// 创建不可变列表
val a = ListBuffer(1,2,3)
// 获取第一个元素
a(0)
// 追加一个元素
a += 4
// 追加一个列表
a ++= List(5,6,7)
// 删除元素
a -= 7
// 转换为不可变列表
a.toList
// 转换为数组
a.toArray
```

### 11.4 列表常用操作

```
val a = List(1,2,3,4)

// 1 判断列表是否为空
a.isEmpty    //false

// 2 拼接两个列表
val b = List(4,5,6)
a ++ b       //List(1, 2, 3, 4, 5, 6)

// 3 获取列表的首个元素和剩余部分
a.head      //1
a.tail      //List(2, 3)

// 4 反转列表
a.reverse   //List(3, 2, 1)

// 5 获取列表前缀和后缀
//使用take方法获取前缀（前三个元素）,使用drop方法获取后缀（除前三个以外的元素）
val a = List(1,2,3,4,5)
a.take(3)      //List(1, 2, 3)
a.drop(3)      //List(4, 5)

// 6 扁平化(压平)
// 扁平化表示将列表中的所有元素放到一个列表中。
// 有一个列表，列表中又包含三个列表，分别为：List(1,2)、List(3)、List(4,5)<br/>
// 使用flatten将这个列表转换为List(1,2,3,4,5)
val a = List(List(1,2), List(3), List(4,5))
a.flatten      //List(1, 2, 3, 4, 5)

// 7 拉链与拉开
// 拉链：使用zip将两个列表，组合成一个元素为元组的列表
// 拉开：将一个包含元组的列表，解开成包含两个列表的元组
val a = List("zhangsan", "lisi", "wangwu")
val b = List(19, 20, 21)
res1 = a.zip(b)      //List((zhangsan,19), (lisi,20), (wangwu,21))
res1.unzip           //(List(zhangsan, lisi, wangwu),List(19, 20, 21))

// 8 转换字符串
val a = List(1,2,3,4)
a.toString              //List(1, 2, 3, 4)

// 9 生成字符串
List(1,2,3,4)
a.mkString              //1234
a.mkString(":")         //1:2:3:4

// 10 并集
> union表示对两个列表取并集，不去重
> distinct，去除重复的元素
val a1 = List(1,2,3,4)
val a2 = List(3,4,5,6)
a1.union(a2)             //List(1, 2, 3, 4, 3, 4, 5, 6)
a1.union(a2).distinct    //List(1, 2, 3, 4, 5, 6)

// 11 交集
> intersect表示对两个列表取交集
val a1 = List(1,2,3,4)
val a2 = List(3,4,5,6)
a1.intersect(a2)         //List(3, 4)

// 12 差集
> diff表示对两个列表取差集，例如： a1.diff(a2)，表示获取a1在a2中不存在的元素
val a1 = List(1,2,3,4)
val a2 = List(3,4,5,6)
a1.diff(a2)             //List(1, 2)
```

## 12 Set

> 元素不重复

> 不保证插入顺序

### 12.1 不可变集

#### 12.1.1 定义

```
//创建一个空的不可变集，语法格式：
val/var 变量名 = Set[类型]()
//给定元素来创建一个不可变集，语法格式：
val/var 变量名 = Set(元素1, 元素2, 元素3...)
```

```
val a = Set[Int]()
val a = Set(1,1,3,2,4,8)  //Set(1, 2, 3, 8, 4)
```

#### 12.1.2 基本操作

```
// 创建集
val a = Set(1,1,2,3,4,5)

// 获取集的大小
a.size    //5

// 遍历集
for(i <- a) println(i)

// 删除一个元素
a - 1       //Set(5, 2, 3, 4)

// 拼接两个集
a ++ Set(6,7,8)    //Set(5, 1, 6, 2, 7, 3, 8, 4)

// 拼接集和列表
a ++ List(6,7,8,9)    //Set(5, 1, 6, 9, 2, 7, 3, 8, 4)
```

### 12.2 可变集

> 可变集合和不可变集的创建方式一致，只不过需要提前导入一个可变集类。

> 手动导入：`import scala.collection.mutable.Set`

```
val a = Set(1,2,3,4)

// 添加元素
a += 5     //Set(1, 5, 2, 3, 4)

// 删除元素
a -= 1      //Set(5, 2, 3, 4)
```

## 13 映射

### 13.1 不可变Map

```
val/var map = Map(键->值, 键->值, 键->值...)  // 推荐，可读性更好
val/var map = Map((键, 值), (键, 值), (键, 值), (键, 值)...)
```

```
val map = Map("zhangsan"->30, "lisi"->40)
val map = Map(("zhangsan", 30), ("lisi", 30))
```

### 13.2 可变Map

> 定义语法与不可变Map一致。但定义可变Map需要手动导入`import scala.collection.mutable.Map`

### 13.3 Map基本操作

```
val map = Map("zhangsan"->30, "lisi"->40)
// 获取zhagnsan的年龄
map("zhangsan")               //30

// 获取所有的学生姓名
map.keys                      //Set(zhangsan, lisi)

// 获取所有的学生年龄
map.values                    //Iterable(30, 40)

// 打印所有的学生姓名和年龄
for((x,y) <- map) println(s"$x $y")  //zhangsan 30 lisi 40

// 获取wangwu的年龄，如果wangwu不存在，则返回-1
map.getOrElse("wangwu", -1)   //-1

// 新增一个学生：wangwu, 35
map += "wangwu"->35
for((x,y) <- map) println(s"$x $y")

// 将lisi从可变映射中移除
map -= "lisi"
for((x,y) <- map) println(s"$x $y")
```

## 14 iterator迭代器

> 使用iterator方法可以从集合获取一个迭代器

> 迭代器的两个基本操作
>> `hasNext`——查询容器中是否有下一个元素
> 
>> `next`——返回迭代器的下一个元素，如果没有，抛出`NoSuchElementException`

> 每一个迭代器都是有状态的
>> 迭代完后保留在最后一个元素的位置
> 
>> 再次使用则抛出NoSuchElementException

> 可以使用while或者for来逐个返回元素

```
val a = List(1,2,3,4,5)

//使用迭代器遍历
val ite = a.iterator
while(ite.hasNext) {
  println(ite.next)
}

//使用for遍历
for(i <- a) println(i)
```

## 15 函数式编程

```
val a = List(1,2,3,4)

// 1 遍历
a.foreach((x:Int) => println(x))

//遍历(省略参数类型)
a.foreach(x => println(x))

//遍历(使用下划线简化：当函数参数，只在函数体中出现一次，而且函数体没有嵌套调用时，可以使用下划线来简化函数定义)
a.foreach(println(_))

// 2 map
val b = a.map(x=>x+1)        //List(2, 3, 4, 5) ,这时候a依然是List(1, 2, 3, 4)
val c = a.map(_ + 1)         //List(2, 3, 4, 5) ,这时候a依然是List(1, 2, 3, 4)

// 3 flatMap
val d = List("hadoop hive spark flink flume", "kudu hbase sqoop storm")
println(d.flatMap(_.split(" ")))  //List(hadoop, hive, spark, flink, flume, kudu, hbase, sqoop, storm)
println(d)                               //List(hadoop hive spark flink flume, kudu hbase sqoop storm)

// 4 filter
val e = a.filter(_ % 2 == 0)            //List(2, 4) ,这时候a依然是List(1, 2, 3, 4)

// 5 sorted
List(3,1,2,9,7).sorted                  //List(1, 2, 3, 7, 9)

// 6 sortBy
val f = List("01 hadoop", "02 flume", "03 hive", "04 spark")
f.sortBy(_.split(" ")(1))       //List(02 flume, 01 hadoop, 03 hive, 04 spark) ,这时候f依然是List("01 hadoop", "02 flume", "03 hive", "04 spark")

// 7 自定义排序
val g = List(2,3,1,6,4,5)
var h = g.sortWith((x,y) => if(x<y)true else false)  //List(1, 2, 3, 4, 5, 6)
h.reverse                                            //List(6, 5, 4, 3, 2, 1)，h依然是List(1, 2, 3, 4, 5, 6)

// 8 groupBy
val j = List("张三"->"男", "李四"->"女", "王五"->"男")
// 按照性别分组
val k = j.groupBy(_._2)                   //HashMap(男 -> List((张三,男), (王五,男)), 女 -> List((李四,女)))
// 将分组后的映射转换为性别/人数元组列表
println(k.map(x => x._1 -> x._2.size))    //HashMap(男 -> 2, 女 -> 1)

// 9 reduce:传入一个函数进行聚合计算
val m = List(1,2,3,4,5,6,7,8,9,10)
m.reduce((x,y) => x + y)                              //55
// 第一个下划线表示第一个参数，就是历史的聚合数据结果
// 第二个下划线表示第二个参数，就是当前要聚合的数据元素
m.reduce(_ + _)                                       //55
// 与reduce一样，从左往右计算
m.reduceLeft(_ + _)                                   //55
// 从右往左聚合计算
m.reduceRight(_ + _)                                  //55

// 10 fold:折叠
//fold与reduce很像，但是多了一个指定初始值参数
val n = List(1,2,3,4,5,6,7,8,9,10)
n.fold(100)(_ + _)                                   //155
```

## 16 类和对象

```
object test {
  // 创建类
  class Person{}
  def main(args: Array[String]): Unit = {
     // 创建对象
     val p = new Person()
     println(p)      //com.ma.test$Person@763d9750
  }
}
```

> 如果类是空的，没有任何成员，可以省略{}<br/>
> 如果构造器的参数为空，可以省略()

```
object test {
  // 创建类，省略花括号
  class Person
  def main(args: Array[String]): Unit = {
     // 创建对象，省略括号
     val p = new Person
     println(p)      //com.ma.test$Person@763d9750
  }
}
```

## 17 成员变量

```
object test {
  class Person {
    // 定义成员变量
   var name = ""
   var age = 0
  }
 def main(args: Array[String]): Unit = {
   // 创建Person对象
   val person = new Person
   person.name = "zhangsan"
   person.age = 20
   // 获取变量值
   println(person.name)
   println(person.age)
  }
}
```

> `使用下划线初始化成员变量`
> 在定义var类型的成员变量时，可以使用_来初始化成员变量

```
String => null
Int => 0
Boolean => false
Double => 0.0
```

> val类型的成员变量，必须要自己手动初始化

```
object test {
  class Person{
     // 使用下划线进行初始化
     var name:String = _
     var age:Int = _
  }
  def main(args: Array[String]): Unit = {
     val person = new Person
     println(person.name)     //null
     println(person.age)      //0
  }
}
```

## 18 成员方法

```
object test {
  class Customer {
     var name:String = _
     var sex:String = _
     // 定义成员方法
     def sayHi(msg:String) = {
        println(msg+":"+name+":"+sex)
        }
  }
  def main(args: Array[String]): Unit = {
     val customer = new Customer
     customer.name = "张三"
     customer.sex = "男"
     customer.sayHi("你好")   //你好:张三:男
  }
}
```

## 19 访问修饰符

> Java中的访问控制，同样适用于scala，可以在成员前面添加`private/protected`关键字来`控制成员的可见性`。

> 但在scala中，`没有public`关键字，`任何没有被标为private或protected的成员都是公共的`

```
class Person {
     // 定义私有成员变量
     private var name:String = _
     private var age:Int = _
     def getName() = name
     def setName(name:String) = this.name = name
     def getAge() = age
     def setAge(age:Int) = this.age = age
     // 定义私有成员方法
     private def getNameAndAge = {
        name -> age
     }
    def getNameAndAgePublic() ={
      getNameAndAge
    }
  }
  def main(args: Array[String]): Unit = {
    val person = new Person
    person.setName("张三")
    person.setAge(10)
    println(person.getName())              //张三
    println(person.getAge())               //10
    println(person.getNameAndAgePublic())  //(张三,10)
  }
}
```

## 20 构造器

> `主构造器`
>> 主构造器的参数列表是直接定义在类名后面，添加了val/var表示直接通过主构造器定义成员变量
> 
>> 构造器参数列表可以指定默认值
> 
>> 创建实例，调用构造器可以指定字段进行初始化
> 
>> 整个class中除了字段定义和方法定义的代码都是构造代码

```
object test {
  // 定义类的主构造器
  // 指定默认值
  class Person(var name:String = "", var age:Int = 0) {
     println("调用主构造器")
  }
  def main(args: Array[String]): Unit = {
     // 给构造器传入参数
     val zhangsan = new Person("张三", 20)
     println(zhangsan.name)   //张三
     println(zhangsan.age)    //20

     // 不传入任何参数
     val empty = new Person
     println(empty.name)      //
     println(empty.age)       //0

     // 指定字段进行初始化
     val man = new Person(age = 40)
     println(man.name)       //
     println(man.age)        //40
  }
}
```

> `辅助构造器`
>> 我们把除了主构造器之外的构造器称为辅助构造器。
> 
>> 定义辅助构造器与定义方法一样，也使用def关键字来定义
> 
>> 这个方法的名字为this

```
def this(参数名:类型, 参数名:类型) {
    // 第一行需要调用主构造器或者其他构造器
    // 构造器代码
}
```

```
object test {
  class Customer(var name:String = "", var address:String = "") {
     // 定义辅助构造器
     def this(arr:Array[String]) = {
      // 辅助构造器必须要调用主构造器或者其他辅助构造器
      this(arr(0), arr(1))
      }
  }
  def main(args: Array[String]): Unit = {
     val zhangsan = new Customer(Array("张三", "北京"))
     println(zhangsan.name)         //张三
     println(zhangsan.address)      //北京
  }
}
```

## 21 单例对象

> scala中没有Java中的静态成员，我们想要定义类似于Java的static变量、static方法，就要使用到scala中的`单例对象——object`.

### 21.1 定义单例对象

> 单例对象表示全局仅有一个对象（类似于Java static概念）
>> 定义单例对象和定义类很像，就是把class换成object
> 
>> 在object中定义的成员变量类似于Java的静态变量
> 
>> 可以使用object直接引用成员变量

```
object test {
  // 定义一个单例对象
  object Dog {
     // 定义腿的数量
     val LEG_NUM = 4
  }
  def main(args: Array[String]): Unit = {
     println(Dog.LEG_NUM)   //4
  }
}
```

### 21.2 定义成员方法

> 在object中定义的成员方法类似于Java的静态方法

```
object test {
  object PrintUtil {
     // 打印分割线
     def printSpliter() = {
        // 字符串乘法，表示返回多少个字符串
        println("-" * 10)
        }
  }
  def main(args: Array[String]): Unit = {
     PrintUtil.printSpliter()   //----------
  }
}
```

### 21.3 时间工具类案例

```
object test {
  object DateUtils {
     // 在object中定义的成员变量，相当于Java中定义一个静态变量
     // 定义一个SimpleDateFormat日期时间格式化对象
     val simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm")
     // 相当于Java中定义一个静态方法
     def format(date: Date) = simpleDateFormat.format(date)
  }
  // main是一个静态方法，所以必须要写在object中
  def main(args: Array[String]): Unit = {
     println(DateUtils.format(new Date()))
  }
}
```

## 22 main方法

> cala和Java一样，如果要运行一个程序，必须有一个main方法。而在Java中main方法是静态的，而在scala中没有静态方法。在scala中，这个main方法必须放在一个单例对象中。

```
object test {
  def main(args:Array[String]) = {
     println("hello, girl")
  }
}
```

> 创建一个object，继承自`App Trait`（特质），然后将需要编写在main方法中的代码，写在object的构造方法体内。

```
object test extends App {
  println("hello，girl")
}
```

## 23 伴生对象

> 在Java中，经常会有一些类，`同时有实例成员又有静态成员`。例如：
> 在scala中，要实现类似的效果，可以使用`伴生对象`来实现。
> 我们还可以`使用伴生对象来实现快速创建对象`，例如：

```
// 无需使用new就可以快速来创建对象
val a = Array(1,2,3)
val b = Set(1,2,3)
```

### 23.1 定义伴生对象

> 一个`class和object具有同样的名字`。这个object称为`伴生对象`，这个class称为`伴生类`
>> `伴生对象必须要和伴生类一样的名字`
> 
>> `伴生对象和伴生类在同一个scala源文件中`
> 
>> `伴生对象和伴生类可以互相访问private属性`

```
object test {
  // 1编写一个英雄Hero类，
  class Hero {
    // 2有一个作战方法，打印  "我要用【方天画戟】作战了"
    def fight() = {
      println(s"我要用【${Hero.WEAPON}】作战了")
    }
  }
  // 3编写一个Hero伴生对象，定义一个私有变量，用于保存作战武器的名称
  object Hero{
    private val WEAPON = "方天画戟"
  }

  // 4main方法中创建Hero类的对象吕布，调用fight方法
  def main(args: Array[String]): Unit = {
    val lvbu = new Hero
    lvbu.fight
  }
}
```

### 23.2 private[this]访问权限

> 如果某个成员的权限设置为`private[this]`，表示`只能在当前类中访问`。`伴生对象也不可以访问 `

```
object test {
  class Person(private[this] var name:String)
  
  object Person {
     def printPerson(person:Person): Unit = {
        println(person.name)     //person.name会编译报错。但移除掉上面[this]就可以访问了
      }
  }
  
  def main(args: Array[String]): Unit = {
     val person = new Person("张三")
     Person.printPerson(person)
  }
}
```

### 23.3 apply方法

>  我们之前使用过这种方式来创建一个Array对象。

```
// 创建一个Array对象
val a = Array(1,2,3,4)
```

> 这种写法非常简便，不需要再写一个new，然后敲一个空格，再写类名。我们可以通过伴生对象的apply方法来实现。

```
object test {
  class Person(var name:String = "", var age:Int = 0)
  object Person {
     // 定义apply方法，接收两个参数
     def apply(name:String, age:Int) = new Person(name, age)
  }
  def main(args: Array[String]): Unit = {
     // 使用伴生对象名称来创建对象
     val zhangsan = Person("张三", 20)
     println(zhangsan.name)
     println(zhangsan.age)
  }
}
```

## 24 继承

### 24.1 类继承

```
class Person {
  var name = "super"
  def getName = this.name
}
class Student extends Person
object test {
  def main(args: Array[String]): Unit = {
     val p1 = new Person()
     val p2 = new Student()
     p2.name = "张三"
     println(p2.getName)
  }
}
```

### 24.2 单例对象继承

```
class Person {
  var name = "super"
  def getName = this.name
}
object Student extends Person
object test {
  def main(args: Array[String]): Unit = {
     println(Student.getName)
  }
}
```

### 24.3 override和super

> 类似于Java语言，我们在`子类中使用override可以来来重写父类的成员`，可以`使用super来引用父类`
>> `子类要覆盖父类中的一个方法，必须要使用override关键字`
> 
>> `使用override来重写一个val字段`
> 
>> `使用super关键字来访问父类的成员方法`

```
class Person {
  val name = "super"
  def getName = name
}
class Student extends Person {
  // 重写val字段
  override val name: String = "child"
  // 重写getName方法
  override def getName: String = "hello, " + super.getName
}
object test {
  def main(args: Array[String]): Unit = {
     println(new Student().getName)  //hello, child
  }
}
```

## 25 类型判断

### 25.1 isInstanceOf/asInstanceOf

> 在Java中，我们可以使用instanceof关键字来判断类型、以及(类型)object【比如（Map）new HashMap将HashMap转换为Map】来进行强制类型转换，在scala中如何实现呢？

> scala中对象提供`isInstanceOf`和`asInstanceOf`方法。
>> `isInstanceOf判断对象是否为指定类的对象`
>> `asInstanceOf将对象转换为指定类型`

```
class Fuck
class Person
class Student extends Person
object test {
  def main(args: Array[String]): Unit = {
    val s1:Person = new Student
    // 判断s1是否为Student类型
    s1.isInstanceOf[Student]           //true
    s1.isInstanceOf[Person]            //true
    s1.isInstanceOf[Fuck]              //false
    if(s1.isInstanceOf[Student]) {     //true
       // 将s1转换为Student类型
       val s2 = s1.asInstanceOf[Person]
    }
  }
}
```

### 25.2 getClass和classOf

> `isInstanceOf` 只能判断对象是否为指定类以及其子类的对象，而不能精确的判断出，对象就是指定类的对象。如果要求精确地判断出对象就是指定类的对象，那么就只能使用 `getClass` 和 `classOf` 。
>> `p.getClass可以精确获取对象的类型`
> 
>> `classOf[x]可以精确获取类型`
> 
>> `使用==操作符可以直接比较类型`

```
class Person
class Student extends Person
object Student{
  def main(args: Array[String]) {
    val p:Person=new Student
    
    //判断p是否为Person类的实例
    println(p.isInstanceOf[Person])//true
    
    //判断p的类型是否为Person类
    println(p.getClass == classOf[Person])//false
    
    //判断p的类型是否为Student类
    println(p.getClass == classOf[Student])//true
  }
}
```

## 26 抽象类

### 26.1 抽象方法

> 如果类的某个成员在当前类中的定义是不包含完整的，它就是一个`抽象类`
>> 不完整定义有两种情况：
>> `1.方法没有方法体（抽象方法）`
>> `2.变量没有初始化（抽象字段）`

> 定义抽象类和Java一样，在类前面加上`abstract`关键字

```
// 创建形状抽象类
abstract class Shape {
   def area:Double
}
// 创建正方形类
class Square(var edge:Double /*边长*/) extends Shape {
   // 实现父类计算面积的方法
   override def area: Double = edge * edge
}
// 创建长方形类
class Rectangle(var length:Double /*长*/, var width:Double /*宽*/) extends Shape {
   override def area: Double = length * width
}
// 创建圆形类
class Cirle(var radius:Double /*半径*/) extends Shape {
   override def area: Double = Math.PI * radius * radius
}
object test {
   def main(args: Array[String]): Unit = {
       val s1:Shape = new Square(2)
       val s2:Shape = new Rectangle(2,3)
       val s3:Shape = new Cirle(2)
       println(s1.area)  //4.0
       println(s2.area)  //6.0
       println(s3.area)  //12.566370614359172
  }
}
```

### 26.2 抽象字段

> 在scala中，也可以定义抽象的字段。如果一个成员变量是没有初始化，我们就认为它是抽象的。

```
// 定义一个人的抽象类
abstract class  Person {
   // 没有初始化的val字段就是抽象字段
   val WHO_AM_I:String
}
class Student extends  Person {
   override val WHO_AM_I: String = "学生"
}
class Policeman6 extends  Person {
   override val WHO_AM_I: String = "警察"
}
object Main6 {
   def main(args: Array[String]): Unit = {
       val p1 = new Student
       val p2 = new Policeman6
       println(p1.WHO_AM_I)   //学生
       println(p2.WHO_AM_I)   //警察
  }
}
```

## 27 匿名内部类

> `匿名内部类`是没有名称的子类，`直接用来创建实例对象`。Spark的源代码中有大量使用到匿名内部类。

> `scala中的匿名内部类使用与Java一致。`

```
abstract class  Person {
   def sayHello:Unit
}
object test {
   def main(args: Array[String]): Unit = {
       // 直接用new来创建一个匿名内部类对象
       val p1 = new  Person {
           override def sayHello: Unit = println("我是一个匿名内部类")
         }
       p1.sayHello   //我是一个匿名内部类
  }
}
```

## 28 特质(trait)

> scala中没有Java中的接口（interface），替代的概念是——特质

### 28.1 定义

> 特质是scala中代码复用的基础单元

> 它可以将方法和字段定义封装起来，然后添加到类中

> 与类继承不一样的是，类继承要求每个类都只能继承一个超类，而一个类可以添加任意数量的特质。

> 特质的定义和抽象类的定义很像，但它是使用trait关键字

```
定义特质
trait 名称 {
    // 抽象字段
    // 抽象方法
}
```

```
继承特质
class 类 extends 特质1 with 特质2 {
    // 字段实现
    // 方法实现
}
```

> `使用extends来继承trai`t（scala不论是类、还是抽象类、还是特质，都是使用extends关键字）

> `如果要继承多个trait，则使用with关键字`

### 28.2 trait作为接口使用

> 示例一：继承单个trait

```
/**
  1.创建一个Logger特质，添加一个接受一个String类型参数的log抽象方法
  2.创建一个ConsoleLogger类，继承Logger特质，实现log方法，打印消息
  3.添加main方法，创建ConsoleLogger对象，调用log方法
*/

object test{
  trait Logger {
       // 抽象方法
       def log(message:String)
  }
  class ConsoleLogger extends Logger {
     override def log(message: String): Unit = println("控制台日志:" + message)
  }
  def main(args: Array[String]): Unit = {
     val logger = new ConsoleLogger
     logger.log("这是一条日志")       //控制台日志:这是一条日志
  }
}
```

> 示例二：继承多个trait

```
/**
  1.创建一个MessageSender特质，添加send方法
  2.创建一个MessageReceiver特质，添加receive方法
  3.创建一个MessageWorker实现这两个特质
  4.在main中调用，分别调用send方法、receive方法
*/

object test{
  trait MessageSender {
       def send(msg:String)
  }
  trait MessageReceive {
       def receive():String
  }
  class MessageWorker extends MessageSender with MessageReceive {
       override def send(msg: String): Unit = println(s"发送消息:${msg}")
       override def receive(): String = "你好！我叫一个好人！"
  }
  def main(args: Array[String]): Unit = {
       val worker = new MessageWorker
       worker.send("hello")    //发送消息:hello
       println(worker.receive())     //你好！我叫一个好人！
  }
}
```

> 示例三：object继承trait

```
/**
  1.创建一个Logger特质，添加一个log抽象方法
  2.创建一个ConsoleLogger的object，继承Logger特质，实现log方法，打印消息
  3.编写main方法，调用ConsoleLogger的log方法
*/

object test{
  trait Logger {
       def log(message:String)
  }
  object ConsoleLogger extends Logger {
       override def log(message: String): Unit = println("控制台消息:" + message)
  }
  def main(args: Array[String]): Unit = {
       ConsoleLogger.log("程序退出!")
  }
}
```

### 28.3 定义具体的方法

> 和类一样，trait中还可以定义具体的方法

```
/**
  1.定义一个Logger特质，添加具体的log方法
  2.定义一个UserService类，继承Logger特质
        添加add方法，在add方法中调用log方法，打印"添加用户"
  3.添加main方法
        创建UserService对象实例
        调用add方法
*/

trait LoggerDetail {
  // 在trait中定义具体方法
  def log(msg:String) = println(msg)
}
class UserService extends LoggerDetail {
  def add() = log("添加用户")
}
object MethodInTrait {
  def main(args: Array[String]): Unit = {
    val userService = new UserService
    userService.add()  //添加用户
  }
}
```

### 28.4 定义具体的字段和抽象的字段

> 在trait中既可以定义抽象字段，又可以定义具体字段

> 继承trait的子类自动拥有trait中定义的字段

> 字段直接被添加到子类中

```
/**
  1.创建Logger特质
      定义一个具体的SimpleDateFormat字段，用来格式化日期（显示到时间）
      定义一个抽象字段TYPE，用于表示日志的级别
      创建一个log抽象方法，用于输出日志
  2.创建ConsoleLogger类，重写TYPE抽象字段（和log方法）
  3.添加main方法
      创建ConsoleLogger类对象
      调用log方法 
*/

object test {
  //1.创建Logger特质
  //定义一个具体的SimpleDateFormat字段，用来格式化日期（显示到时间）
  //定义一个抽象字段TYPE，用于表示日志的级别
  //创建一个抽象方法log，用于输出完整日志
  trait Logger{
    val sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
    val TYPE :String
    def log(msg:String)
  }

  //2.创建ConsoleLogger类，重写TYPE抽象字段和log方法
  class ConsoleLogger extends Logger{
    override val TYPE: String = "info"
    override def log(msg: String): Unit = {
      val info = s"${TYPE} ${sdf.format(new Date)}:"+msg
      println(info)
    }

  }
  //3.添加main方法
  //创建ConsoleLogger类对象
  //调用log方法
  def main(args: Array[String]): Unit = {
    val consoleLogger = new ConsoleLogger
    consoleLogger.log("用户A登录了系统。")   //info 2021-02-14 17:09:51:用户A登录了系统。
  }
}
```

### 28.5 实现模板模式

```
/**
  使用trait实现模板模式
    示例说明
      编写一个日志输出工具，分别有info、warn、error三个级别的日志输出
      日志输出的方式要求设计为可扩展的，例如：可以输出到控制台、将来也可以扩展输出到文件、数据库等
    实现步骤
    1.添加一个Logger特质
        添加一个log抽象方法
        添加三个info、warn、error具体方法，他们都调用log抽象方法。就像信用卡提前透支，此处提前使用log方法，log()的具体内容留给以后再去实现。
    2.创建ConsoleLogger类，继承Logger特质，实现log方法，打印入参即可
    3.添加main方法
        创建ConsoleLogger类对象
        分别调用info、warn、error方法输出日志
 */
 
object test {
  // 1.  添加一个Logger特质
  trait Logger {
    //   添加一个log抽象方法
    def log(msg: String)
    //   添加三个info、warn、error具体方法，他们都调用log抽象方法。就像信用卡提前透支，此处提前使用log方法，log()的具体内容留给以后再去实现。
    def info(msg:String)=log("INFO:"+msg)
    def warn(msg:String)=log("WARN:"+msg)
    def error(msg:String)=log("ERROR:"+msg)
  }

  // 2.  创建ConsoleLogger类，继承Logger特质，实现log方法，打印入参即可
  class ConsoleLogger extends Logger{
    override def log(msg: String): Unit = println(msg)
  }
  // 3.  添加main方法
  def main(args: Array[String]): Unit = {
    //  创建ConsoleLogger类对象
    val c = new ConsoleLogger
    //  分别调用info、warn、error方法输出日志
    c.info("信息日志")  //INFO:信息日志
    c.warn("警告日志")  //WARN:警告日志
    c.error("错误日志") //ERROR:错误日志
  }
}
```

### 28.6 对象混入trait

> scala中可以将trait混入到对象中，就是将trait中定义的方法、字段添加到一个对象中

```
/**
  1.创建一个Logger特质
      添加一个log实现方法，打印参数
  2.定义一个UserService类
  3.添加main方法
      创建UserService对象，混入Logger特质
      调用log方法
 */
 
object test{
  trait Logger {
     def log(msg:String) = println(msg)
  }
   class UserService
   def main(args: Array[String]): Unit = {
     val service = new UserService with Logger
     service.log("混入的方法")  //混入的方法
  }
}
```

### 28.7 责任链模式

> 我们如果要开发一个支付功能，往往需要执行一系列的验证才能完成支付。例如：
>> 1.进行支付签名校验
> 
>> 2.数据合法性校验
> 
>> 3....

> 如果将来因为第三方接口支付的调整，需要增加更多的校验规则，此时如何不修改之前的校验代码，来实现扩展呢？

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/11.png)

### 28.8 调用链

> 类继承了多个trait后，可以依次调用多个trait中的同一个方法，只要让多个trait中的同一个方法在最后都依次执行super关键字即可。类中调用多个tait中都有这个方法时，首先会从最右边的trait方法开始执行，然后依次往左执行，形成一个调用链条。

> 实现一个模拟支付过程的调用链

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/12.png)

```
object test{
  trait HandlerTrait {
     def handle(data:String) = println("处理数据...")
  }

  trait DataValidHanlderTrait extends HandlerTrait {
    override def handle(data:String): Unit = {
      println("验证数据...")
      super.handle(data)
    }
  }

  trait SignatureValidHandlerTrait extends HandlerTrait {
    override def handle(data: String): Unit = {
      println("校验签名...")
      super.handle(data)
    }
  }

  class PayService extends DataValidHanlderTrait with SignatureValidHandlerTrait {
    override def handle(data: String): Unit = {
      println("准备支付...")
      super.handle(data)
    }
  }

  def main(args: Array[String]): Unit = {
    val service = new PayService
    service.handle("支付参数")

    //准备支付...
    //校验签名...
    //验证数据...
    //处理数据...
  }
}
```

### 28.9 构造机制

> 如果一个类继承了多个trait，那这些trait是如何构造的呢？

> 定义
>> trait也有构造代码，但和类不一样，特质不能有构造器参数
> 
>> 每个特质只有一个无参数的构造器。
> 
>> 一个类继承另一个父类A、以及多个trait，当创建该类的实例时，它的构造顺序如下：
>> 
>>> 1.执行父类A的构造器
>> 
>>> 2.从左到右依次执行trait的构造器
>>  
>>> 3.如果trait有父trait，先构造父trait，如果多个trait有同样的父trait，则只初始化一次
>>  
>>> 4.执行子类构造器

```
object test{
  trait Logger {
       println("执行Logger构造器")
  }

  trait MyLogger extends Logger {
       println("执行MyLogger构造器")
  }

  trait TimeLogger extends Logger {
       println("执行TimeLogger构造器")
  }

  class Person{
       println("执行Person构造器")
  }

  class Student extends Person with TimeLogger with MyLogger {
       println("执行Student构造器")
  }

  def main(args: Array[String]): Unit = {
       new Student
       //执行Person构造器
       //执行Logger构造器
       //执行TimeLogger构造器
       //执行MyLogger构造器
       //执行Student构造器
  }
}

```

### 28.10 继承class

> trait也可以继承class的。特质会将class中的成员都继承下来。

```
object test{
  class MyUtil {
       def printMsg(msg:String) = println(msg)
  }

  trait Logger extends MyUtil {
       def log(msg:String) = printMsg("Logger:" + msg)
  }

  class Person extends Logger {
       def sayHello() = log("你好")
  }

  def main(args: Array[String]): Unit = {
       val person = new Person
       person.sayHello()    //Logger:你好
  }
}
```

## 29 样例类

> 样例类是一种特殊类，它可以用来快速定义一个用于保存数据的类（类似于Java POJO类），并发编程和spark、flink这些框架也都会经常使用它。

### 29.1 定义样例类

> `语法格式`

```
case class 样例类名([var/val] 成员变量名1:类型1, 成员变量名2:类型2, 成员变量名3:类型3)
```

> `在声明样例类时，下面的过程自动发生了`：
>> 构造器的每个参数都默认成为val修饰（val可以省略），除非显式被声明为var，但是并不推荐这么做；
> 
>> 自动在伴生对象中提供了apply方法，所以后面我们可以不使用new关键字就可构建对象；
> 
>> 自动提供了unapply方法，让我们可以通过模式匹配来获取类属性，是Scala中抽取器的实现和模式匹配的关键方法
> 
>> 自动生成了toString、equals、hashCode和copy方法，除非我们另外再显示给出这些方法的定义。

```
object test{
  case class Person(name:String, age:Int)
   def main(args: Array[String]): Unit = {
       val zhangsan = Person("张三", 20)
       println(zhangsan)      //Person(张三,20)
   }
}
```

### 29.2 可变成员变量

```
object test{
  case class Person(var name:String, var age:Int)
   def main(args: Array[String]): Unit = {
       val zhangsan = Person("张三", 20)
       zhangsan.age = 23
       println(zhangsan)    //Person(张三,23)
  }
}
```

### 29.3 样例类方法

```
case class CasePerson(name:String, age:Int)
object test {
   def main(args: Array[String]): Unit = {

     //toString：toString返回样例类名称(成员变量1, 成员变量2, 成员变量3....)，我们可以更方面查看样例类的成员
     val lisi = CasePerson("李四", 21)
     println(lisi.toString)      //CasePerson(李四,21)

     //equals：样例类自动实现了equals方法，可以直接使用==比较两个样例类是否相等，即所有的成员变量是否相等
     val lisi1 = CasePerson("李四", 21)
     val lisi2 = CasePerson("李四", 21)
     println(lisi1 == lisi2)       //true
     println(lisi1.eq(lisi2))      //false
     println(lisi1.equals(lisi2))  //true

     //hashCode：样例类自动实现了hashCode方法，如果所有成员变量的值相同，则hash值相同，只要有一个不一样，则hash值不一样。
     println(lisi1.hashCode())     //1473348919
     println(lisi2.hashCode())     //1473348919

     //copy：样例类实现了copy方法，可以快速创建一个相同的实例对象，可以使用带名参数指定给成员进行重新赋值
     val wangwu = lisi1.copy(name="王五")
     println(wangwu)               //CasePerson(王五,21)
  }
}
```

## 30 样例对象

> 它主要用在两个地方：
>> `1.定义枚举`
>> `2.作为没有任何参数的消息传递（后面Akka编程会讲到）`

> 使用case object可以创建样例对象。样例对象是单例的，而且它没有主构造器

```
case object 样例对象名
```

```
trait Sex /*定义一个性别特质*/
case object Male extends Sex        // 定义一个样例对象并实现了Sex特质
case object Female extends Sex
case class Person(name:String, sex:Sex)
object test {
   def main(args: Array[String]): Unit = {
       val zhangsan = Person("张三", Male)
       println(zhangsan)       //Person(张三,Male)
  }
}
```

## 31 模式匹配:match

> `scala用match替代java中的switch`

> java只支持相等值的模式匹配，而scala支持更多场景的模式匹配

> scala中有一个非常强大的模式匹配机制，可以应用在很多场景：
>> `switch语句`
> 
>> `类型查询`
> 
>> `使用模式匹配快速获取数据`

### 31.1 简单模式匹配

```
变量 match {
    case "常量1" => 表达式1
    case "常量2" => 表达式2
    case "常量3" => 表达式3
    case _ => 表达式4      // 默认配
}
```

```
object test {
  def main(args: Array[String]): Unit = {
    print("请输出一个词：")
    // StdIn.readLine表示从控制台读取一行文本
    val name = StdIn.readLine()
    val result = name match {
       case "hadoop" => "大数据分布式存储和计算框架"
       case "zookeeper" => "大数据分布式协调服务框架"
       case "spark" => "大数据分布式内存计算框架"
       case _ => "未匹配"
    }
    println(result)
  }
}
```

### 31.2 匹配类型

```
变量 match {
    case 类型1变量名: 类型1 => 表达式1
    case 类型2变量名: 类型2 => 表达式2
    case 类型3变量名: 类型3 => 表达式3
    ...
    case _ => 表达式4
}
```

```
object test {
  def main(args: Array[String]): Unit = {
    val a:Any = "hadoop"
    val result = a match {
       case _:String => "String"
       case _:Int => "Int"
       case _:Double => "Double"
    }
    println(result)
  }
}
```

### 31.3 守卫

```
java

int a = 0;
switch(a) {
    case 0: a += 1;
    case 1: a += 1;
    case 2: a += 1;
    case 3: a += 1;
    case 4: a += 2;
    case 5: a += 2;
    case 6: a += 2;
    case 7: a += 2;
    default: a = 0;
}
```

```
scala

object test {
  def main(args: Array[String]): Unit = {
    print("请输入一个整数：")
    val a = StdIn.readInt()
    a match {
       case _ if a >= 0 && a <= 3 => println("[0-3]")
       case _ if a >= 4 && a <= 8 => println("[4-8]")
       case _ => println("未匹配")
    }
  }
}
```

### 31.4 匹配样例类

> scala可以使用模式匹配来`匹配样例类`，从而可以`快速获取样例类中的成员数据`。在开发Akka案例时，会用到。

```
object test {
  // 1. 创建两个样例类
  case class Person(name:String, age:Int)
  case class Order(id:String)
  def main(args: Array[String]): Unit = {
     // 2. 创建样例类对象，并赋值为Any类型
     val zhangsan:Any = Person("张三", 20)
     val order1:Any = Order("001")
     // 3. 使用match...case表达式来进行模式匹配
     // 获取样例类中成员变量
     order1 match {
         case Person(name, age) => println(s"姓名：${name} 年龄：${age}")
         case Order(id1) => println(s"ID为：${id1}")
         case _ => println("未匹配")
     }
  }
}
```

### 31.5 匹配集合

> scala中的模式匹配，还能用来匹配集合。

### 31.6 匹配数组

> 依次修改代码定义以下三个数组

```
Array(1,x,y)   // 以1开头，后续的两个元素不固定
Array(0)       // 只匹配一个0元素的元素
Array(0, ...)  // 可以任意数量，但是以0开头
```

```
object test {
  def main(args: Array[String]): Unit = {
    val arr = Array(1, 3, 5)
    arr match {
       case Array(1, x, y) => println(x + " " + y)
       case Array(0) => println("only 0")
       case Array(0, _*) => println("0 ...")
       case _ => println("something else")
    }
  }
}
```

### 31.7 匹配列表

> 依次修改代码定义以下三个列表

```
List(0)             // 只保存0一个元素的列表
List(0,...)         // 以0开头的列表，数量不固定
List(x,y)           // 只包含两个元素的列表
```

```
object test {
  def main(args: Array[String]): Unit = {
    val list = List(0, 1, 2)
    list match {
       case 0 :: Nil => println("只有0的列表")
       case 0 :: tail => println("0开头的列表")
       case x :: y :: Nil => println(s"只有另两个元素${x}, ${y}的列表")
       case _ => println("未匹配")
    }
  }
}
```

### 31.8 匹配元组

> 依次修改代码定义以下两个元组

```
(1, x, y)       // 以1开头的、一共三个元素的元组
(x, y, 5)       // 一共有三个元素，最后一个元素为5的元组
```

```
object test {
  def main(args: Array[String]): Unit = {
    val tuple = (2, 2, 5)
    tuple match {
       case (1, x, y) => println(s"三个元素，1开头的元组：1, ${x}, ${y}")
       case (x, y, 5) => println(s"三个元素，5结尾的元组：${x}, ${y}, 5")
       case _ => println("未匹配")
    }
  }
}
```

### 31.9 变量声明中的模式匹配

> 在定义变量的时候，可以使用模式匹配快速获取数据

```
/**
    获取数组中的元素:生成包含0-10数字的数组，使用模式匹配分别获取第二个、第三个、第四个元素
 */
object test {
  def main(args: Array[String]): Unit = {
    val arr = (1 to 10).toArray
    //思路一
    val Array(a,b,c,d,e,f,g,h,i,j)=arr
    //思路二
    val Array(_, x, y, z, _*) = arr
    println(b,c,d)
    println(x, y, z)
  }
}
```

```
/**
    获取List中的数据:生成包含0-10数字的列表，使用模式匹配分别获取第一个、第二个元素，其他元素
*/
object test {
  def main(args: Array[String]): Unit = {
    val list = (1 to 10).toList
    val x :: y :: tail = list
    println(x, y,tail)  //(1,2,List(3, 4, 5, 6, 7, 8, 9, 10))
  }
}
```

## 32 Option类型

> 使用Option类型，可以用来有效避免空引用(null)异常。也就是说，将来我们返回某些数据时，可以返回一个Option类型来替代。

> scala中，Option类型来表示可选值。这种类型的数据有两种形式：
>> `Some(x)`：表示实际的值
>> `None`：表示没有值
>> 使用`getOrElse`方法，当值为None是可以指定一个默认值

```
/**
    示例一：
    定义一个两个数相除的方法，使用Option类型来封装结果
    然后使用模式匹配来打印结果
        不是除零，打印结果
        除零打印异常错误
 */
object test {
  def dvi(a:Double, b:Double):Option[Double] = {
    if(b != 0) {
      Some(a / b)
    }
    else {
      None
    }
  }
  def main(args: Array[String]): Unit = {
    val result1 = dvi(1.0, 5)
    result1 match {
     case Some(x) => println(x)
     case None => println("除零异常")
    }
  }
}
```

```
/**
  重写上述案例，使用getOrElse方法，当除零时，或者默认值为0
 */
object test {
  def dvi(a:Double, b:Double) = {
    if(b != 0) {
      Some(a / b)
     }
    else {
      None
     }
  }
  def main(args: Array[String]): Unit = {
    val result = dvi(1, 0).getOrElse(0)
    println(result)
  }
}
```

## 33 偏函数

> 偏函数可以提供了简洁的语法，可以简化函数的定义。配合集合的函数式编程，可以让代码更加优雅。

> 被包在花括号内没有match的一组case语句是一个偏函数

> 偏函数是PartialFunction[A, B]的一个实例
>> A代表输入参数类型
> 
>> B代表返回结果类型

```
/**
  案例一
 */
object test {
  def main(args: Array[String]): Unit = {
    // func1是一个输入参数为Int类型，返回值为String类型的偏函数
    val func1: PartialFunction[Int, String] = {
         case 1 => "一"
         case 2 => "二"
         case 3 => "三"
         case _ => "其他"
    }
    println(func1(2))
  }
}
```

```
/**
  案例二
      定义一个列表，包含1-10的数字
      请将1-3的数字都转换为[1-3]
      请将4-8的数字都转换为[4-8]
      将其他的数字转换为(9-*]
 */
object test {
  def main(args: Array[String]): Unit = {
    val list = (1 to 10).toList
    val list2 = list.map({
       case x if x >= 1 && x <= 3 => "[1-3]"
       case x if x >= 4 && x <= 8 => "[4-8]"
       case x if x > 8 => "(9-*]"
    })
    println(list2)  //List([1-3], [1-3], [1-3], [4-8], [4-8], [4-8], [4-8], [4-8], (9-*], (9-*])
  }
}
```

## 34 正则表达式

> Regex类
>> scala中提供了Regex类来定义正则表达式
>> 要构造一个RegEx对象，`直接使用String类的r方法即可`
>> `建议使用三个双引号来表示正则表达式，不然就得对正则中的反斜杠来进行转义`

```
val regEx = """正则表达式""".r
```
> `findAllMatchIn方法`
>> 使用findAllMatchIn方法可以获取到所有正则匹配到的字符串

```
/**
 * 示例一
  定义一个正则表达式，来匹配邮箱是否合法
  合法邮箱测试：qq12344@163.com
  不合法邮箱测试：qq12344@.com
 */
object test {
  def main(args: Array[String]): Unit = {
    val r:Regex = """.+@.+\..+""".r
    val eml1 = "qq12344@163.com"
    val eml2 = "qq12344@.com"
    if(r.findAllMatchIn(eml1).size > 0) {
         println(eml1 + "邮箱合法")
    }
    else {
         println(eml1 + "邮箱不合法")
    }
    if(r.findAllMatchIn(eml2).size > 0) {
         println(eml2 + "邮箱合法")
    }
    else {
         println(eml2 + "邮箱不合法")
    }
  }
}

//qq12344@163.com邮箱合法
qq12344@.com邮箱不合法
```

```
/**
 * 示例二
  找出以下列表中的所有不合法的邮箱
 */
object test {
  def main(args: Array[String]): Unit = {
    val emlList =
      List("38123845@qq.com", "a1da88123f@gmail.com", "zhansan@163.com", "123afadff.com")
    val regex = """.+@.+\..+""".r
    val invalidEmlList = emlList.filter {
         x =>
         if (regex.findAllMatchIn(x).size < 1) true else false
    }
    println(invalidEmlList)  //List(123afadff.com)
  }
}
```

```
/**
 * 示例三
  使用正则表达式进行模式匹配，匹配出来邮箱运营商的名字
 */
object test {
  def main(args: Array[String]): Unit = {
    // 使用括号表示一个分组
    val regex = """.+@(.+)\..+""".r
    val emlList =
      List("38123845@qq.com", "a1da88123f@gmail.com", "zhansan@163.com", "123afadff.com")
    //注意，下面的@跟邮箱没半毛钱关系。是固定语法需要。
    val emlCmpList = emlList.map {
         case x @ regex(company) => s"${x} => ${company}"
         case x => x + "=>未知"
    }
    println(emlCmpList)  //List(38123845@qq.com => qq, a1da88123f@gmail.com => gmail, zhansan@163.com => 163, 123afadff.com=>未知)
  }
}
```

## 35 异常处理

### 35.1 捕获异常

```
try {
    // 代码
}
catch {
    case ex:异常类型1 => // 代码
    case ex:异常类型2 => // 代码
}
finally {
    // 代码
}
```

```
object test {
  def main(args: Array[String]): Unit = {
    try {
         val i = 10 / 0
         println("你好！")
    } catch {
         case ex: Exception => println(ex.getMessage)
    }
  }
}
```

### 35.2 抛出异常

> scala不需要在方法上声明要抛出的异常，它已经解决了在Java中被认为是设计失败的检查型异常

```
java

public static void main(String[] args) throws Exception {
    throw new Exception("这是一个异常");
}
```

```
scala

object test {
  def main(args: Array[String]): Unit = {
    throw new Exception("这是一个异常")
  }
}
```

## 36 提取器(Extractor)

> 之前我们学习过了，实现一个类的伴生对象中的apply方法，可以用类名来快速构建一个对象。伴生对象中，还有一个unapply方法。与apply相反，unapply是将该类的对象，拆解为一个个的元素。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/13.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/14.png)

> 要实现一个类的提取器，只需要在该类的伴生对象中实现一个unapply方法即可。

```
override def unapply(stu:Student):Option[(类型1, 类型2, 类型3...)] = {
    if(stu != null) {
        Some((变量1, 变量2, 变量3...))
    }
    else {
        None
    }
}
```

```
/**
  创建一个Student类，包含姓名年龄两个字段
  定义半生对象，重写unapply方法，实现一个类的解构器，并使用match表达式进行模式匹配，提取类中的字段。。
 */
object test {
  class Student(var name:String, var age:Int)
  object Student {
     def apply(name:String, age:Int) = {
       new Student(name, age)
     }
     def unapply(student:Student) = {
       val tuple = (student.name, student.age)
       Some(tuple)
     }
  }
  def main(args: Array[String]): Unit = {
     val zhangsan = Student("张三", 20)
     zhangsan match {
       case Student(name, age) => println(s"${name} => ${age}")  //张三 => 20
     }
  }
}
```

## 37 泛型

### 37.1 泛型方法 

```
object test {
  def getMiddleElement[T](array: Array[T]) =
    array(array.length / 2)

  def main(args: Array[String]): Unit = {
    println(getMiddleElement(Array(1, 2, 3, 4, 5, 6)))         //4
    println(getMiddleElement(Array("a", "b", "c", "d", "e")))  //c
  }
}
```

### 37.2 泛型类

```
object test {
  case class Pair[T](var a:T, var b:T)
  def main(args: Array[String]): Unit = {
     val pairList = List(
         Pair("Hadoop", "Storm"),
         Pair("Hadoop", 2008),
         Pair(1.0, 2.0),
         Pair("Hadoop", Some(1.9))
     )
     println(pairList) //List(Pair(Hadoop,Storm), Pair(Hadoop,2008), Pair(1.0,2.0), Pair(Hadoop,Some(1.9)))
  }
}
```

### 37.3 上下界

> 我们在定义方法/类的泛型时，限定必须从哪个类继承、或者必须是哪个类的父类。此时，就需要使用到上下界。

#### 37.3.1 上界

> 使用<: 类型名表示给类型添加一个上界，表示泛型参数必须要从该类（或本身）继承

```
[T <: 类型]
```

```
object test {
  class Person
  class Student extends Person
  def demo[T <: Person](a:Array[T]) = println(a)
  def main(args: Array[String]): Unit = {
       demo(Array(new Person))    // [Lcom.ma.test$Person;@763d9750
       demo(Array(new Student))   // [Lcom.ma.test$Student;@5c0369c4
       //demo(Array("hadoop"))    // 编译出错，必须是Person的子类
  }
}
```

#### 37.3.2 下界

> 上界是要求必须是某个类的子类，或者必须从某个类继承，而下界是必须是某个类的父类（或本身）

```
孙子 extends 儿子  extends 哥哥 extends 老子 extends 爷爷
[T >: 孙子]

//下界写在前面，上界写在后面

[T >:儿子 T<:老子]
```

> 如果类既有上界、又有下界。下界写在前面，上界写在后面

```
object test {
  class Person
  class Policeman extends Person
  class Superman extends Policeman
  def demo[T >: Policeman](array:Array[T]) = println(array)
  def main(args: Array[String]): Unit = {
     demo(Array(new Person))      // [Lcom.ma.test$Person;@763d9750
     demo(Array(new Policeman))   // [Lcom.ma.test$Policeman;@5c0369c4
     demo(Array(new Superman))    // 编译出错：Superman是Policeman的子类
  }
}
```

### 37.4 协变、逆变、非变

> spark的源代码中大量使用到了协变、逆变、非变，学习该知识点对我们将来阅读spark源代码很有帮助。

> 来看一个类型转换的问题：

```
class Pair[T]
object Pair {
  def main(args: Array[String]): Unit = {
    val p1 = Pair("hello")
    val p2:Pair[AnyRef] = p1  // 编译报错，无法将p1转换为p2
    println(p2)
  }
}
```
> 如何让带有泛型的类支持类型转换呢？

#### 37.4.1 非变

```
class Pair[T]{}
```

> 默认泛型类是非变的

> 类型B是A的子类型，Pair[A]和Pair[B]没有任何从属关系

> Java是一样的

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/15.png)

#### 37.4.2 协变

```
class Pair[+T]
```
> 表示：如果已知类型B是A的子类型，那么Pair[B]也是Pair[A]的子类型

> 参数化类型的方向和类型的方向是一致的。

#### 37.4.3 逆变

```
class Pair[-T]
```

> 表示：如果已知类型B是A的子类型，Pair[A]反过来可以认为是Pair[B]的子类型

> 参数化类型的方向和类型的方向是相反的

```
object test {
  class Super
  class Sub extends Super
  class Temp1[T]
  class Temp2[+T]
  class Temp3[-T]
  def main(args: Array[String]): Unit = {
       val a:Temp1[Sub] = new Temp1[Sub]
       // 非变
       //val b:Temp1[Super] = a  // 编译报错
       // 协变
       val c: Temp2[Sub] = new Temp2[Sub]
       val d: Temp2[Super] = c
       // 逆变
       val e: Temp3[Super] = new Temp3[Super]
       val f: Temp3[Sub] = e
  }
```

## 38 高阶函数

> scala 混合了面向对象和函数式的特性，在函数式编程语言中，函数是“头等公民”，它和Int、String、Class等其他类型处于同等的地位，可以像其他类型的变量一样被传递和操作。

### 38.1 作为值的函数

> 在scala中，函数就像和数字、字符串一样，可以将函数传递给一个方法。我们可以对算法进行封装，然后将具体的动作传递给方法，这种特性很有用。

> 我们之前学习过List的map方法，它就可以接收一个函数，完成List的转换。

```
/**
    1.创建一个函数，用于将数字装换为指定个数的小星星
    2.创建一个列表，调用map方法，将上面的函数，传递给map方法。
    3.打印转换为的列表
*/
object test {
   def main(args: Array[String]): Unit = {
     val func: Int => String = (num:Int) => "*" * num
     println((1 to 10).map(func))  //Vector(*, **, ***, ****, *****, ******, *******, ********, *********, **********)
  }
}
```

### 38.2 匿名函数

> 上面的代码，给`(num:Int) => "*" * num`函数赋值给了一个变量，但是这种写法有一些啰嗦。在scala中，可以不需要给函数赋值给变量，没有赋值给变量的函数就是`匿名函数`

```
object test {
   def main(args: Array[String]): Unit = {
     val list = List(1, 2, 3, 4)
     // 字符串*方法，表示生成指定数量的字符串
     val func_num2star = (num:Int) => "*" * num
     print(list.map(func_num2star))
  }
}
```

> 使用匿名函数优化上述代码

```
object test {
   def main(args: Array[String]): Unit = {
     println((1 to 10).map(num => "*" * num))  //Vector(*, **, ***, ****, *****, ******, *******, ********, *********, **********)
     // 因为此处num变量只使用了一次，而且只是进行简单的计算，所以可以省略参数列表，使用_替代参数
     println((1 to 10).map("*" * _))           //Vector(*, **, ***, ****, *****, ******, *******, ********, *********, **********)
  }
}
```

### 38.3 柯里化

> 在scala和spark的源代码中，大量使用到了柯里化。为了后续方便阅读源代码，我们需要来了解下柯里化。

> 柯里化（Currying）是指将原先接受多个参数的方法转换为多个只有一个参数的参数列表的过程。

```
def method ( a: Int , b:String , c:Double …..)={}
def method(a: Int)( b:String)( c:Double) …={}
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/16.png)

> 柯里化过程解析

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_scala/17.png)

```
/**
    示例一：
    1.编写一个方法calc_method(参数1,参数2)，有一个括号参数列表，接收2个数字
    2.将上面的方法用柯里化方式再定义一次。定义接收2个括号参数列表的柯里化方法calc_method_curring(参数列表1#)(参数列表2#)，方法体和上面一样。
    3.将柯里化方法转换成函数func1(参数列表1#)(参数列表2#)
    4.在main方法中，给函数func1的第一个参数列表传入具体对象，第二个参数列表不动，从而将其转换成接收一个参数列表的函数func2(参数列表2#)。
    5.调用函数func2，进行计算，打印结果
    6.将3中func1的参数列表1#传入不同的值对象，得到不同的func2，再试一次。
*/
object test {
  def calc_carried(x:Double, y:Double)(func_calc:(Double, Double)=>Double) = {
     func_calc(x, y)
  }

  def main(args: Array[String]): Unit = {
     println(calc_carried(10.1, 10.2){
           (x,y) => x + y
       })                                        //20.299999999999997
     println(calc_carried(10, 10)(_ + _))        //20.0
     println(calc_carried(10.1, 10.2)(_ * _))    //103.02
     println(calc_carried(100.2, 10)(_ - _))     //90.2
  }
}

```

```
/**
    示例二：
    1.编写一个方法calc_method(参数1,参数2)，有一个括号参数列表，接收2个数字
    2.将上面的方法转换成接收2个括号参数列表的柯里化方法calc_method_curring(参数列表1#)(参数列表2#)，两个方法的方法体是一样的。
    3.将柯里化方法转换成函数func1(参数列表1#)(参数列表2#)
    4.在main方法中，给函数func1的第一个参数列表传入具体对象，第二个参数列表不动，从而将其转换成接收一个参数列表的函数func2(参数列表2#)。
    5.调用函数func2，进行计算，打印结果
    6.将3中func1的参数列表1#传入不同的值对象，得到不同的func2，再试一次。
*/
object test {
  //1  编写一个方法calc_method(参数1,参数2,参数3)，有一个括号参数列表,接收3个参数，第一个参数接收一个函数对象，其余二个接收2个数字
  def calc_method(fun_calc: (Int, Int) => Int , a: Int, b: Int) = fun_calc(a, b)

  //2  将上面的方法转换成接收2个括号参数列表的柯里化方法calc_method_curring(参数列表1#)(参数列表2#)，
  //   第一个列表接收上一步骤的参数1函数，第二个列表接收2个数字,方法体和上一步骤是一样的。
  def calc_method_curring(fun_calc: (Int, Int) => Int)(a: Int, b: Int) = fun_calc(a, b)

  //3  将柯里化方法转换成函数func1(参数列表1#)(参数列表2#)
  val func1 = calc_method_curring _

  def main(args: Array[String]): Unit = {
    //4  在main方法中，给函数func1的第一个参数列表传入具体函数对象，第二个参数列表不动，从而将其转换成接收一个参数列表的函数func2(参数列表2#)。
    //传入一个做加法的函数对象
    val func2_add = func1((x, y) => x + y)

    //5  调用函数func2，进行计算，打印结果
    println(func2_add(4, 7))

    //6  将4中func1的参数列表1#传入不同的函数对象，得到不同的func2，再试一次。
    //传入一个做减法的函数对象
    val func2_minus = func1((x, y) => x - y)
    println(func2_minus(7, 4))
  }
}
```

### 38.4 闭包

> 闭包其实就是一个函数，只不过这个函数的返回值依赖于声明在函数外部的变量。

> 可以简单认为，就是可以访问不在当前作用域范围的一个函数。

```
/**
    示例一：定义一个闭包：add函数就是一个闭包
*/
object test {
  def main(args: Array[String]): Unit = {
    val y=10
    val add=(x:Int)=>{
         x+y
    }
    println(add(5))     // 15
  }
}
```

```
/**
    示例二：柯里化就是一个闭包
*/
object test {
  def add(x:Int)(y:Int) = {
    x + y
  }
  //上述代码相当于
  def add1(x:Int) = {
    (y:Int) => x + y
  }
}
```

## 39 隐式转换和隐式参数

> 隐式转换和隐式参数是scala非常有特色的功能，也是Java等其他编程语言没有的功能。我们可以很方便地利用隐式转换来丰富现有类的功能。后面在编写Akka并发编程、Spark SQL、Flink都会看到隐式转换和隐式参数的身影。

### 39.1 定义

> 所谓`隐式转换`，是指以`implicit`关键字声明的带有单个参数的方法。它是自动被调用的，自动将某种类型转换为另外一种类型。

> 使用步骤
>> 1.在object中定义隐式转换方法（使用implicit）
>> 2.在需要用到隐式转换的地方，手动引入隐式转换（使用import）
>> 3.编译器自动调用隐式转化后的方法

```
/**
    1.在模块根目录下创建./data/1.txt，输入一些文本内容。
    2.定义一个类RichFile，主构造器中包含一个File类型变量，包含一个方法read，可以返回一个文件的文本内容
    3.定义RichFile类的伴生对象
    4.伴生对象中定义一个方法file2RichFile，返回RichFile的一个对象实例，这个方法用implicit修饰。
    5.main方法中，定义文件对象
    6.手动引入RichFile的file2RichFile方法
    在file上调用read()方法，打印结果
*/
object test {
  class RichFile(val file:File) {
     // 读取文件为字符串
     def read() = {
         Source.fromFile(file).mkString
     }
  }
  object RichFile {
     // 定义隐式转换方法
     implicit def file2RichFile(file:File) = new RichFile(file)
  }
  def main(args: Array[String]): Unit = {
     // 加载文件
     val file = new File("./data/1.txt")
     // 导入隐式转换
     import RichFile.file2RichFile
     // file对象具备有read方法
     println(file.read())
  }
}

```

### 39.2 隐式转换的时机

> 当对象调用类中不存在的方法或者成员时，编译器会自动将对象进行隐式转换

> 当方法中的参数的类型与目标类型不一致时

### 39.3 自动导入隐式转换方法

> 在scala中，如果在当前作用域中有隐式转换方法，会自动导入隐式转换。

```
/**
    将隐式转换方法定义在main所在的object中
*/
class RichFile(val f:File) {
  // 将文件中内容读取成字符串
  def read() = Source.fromFile(f).mkString
}
object ImplicitConvertDemo {
  // 定义隐式转换方法
  implicit def file2RichFile(f:File) = new RichFile(f)
  def main(args: Array[String]): Unit = {
    val f = new File("./data/1.txt")
    // 调用的其实是RichFile的read方法
    println(f.read())
  }
}
```

### 39.4 隐式参数

> 方法可以带有一个标记为implicit的参数列表。这种情况，编译器会查找缺省值，提供给该方法。
>> 1.隐式参数一般用在柯里化方法。
> 
>> 2.在方法后面添加一个参数列表，参数使用implicit修饰
> 
>> 3.在object中定义implicit修饰的隐式值
> 
>> 4.调用方法，可以不传入implicit修饰的参数列表，编译器会自动查找缺省值

> 1.和隐式转换一样，可以使用import手动导入隐式参数<br/>
> 2.如果在当前作用域定义了隐式值，会自动进行导入

```
/**
  隐式参数
  1 定义一个柯里化方法quote(String)(Tuple)，可将传入的值，使用一个分隔符前缀、后缀包括起来
  2 定义一个object，包含一个成员变量，用implicit修饰，表示隐式参数，也是步骤1的潜在的分隔符
  3 调用该方法，并打印测试
*/
object test {
  //1  定义一个柯里化方法quote(String)(Tuple)，可将传入的值，使用一个分隔符前缀、后缀包括起来
  def quote(str:String)( implicit delimiter:(String,String))=delimiter._1+str+delimiter._2
  //2 定义一个object，包含一个成员变量，用implicit修饰，表示隐式参数，也是步骤1的潜在的分隔符
  object ImplicitParam{
    implicit val DEFAULT_DELIMITERS=("<<<",">>>")
  }
  def main(args: Array[String]): Unit = {
    //3  调用该方法，并打印测试
    import ImplicitParam.DEFAULT_DELIMITERS
    println(quote("你好"))
  }
}

```

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

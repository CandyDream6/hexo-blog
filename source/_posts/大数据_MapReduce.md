---
title: MapReduce 分布式计算框架
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(39).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(39).jpg
tags:
  - Cenos7
  - 大数据
  - Hadoop
  - MapReduce
category:
  - - 编程
    - 大数据
 
date: 2019-08-18 18:57:13
---

MapReduce是一种编程模型，用于大规模数据集（大于1TB）的并行运算。

概念"Map（映射）"和"Reduce（归约）"，是它们的主要思想，都是从函数式编程语言里借来的，还有从矢量编程语言里借来的特性。

它极大地方便了编程人员在不会分布式并行编程的情况下，将自己的程序运行在分布式系统上。

当前的软件实现是指定一个Map（映射）函数，用来把一组键值对映射成一组新的键值对，指定并发的Reduce（归约）函数，用来保证所有映射的键值对中的每一个共享相同的键组。 

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 MapReduce基本概念

> 分布式计算框架，是hadoop的一部分。**分而治之**，比如`人口普查`

### 1.1 名词解释

#### 1.1.1 Map

> **Map负责“分”**，即把复杂的任务分解为若干个“简单的任务”来并行处理。可以进行拆分的前提是这些小任务可以并行计算，彼此间几乎没有依赖关系。

<details>
<summary>这个具有强依赖关系，就不行</summary>

```
- 第一个人拿走：前300个同学的成绩 进行求平均分 ，等到平均分(70+80+90...+92.5)/300 = 80.5
- 第二个人拿走：300-2000个同学成绩，等到(70+85+90...+95)/1701= 69
- 第三个人拿走：2000-5000个同学成绩，等到(73+81+94...+92)/3000= 74.5
- 最终合并局部结果： (80.5+69+74.5)/3 是真正的平均值吗 ？ 肯定不是...
```

</details>

#### 1.1.1 Reduce

> **Reduce负责“合”**，即对map阶段的结果进行全局汇总。

### 1.2 设计构思

#### 1.2.1 如何对付大数据处理

> 对相互间不具有计算依赖关系的大数据，实现并行最自然的办法就是采取分而治之的策略。并行计算的第一个重要问题是如何划分计算任务或者计算数据以便对划分的子任务或数据块同时进行计算。不可分拆的计算任务或相互间有依赖关系的数据无法进行并行计算！

#### 1.2.2 构建抽象模型

> MapReduce借鉴了函数式语言中的思想，用Map和Reduce两个函数提供了高层的并行编程抽象模型。<br/>

> Map: 对一组数据元素进行某种重复式的处理；<br/>

> Reduce: 对Map的中间结果进行某种进一步的结果整理。<br/>

> MapReduce中定义了如下的Map和Reduce两个抽象的编程接口，由用户去编程实现:<br/>

> map: (k1; v1) → [(k2; v2)]<br/>

> reduce: (k2; [v2]) → [(k3; v3)]<br/>

> Map和Reduce为程序员提供了一个清晰的操作接口抽象描述。通过以上两个编程接口，大家可以看出MapReduce处理的数据类型是<key,value>键值对。

#### 1.2.3 统一构架，隐藏系统层细节

> 如何提供统一的计算框架，如果没有统一封装底层细节，那么程序员则需要考虑诸如数据存储、划分、分发、结果收集、错误恢复等诸多细节；为此，MapReduce设计并提供了统一的计算框架，为程序员隐藏了绝大多数系统层面的处理细节。

> MapReduce最大的亮点在于通过抽象模型和计算框架把需要做什么(what need to do)与具体怎么做(how to do)分开了，为程序员提供一个抽象和高层的编程接口和框架。程序员仅需要关心其应用层的具体计算问题，仅需编写少量的处理应用本身计算问题的程序代码。如何具体完成这个并行计算任务所相关的诸多系统层细节被隐藏起来,交给计算框架去处理：从分布代码的执行，到大到数千小到单个节点集群的自动调度使用。

### 1.3 核心功能

> 将用户编写的业务逻辑代码和MapReduce本身自带的组件整合到一个完整的分布式计算程序。

![MR构思](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/1.png)

> block 数据分块，是hdfs中的概念，物理上进行数据的分割，（默认128M）分成一个数据块，split是mapreduce中的一个逻辑的概念，mapTask处理数据的一个分片，具体怎么分片，是和 InputFormat。每个分片对应的是一个map任务。

## 2 MapReduce编程规范

> **数据传输的过程中，都是以 key - value的键值对出现的。**

### 2.1 三个阶段

#### 2.1.1 map阶段

1. 读取数据，将数据转换成 k1 和 v1
2. 自定义 map逻辑， 将 k1 和 v1 转换成 k2 和 v2

![map阶段](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/2.png)

#### 2.1.2 shuffle阶段

1. 分区： 将相同的k2的数据发送给同一个reduce程序
2. 排序：根据k2的数据，进行排序操作（按照字典顺序）
3. 规约combine：是局部聚合，是MapReduce的优化步骤
4. 分组：将相同的k2的值进行合并成为一个集合

![shuffle阶段](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/3.png)

#### 2.1.3 reduce阶段

1. 自定义 reduce 任务的逻辑，将 shuffle 的 k2 和v2 进行转换操作得到 k3 和 v3
2. 输出操作：将k3 和v3 输出到指定的文件目录

![reduce阶段](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/4.png)

### 2.2 编程步骤

> 用户编写的程序分成三个部分：Mapper，Reducer，Driver(提交运行mr程序的客户端)

#### 2.2.1 Mapper

(1)自定义类继承Mapper类<br/>
(2)重写自定义类中的map方法，在该方法中将K1和V1转为K2和V2<br/>
(3)将生成的K2和V2写入上下文中

#### 2.2.2 Reducer

(1)自定义类继承Reducer类<br/>
(2)重写Reducer中的reduce方法，在该方法中将K2和[V2]转为K3和V3<br/>
(3)将K3和V3写入上下文中

#### 2.2.3 Driver

> 整个程序需要一个Drvier来进行提交，提交的是一个描述了各种必要信息的job对象<br/>

（1）定义类，编写main方法<br/>
（2）在main方法中指定以下内容:<br/>
1、创建建一个job任务对象<br/>
2、指定job所在的jar包<br/>
3、指定源文件的读取方式类和源文件的读取路径<br/>
4、指定自定义的Mapper类和K2、V2类型<br/>
5、指定自定义分区类（如果有的话）<br/>
6、指定自定义分组类（如果有的话）<br/>
7、指定自定义的Reducer类和K3、V3的数据类型<br/>
8、指定输出方式类和结果输出路径<br/>
9、将job提交到yarn集群

### 2.3 编程示例(单词统计)

#### 2.3.1 实现思路

![单词统计实现思路](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/5.png)

![单词统计实现思路](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/6.png)

![单词统计实现思路](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/10.png)

#### 2.3.2 数据准备

> 1 创建新文件

```
cd /export/server
vim wordcount.txt
```

> 2 向其中放入以下内容并保存

```
hello hello
world world
hadoop hadoop
hello world
hello flume
hadoop hive
hive kafka
flume storm
hive oozie
```

> 3 上传到 HDFS

```
hdfs dfs -mkdir -p /input/wordcount
hdfs dfs -put wordcount.txt /input/wordcount
```

> 4 查看上传结果

```
hdfs dfs -ls -R /input
```

#### 2.3.3 代码实现

<br/>
1.pom文件

<details>
<summary>点击查看pom</summary>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>beijing45-parent</artifactId>
        <groupId>cn.itcast</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>day09_mapreduce1</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-core</artifactId>
            <version>2.7.6</version>
        </dependency>
        <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13</version>
    </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                    <!--    <verbal>true</verbal>-->
                </configuration>
            </plugin>
               <plugin>
                   <groupId>org.apache.maven.plugins</groupId>
                   <artifactId>maven-shade-plugin</artifactId>
                   <version>2.4.3</version>
                   <executions>
                       <execution>
                           <phase>package</phase>
                           <goals>
                               <goal>shade</goal>
                           </goals>
                           <configuration>
                               <minimizeJar>true</minimizeJar>
                           </configuration>
                       </execution>
                   </executions>
               </plugin>
        </plugins>
    </build>
</project>
```

</details>

<br/>
2.WordCoutMapperTask.java

<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.mapreduce;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/22 11:40
 * Desc 1.实现读取文件 2.将数据进行map映射处理
 */
public class WordCoutMapperTask extends Mapper<LongWritable, Text,Text, IntWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        System.out.println("偏移量:"+key.get());
        //1.读取每一行的数据
        String line = value.toString();
        if(StringUtils.isNotEmpty(line)){
            //2.对数据行进行切割操作 ，空格切割
            String[] words = line.split(" ");
            //3.为每个单词进行赋值为1
            for(String word : words){
                context.write(new Text(word),new IntWritable(1));
            }
        }
    }
}

```

</details>
<br/>
3.WordCountReduceTask.java

<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.mapreduce;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/22 11:51
 * Desc TODO
 */
public class WordCountReduceTask extends Reducer<Text, IntWritable,Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {

        int count = 0;
        //1.遍历所有的值
        for(IntWritable v2:values){
            //2.进行累加
            count += v2.get();
        }
        //3.对累加的值进行输出操作
        context.write(key,new IntWritable(count));
    }
}

```

</details>
<br/>
4.WordCountMain.java

<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.mapreduce;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.mapreduce.lib.partition.HashPartitioner;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

/**
 * Author xiaoma
 * Date 2020/8/22 11:59
 * Desc 实现整个业务逻辑的贯通，将map 和 reduce 任务能跑起来，入口类
 * 调用map reduce
 */
public class WordCountMain extends Configured implements Tool {
    public static void main(String[] args) throws Exception{
        //1.基于 tool 调用 run 方法
        Configuration conf = new Configuration();
        //2.运行并返回一个返回码 ,会返回两个值，如果成功就是0 如果失败就非0
        int code = ToolRunner.run(conf, new WordCountMain(), args);
        //3.执行程序，退出程序
        System.exit(code);
    }

    @Override
    public int run(String[] args) throws Exception {
        //1.获取 Job 对象
        Job job = Job.getInstance(super.getConf(), "wordcount");
        //在yarn集群平台运行必备值参数
        job.setJarByClass(WordCountMain.class);
        //2. 天龙八步
        //2.1读取数据的输入格式组件
        job.setInputFormatClass(TextInputFormat.class);
//        TextInputFormat.addInputPath(job,new Path(args[0]));
        TextInputFormat.addInputPath(job,new Path("F:\\Black_House_42\\Blk_BigData_42\\03_Hadoop\\day09_MapReduce\\资料\\wordcount\\input\\wordcount.txt"));
        //2.2设置mapTask
        job.setMapperClass(WordCoutMapperTask.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        //2.3 设置 shuffle 分区 排序  combine规约 分组
        //2.7 设置reduceTask
        job.setReducerClass(WordCountReduceTask.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        //2 .8 输出组件
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job,new Path("F:\\Black_House_42\\Blk_BigData_42\\03_Hadoop\\day09_MapReduce\\资料\\wordcount\\output\\wordcount.txt"));
        //3 提交任务
        boolean flag = job.waitForCompletion(true);
        return flag?0:1;
    }
}

```

</details>

## 3 MapReduce程序运行模式

### 3.1 本地运行模式

> 1 mapreduce程序是被提交给LocalJobRunner在本地以单进程的形式运行<br/>
> 2 而处理的数据及输出结果可以在本地文件系统，也可以在hdfs上<br/>
> 3 本地模式非常便于进行业务逻辑的调试

### 3.2 集群运行模式

> 1 将mapreduce程序提交给yarn集群，分发到很多的节点上并发执行<br/>
> 2 处理的数据和输出结果应该位于hdfs文件系统

> 集群运行程序

修改WordCountMain类中的下面两行代码如下图

```
TextInputFormat.addInputPath(job,new Path(args[0]));
TextOutputFormat.setOutputPath(job,new Path(args[1]));
```

1. 上面pom插件将这个jar包中所有依赖的第三方的 jar 包 导入到当前的jar包中，这个 jar 包就叫 肥包。

2. 将这个 jar 包（不是带第三方依赖的小包）上传到 linux 中的任意目录（比如/root/）

3. 在yarn集群中执行 mapreduce。 
  - 执行的脚本可以使用 hadoop jar （1.X） 也可以使用 yarn jar （2.x）运行。
  - 当前wordcount的集群执行命令

```
# 参数说明
# yarn jar 执行的命令
# original-day09_mapreduce1-1.0-SNAPSHOT.jar jar 包
# cn.xiaoma.mapreduce.WordCountMain 全路径类名
# /input/wordcount/wordcount.txt input参数
# /output/wordcount/ output参数
yarn jar original-day09_mapreduce1-1.0-SNAPSHOT.jar cn.xiaoma.mapreduce.WordCountMain /input/wordcount/wordcount.txt /output/wordcount/
```

```
# 开启 mapreduce 的历史服务器命令，查看历史跑过的所有 mapreduce 任务
mr-jobhistory-daemon.sh start historyserver
```

## 4 MapReduce分区

> 分区： 将相同的 k2 的数据发送到同一个分区中

> 物以类聚，人与群分

在 MapReduce 中, 通过我们指定分区, 会将同一个分区的数据发送到同一个Reduce当中进行处理。例如: 为了数据的统计, 可以把一批类似的数据发送到同一个 Reduce 当中, 在同一个 Reduce 当中统计相同类型的数据, 就可以实现类似的数据分区和统计等

其实就是相同类型的数据, 有共性的数据, 送到一起去处理, 在Reduce过程中，可以根据实际需求（比如按某个维度进行归档，类似于数据库的分组），把Map完的数据Reduce到不同的文件中。分区的设置需要与ReduceTaskNum配合使用。比如想要得到5个分区的数据结果。那么就得设置5个ReduceTask。

### 4.1 案例

#### 4.1.1 分析

需求：将文本文件中的彩票数据进行分区，小于等于15的分到一个区里， 大于15的分到另外一个区里，并最终将数据保存到两个文件中。

![彩票分析](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/7.png)

思路：怎么进行数据的分区？

> mapreduce 默认分区的方式是 hashPartiton

```
(key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;  #默认情况 numReduceTasks =1
```

如何创建自定义分区

1. 创建一个类，继承 Partitioner<K,V>
2. 重写  getPartition 方法
3. 自定义分区规则，小于等于1 5是个规则0，大于15是个规则1
4. 在入口函数在中将分区规则设置到驱动类

#### 4.1.2 实现
<br/>
1.实现分区业务逻辑

<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.partition;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

/**
 * Author xiaoma
 * Date 2020/8/22 15:44
 * Desc
 * 输入：mapper输出
 */
public class MyPartition extends Partitioner<IntWritable, Text> {
    /**
     * 根据输入的值 key2 value2 生成分区号
     * @param lotteryResult 彩票结果
     * @param text 每条彩票记录
     * @param numPartitions 分区的个数
     * @return 分区号
     */
    @Override
    public int getPartition(IntWritable lotteryResult, Text text, int numPartitions) {
        /**
         * 根据彩票号进行分区，如果小于等于15进行分区标记为0 否则1
         */
        if(lotteryResult.get()<=15){
            return 0;
        }else{
            return 1;
        }
    }
}

```

</details>

<br/>
2.实现 map 业务逻辑

<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.partition;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/22 15:53
 * Desc 将当前的文本内容按行进行拆分，得到第6列数据并转换成数字。
 * 默认使用 TextInputFormat 这个格式 key value 默认就是 LongWritable（偏移量）,Text（每行内容）
 * 后面两个IntWritable（根据彩票号）, Text（每行内容）
 * Long,String,Integer,String 对应 LongWritable, Text, IntWritable, Text
 */
public class LotteryMapperTask extends Mapper<LongWritable,Text, IntWritable, Text> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //1.获取一行
        String line = value.toString();
        if(StringUtils.isNotEmpty(line)){
            //2.对每行数据进行数据的切割操作，获取开奖结果
            String lotteryResult = line.split("\t")[5];
            int code = Integer.parseInt(lotteryResult);
            //3.将数据写出去
            context.write(new IntWritable(code),value);
        }
    }
}

```

</details>

<br/>
3.实现 reduce 的业务逻辑

<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.partition;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * Author itcast
 * Date 2020/8/22 16:18
 * Desc IntWritable Iterable<Text> Text NullWritable
 */
public class LotteryReducerTask extends Reducer<IntWritable, Text,Text, NullWritable> {
    @Override
    protected void reduce(IntWritable key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        for(Text value:values){
            context.write(value,NullWritable.get());
        }
    }
}

```

</details>

<br/>
4.实现LotteryMain的业务逻辑 

<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.partition;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

/**
 * Author xiaoma
 * Date 2020/8/22 16:22
 * Desc 彩票的分区需求
 */
public class LotteryMain {
    public static void main(String[] args) throws Exception{
        //1创建job对象
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "Lottery ticket MR");
        // 设置提交到yarn集群的
        job.setJarByClass(LotteryMain.class);
        //2.封装 八大步
        //2.1 设置输入类
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job,new Path(args[0]));
//        TextInputFormat.addInputPath(job,new Path("F:\\Black_House_42\\Blk_BigData_42\\03_Hadoop\\day09_MapReduce\\资料\\自定义分区\\input\\partition.csv"));
        //2.2 设置自定义map类和相关参数
        job.setMapperClass(LotteryMapperTask.class);
        job.setMapOutputKeyClass(IntWritable.class);
        job.setMapOutputValueClass(Text.class);
        //2.3 设置shuffle中的分区
        job.setPartitionerClass(MyPartition.class);
        //2.4 排序、combine、分组
        //2.7 设置reduce类和相关参数
        job.setReducerClass(LotteryReducerTask.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);
        //2.8 设置输出类
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job,new Path(args[1]));
//        TextOutputFormat.setOutputPath(job,new Path("F:\\Black_House_42\\Blk_BigData_42\\03_Hadoop\\day09_MapReduce\\资料\\自定义分区\\output"));
        //2.9 设置使用分区数，自定义2个分区
        job.setNumReduceTasks(2);
        //3 提交任务
        boolean flag = job.waitForCompletion(true);
        int i = flag ? 0 : 1;
        System.exit(i);
    }
}

```

</details>

<br/>

5.数据准备

```
hdfs dfs -mkdir -p /input/partition
hdfs dfs -put partition.txt /input/partition/partition.csv
```

6.执行 shell 脚本

```shell
yarn jar original-day09_mapreduce1-1.0-SNAPSHOT.jar cn.xiaoma.partition.LotteryMain /input/partition/partition.csv /output/partition
```

## 5 MapReduce排序和序列化

### 5.1 概述

`序列化`（Serialization）是指把结构化对象转化为字节流。

`反序列化`（Deserialization）是序列化的逆过程。把字节流转为结构化对象。
当要在进程间传递对象或持久化对象的时候，就需要序列化对象成字节流，反之当要将接收到或从磁盘读取的字节流转换为对象，就要进行反序列化。

Java的序列化（Serializable）是一个重量级序列化框架，一个对象被序列化后，会附带很多额外的信息（各种校验信息，header，继承体系…），不便于在网络中高效传输；所以，hadoop自己开发了一套序列化机制（Writable），精简，高效。不用像java对象类一样传输多层的父子关系，需要哪个属性就传输哪个属性值，大大的减少网络传输的开销。

### 5.1 案例

> 这个就不在服务器跑了，跑的流程都一样，看下上面的案例就好，换下文件的路径，yarn jar 跑一下就好了

#### 5.1.1 需求

> 将数据进行排列，第一列降序排列，如果相等的情况下，第二列进行升序排列，输出到文件

![排序需求](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/8.png)

#### 5.1.2 分析

> 实现SortPojo 类 思路：

- 实现自定义的bean来封装数据，并将bean作为map输出的key来传输
- MR程序在处理数据的过程中会对数据排序(map输出的kv对传输到reduce之前，会排序)，排序的依据是map输出的key。所以，我们如果要实现自己需要的排序规则，则可以考虑将排序因素放到key中，让key实现接口：WritableComparable，然后重写key的compareTo方法。

#### 5.1.3 实现

<br/>
1.SortPojo类
<details>
<summary>点击查看代码</summary>

```java
package cn.itcast.sort;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

/**
 * Author itcast
 * Date 2020/8/22 16:54
 * Desc Sort 对象，实现了字段的排序和比较
 */
public class SortPojo implements WritableComparable<SortPojo> {
    private String first;
    private String second;

    public SortPojo() {
    }

    public String getFirst() {
        return first;
    }

    public void setFirst(String first) {
        this.first = first;
    }

    public String getSecond() {
        return second;
    }

    public void setSecond(String second) {
        this.second = second;
    }

    @Override
    public String toString() {
        return first + "\t" +second;
    }

    /**
     * 排序的方法，如果第一列不相等，根据字典顺序降序排列
     * 如果第一列相同的清空下，第二列升序排列
     * 如果 this.first.comparaTo(o.first)  升序排列
     * 如果 o.first.comparaTo(this.first)  降序排列
     * @param o
     * @return
     */
    @Override
    public int compareTo(SortPojo o) {
        //比较之后有三个值
        //如果是 i>0 代表o.first大于this.first
        //如果是 i<0 代表this.first大于o.first
        //如果i==0 说明两个值相等
        int i = o.first.compareTo(this.first);
        if( i == 0){
            int i1 = this.second.compareTo(o.second);
            return i1;
        }
        return i;
    }

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeUTF(first);
        out.writeUTF(second);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.first = in.readUTF();
        this.second = in.readUTF();
    }
}

```
</details>

<br/>
2.SortMapperTask 类的实现
<details>
<summary>点击查看代码</summary>

```
package cn.itcast.sort;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * Author itcast
 * Date 2020/8/22 16:54
 * Desc
 * 输入: <偏移量,文本,排序对象,空></>
 */
public class SortMapperTask extends Mapper<LongWritable, Text,SortPojo, NullWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //1.获取行数据
        String line = value.toString();
        if(StringUtils.isNotEmpty(line)){
            //2.切割数据
            String[] sortPojoArr = line.split("\t");
            //3.封装对象并发送数据
            SortPojo sortPojo = new SortPojo();
            sortPojo.setFirst(sortPojoArr[0]);
            sortPojo.setSecond(sortPojoArr[1]);
            context.write(sortPojo, NullWritable.get());
        }

    }
}

```
</details>

<br/>
3.SortReduceTask 的实现
<details>
<summary>点击查看代码</summary>

```
package cn.itcast.sort;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * Author itcast
 * Date 2020/8/22 17:13
 * Desc TODO
 */
public class SortReduceTask extends Reducer<SortPojo, NullWritable,SortPojo, NullWritable> {
    @Override
    protected void reduce(SortPojo key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
        for(NullWritable n:values){
            context.write(key,n);
        }
    }
}

```
</details>

<br/>
4.SortMain 类的实现
<details>
<summary>点击查看代码</summary>

```
package cn.itcast.sort;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

/**
 * Author itcast
 * Date 2020/8/22 17:14
 * Desc 实现主要的封装操作
 */
public class SortMain {
    public static void main(String[] args) throws Exception {
        //1.创建job对象
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "sort MR");
        //2.设置 job 的八大步  天龙八部
        //2.1 设置输入类
        job.setInputFormatClass(TextInputFormat.class);
//        TextInputFormat.addInputPath(job,new Path(args[0]));
        TextInputFormat.addInputPath(job,new Path("F:\\Black_House_42\\Blk_BigData_42\\03_Hadoop\\day09_MapReduce\\资料\\排序\\input\\sort.txt"));
        //2.2 设置mapper
        job.setMapperClass(SortMapperTask.class);
        job.setMapOutputKeyClass(SortPojo.class);
        job.setMapOutputValueClass(NullWritable.class);
        //2.3 设置shuffle 分区 排序 combine 分组

        //2.7 设置reducer
        job.setReducerClass(SortReduceTask.class);
        job.setOutputKeyClass(SortPojo.class);
        job.setOutputValueClass(NullWritable.class);
        //3. 文件输出格式
        job.setOutputFormatClass(TextOutputFormat.class);
//        TextOutputFormat.setOutputPath(job,new Path(args[1]));
        TextOutputFormat.setOutputPath(job,new Path("F:\\Black_House_42\\Blk_BigData_42\\03_Hadoop\\day09_MapReduce\\资料\\排序\\output"));
        // 提交任务
        boolean flag = job.waitForCompletion(true);
        //退出
        System.exit(flag?0:1);
    }
}

```
</details>

<br/>

![排序结果](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/9.png)









## 6 MapReduce的combine规约

### 6.1 概念

> 每一个 map 都可能会产生大量的本地输出，Combiner 的作用就是对 map 端的输出先做一次合并，以减少在 map 和 reduce 节点之间的数据传输量，以提高网络IO 性能，是 MapReduce 的一种优化手段之一

> combiner 是 MR 程序中 Mapper 和 Reducer 之外的一种组件

> combiner 组件的父类就是 Reducer

> combiner 和 reducer 的区别在于运行的位置


- Combiner 是在每一个 maptask 所在的节点运行

- Reducer 是接收全局所有 Mapper 的输出结果\

> combiner 的意义就是对每一个 maptask 的输出进行局部汇总，以减小网络传输量

> combine规约和reduce的逻辑是相似的，combine是一个本地局部（所有的所在节点）的汇总，以减小网络传输量；reduce是一个全局的汇总操作。

### 6.2 案例

#### 6.2.1 需求

有三个书架 ,每个书架上都有5本书, 要求 统计出 每种分类的下有几本书??  计算机  武林秘籍  历史

```
  1号书架                 2号书架                         3号书架
<<java入门宝典>>    <<Python入门宝典>>            <<spark入门宝典>>
<<UI入门宝典>>      <<乾坤大挪移>>                <<hive入门宝典>> 
<<天龙八部>>        <<凌波微步>>                  <<葵花点穴手>>
<<史记>>            <<PHP入门宝典>>               <<铁砂掌>>
<<葵花宝典>>        <<hadoop入门宝典>>            <<论清王朝的腐败>>
```

输入

```
<<java入门宝典>>
<<Python入门宝典>>
<<spark入门宝典>>
<<UI入门宝典>>
<<乾坤大挪移>>
<<hive入门宝典>>
<<天龙八部>>
<<凌波微步>>
<<葵花点穴手>>
<<史记>>
<<PHP入门宝典>>
<<铁砂掌>>
<<葵花宝典>>
<<hadoop入门宝典>>
<<论清王朝的腐败>>

```

#### 6.2.2 分析

> 思路：

1. 首先要创建 mapTask reduceTask
2. 重写Reducer提供的reduce方法，实现局部聚合逻辑
3. 在MR的驱动中，main函数中将 combine 的类添加进去。

```
int count = 0;
for(IntWritable value:values){
    count += value.get();
}
//将数据写出去
context.write(key,new IntWritable(count));
```

使用combiner 和不适用 combiner 的reduce端的数据 input 的区别，使用combiner 明显比不使用 combiner的读取数量要小。

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/11.png)

#### 6.2.3 实现

<br/>
1.CombinerMapperTask
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.combine;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/23 9:46
 * Desc 根据不同的图书进行分类，分类之后进行数据的合并
 * 比如说
 * <<java入门宝典>>  <计算机 ， 1>
 * <<Python入门宝典>> <计算机 ， 1>
 * <<spark入门宝典>> <计算机 ， 1>
 * <<UI入门宝典>> <计算机 ， 1>
 * <<乾坤大挪移>> <武林秘籍， 1>
 * <<hive入门宝典>> <计算机 ， 1>
 * <<天龙八部>> <武林秘籍， 1>
 * <<凌波微步>> <武林秘籍， 1>
 * <<葵花点穴手>> <武林秘籍， 1>
 * <<史记>> <历史， 1>
 * <<PHP入门宝典>> <计算机 ， 1>
 * <<铁砂掌>>  <武林秘籍， 1>
 * <<葵花宝典>> <武林秘籍， 1>
 * <<hadoop入门宝典>>  <计算机 ， 1>
 * <<论清王朝的腐败>>  <历史， 1>
 */
public class CombinerMapperTask extends Mapper<LongWritable, Text, Text/*分类名称*/, IntWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String bookName = value.toString();
        //判断是否为空
        if (StringUtils.isNotEmpty(bookName)) {
            if (bookName.contains("入门")) {
                context.write(new Text("计算机"), new IntWritable(1));
            } else if (bookName.contains("史记") || bookName.contains("论清王朝的腐败")) {
                context.write(new Text("历史"), new IntWritable(1));
            } else {
                context.write(new Text("武林秘籍"), new IntWritable(1));
            }
        }
    }
}

```
</details>

<br/>
2.CombinerReducerTask 实现
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.combine;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/23 9:56
 * Desc TODO
 * <h,1><h,2><h,4>
 * <h,[1,2,4]>
 * <m,3>
 * <h,7>
 * <h,10>
 */
public class CombinerReducerTask extends Reducer<Text, IntWritable, Text,IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int count = 0;
        for(IntWritable value:values){
            count += value.get();
        }
        //将数据写出去
        context.write(key,new IntWritable(count));
    }
}

```
</details>

<br/>
3.CombinerTask 的实现
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.combine;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/23 10:30
 * Desc TODO
 */
public class CombinerTask extends Reducer<Text, IntWritable,Text,IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int count = 0;
        for(IntWritable value:values){
            count += value.get();
        }
        //将数据写出去
        context.write(key,new IntWritable(count));
    }
}

```
</details>

<br/>
4.CombinerJobMain 实现
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.combine;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

/**
 * Author xiaoma
 * Date 2020/8/23 9:59
 * Desc TODO
 */
public class CombinerJobMain {
    public static void main(String[] args) throws Exception{
        //1.创建 job 对象
        Job job = Job.getInstance(new Configuration(), "combinerJobMR");
        job.setJarByClass(CombinerJobMain.class);
        //2.设置 mapreduce 的八大步
        //2.1读取数据
        job.setInputFormatClass(TextInputFormat.class);
//        TextInputFormat.addInputPath(job,new Path(args[0]));
        TextInputFormat.addInputPath(job,new Path("F:\\Black_House_42\\Blk_BigData_42\\03_Hadoop\\day10_MapReduce\\资料\\combinner\\input\\combiner.txt"));
        //2.2设置 map
        job.setMapperClass(CombinerMapperTask.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        //2.3 设置shuffle ： 分区 排序 归并combine 分组
//        job.setCombinerClass(CombinerReducerTask.class);
        job.setCombinerClass(CombinerTask.class);
        //2.7 设置 reduce
        job.setReducerClass(CombinerReducerTask.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        //2.8 输出
        job.setOutputFormatClass(TextOutputFormat.class);
//        TextOutputFormat.setOutputPath(job,new Path(args[1]));
        TextOutputFormat.setOutputPath(job,new Path("F:\\Black_House_42\\Blk_BigData_42\\03_Hadoop\\day10_MapReduce\\资料\\combinner\\output"));
        //3 提交任务
        boolean flag = job.waitForCompletion(true);
        //4 退出程序
        System.exit(flag?0:1);
    }
}

```
</details>

#### 6.2.4 运行结果

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/12.png)

## 7 MapReduce的分组

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/13.png)

> 分区和分组的区别

- 分区：将相同的 k2的数据，发送给同一个 reducer 中，这个操作是在 map端执行
- 分组：将相同的 k2的值进行合并形成一个集合操作，在 reduce 中对同一个分区下的数据进行分组操作。

### 7.1 需求

有如下订单数据，现在需要求出每一个订单中成交金额最大的一笔交易，将结果集存储到2个文件

订单id | 商品id | 成交金额
---|---|---
Order_0000001 | Pdt_01 | 222.8
Order_0000001 | Pdt_05 | 25.8
Order_0000002 | Pdt_03 | 522.8
Order_0000002 | Pdt_04 | 122.4
Order_0000002 | Pdt_05 | 722.4
Order_0000003 | Pdt_01 | 222.8

现在需要求出每一个订单中成交金额最大的一笔交易，将结果集存储到2个文件

### 7.2 分析

思路 

- 如果使用 SQL 怎么写？
  select  订单id,max(成交金额) from 订单表 group by 订单id
- 使用mapreduce 怎么来实现呢？

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/14.png)

### 7.3 实现

<br/>
1.OrderBean 实现
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.group;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/23 15:22
 * Desc 订单的对象
 */
public class OrderBean implements WritableComparable<OrderBean> {
    //订单id
    private String orderid;
    //商品id
    private String pid;
    //成交金额
    private Double price;

    public OrderBean() {
    }

    public String getOrderid() {
        return orderid;
    }

    public void setOrderid(String orderid) {
        this.orderid = orderid;
    }

    public String getPid() {
        return pid;
    }

    public void setPid(String pid) {
        this.pid = pid;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return orderid + "\t" + pid + "\t" + price;
    }

    @Override
    public int compareTo(OrderBean o) {
        int i = o.orderid.compareTo(this.orderid);
        if( i == 0 ){
            int i1 = o.price.compareTo(this.price);
            return i1;
        }
        return i;
    }

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeUTF(orderid);
        out.writeUTF(pid);
        out.writeDouble(price);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.orderid = in.readUTF();
        this.pid = in.readUTF();
        this.price=in.readDouble();
    }
}


```
</details>

<br/>
2.GroupMapperTask
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.group;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/23 15:33
 * Desc TODO
 */
public class GroupMapperTask extends Mapper<LongWritable, Text,OrderBean, NullWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //1.读取每一行的数据
        String line = value.toString();
        //2.判断数据不为空的情况下
        if(StringUtils.isNotEmpty(line)){
            //2.1对数据切割
            String[] splits = line.split("\t");
            //2.2数据进行封装
            OrderBean orderBean = new OrderBean();
            orderBean.setOrderid(splits[0]);
            orderBean.setPid(splits[1]);
            orderBean.setPrice(Double.parseDouble(splits[2]));
            //将数据写出去
            context.write(orderBean,NullWritable.get());
        }
    }
}

```
</details>

<br/>
3.MyPartition
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.group;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Partitioner;

/**
 * Author xiaoma
 * Date 2020/8/23 15:39
 * Desc TODO
 */
public class MyPartition extends Partitioner<OrderBean, NullWritable> {
    @Override
    public int getPartition(OrderBean orderBean, NullWritable nullWritable, int numPartitions) {
        String orderid = orderBean.getOrderid();
        return (orderid.hashCode() & Integer.MAX_VALUE) % numPartitions;
    }
}

```
</details>

<br/>
4.MyGroup  分组实现
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.group;

import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;

/**
 * Author xiaoma
 * Date 2020/8/23 15:44
 * Desc TODO
 */
public class MyGroup extends WritableComparator {
    //将我们自定义的OrderBean注册到我们自定义的MyGroup当中来
    //表示我们的分组器在分组的时候，对OrderBean这一种类型的数据进行分组
    //传入作为key的bean的class类型，以及指定需要让框架做反射获取实例对象
    public MyGroup(){
        //告诉分组组件，k2 是什么类型，允许 k2 这个对象能够被创建出来
        super(OrderBean.class,true);
    }
    @Override
    public int compare(WritableComparable a, WritableComparable b) {
        OrderBean a1 = (OrderBean) a;
        OrderBean b1 = (OrderBean) b;
        return a1.getOrderid().compareTo(b1.getOrderid());
    }
}

```
</details>

<br/>
5.GroupReduceTask
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.group;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/23 15:55
 * Desc TODO
 */
public class GroupReduceTask extends Reducer<OrderBean, NullWritable, OrderBean, NullWritable> {
    @Override
    protected void reduce(OrderBean key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException {
        //控制当前只取一条数据
        /**
         * reduce 就是将前面 map 分区 排序 分组得到的结果集拿到
         * for 循环，就会将所有的数据都输出到文件中
         * 这个时候用一个变量来确定，只输出一个值，
         * 来确保降序排列价格中最高的值
         */
        //size 代表当前我要输出到文件的订单个数
        //int size = 1;
        //length 当前的输出的个数
        int length = 1;
        for (NullWritable value : values) {
            context.write(key, value);
            if (length == 1) {
                break;
            }
            //length++;
        }
    }
}

```
</details>
 
<br/>
6.GroupJobMain
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.group;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
/**
 * Author xiaoma
 * Date 2020/8/23 16:18
 * Desc TODO
 */
public class GroupJobMain {
    public static void main(String[] args) throws Exception{
        //1创建job对象
        Job job = Job.getInstance(new Configuration(), "GroupJobMR");
        //2 实现mr 八大步
        //2.1 读取数据
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job,new Path(args[0]));
        //2.2 封装 mapper
        job.setMapperClass(GroupMapperTask.class);
        job.setMapOutputKeyClass(OrderBean.class);
        job.setMapOutputValueClass(NullWritable.class);
        //2.3 分区操作
        job.setPartitionerClass(MyPartition.class);
        //2.4 设置排序，规约
        //2.6 设置分组操作
        job.setGroupingComparatorClass(MyGroup.class);
        //2.7 设置reduce
        job.setReducerClass(GroupReduceTask.class);
        job.setOutputKeyClass(OrderBean.class);
        job.setOutputValueClass(NullWritable.class);
        //2.8 输出操作
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job,new Path(args[1]));
        //3.设置 2个reduce task
        job.setNumReduceTasks(2);
        //4 提交任务
        boolean flag = job.waitForCompletion(true);
        //5 退出
        System.exit(flag?0:1);
    }
}

```
</details>

## 8 MapReduce的底层运行机制

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/15.png)

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/28.png)

> mapTask的并行机制

- mapTask在运行的时候，开启多个map由谁来决定？

默认情况：mapTask 的数量和读取 HDFS 中的数据块 block 的数量相等

- block块：HDFS 中文件各个小数据块（默认 128m ）（物理划分）
- FileSplit： 在MapReduce 读取每一个块称为 fileSplit（文件切片）（逻辑划分）

block 的数量 和 文件分片的数量一样，大小也是一样。

### 8.1 MapTask运行机制

简单概述：inputFile通过split被逻辑切分为多个split文件，通过Record按行读取内容给map（用户自己实现的）进行处理，数据被map处理结束之后交给OutputCollector收集器，对其结果key进行分区（默认使用hash分区），然后写入buffer，每个map task都有一个内存缓冲区，存储着map的输出结果，当缓冲区快满的时候需要将缓冲区的数据以一个临时文件的方式存放到磁盘，当整个map task结束后再对磁盘中这个map task产生的所有临时文件做合并，生成最终的正式输出文件，然后等待reduce task来拉数据。

#### 8.1.1 详细步骤

> 1 读取数据组件 InputFormat (默认 TextInputFormat) 会通过 getSplits 方法对输入目录中文件进行逻辑切片规划得到 block, 有多少个 block就对应启动多少个 MapTask. 

> 2 将输入文件切分为 block 之后, 由 RecordReader 对象 (默认是LineRecordReader) 进行读取, 以 \n 作为分隔符, 读取一行数据, 返回 <key，value>. Key 表示每行首字符偏移值, Value 表示这一行文本内容

> 3 读取 block 返回 <key,value>, 进入用户自己继承的 Mapper 类中，执行用户重写的 map 函数, RecordReader 读取一行这里调用一次

> 4 Mapper 逻辑结束之后, 将 Mapper 的每条结果通过 context.write 进行collect数据收集. 在 collect 中, 会先对其进行分区处理，默认使用 HashPartitioner

`MapReduce 提供 Partitioner 接口, 它的作用就是根据 Key 或 Value 及 Reducer 的数量来决定当前的这对输出数据最终应该交由哪个 Reduce task 处理, 默认对 Key Hash 后再以 Reducer 数量取模. 默认的取模方式只是为了平均 Reducer 的处理能力, 如果用户自己对 Partitioner 有需求, 可以订制并设置到 Job 上`

> 5 接下来, 会将数据写入内存, 内存中这片区域叫做环形缓冲区, 缓冲区的作用是批量收集 Mapper 结果, 减少磁盘 IO 的影响. 我们的 Key/Value 对以及 Partition 的结果都会被写入缓冲区. 当然, 写入之前，Key 与 Value 值都会被序列化成字节数组

`环形缓冲区其实是一个数组, 数组中存放着 Key, Value 的序列化数据和 Key, Value 的元数据信息, 包括 Partition, Key 的起始位置, Value 的起始位置以及 Value 的长度. 环形结构是一个抽象概念`

`缓冲区是有大小限制, 默认是 100MB. 当 Mapper 的输出结果很多时, 就可能会撑爆内存, 所以需要在一定条件下将缓冲区中的数据临时写入磁盘, 然后重新利用这块缓冲区. 这个从内存往磁盘写数据的过程被称为 Spill, 中文可译为溢写. 这个溢写是由单独线程来完成, 不影响往缓冲区写 Mapper 结果的线程. 溢写线程启动时不应该阻止 Mapper 的结果输出, 所以整个缓冲区有个溢写的比例 spill.percent. 这个比例默认是 0.8, 也就是当缓冲区的数据已经达到阈值 buffer size * spill percent = 100MB * 0.8 = 80MB, 溢写线程启动, 锁定这 80MB 的内存, 执行溢写过程. Mapper 的输出结果还可以往剩下的 20MB 内存中写, 互不影响`

> 6 当溢写线程启动后, 需要对这 80MB 空间内的 Key 做排序 (Sort). 排序是 MapReduce 模型默认的行为, 这里的排序也是对序列化的字节做的排序

`如果 Job 设置过 Combiner, 那么现在就是使用 Combiner 的时候了. 将有相同 Key 的 Key/Value 对的 Value 加起来, 减少溢写到磁盘的数据量. Combiner 会优化 MapReduce 的中间结果, 所以它在整个模型中会多次使用`

`那哪些场景才能使用 Combiner 呢? 从这里分析, Combiner 的输出是 Reducer 的输入, Combiner 绝不能改变最终的计算结果. Combiner 只应该用于那种 Reduce 的输入 Key/Value 与输出 Key/Value 类型完全一致, 且不影响最终结果的场景. 比如累加, 最大值等. Combiner 的使用一定得慎重, 如果用好, 它对 Job 执行效率有帮助, 反之会影响 Reducer 的最终结果`

> 7 合并溢写文件, 每次溢写会在磁盘上生成一个临时文件 (写之前判断是否有 Combiner), 如果 Mapper 的输出结果真的很大, 有多次这样的溢写发生, 磁盘上相应的就会有多个临时文件存在. 当整个数据处理结束之后开始对磁盘中的临时文件进行 Merge 合并, 因为最终的文件只有一个, 写入磁盘, 并且为这个文件提供了一个索引文件, 以记录每个reduce对应数据的偏移量

#### 8.1.2 配置

配置 | 默认值 | 解释
---|---|---
mapreduce.task.io.sort.mb | 100 | 设置环型缓冲区的内存值大小
mapreduce.map.sort.spill.percent | 0.8 | 设置溢写的比例
mapreduce.cluster.local.dir | ${hadoop.tmp.dir}/mapred/local | 溢写数据目录
mapreduce.task.io.sort.factor | 10 | 设置一次合并多少个溢写文件



### 8.2 ReduceTask运行机制

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/16.png)

Reduce大致分为copy、sort、reduce三个阶段，重点在前两个阶段。copy阶段包含一个eventFetcher来获取已完成的map列表，由Fetcher线程去copy数据，在此过程中会启动两个merge线程，分别为inMemoryMerger和onDiskMerger，分别将内存中的数据merge到磁盘和将磁盘中的数据进行merge。待数据copy完成之后，copy阶段就完成了，开始进行sort阶段，sort阶段主要是执行finalMerge操作，纯粹的sort阶段，完成之后就是reduce阶段，调用用户定义的reduce函数进行处理。

详细步骤：

> 1 Copy阶段

简单地拉取数据。Reduce进程启动一些数据copy线程(Fetcher)，通过HTTP方式请求maptask获取属于自己的文件。

> 2 Merge阶段

这里的merge如map端的merge动作，只是数组中存放的是不同map端copy来的数值。Copy过来的数据会先放入内存缓冲区中，这里的缓冲区大小要比map端的更为灵活。merge有三种形式：内存到内存；内存到磁盘；磁盘到磁盘。默认情况下第一种形式不启用。当内存中的数据量到达一定阈值，就启动内存到磁盘的merge。与map 端类似，这也是溢写的过程，这个过程中如果你设置有Combiner，也是会启用的，然后在磁盘中生成了众多的溢写文件。第二种merge方式一直在运行，直到没有map端的数据时才结束，然后启动第三种磁盘到磁盘的merge方式生成最终的文件。

> 3 合并排序

把分散的数据合并成一个大的数据后，还会再对合并后的数据排序。

> 4 对排序后的键值对调用reduce方法

键相等的键值对调用一次reduce方法，每次调用会产生零个或者多个键值对，最后把这些输出的键值对写入到HDFS文件中。

### 8.3 shuffle运行机制

map阶段处理的数据如何传递给reduce阶段，是MapReduce框架中最关键的一个流程，这个流程就叫shuffle。
shuffle: 洗牌、发牌——（核心机制：数据分区，排序，分组，规约，合并等过程）。

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/17.png)

shuffle是Mapreduce的核心，它分布在Mapreduce的map阶段和reduce阶段。一般把从Map产生输出开始到Reduce取得数据作为输入之前的过程称作shuffle。

> 1 Collect阶段

将MapTask的结果输出到默认大小为100M的环形缓冲区，保存的是key/value，Partition分区信息等。

> 2 Spill阶段

当内存中的数据量达到一定的阀值的时候，就会将数据写入本地磁盘，在将数据写入磁盘之前需要对数据进行一次排序的操作，如果配置了combiner，还会将有相同分区号和key的数据进行排序。

> 3 Merge阶段

把所有溢出的临时文件进行一次合并操作，以确保一个MapTask最终只产生一个中间数据文件。

> 4 Copy阶段

ReduceTask启动Fetcher线程到已经完成MapTask的节点上复制一份属于自己的数据，这些数据默认会保存在内存的缓冲区中，当内存的缓冲区达到一定的阀值的时候，就会将数据写到磁盘之上。

> 5 Merge阶段

在ReduceTask远程复制数据的同时，会在后台开启两个线程对内存到本地的数据文件进行合并操作。

> 6 Sort阶段

在对数据进行合并的同时，会进行排序操作，由于MapTask阶段已经对数据进行了局部的排序，ReduceTask只需保证Copy的数据的最终整体有效性即可。

Shuffle中的缓冲区大小会影响到mapreduce程序的执行效率，原则上说，缓冲区越大，磁盘io的次数越少，执行速度就越快

缓冲区的大小可以通过参数调整,  参数：mapreduce.task.io.sort.mb  默认100M

## 9 高阶案例

### 9.1 上网流量统计

数据格式如下：

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/18.png)

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/19.png)

#### 9.1.1 统计求和

##### 9.1.1.1 需求

> 统计上网流量中，每个手机号的上行流量、下行流量、上行总流量、下行总流量的和。

##### 9.1.1.2 分析

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/20.png)

##### 9.1.1.3 实现

<br/>
1.FlowBean
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.flow;

import org.apache.hadoop.io.Writable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/23 16:45
 * Desc TODO
 */
public class FlowBean implements Writable {
    private Integer upFlow;
    private Integer downFlow;
    private Integer upCountFlow;
    private Integer downCountFlow;

    public FlowBean() {
    }

    public FlowBean(Integer upFlow, Integer downFlow, Integer upCountFlow, Integer downCountFlow) {
        this.upFlow = upFlow;
        this.downFlow = downFlow;
        this.upCountFlow = upCountFlow;
        this.downCountFlow = downCountFlow;
    }

    public Integer getUpFlow() {
        return upFlow;
    }

    public void setUpFlow(Integer upFlow) {
        this.upFlow = upFlow;
    }

    public Integer getDownFlow() {
        return downFlow;
    }

    public void setDownFlow(Integer downFlow) {
        this.downFlow = downFlow;
    }

    public Integer getUpCountFlow() {
        return upCountFlow;
    }

    public void setUpCountFlow(Integer upCountFlow) {
        this.upCountFlow = upCountFlow;
    }

    public Integer getDownCountFlow() {
        return downCountFlow;
    }

    public void setDownCountFlow(Integer downCountFlow) {
        this.downCountFlow = downCountFlow;
    }

    @Override
    public String toString() {
        return upFlow +"\t"+ downFlow+"\t"+ upCountFlow+"\t"+ downCountFlow;
    }

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeInt(upFlow);
        out.writeInt(downFlow);
        out.writeInt(upCountFlow);
        out.writeInt(downCountFlow);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.upFlow = in.readInt();
        this.downFlow = in.readInt();
        this.upCountFlow = in.readInt();
        this.downCountFlow = in.readInt();
    }
}

```
</details>

<br/>
2.FlowMapperTask
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.flow;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/23 16:47
 * Desc TODO
 */
public class FlowMapperTask extends Mapper<LongWritable,Text, Text/*手机号*/,FlowBean/*流量对象*/> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //1.读取数据
        String line = value.toString();
        //2.判断是否为空
        if(StringUtils.isNotEmpty(line)){
            String[] splits = line.split("\t");
            //截取出来四个字段 上传流量 下载流量 上传总流量 下载总流量
            String iphone = splits[1];
            String upFlow = splits[6];
            String downFlow = splits[7];
            String upTotalFlow = splits[8];
            String downTotalFlow = splits[9];
            FlowBean flowBean = new FlowBean(
                    Integer.parseInt(upFlow),
                    Integer.parseInt(downFlow),
                    Integer.parseInt(upTotalFlow),
                    Integer.parseInt(downTotalFlow)
            );
            //将数据写出去
            context.write(new Text(iphone),flowBean);
        }
    }
}

```
</details>

<br/>
3.FlowReduceTask
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.flow;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/23 16:53
 * Desc TODO
 */
public class FlowReduceTask extends Reducer<Text, FlowBean, Text, NullWritable> {
    @Override
    protected void reduce(Text key, Iterable<FlowBean> values, Context context) throws IOException, InterruptedException {
        //用于接收总的上传流量
        Integer upFlow = 0;
        Integer downFlow = 0;
        Integer upTotalFlow = 0;
        Integer downTotalFlow = 0;
        //变量flowBean
        for (FlowBean value : values) {
            upFlow += value.getUpFlow();
            downFlow += value.getDownFlow();
            upTotalFlow += value.getUpCountFlow();
            downTotalFlow += value.getDownCountFlow();
        }
        //将值写出去
        String k3 = key.toString() + "\t" + upFlow + "\t" + downFlow + "\t" + upTotalFlow + "\t" + downTotalFlow;
        context.write(new Text(k3), NullWritable.get());
    }
}

```
</details>

<br/>
4.FlowJobMain
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.flow;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

/**
 * Author xiaoma
 * Date 2020/8/23 17:00
 * Desc TODO
 */
public class FlowJobMain {
    public static void main(String[] args) throws Exception{
        //1创建job对象
        Job job = Job.getInstance(new Configuration(), "FlowJobMR");
        //2 实现mr 八大步
        //2.1 读取数据
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job,new Path(args[0]));
        //2.2 封装 mapper
        job.setMapperClass(FlowMapperTask.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(FlowBean.class);
        //2.3 分区操作
        //job.setPartitionerClass(MyPartition.class);
        //2.4 设置排序，规约
        //2.6 设置分组操作
        //job.setGroupingComparatorClass(MyGroup.class);
        //2.7 设置reduce
        job.setReducerClass(FlowReduceTask.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);
        //2.8 输出操作
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job,new Path(args[1]));
        //3.设置 2个reduce task
        //job.setNumReduceTasks(2);
        //4 提交任务
        boolean flag = job.waitForCompletion(true);
        //5 退出
        System.exit(flag?0:1);
    }
}

```
</details>

<br/>

#### 9.1.2 上行流量倒序排序（递减排序）

#####  9.1.2.1 需求

> 以需求一的输出数据作为排序的输入数据，自定义FlowBean,以FlowBean为map输出的key，以手机号作为Map输出的value，因为MapReduce程序会对Map阶段输出的key进行排序

<br/>
1.定义FlowBean实现WritableComparable实现比较排序
<details>
<summary>点击查看代码</summary>

```java
public class FlowBean implements WritableComparable<FlowBean> {
    private Integer upFlow;
    private Integer  downFlow;
    private Integer upCountFlow;
    private Integer downCountFlow;
    public FlowBean() {
    }
    public FlowBean(Integer upFlow, Integer downFlow, Integer upCountFlow, Integer downCountFlow) {
        this.upFlow = upFlow;
        this.downFlow = downFlow;
        this.upCountFlow = upCountFlow;
        this.downCountFlow = downCountFlow;
    }
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeInt(upFlow);
        out.writeInt(downFlow);
        out.writeInt(upCountFlow);
        out.writeInt(downCountFlow);
    }
    @Override
    public void readFields(DataInput in) throws IOException {
        upFlow = in.readInt();
        downFlow = in.readInt();
        upCountFlow = in.readInt();
        downCountFlow = in.readInt();
    }
    public Integer getUpFlow() {
        return upFlow;
    }
    public void setUpFlow(Integer upFlow) {
        this.upFlow = upFlow;
    }
    public Integer getDownFlow() {
        return downFlow;
    }
    public void setDownFlow(Integer downFlow) {
        this.downFlow = downFlow;
    }
    public Integer getUpCountFlow() {
        return upCountFlow;
    }
    public void setUpCountFlow(Integer upCountFlow) {
        this.upCountFlow = upCountFlow;
    }
    public Integer getDownCountFlow() {
        return downCountFlow;
    }
    public void setDownCountFlow(Integer downCountFlow) {
        this.downCountFlow = downCountFlow;
    }
    @Override
    public String toString() {
        return upFlow+"\t"+downFlow+"\t"+upCountFlow+"\t"+downCountFlow;
    }
    @Override
    public int compareTo(FlowBean o) {
        return this.upCountFlow > o.upCountFlow ?-1:1;
    }
}
```
</details>

<br/>
2.定义FlowMapper
<details>
<summary>点击查看代码</summary>

```java
public class FlowMapper extends Mapper<LongWritable,Text,FlowBean,Text> {
     Text outKey = new Text();
     FlowBean flowBean = new FlowBean();
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String[] split = value.toString().split("\t");
        flowBean.setUpFlow(Integer.parseInt(split[1]));
        flowBean.setDownFlow(Integer.parseInt(split[2]));
        flowBean.setUpCountFlow(Integer.parseInt(split[3]));
        flowBean.setDownCountFlow(Integer.parseInt(split[4]));
        outKey.set(split[0]);
        context.write(flowBean,outKey);
    }
}
```
</details>

<br/>
3.定义FlowReducer
<details>
<summary>点击查看代码</summary>

```java
public class FlowReducer extends Reducer<FlowBean,Text,Text,FlowBean> {
    FlowBean flowBean = new FlowBean();
    @Override
    protected void reduce(FlowBean key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
       context.write(values.iterator().next(),key);
    }
}
```
</details>

<br/>
4.程序main函数入口
<details>
<summary>点击查看代码</summary>

```java
public class FlowMain extends Configured implements Tool {
    @Override
    public int run(String[] args) throws Exception {
        Configuration conf = super.getConf();
        conf.set("mapreduce.framework.name","local");
        Job job = Job.getInstance(conf, FlowMain.class.getSimpleName());
        job.setJarByClass(FlowMain.class);
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job,new Path("file:///D:\\flowcount\\output"));
        job.setMapperClass(FlowMapper.class);
        job.setMapOutputKeyClass(FlowBean.class);
        job.setMapOutputValueClass(Text.class);
        job.setReducerClass(FlowReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);
        TextOutputFormat.setOutputPath(job,new Path("file:///D:\\output\\flowcount_sort"));
        job.setOutputFormatClass(TextOutputFormat.class);
        boolean b = job.waitForCompletion(true);
        return b?0:1;
    }
    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        int run = ToolRunner.run(configuration, new FlowMain(), args);
        System.exit(run);
    }
}
```
</details>

### 9.2 社交粉丝数据分析

数据如下：

以下是qq的好友列表数据，冒号前是一个用户，冒号后是该用户的所有好友（数据中的好友关系是单向的）

```
A:B,C,D,F,E,O
B:A,C,E,K
C:F,A,D,I
D:A,E,F,L
E:B,C,D,M,L
F:A,B,C,D,E,O,M
G:A,C,D,E,F
H:A,C,D,E,O
I:A,O
J:B,O
K:A,C,D
L:D,E,F
M:E,F,G
O:A,H,I,J
```

#### 9.2.1 需求

> 计算出来两两用户之间，有哪些共同好友？

#### 9.2.2 分析

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/21.png)

```
#案例数据
A: B C D
B: C D
C: A B D
D: A C

#某一个用户，在哪些好友列表中存在
#先mapper任务
<k1,v1> => <k2,v2>
//k1是偏移量,v1是整个文本文件
//从左往右看,是:B出现在A的好友列表,
			  C出现在A的好友列表,
			  D出现在A的好友列表
<k2,  v2>
 B    A
 C    A
 D    A
 C    B
 D    B
 A    C
 B    C
 D    C
 A    D
 C    D
#reduce任务
//reduce的任务就是对mapperde 结果进行归类统计
		B出现在了A,C的好友列表
		C出现在了A,B,D的好友列表
		D出现在了A,B,C的好友列表
		A出现在了C,D的好友列表
 B:{A,C}
 C:{A,B,D}
 D:{A,B,C}
 A:{C,D}
 
 #写第二个MR
 #mapper任务
 //  A-B的共同好友  c
 	 A-B的共同好友  D
 	 A-C的共同好友  B
 	 
 	 
 <k2, V2>
 A-B  C
 A-B  D
 A-C  B
 A-C  D
 A-D  C
 B-C  D
 B-D  C
 C-D  A
 #reduce任务，先分组在聚合
 //对mapper的结果进行合并统计,对相同的k2进行value的汇总
 A-B:{C,D}
 A-C:{B,D}
 A-D:{C}
 B-C:{D}
 B-D:{C}
 C-D:{A}
```

#### 9.2.3 实现

<br/>
1.Friend1MapperTask 实现
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.friend;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/25 10:03
 * Desc TODO
 */
public class Friend1MapperTask extends Mapper<LongWritable, Text,Text,Text> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //1.读取每一行的数据
        String line = value.toString();
        //2.判断每一行是否存在
        if(StringUtils.isNotEmpty(line)){
            //2.1对数据进行切割
            String[] splits = line.split(":");
            String[] friends = splits[1].split(",");
            //2.2遍历好友列表
            for(String friend:friends){
                //2.3写出去
                context.write(new Text(friend),new Text(splits[0]));
            }
        }
    }
}

```
</details>

<br/>
2.Friend1ReducerTask 实现
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.friend;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/25 10:12
 * Desc TODO
 */
public class Friend1ReducerTask extends Reducer<Text,Text,Text,Text> {
    @Override
    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        String v3 = "";
        //1.遍历好友信息列表
        for(Text friend:values){
            v3 += friend + "-";
        }
        //2.写出去
        context.write(key,new Text(v3));
    }
}

```
</details>

<br/>
3.Friend1JobMain
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.friend;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

/**
 * Author xiaoma
 * Date 2020/8/25 10:17
 * Desc TODO
 */
public class Friend1JobMain {
    public static void main(String[] args) throws Exception {
        //1.创建job对象
        Job job = Job.getInstance(new Configuration(), "Friend1JobMR");
        //2.组装八大步
        //2.1输入
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job,new Path(args[0]));
        //2.2设置mapper
        job.setMapperClass(Friend1MapperTask.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);
        //2.3 shuffle 分区 排序 规约 分组
        //2.7 设置reduce
        job.setReducerClass(Friend1ReducerTask.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        //2.8 输出
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job,new Path(args[1]));
        //3 提交任务
        boolean flag = job.waitForCompletion(true);
        //4 退出执行
        System.exit(flag?0:1);
    }
}

```
</details>

<br/>
4.Friend2MapperTask
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.friend;

import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;
import java.util.Arrays;

/**
 * Author xiaoma
 * Date 2020/8/25 10:39
 * Desc TODO
 */
public class Friend2MapperTask extends Mapper<LongWritable,Text, Text,Text> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString();
        //判空
        if(StringUtils.isNotEmpty(line)){
            //切割
            String[] splits = line.split("\t");
            String[] friends = splits[1].split("-");
            //从小到大排序
            Arrays.sort(friends);
            //遍历 A  B-C-F-G-H-I-K-O
            for(int i=0;i<friends.length-1;i++){
                for(int j=i+1;j<friends.length;j++){
                    String k2 = friends[i] + "-" + friends[j];
                    //写出去
                    context.write(new Text(k2),new Text(splits[0]));
                }
            }
        }
    }
}

```
</details>

<br/>
5.Friend2ReducerTask
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.friend;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

import java.io.IOException;

/**
 * Author xiaoma
 * Date 2020/8/25 10:50
 * Desc TODO
 */
public class Friend2ReducerTask extends Reducer<Text,Text,Text,Text> {
    @Override
    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        String v3="";
        //1.遍历
        for(Text value:values){
            v3 += value+",";
        }
            //2.写出去
        context.write(key, new Text(v3));
    }
}

```
</details>

<br/>
6.Friend2JobMain 实现
<details>
<summary>点击查看代码</summary>

```java
package cn.xiaoma.friend;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

/**
 * Author xiaoma
 * Date 2020/8/25 10:17
 * Desc TODO
 */
public class Friend2JobMain {
    public static void main(String[] args) throws Exception {
        //1.创建job对象
        Job job = Job.getInstance(new Configuration(), "Friend2JobMR");
        //2.组装八大步
        //2.1输入
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job,new Path(args[0]));
        //2.2设置mapper
        job.setMapperClass(Friend2MapperTask.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);
        //2.3 shuffle 分区 排序 规约 分组
        //2.7 设置reduce
        job.setReducerClass(Friend2ReducerTask.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        //2.8 输出
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job,new Path(args[1]));
        //3 提交任务
        boolean flag = job.waitForCompletion(true);
        //4 退出执行
        System.exit(flag?0:1);
    }
}

```
</details>

<br/>

总结： 第一个MR的文件的输出是第二个MR的输入，两个任务是串联的关系。
<br/>

> 社交将某个人所有的好友列表显示出来

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/22.png)

> 将最终两两之间的共同好友展示出来

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/23.png)

### 9.3 倒序索引

#### 9.3.1 倒排索引介绍

> **倒排索引**是文档检索系统中最常用的数据结构，被广泛应用于全文搜索引擎。倒排索引主要用来存储某个单词（或词组）在一组文档中的存储位置的映射，提供了可以根据内容来查找文档的方式，而不是根据文档来确定内容，因此称为倒排索引（Inverted Index）。带有倒排索引的文件我们称为倒排索引文件，简称倒排文件(Inverted File)。

> 通常情况下，倒排文件由一个单词（或词组）和相关联的文档列表组成，如图所示。

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/24.png)

> 从图可以看出，建立倒排索引的目的是为了更加方便的搜索。例如，单词1出现在文档1、文档4、文档13等；单词2出现在文档2、文档6、文档10等；而单词3出现在文档3、文档7等。

> 在实际应用中，还需要给每个文档添加一个权值，用来指出每个文档与搜索内容的相关度。最常用的是使用词频作为权重，即记录单词或词组在文档中出现的次数，用户在搜索相关文档时，就会把权重高的推荐给客户。下面以英文单词倒排索引为例，如图所示。

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/25.png)

> 从图可以看出，加权倒排索引文件中，文件每一行内容对每一个单词进行了加权索引，统计出单词出现的文档和次数。例如索引文件中的第一行，表示“hadoop”这个单词在文本file1.txt中出现过1次，file4.txt中出现过2次，file13.txt中出现过1次。

#### 9.3.2 案例需求及分析

> 现假设有三个源文件a.txt、b.txt和c.txt，需要使用倒排索引的方式对这三个源文件内容实现倒排索引，并将最后的倒排索引文件输出，整个过程要求实现如下转换，如图所示。

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/26.png)

> 接下来，我们就根据上面案例的需求结合倒排索引的实现，对该倒排索引案例的实现进行分析，具体如下。

1. 首先使用默认的TextInputFormat类对每个输入文件进行处理，得到文本中每行的偏移量及其内容。Map过程首先分析输入的<key，value>键值对，经过处理可以得到倒排索引中需要的三个信息：单词、文档名称和词频，如图所示。

![image](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_MapReduce/27.png)

2. 思路分析：

- 首选将文档的内容全部读取出来，加上文档的名字作为key，文档的value为1，组织成这样的一种形式的数据

- map端数据输出
```
hello-a.txt 1
hello-a.txt 1
hello-a.txt 1
```
- reduce端数据输出

```
hello-a.txt 3
```

#### 9.3.3 实现

<br/>
<details>
<summary>点击查看代码</summary>

```java
public class IndexCreate extends Configured implements Tool {
    public static void main(String[] args) throws Exception {
        ToolRunner.run(new Configuration(),new IndexCreate(),args);
    }
    @Override
    public int run(String[] args) throws Exception {
        Job job = Job.getInstance(super.getConf(), IndexCreate.class.getSimpleName());
        job.setInputFormatClass(TextInputFormat.class);
        TextInputFormat.addInputPath(job,new Path("file:///F:\\传智播客大数据离线阶段课程资料\\倒排索引\\input"));
        job.setMapperClass(IndexCreateMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        job.setReducerClass(IndexCreateReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        job.setOutputFormatClass(TextOutputFormat.class);
        TextOutputFormat.setOutputPath(job,new Path("file:///F:\\传智播客大数据离线阶段课程资料\\5、大数据离线第五天\\倒排索引\\outindex"));
        boolean bool = job.waitForCompletion(true);
        return bool?0:1;
    }
    public static class IndexCreateMapper extends Mapper<LongWritable,Text,Text,IntWritable>{
        Text text = new Text();
        IntWritable v = new IntWritable(1);
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            //获取文件切片
            FileSplit fileSplit  = (FileSplit) context.getInputSplit();
            //通过文件切片获取文件名
            String name = fileSplit.getPath().getName();
            String line = value.toString();
            String[] split = line.split(" ");
            //输出 单词--文件名作为key  value是1
            for (String word : split) {
               text.set(word+"--"+name);
                context.write(text,v);
            }
        }
    }
    public static class IndexCreateReducer extends Reducer<Text,IntWritable,Text,IntWritable>{
        IntWritable value = new IntWritable();
        @Override
        protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
            int count = 0;
            for (IntWritable value : values) {
                count += value.get();
            }
            value.set(count);
            context.write(key,value);
        }
    }
}
```
</details>

## 10 MapReduce并行读机制

### 10.1 MapTask并行读机制

> MapTask的并行度指的是map阶段有多少个并行的task共同处理任务。map阶段的任务处理并行度，势必影响到整个job的处理速度。那么，MapTask并行实例是否越多越好呢？其并行度又是如何决定呢？

> 一个MapReducejob的`map阶段并行度由客户端在提交job时决定`，即客户端提交job之前会对待处理数据进行`逻辑切片`。切片完成会形成`切片规划文件（job.split）`，每个逻辑切片最终对应启动一个maptask。

> 逻辑切片机制由FileInputFormat实现类的`getSplits()`方法完成。

#### 10.1.1 FileInputFormat切片机制

> FileInputFormat中默认的切片机制：

- 切片大小，默认等于block大小，即128M
- block是HDFS上物理上存储的存储的数据，切片是对数据逻辑上的划分。
- 在FileInputFormat中，计算切片大小的逻辑：

```
Math.max(minSize, Math.min(maxSize, blockSize));  
```
- 切片举例

比如待处理数据有两个文件：

```
file1.txt 320M
file2.txt 10M	
```

经过FileInputFormat的切片机制运算后，形成的切片信息如下： 

```
file1.txt.split1—0M~128M
file1.txt.split2—128M~256M
file1.txt.split3—256M~320M
file2.txt.split1—0M~10M 
```

`FileInputFormat中切片的大小的由这几个值来运算决定`：

在 FileInputFormat 中，计算切片大小的逻辑： 
```
long splitSize = computeSplitSize(blockSize, minSize, maxSize)
```
切片主要由这几个值来运算决定：

```
blocksize：默认是 128M，可通过 dfs.blocksize 修改

minSize：默认是 1，可通过 mapreduce.input.fileinputformat.split.minsize 修改

maxsize：默认是 Long.MaxValue，可通过 mapreduce.input.fileinputformat.split.maxsize 修改
```
如果设置的最大值maxsize比blocksize值小，则按照maxSize切数据

如果设置的最小值minsize比blocksize值大，则按照minSize切数据

但是，不论怎么调参数，都不能让多个小文件“划入”一个 split

#### 10.1.1 FileInputFormat切片参数设置

第一种情况（切片大小为256M）：

```
FileInputFormat.setInputPaths(job, new Path(input));
FileInputFormat. setMaxInputSplitSize(job,1024*1024*500) ; //设置最大分片大小
FileInputFormat.setMinInputSplitSize(job,1024*1024*256); //设置最小分片大小
```

第二种情况(切片大小为100M)：

```
FileInputFormat.setInputPaths(job, new Path(input));
FileInputFormat.setMaxInputSplitSize(job,1024*1024*100) ; //设置最大分片大小
FileInputFormat.setMinInputSplitSize(job,1024*1024*80); //设置最小分片大小
```

整个切片的核心过程在getSplit()方法中完成。

数据切片只是在逻辑上对输入数据进行分片，并不会再磁盘上将其切分成分片进行存储。InputSplit只记录了分片的元数据信息，比如起始位置、长度以及所在的节点列表等。

### 10.2 Reducetask并行度机制

reducetask并行度同样影响整个job的执行并发度和执行效率，与maptask的并发数由切片数决定不同，`Reducetask数量的决定是可以直接手动设置`：

```
job.setNumReduceTasks(4);
```
如果数据分布不均匀，就有可能在reduce阶段产生数据倾斜。

注意： reducetask数量并不是任意设置，还要考虑业务逻辑需求，有些情况下，需要计算全局汇总结果，就只能有1个reducetask。

## 11 MapReduce性能优化策略

> 使用Hadoop进行大数据运算，当数据量极其大时，那么对MapReduce性能的调优重要性不言而喻，尤其是Shuffle过程中的参数配置对作业的总执行时间影响特别大。下面总结一些和MapReduce相关的性能调优方法，主要从五个方面考虑：`数据输入`、`Map阶段`、`Reduce阶段`、`Shuffle阶段`和`其他调优属性`。

### 11.1 数据输入

> 在执行MapReduce任务前，将小文件进行合并，大量的小文件会产生大量的map任务，增大map任务装载的次数，而任务的装载比较耗时，从而导致MapReduce运行速度较慢。因此我们采用CombineTextInputFormat来作为输入，解决输入端大量的小文件场景。

### 11.2 Map阶段

> （1）减少溢写（spill）次数：通过调整io.sort.mb及sort.spill.percent参数值，增大触发spill的内存上限，减少spill次数，从而减少磁盘IO。

> （2）减少合并（merge）次数：通过调整io.sort.factor参数，增大merge的文件数目，减少merge的次数，从而缩短mr处理时间。

> （3）在map之后，不影响业务逻辑前提下，先进行combine处理，减少 I/O。
我们在上面提到的那些属性参数，都是位于mapred-default.xml文件中，这些属性参数的调优方式如表所示。


属性名称 | 类型 | 默认值 | 说明
---|---|---|---
mapreduce.task.io.sort.mb &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | int &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  | 100 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  | 配置排序map输出时使用的内存缓冲区的大小，默认100Mb，实际开发中可以设置大一些。 
mapreduce.map.sort.spill.percent | float  | 0.80  | map输出内存缓冲和用来开始磁盘溢出写过程的记录边界索引的阈值，即最大使用环形缓冲内存的阈值。一般默认是80%。也可以直接设置为100%  
mapreduce.task.io.sort.factor | int  | 10  | 排序文件时，一次最多合并的流数，实际开发中可将这个值设置为100。 
mapreduce.task.min.num.spills.for.combine | int  | 3  | 运行combiner时，所需的最少溢出文件数(如果已指定combiner) 


### 11.3 Reduce阶段

> （1）合理设置map和reduce数：两个都不能设置太少，也不能设置太多。太少，会导致task等待，延长处理时间；太多，会导致 map、reduce任务间竞争资源，造成处理超时等错误。

> （2）设置map、reduce共存：调整slowstart.completedmaps参数，使map运行到一定程度后，reduce也开始运行，减少reduce的等待时间。

> （3）规避使用reduce：因为reduce在用于连接数据集的时候将会产生大量的网络消耗。通过将MapReduce参数setNumReduceTasks设置为0来创建一个只有map的作业。

> （4）合理设置reduce端的buffer：默认情况下，数据达到一个阈值的时候，buffer中的数据就会写入磁盘，然后reduce会从磁盘中获得所有的数据。也就是说，buffer和reduce是没有直接关联的，中间多一个写磁盘->读磁盘的过程，既然有这个弊端，那么就可以通过参数来配置，使得buffer中的一部分数据可以直接输送到reduce，从而减少IO开销。这样一来，设置buffer需要内存，读取数据需要内存，reduce计算也要内存，所以要根据作业的运行情况进行调整。

我们在上面提到的属性参数，都是位于mapred-default.xml文件中，这些属性参数的调优方式如表所示。


属性名称 | 类型 | 默认值 | 说明
---|---|---|---
mapreduce.job.reduce.slowstart.completedmaps &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | float &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | 0.05  &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | 当map task在执行到5%，就开始为reduce申请资源。开始执行reduce操作，reduce可以开始拷贝map结果数据和做reduce shuffle操作。
mapred.job.reduce.input.buffer.percent | float | 0.0 | 在reduce过程，内存中保存map输出的空间占整个堆空间的比例。如果reducer需要的内存较少，可以增加这个值来最小化访问磁盘的次数。 

### 11.4 Shuffle阶段

> Shuffle阶段的调优就是给Shuffle过程尽量多地提供内存空间，以防止出现内存溢出现象，可以由参数mapred.child.java.opts来设置，任务节点上的内存大小应尽量大。

> 我们在上面提到的属性参数，都是位于mapred-site.xml文件中，这些属性参数的调优方式如表所示。

属性名称 | 类型 | 默认值 | 说明
---|---|---|---
mapred.map.child.java.opts &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | | -Xmx200m &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | 当用户在不设置该值情况下，会以最大1G jvm heap size启动map task，有可能导致内存溢出，所以最简单的做法就是设大参数，一般设置为-Xmx1024m
mapred.reduce.child.java.opts |  | -Xmx200m | 当用户在不设置该值情况下，会以最大1G jvm heap size启动Reduce task，也有可能导致内存溢出，所以最简单的做法就是设大参数，一般设置为-Xmx1024m 

### 11.5 其他调优属性

除此之外，MapReduce还有一些基本的资源属性的配置，这些配置的相关参数都位于mapred-default.xml文件中，我们可以合理配置这些属性提高MapReduce性能，表4-4列举了部分调优属性。

属性名称 | 类型 | 默认值 | 说明
---|---|---|---
mapreduce.map.memory.mb &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | int &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | 1024 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; | 一个Map Task可使用的资源上限。如果Map Task实际使用的资源量超过该值，则会被强制杀死。
mapreduce.reduce.memory.mb | int | 1024 | 一个Reduce Task可使用的资源上限。如果Reduce Task实际使用的资源量超过该值，则会被强制杀死。
mapreduce.map.cpu.vcores | int | 1 | 每个Map task可使用的最多cpu core数目
mapreduce.reduce.cpu.vcores | int | 1 | 每个Reduce task可使用的最多cpu core数目
mapreduce.reduce.shuffle.parallelcopies | int | 5 | 每个reduce去map中拿数据的并行数。
mapreduce.map.maxattempts | int | 4 | 每个Map Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败
mapreduce.reduce.maxattempts | int | 4 | 每个Reduce Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败


## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
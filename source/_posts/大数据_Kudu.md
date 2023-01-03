---
title: Kudu 开源的分布式数据存储引擎
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(66).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(66).jpg
tags:
  - 大数据
  - Kudu
category:
  - - 编程
    - 大数据
date: 2020-07-14 15:48:38
---

Apache Kudu 是一个`开源的分布式数据存储引擎`，使得`快速分析和更改数据变得容易`。

Kudu `提供了快速插入/更新和高效的柱状扫描的组合`，以`支持跨一个存储层的多个实时分析工作负载`。Kudu 为架构师提供了灵活性，可以处理更多种类的用例，而不需要特殊的解决方案和必需的外部服务依赖。

Kudu 是专门`为需要对快速(快速变化的)数据进行快速分析的用例而设计的`。Kudu 利用下一代硬件和内存处理的优势，显著降低了 Apache Impala、 Apache NiFi、 Apache Spark、 Apache Flink 等引擎的查询延迟。

Apache Kudu 由 `Apache 大数据生态系统的长期贡献者建立`，是 Apache 2许可下发布的`顶级 Apache软件基金会项目`，重视社区参与作为其长期成功的一个重要因素。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 ETL实现方案

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/1.png)

### 1.1 ETL处理流程图

### 1.2 为什么使用Kudu作为存储介质

> `数据库数据上的快速分析`
目前很多业务使用事务型数据库（MySQL、Oracle）做数据分析，把数据写入数据库，然后使用 SQL 进行有效信息提取，当数据规模很小的时候，这种方式确实是立竿见影的，但是当数据量级起来以后，会发现数据库吃不消了或者成本开销太大了，此时就需要把数据从事务型数据库里拷贝出来或者说剥离出来，装入一个分析型的数据库里。发现对于实时性和变更性的需求，目前只有 Kudu 一种组件能够满足需求，所以就产生了这样的一种场景：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/2.png)

> MySQL 数据库增、删、改的数据通过 Binlog 实时的被同步到 Kudu 里，同时在 Impala（或者其他计算引擎如 Spark、Hive、Presto、MapReduce）上可以实时的看到。

> 这种场景也是目前业界使用最广泛的，认可度最高。

> `用户行为日志的快速分析`

> 对于用户行为日志的实时性敏感的业务，比如电商流量、AB 测试、优惠券的点击反馈、广告投放效果以及秒级导入秒级查询等需求，按 Kudu 出现以前的架构基本上都是这张图的模式：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/3.png)

> 不仅链路长而且实时性得不到有力保障，有些甚至是 T + 1 的，极大的削弱了业务的丰富度。

> 引入 Kudu 以后，大家看，数据的导入和查询都是在线实时的：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/4.png)

> 这种场景目前也是网易考拉和hub在使用的，其中hub甚至把 Kudu 当 HBase 来作点查使用。

## 2 Kudu入门

### 2.1 Kudu介绍

#### 2.1.1 背景介绍

> 在Kudu之前，大数据主要以两种方式存储；

> `静态数据`：
>> 以 HDFS 引擎作为存储引擎，适用于高吞吐量的离线大数据分析场景。
> 
>> 这类存储的局限性是数据无法进行随机的读写。

> 动态数据：
>> 以 HBase、Cassandra 作为存储引擎，适用于大数据随机读写场景。
> 
>> 这类存储的局限性是批量读取吞吐量远不如 HDFS，不适用于批量数据分析的场景。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/5.png)

> 从上面分析可知，这两种数据在`存储方式上`完全不同，进而导致使用场景完全不同，但在真实的场景中，边界可能没有那么清晰，面对既需要随机读写，又需要批量分析的大数据场景，该如何选择呢？这个场景中，单种存储引擎无法满足业务需求，我们需要通过多种大数据工具组合来满足这一需求。

> 如上图所示，数据实时写入 HBase，实时的数据更新也在 HBase 完成，为了应对 OLAP 需求，我们定时（通常是 T+1 或者 T+H）将 HBase 数据写成静态的文件（如：Parquet）导入到 OLAP 引擎（如：HDFS）。这一架构能满足既需要随机读写，又可以支持 OLAP 分析的场景，但它有如下缺点：
>> `架构复杂`。从架构上看，数据在HBase、消息队列、HDFS 间流转，涉及环节太多，运维成本很高。并且每个环节需要保证高可用，都需要维护多个副本，存储空间也有一定的浪费。最后数据在多个系统上，对数据安全策略、监控等都提出了挑战。
> 
>> `时效性低`。数据从HBase导出成静态文件是周期性的，一般这个周期是一天（或一小时），在时效性上不是很高。
> 
>> `难以应对后续的更新`。真实场景中，总会有数据是延迟到达的。如果这些数据之前已经从HBase导出到HDFS，新到的变更数据就难以处理了，一个方案是把原有数据应用上新的变更后重写一遍，但这代价又很高。

> 为了解决上述架构的这些问题，Kudu应运而生。`Kudu的定位是Fast Analytics on Fast Data，是一个既支持随机读写、又支持 OLAP 分析的大数据存储引擎`。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/6.png)

> 从上图可以看出，KUDU 是一个折中的产品，在 HDFS 和 HBase 这两个偏科生中平衡了随机读写和批量分析的性能。从 KUDU 的诞生可以说明一个观点：`底层的技术发展很多时候都是上层的业务推动的，脱离业务的技术很可能是空中楼阁`。

#### 2.1.2 新的硬件设备

> 内存（RAM）的技术发展非常快，它变得越来越便宜，容量也越来越大。Cloudera的客户数据显示，他们的客户所部署的服务器，2012年每个节点仅有32GB RAM，现如今增长到每个节点有128GB或256GB RAM。存储设备上更新也非常快，在很多普通服务器中部署SSD也是屡见不鲜。HBase、HDFS、以及其他的Hadoop工具都在不断自我完善，从而适应硬件上的升级换代。然而，从根本上，HDFS基于03年GFS，HBase基于05年BigTable，在当时系统瓶颈主要取决于底层磁盘速度。当磁盘速度较慢时，CPU利用率不足的根本原因是磁盘速度导致的瓶颈，当磁盘速度提高了之后，CPU利用率提高，这时候CPU往往成为系统的瓶颈。HBase、HDFS由于年代久远，已经很难从基本架构上进行修改，而Kudu是基于全新的设计，因此可以更充分地利用RAM、I/O资源，并优化CPU利用率。

> 我们可以理解为：`Kudu相比与以往的系统，CPU使用降低了，I/O的使用提高了，RAM的利用更充分了`。

#### 2.1.3 Kudu是什么

> Apache Kudu是由Cloudera开源的存储引擎，可以同时提供低延迟的随机读写和高效的数据分析能力。它是一个融合HDFS和HBase的功能的新组件，具备介于两者之间的新存储组件。

> Kudu支持水平扩展，并且`与Cloudera Impala和Apache Spark`等当前流行的大数据查询和分析工具结合紧密。

#### 2.1.4 Kudu的应用场景

> Kudu的很多特性跟HBase很像，它支持索引键的查询和修改。Cloudera曾经想过基于Hbase进行修改，然而结论是对HBase的改动非常大，Kudu的数据模型和磁盘存储都与Hbase不同。HBase本身成功的适用于大量的其它场景，因此修改HBase很可能吃力不讨好。最后Cloudera决定开发一个全新的存储系统。
>> Strong performance for both scan and random access to help customers simplify complex hybrid architectures（`适用于那些既有随机访问，也有批量数据扫描的复合场景`）
> 
>> High CPU efficiency in order to maximize the return on investment that our customers are making in modern processors（`高计算量的场景`）
> 
>> High IO efficiency in order to leverage modern persistent storage（`使用了高性能的存储设备，包括使用更多的内存`）
> 
>> The ability to upDATE data in place, to avoid extraneous processing and data movement（`支持数据更新，避免数据反复迁移`）
> 
>> The ability to support active-active replicated clusters that span multiple data centers in geographically distant locations（`支持跨地域的实时数据备份和查询`）

#### 2.1.5 Kudu的架构

> 下图显示了一个具有三个 `master` 和多个 `tablet server` 的 `Kudu 集群`，每个服务器都支持多个 tablet。

> 它说明了如何使用 Raft 共识来允许 master 和 tablet server 的 `leader` 和 `follow`。

> 此外，tablet server 可以成为某些 tablet 的 leader，也可以是其他 tablet 的 follower。leader 以金色显示，而 follower 则显示为蓝色。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/7.png)

> 下面是一些基本概念：

角色 | 作用
---|---
Master | 集群中的老大，负责集群管理、元数据管理等功能
Tablet Server | 集群中的小弟，负责数据存储，并提供数据读写服务<br/>一个 tablet server 存储了table表的tablet 和为 tablet 向 client 提供服务。对于给定的 tablet，一个tablet server 充当 leader，其他 tablet server 充当该 tablet 的 follower 副本。<br/>只有 leader服务写请求，然而 leader 或 followers 为每个服务提供读请求 。一个 tablet server 可以服务多个 tablets ，并且一个 tablet 可以被多个 tablet servers 服务着。
Table（表） | 一张table是数据存储在Kudu的tablet server中。表具有 schema 和全局有序的primary key（主键）。table 被分成称为 tablets 的 segments。
Tablet | 一个 tablet 是一张 table连续的segment，tablet是kudu表的水平分区，类似于google Bigtable的tablet，或者HBase的region。每个tablet存储着一定连续range的数据（key），且tablet两两间的range不会重叠。一张表的所有tablet包含了这张表的所有key空间。与其它数据存储引擎或关系型数据库中的 partition（分区）相似。给定的tablet 冗余到多个 tablet 服务器上，并且在任何给定的时间点，其中一个副本被认为是leader tablet。任何副本都可以对读取进行服务，并且写入时需要在为 tablet 服务的一组 tablet server之间达成一致性。

### 2.2 Kudu的分区

> 为了提供可扩展性，Kudu 表被划分为称为 tablets 的单元，并分布在许多 tablet servers 上。行总是属于单个tablet 。将行分配给 tablet 的方法由在表创建期间设置的表的分区决定。

> kudu提供了3种分区方式。

#### 2.2.1 Hash Partitioning(哈希分区)

> 哈希分区通过哈希值将行分配到许多 buckets ( 存储桶 )之一； 哈希分区是一种有效的策略，当不需要对表进行有序访问时。哈希分区对于在 tablet 之间随机散布这些功能是有效的，这有助于减轻热点和 tablet 大小不均匀。

> `优点`：可以解决数据的热点问题（数据倾斜）

> `缺点`：数据是无序的

#### 2.2.2 Range Partitioning(范围分区)

> 范围分区可以根据存入数据的数据量，均衡的存储到各个机器上，防止机器出现负载不均衡现象.

> `优点`：可以解决数据全局有序问题

> `缺点`：导致数据热点问题

#### 2.2.3 Multilevel Partitioning(多级分区)

> Kudu 允许一个表在单个表上组合多级分区。

> 当正确使用时，多级分区可以保留各个分区类型的优点，同时减少每个分区的缺点 需求.

> `hashPartition+ranagePartiotin=混合分区`

> `优点`：既解决了热点问题也解决了数据有序问题

### 2.3 Java操作Kudu

#### 2.3.1 pom

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.xiaoma</groupId>
    <artifactId>xiaoma_kudu</artifactId>
    <version>1.0-SNAPSHOT</version>

    <repositories>
        <repository>
            <id>cloudera</id>
            <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.apache.kudu</groupId>
            <artifactId>kudu-client</artifactId>
            <version>1.9.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.kudu</groupId>
            <artifactId>kudu-client-tools</artifactId>
            <version>1.9.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.kudu/kudu-spark2 -->
        <dependency>
            <groupId>org.apache.kudu</groupId>
            <artifactId>kudu-spark2_2.11</artifactId>
            <version>1.9.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-sql -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.11</artifactId>
            <version>2.1.0</version>
        </dependency>

        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>RELEASE</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>
</project>
```

#### 2.3.2 创建表

```
package cn.xiaoma;

import org.apache.kudu.ColumnSchema;
import org.apache.kudu.Schema;
import org.apache.kudu.Type;
import org.apache.kudu.client.CreateTableOptions;
import org.apache.kudu.client.KuduClient;
import org.apache.kudu.client.KuduException;
import org.junit.Before;
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;

/**
 * 创建一张kudu表
 */
public class CreateTable {
    //定义kudu的连接地址
    private static String masterAddress = "node2.xiaoma.cn";
    //定义kudu操作的客户端对象
    private static KuduClient kuduClient;
    //定义kudu表名
    private static String tableName = "person1";

    /**
     * 初始化方法
     */
    @Before
    public void  init(){
        System.out.println("---初始化kuduclient对象--");
        //实例化kudu的客户端对象
        kuduClient = new KuduClient.KuduClientBuilder(masterAddress).defaultSocketReadTimeoutMs(60000).build();
    }

    /**
     * 添加列
     * @param name 列名
     * @param type 列的类型
     * @param isKey 是否主键
     * @return
     */
    private ColumnSchema newColumn(String name, Type type, Boolean isKey){
        //创建列的描述对象
        ColumnSchema.ColumnSchemaBuilder columnSchemaBuilder = new ColumnSchema.ColumnSchemaBuilder(name, type);
        columnSchemaBuilder.key(isKey);
        return columnSchemaBuilder.build();
    }
    /**
     * 创建表
     * 创建表的时候需要指定表的分区方式：hash分区、ranage分区、混合分区
     * kudu的表是多分区多副本
     */
    @Test
    public void CreateTable(){
        //构建表的列的集合
        List<ColumnSchema> columnSchemaList = new ArrayList<ColumnSchema>();
        columnSchemaList.add(newColumn("id", Type.INT32, true));
        columnSchemaList.add(newColumn("name", Type.STRING, false));
        columnSchemaList.add(newColumn("gender", Type.STRING, false));
        columnSchemaList.add(newColumn("phone", Type.STRING, false));

        //创建表结构（列的信息）
        Schema schema = new Schema(columnSchemaList);

        //创建表选项对象
        CreateTableOptions createTableOptions = new CreateTableOptions();

        //指定分区的列
        List<String> partitionColumnList = new ArrayList<String>();
        partitionColumnList.add("id");

        //指定分区的方式：按照hash分区，同时指定分区的字段是哪个
        createTableOptions.addHashPartitions(partitionColumnList, 3);
        //副本数量要小于等于从节点数量，因此设置3个副本一定报错
        //org.apache.kudu.client.NonRecoverableException: not enough live tablet servers to create a table with the requested replication factor 3; 1 tablet servers are alive
        createTableOptions.setNumReplicas(1);

        try {
            //创建表
            kuduClient.createTable(tableName, schema, createTableOptions);
        } catch (KuduException e) {
            e.printStackTrace();
        }
    }
}
```

#### 2.3.3 数据操作

```
package cn.xiaoma;

import org.apache.kudu.client.*;
import org.junit.Before;
import org.junit.Test;

/**
 * kudu表的操作
 * 1）写入数据
 * 2）查询数据
 * 3）修改数据
 * 4）删除数据
 */
public class KuduDemo {
    //定义kudu的连接地址
    private static String masterAddress = "node2.xiaoma.cn";
    //定义kudu操作的客户端对象
    private static KuduClient kuduClient;
    //定义kudu表名
    private static String tableName = "person";

    /**
     * 初始化方法
     */
    @Before
    public void  init(){
        System.out.println("---初始化kuduclient对象--");
        //实例化kudu的客户端对象
        kuduClient = new KuduClient.KuduClientBuilder(masterAddress).defaultSocketReadTimeoutMs(60000).build();
    }

    /**
     * 加载数据
     */
    @Test
    public  void  loadData() throws KuduException {
        //打开表
        KuduTable kuduTable = kuduClient.openTable(tableName);

        //创建kudusession对象，通过kudusession对象写入数据
        KuduSession kuduSession = kuduClient.newSession();
        
        //准备数据进行写入
        for (int i = 0; i <100 ; i++) {
            //基于表对象创建插入对象
            Insert insert = kuduTable.newInsert();
            insert.getRow().addInt("id", i+1);
            insert.getRow().addString("name", "zhangsan");
            insert.getRow().addString("gender", "male");
            insert.getRow().addString("phone", "12213213232");
            //kuduSession.flush();
            kuduSession.apply(insert);
        }
        kuduSession.close();
        kuduClient.close();
    }

    /**
     * 查询数据
     */
    @Test
    public void queryData() throws KuduException {
        //打开表
        KuduTable kuduTable = kuduClient.openTable(tableName);

        //创建scanBuilder对象扫描数据
        KuduScanner.KuduScannerBuilder scannerBuilder = kuduClient.newScannerBuilder(kuduTable);
        KuduScanner scanner = scannerBuilder.build();

        //遍历数据
        while (scanner.hasMoreRows()){
            //返回行集合数据
            RowResultIterator rowResults = scanner.nextRows();
            while (rowResults.hasNext()){
                RowResult result = rowResults.next();
                System.out.println(result.getInt("id"));
                System.out.println(result.getString("name"));
                System.out.println(result.getString("gender"));
                System.out.println(result.getString("phone"));
            }
        }

        //释放资源
        scanner.close();
        kuduClient.close();
    }

    /**
     * 修改数据
     */
    @Test
    public void updateData() throws KuduException {
        //打开表
        KuduTable kuduTable = kuduClient.openTable(tableName);

        //创建kudusession对象
        KuduSession kuduSession = kuduClient.newSession();
        //创建修改对象
        Update update = kuduTable.newUpdate();
        //获取到指定行的数据，修改其字段
        PartialRow row = update.getRow();
        row.addInt("id", 100);
        row.addString("name", "lisi");
        row.addString("phone", "1111111111");

        //操作update对象
        kuduSession.apply(update);
        kuduSession.close();
        kuduClient.close();
    }

    /**
     * 删除数据
     */
    @Test
    public void deleteData() throws KuduException {
        //打开表
        KuduTable kuduTable = kuduClient.openTable(tableName);

        KuduSession kuduSession = kuduClient.newSession();
        Delete delete = kuduTable.newDelete();

        //获取要删除的行数据
        PartialRow row = delete.getRow();
        row.addInt("id", 1);
        kuduSession.apply(delete);
        kuduSession.close();
        kuduClient.close();
    }


    /**
     * 修改数据
     * 如果主键不存在，则执行数据的插入操作
     * 如果主键存在，则进行数据的更新操作
     *
     * 定义表的时候，一旦指定了字段不能为空，那么在upsert的时候，该字段一定要赋值（否则插入和更新操作都会失败）
     */
    @Test
    public void upSertData() throws KuduException {
        //打开表
        KuduTable kuduTable = kuduClient.openTable(tableName);

        //创建kudusession对象
        KuduSession kuduSession = kuduClient.newSession();
        //创建修改对象
        Upsert upsert = kuduTable.newUpsert();
        //获取到指定行的数据，修改其字段
        PartialRow row = upsert.getRow();
        row.addInt("id", 100);
        row.addString("name", "zhangsan");
        row.addString("gender", "male");
        row.addString("phone", "1111111111");

        //操作update对象
        kuduSession.apply(upsert);
        kuduSession.close();
        kuduClient.close();
    }
}

```

#### 2.3.4 表操作

```
package cn.xiaoma;

import org.apache.kudu.ColumnSchema;
import org.apache.kudu.Schema;
import org.apache.kudu.Type;
import org.apache.kudu.client.*;
import org.junit.Before;
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;

/**
 * 修改表操作
 * 1）添加列
 * 2）删除列
 * 3）添加新的分区
 * 4）删除表
 */
public class AlterTable {
    //定义kudu的连接地址
    private static String masterAddress = "node2.xiaoma.cn";
    //定义kudu操作的客户端对象
    private static KuduClient kuduClient;
    //定义kudu表名
    private static String tableName = "person";

    /**
     * 初始化方法
     */
    @Before
    public void  init(){
        System.out.println("---初始化kuduclient对象--");
        //实例化kudu的客户端对象
        kuduClient = new KuduClient.KuduClientBuilder(masterAddress).defaultSocketReadTimeoutMs(60000).build();
    }

    /**
     * 添加列
     */
    @Test
    public void alterTableAddColumn(){
        AlterTableOptions alterTableOptions = new AlterTableOptions();
        //添加一个叫做address的新列，可以为空
        alterTableOptions.addColumn(new ColumnSchema.ColumnSchemaBuilder("address", Type.STRING).nullable(true).build());

        try {
            kuduClient.alterTable(tableName,alterTableOptions);
        } catch (KuduException e) {
            e.printStackTrace();
        }
    }

    /**
     * 删除列
     */
    @Test
    public void alterTableDeleteColumn(){
        AlterTableOptions alterTableOptions = new AlterTableOptions().dropColumn("address");

        try {
            kuduClient.alterTable(tableName, alterTableOptions);
        } catch (KuduException e) {
            e.printStackTrace();
        }
    }

    /**
     * 添加新的分区
     */
    @Test
    public void alterTableAddRangePartition(){
        int lowerValue = 110;
        int upperValue = 120;
        try {
            //打开表
            KuduTable kuduTable = kuduClient.openTable("person_range");

            boolean flag = true;
            //查询到所有的分区(分区列表)
            List<Partition> rangePartitions = kuduTable.getRangePartitions(6000);
            //遍历所有的分区列表
            for(Partition partition : rangePartitions){
                //获取到每个分区的下界（起始值）
                int startKey = partition.getDecodedRangeKeyStart(kuduTable).getInt("id");
                int endKey = partition.getDecodedRangeKeyEnd(kuduTable).getInt("id");
                if(startKey == lowerValue){
                    //说明下界存在（分区已经创建过）
                    flag = false;
                }
            }
//            //确定分区的起始范围
//            Schema schema = kuduTable.getSchema();
//            PartialRow lower = new PartialRow(schema);
//            lower.addInt("id", count);
            if(flag) {
                //指定范围分区的下界
                PartialRow lower = kuduTable.getSchema().newPartialRow();
                lower.addInt("id", lowerValue);

                //指定范围分区的上界
                PartialRow upper = kuduTable.getSchema().newPartialRow();
                upper.addInt("id", upperValue);

                kuduClient.alterTable("person_range", new AlterTableOptions().addRangePartition(lower, upper));
            }else {
                System.out.println("表分区已经创建过，无需再次创建....");
            }
        } catch (KuduException e) {
            e.printStackTrace();
        } catch (Exception exception) {
            exception.printStackTrace();
        }
    }

    /**
     * 删除表
     */
    @Test
    public void dropTable() throws KuduException {
        kuduClient.deleteTable(tableName);
    }
}
```

#### 2.3.5 分区演示

```
package cn.xiaoma;

import org.apache.kudu.ColumnSchema;
import org.apache.kudu.Schema;
import org.apache.kudu.Type;
import org.apache.kudu.client.CreateTableOptions;
import org.apache.kudu.client.KuduClient;
import org.apache.kudu.client.KuduException;
import org.apache.kudu.client.PartialRow;
import org.junit.Before;
import org.junit.Test;


import javax.print.DocFlavor;
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

/**
 * kudu的分区策略
 * 1）hash分区
 * 2）rangePartition分区
 * 3）混合分区
 */
public class KuduPartition {
    //定义kudu的连接地址
    private static String masterAddress = "node2.xiaoma.cn";
    //定义kudu操作的客户端对象
    private static KuduClient kuduClient;
    //定义kudu表名
    private static String tableName = "person";

    /**
     * 初始化方法
     */
    @Before
    public void  init(){
        System.out.println("---初始化kuduclient对象--");
        //实例化kudu的客户端对象
        kuduClient = new KuduClient.KuduClientBuilder(masterAddress).defaultSocketReadTimeoutMs(60000).build();
    }

    /**
     * 添加列
     * @param name 列名
     * @param type 列的类型
     * @param isKey 是否主键
     * @return
     */
    private ColumnSchema newColumn(String name, Type type, Boolean isKey){
        //创建列的描述对象
        ColumnSchema.ColumnSchemaBuilder columnSchemaBuilder = new ColumnSchema.ColumnSchemaBuilder(name, type);
        columnSchemaBuilder.key(isKey);
        return columnSchemaBuilder.build();
    }
    /**
     * hash分区
     */
    @Test
    public void testHashPartition(){
        //构建表的列的集合
        List<ColumnSchema> columnSchemaList = new ArrayList<ColumnSchema>();
        columnSchemaList.add(newColumn("id", Type.INT32, true));
        columnSchemaList.add(newColumn("name", Type.STRING, false));
        columnSchemaList.add(newColumn("gender", Type.STRING, false));
        columnSchemaList.add(newColumn("phone", Type.STRING, false));

        //创建表结构（列的信息）
        Schema schema = new Schema(columnSchemaList);

        //创建表选项对象
        CreateTableOptions createTableOptions = new CreateTableOptions();

        //指定分区的列
        List<String> partitionColumnList = new ArrayList<String>();
        partitionColumnList.add("id");

        //指定分区的方式：按照hash分区，同时指定分区的字段是哪个
        createTableOptions.addHashPartitions(partitionColumnList, 3);
        //副本数量要小于等于从节点数量，因此设置3个副本一定报错
        //org.apache.kudu.client.NonRecoverableException: not enough live tablet servers to create a table with the requested replication factor 3; 1 tablet servers are alive
        createTableOptions.setNumReplicas(1);

        try {
            //创建表
            kuduClient.createTable(tableName, schema, createTableOptions);
        } catch (KuduException e) {
            e.printStackTrace();
        }
    }

    /**
     * 范围分区
     * 分区1：1-100
     * 分区2：101-200
     * 分区3: 201-300...
     */
    @Test
    public void testRangePartition(){
        //构建表的列的集合
        List<ColumnSchema> columnSchemaList = new ArrayList<ColumnSchema>();
        columnSchemaList.add(newColumn("id", Type.INT32, true));
        columnSchemaList.add(newColumn("name", Type.STRING, false));
        columnSchemaList.add(newColumn("gender", Type.STRING, false));
        columnSchemaList.add(newColumn("phone", Type.STRING, false));

        //创建表结构（列的信息）
        Schema schema = new Schema(columnSchemaList);

        //创建表选项对象
        CreateTableOptions createTableOptions = new CreateTableOptions();

        //指定分区的列
        List<String> partitionColumnList = new ArrayList<String>();
        partitionColumnList.add("id");

        //指定分区的方式：按照range分区，同时指定分区的字段是哪个
        createTableOptions.setRangePartitionColumns(partitionColumnList);

        //副本数量要小于等于从节点数量，因此设置3个副本一定报错
        //org.apache.kudu.client.NonRecoverableException: not enough live tablet servers to create a table with the requested replication factor 3; 1 tablet servers are alive
        createTableOptions.setNumReplicas(1);

        try {
            int count = 0;
            for (int i = 0; i < 10; i++) {
                //确定分区的起始范围
                PartialRow lower = new PartialRow(schema);
                lower.addInt("id", count);

                //确定分区的结束范围
                PartialRow upper = new PartialRow(schema);
                count+= 10;
                upper.addInt("id", count);
                createTableOptions.addRangePartition(lower, upper);
            }
            //创建表
            kuduClient.createTable(tableName+"_range", schema, createTableOptions);
        } catch (KuduException e) {
            e.printStackTrace();
        }
    }

    /**
     * 混合分区：hash+range
     * hash分区：
     *  优点：吞吐量高，有利于提高数据的写入的吞吐量
     * range分区：
     *  优点：可以保证数据的全局有序
     *  缺点：会因为指定范围内的数据不均衡导致数据的倾斜
     * 混合分区：
     *  既可以解决数据热点问题也能解决吞吐量问题，从而提高kudu的性能
     */
    @Test
    public void testMultilevelPartition() throws KuduException {
        //构建表的列的集合
        List<ColumnSchema> columnSchemaList = new ArrayList<ColumnSchema>();
        columnSchemaList.add(newColumn("id", Type.INT32, true));
        columnSchemaList.add(newColumn("name", Type.STRING, false));
        columnSchemaList.add(newColumn("gender", Type.STRING, false));
        columnSchemaList.add(newColumn("phone", Type.STRING, false));

        //创建表结构（列的信息）
        Schema schema = new Schema(columnSchemaList);

        //创建表选项对象
        CreateTableOptions createTableOptions = new CreateTableOptions();

        //指定需要分区的列
        List<String> columns = new LinkedList<String>();
        columns.add("id");

        //设置副本数量
        createTableOptions.setNumReplicas(1);
        //指定范围分区的规则
        createTableOptions.addHashPartitions(columns, 3);
        int count = 0;
        for (int i = 0; i < 3; i++) {
            //确定分区的起始范围
            PartialRow lower = new PartialRow(schema);
            lower.addInt("id", count);

            //确定分区的结束范围
            PartialRow upper = new PartialRow(schema);
            count+= 10;
            upper.addInt("id", count);
            createTableOptions.addRangePartition(lower, upper);
        }

        try {
            kuduClient.createTable(tableName+"_multilevel", schema, createTableOptions);
        } catch (KuduException e) {
            e.printStackTrace();
        }
        kuduClient.close();
    }
}
```

### 2.4 Spark操作Kudu

```
package cn.xiaoma

import java.util

import org.apache.kudu.{ColumnSchema, Schema, Type}
import org.apache.kudu.client.{AlterTableOptions, CreateTableOptions, KuduClient}
import org.apache.kudu.spark.kudu.{KuduContext, KuduWriteOptions}
import org.apache.parquet.schema.Types
import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.{DataFrame, Row, SaveMode, SparkSession}
import org.apache.spark.sql.types.{IntegerType, StringType, StructField, StructType}

/**
 * 使用spark操作kudu
 * 1）创建表
 * 2）插入数据
 * 3）删除数据
 * 4）修改数据
 * 5）使用dataFrame读取kudu表数据，dataFrame=schema+rdd
 * 6）将dataFrame的数据写入到kudu表中
 * 7）使用sparksql将数据写入到kudu表中
 * 8）使用kudu原生的kuduRdd操作kudu
 * 9) 使用spark修改kudu表
 */
object SparkDemo {
  //定义kudu表名
  val tableName:String = "person_spark1"
  val kuduMaster = "node2.xiaoma.cn"

  /**
   * 入口函数
   *
   * @param args
   */
  def main(args: Array[String]): Unit = {
    /**
     * 实现步骤：
     * 1）创建sparkConf对象
     * 2）创建sparkSession对象
     * 3）使用spark操作kudu
       * 1）创建表
       * 2）插入数据
       * 3）删除数据
       * 4）使用dataFrame读取kudu表数据，dataFrame=schema+rdd
       * 5）将dataFrame的数据写入到kudu表中
       * 6）使用sparksql将数据写入到kudu表中
       * 7）使用kudu原生的kuduRdd操作kudu
       * 8) 使用spark修改kudu表
     */

    //TODO 1）创建sparkConf对象
    val sparkConf: SparkConf = new SparkConf().setMaster("local[*]").setAppName(this.getClass.getSimpleName)

    //TODO 2）创建sparkSession对象
    val sparkSession: SparkSession = SparkSession.builder().config(sparkConf).getOrCreate()

    //TODO 3）spark操作kudu
    //创建kuduContext
    val kuduContext: KuduContext = new KuduContext("node2.xiaoma.cn:7051", sparkSession.sparkContext)

    //3.1：创建表
//    createTable(kuduContext)

    //3.2：插入数据
//    loadData(sparkSession, kuduContext)

    //3.3:删除数据
//    deleteData(sparkSession, kuduContext)

    //3.4：修改数据
    //updateData(sparkSession, kuduContext)

    //3.5：使用dataFrame读取kudu表数据，dataFrame=schema+rdd
//    getTableData(sparkSession)

    //3.6：将dataFrame的数据写入到kudu表中
    dataFrame2Kudu(sparkSession)

    //3.7：使用sparksql将数据写入到kudu表中
    //sparkSql2Kudu(sparkSession)

    //3.8：使用kudu原生的kuduRdd操作kudu
    //kuduNativeRdd(sparkSession, kuduContext)

    //3.9:使用spark修改kudu表
//    addColumn(kuduContext)

    //停止任务
    sparkSession.stop()
  }

  /**
   * 创建kudu表
   * DataFrame = Schema（StructType【StructField】）+RDD
   * @param kuduContext
   * @return
   */
  def createTable(kuduContext: KuduContext) = {
    //如果表不存在则创建
    if(!kuduContext.tableExists(tableName)){
      //定义表的结构信息
      val schema: StructType = StructType(
        StructField("id", IntegerType, false) ::
        StructField("name", StringType, true) ::
        StructField("gender", StringType, true) ::
        StructField("phone", StringType, true) :: Nil
      )

      //指定分区的字段
      val keys: List[String] = List("id")

      //定义表的选项
      val createTableOptions = new CreateTableOptions
      //设定副本数量
      createTableOptions.setNumReplicas(1)

      //定义分区的字段
      val partitionColumnList = new util.ArrayList[String]()
      partitionColumnList.add("id")

      //指定分区方式及分区字段
      createTableOptions.addHashPartitions(partitionColumnList, 3)

      //创建表
      kuduContext.createTable(tableName, schema, keys, createTableOptions)
    }
  }


  /**
   * 插入数据
   * @param sparkSession
   * @param kuduContext
   * @return
   */
  def loadData(sparkSession: SparkSession, kuduContext: KuduContext) = {
    //构建dataFrame对象(能否将rdd转换成dataFrame——>需要rdd的数据是有格式)
    val data: List[Person] = List(Person(1, "zhangsan", "male", "2323232"),Person(2, "lisi", "male", "2323232"))
    //将集合转换成rdd对象
    val personRDD: RDD[Person] = sparkSession.sparkContext.makeRDD(data)
    //导入隐式转换
    import  sparkSession.implicits._
    val personDF: DataFrame = personRDD.toDF

    //将数据写入到kudu表中
    kuduContext.insertRows(personDF, tableName)
  }

  /**
   * 删除数据
   * @param sparkSession
   * @param kuduContext
   * @return
   */
  def deleteData(sparkSession: SparkSession, kuduContext: KuduContext) = {
    //构建dataFrame对象(能否将rdd转换成dataFrame——>需要rdd的数据是有格式)
    val data: List[Person] = List(Person(2, "lisi", "male", "2323232"))
    //将集合转换成rdd对象
    val personRDD: RDD[Person] = sparkSession.sparkContext.makeRDD(data)
    //导入隐式转换
    import  sparkSession.implicits._
    //DELETE should not have a value for column: name STRING NULLABLE (error 0)
    //删除数据的时候需要根据指定的主键进行删除
    val personDF: DataFrame = personRDD.toDF.select("id")

    //删除数据
    kuduContext.deleteRows(personDF, tableName)

  }
  /**
   * 修改数据
   * @param sparkSession
   * @param kuduContext
   * @return
   */
  def updateData(sparkSession: SparkSession, kuduContext: KuduContext) = {
    //构建dataFrame对象(能否将rdd转换成dataFrame——>需要rdd的数据是有格式)
    val data: List[Person] = List(Person(1, "zhangsan", "male", "11111111111111111"),Person(2, "lisi", "male", "2222"))
    //将集合转换成rdd对象
    val personRDD: RDD[Person] = sparkSession.sparkContext.makeRDD(data)
    //导入隐式转换
    import  sparkSession.implicits._
    val personDF: DataFrame = personRDD.toDF

    //将数据写入到kudu表中
    //sample errors: Not found: key not found (error 0)
    //两个问题：
    //1：kudu不支持事务，有的成功有的失败（没有事务保障）
    //2：主键如果存在则更新，如果不存在则抛出异常
    //kuduContext.updateRows(personDF, tableName)
    //主键存在则更新，主键不存在则插入（upsert）
    kuduContext.upsertRows(personDF, tableName)
    //一般情况下数据是不允许被删除的（仅仅是逻辑删除，不是物理删除，因此逻辑删除实际上就是一个update操作）
  }

  /**
   * 使用dataframe读取kudu的数据
   * @param sparkSession
   * @return
   */
  def getTableData(sparkSession: SparkSession) = {
    //定义map的访问参数集合
    val options: Map[String, String] = Map(
      "kudu.master" ->kuduMaster,
      "kudu.table"-> tableName
    )

    //导入隐式转换
    import org.apache.kudu.spark.kudu._
    sparkSession.read.options(options).kudu.show()
  }

  /**
   * 将dataFrame的数据写入到kudu表中
   * @param sparkSession
   * @return
   */
  def dataFrame2Kudu(sparkSession: SparkSession) = {
    //构建dataFrame对象(能否将rdd转换成dataFrame——>需要rdd的数据是有格式)
    val data: List[Person] = List(Person(3, "wangwu", "male", "11111111111111111"),Person(4, "zhaoliu", "male", "2222"))
    //将集合转换成rdd对象
    val personRDD: RDD[Person] = sparkSession.sparkContext.makeRDD(data)
    //导入隐式转换
    import  sparkSession.implicits._
    val personDF: DataFrame = personRDD.toDF

    //定义map的访问参数集合
    val options: Map[String, String] = Map(
      "kudu.master" ->kuduMaster,
      "kudu.table"-> tableName
    )

    import org.apache.kudu.spark.kudu._
    personDF.write.mode(SaveMode.Append).options(options).kudu
  }

  /**
   * 使用sparksql操作kudu
   * @param sparkSession
   * @return
   */
  def sparkSql2Kudu(sparkSession: SparkSession) = {
    //将数据写入到kudu的表中
    //构建dataFrame对象(能否将rdd转换成dataFrame——>需要rdd的数据是有格式)
    val data: List[Person] = List(Person(5, "laowang", "male", "11111111111111111"),Person(6, "laoli", "male", "2222"))
    //将集合转换成rdd对象
    val personRDD: RDD[Person] = sparkSession.sparkContext.makeRDD(data)
    //导入隐式转换
    import  sparkSession.implicits._
    val personDF: DataFrame = personRDD.toDF

    //将dataFrame注册成表或者视图
    personDF.createTempView("tmp1")

    //定义map的访问参数集合
    val options: Map[String, String] = Map(
      "kudu.master" ->kuduMaster,
      "kudu.table"-> tableName
    )

    //导入隐式转换
    import org.apache.kudu.spark.kudu._
    val kuduDataFrame: DataFrame = sparkSession.read.options(options).kudu

    //将kududataFrame注册成视图或者表
    kuduDataFrame.createTempView("tmp2")

    //将tmp1表中的数据写入到tmp2表中
    sparkSession.sql("insert into table tmp2 select * from tmp1")

  }

  /**
   * 使用原生的kudurdd操作kudu
   * @param sparkSession
   * @return
   */
  def kuduNativeRdd(sparkSession: SparkSession,kuduContext: KuduContext) = {
    val columnList = List("id", "name", "gender", "phone")
    val rowRdd: RDD[Row] = kuduContext.kuduRDD(sparkSession.sparkContext, tableName, columnList)
    rowRdd.foreach(println(_))
  }

  /**
   * spark添加kudu列
   * @param kuduContext
   * @return
   */
  def addColumn(kuduContext: KuduContext) = {
//    //同步修改表操作
//    val client: KuduClient = kuduContext.syncClient
//    //异步修改表操作
//    kuduContext.asyncClient
    val alterTableOptions = new AlterTableOptions
    alterTableOptions.addColumn(new ColumnSchema.ColumnSchemaBuilder("address", Type.STRING).nullable(true).build())

    kuduContext.syncClient.alterTable(tableName, alterTableOptions)
  }


  /**
   * 样例类与普通类的区别：
   * 1）样例类不需要new
   * 2）默认实现了序列化
   * 3）可以进行模式匹配
   */
  case class Person(id:Int, name:String, gender:String, phone:String)
}
```

## 3 Kudu原理

### 3.1 表与schema

> Kudu设计是`面向结构化存储`的，因此Kudu的表需要用户在建表时定义它的`Schema`信息，这些Schema信息包含：
>> `列定义（含类型）`
> 
>> `Primary Key定义（用户指定的若干个列的有序组合）`

> 数据的唯一性，依赖于用户所提供的`Primary Key`中的Column组合的值的唯一性。

> Kudu提供了`Alter`命令来增删列，但位于Primary Key中的列是不允许删除的。

> `Kudu当前并不支持二级索引`。

> 从用户角度来看，Kudu是一种存储结构化数据表的存储系统。

> 在一个Kudu集群中可以定义任意数量的`table`，每个table都需要预先定义好`schema`。每个table的列数是确定的，每一列都需要有名字和类型，每个表中可以把其中一列或多列定义为主键。这么看来，Kudu更像关系型数据库，而不是像HBase、Cassandra和MongoDB这些NoSQL数据库。

> 不过Kudu目前还不能像关系型数据一样支持二级索引。

> Kudu使用确定的列类型，而不是类似于NoSQL的“`everything is byte”`（一切都是字节）。

> `这可以带来两点好处`：
>> 确定的列类型使Kudu可以进行类型特有的编码
> 
>> 可以提供 SQL-like 元数据给其他上层查询工具使用，比如BI工具。

### 3.2 Kudu的底层数据模型

> Kudu的底层数据文件的存储，未采用HDFS这样的较高抽象层次的分布式文件系统，而是自行开发了一套`可基于Table/Tablet/Replica视图级别的底层存储系统`

> `这套实现基于如下的几个设计目标`：
>> 可提供快速的列式查询
> 
>> 可支持快速的随机更新
> 
>> 可提供更为稳定的查询性能保障
> 
>> ![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/8.png) 
> 
>> 一个Table会被分成若干个`tablet`，其中`Tablet`的数量是根据`hash`或者是`range`进行设置的
> 
>> 一个Tablet中包含`MetaData信息`和`多个RowSet信息`，其中MetaData信息是`block`和block在data中的位置。
> 
>> 一个RowSet包含`一个MemRowSet`和`多个DiskRowSet`，其中MemRowSet用于存储`insert数据和update后的数据`，写满后会刷新到磁盘中也就是多个DiskRowSet中，`默认是1G刷新一次或者是2分钟。`
> 
>> DiskRowSet用于老数据的`mutation`（改变），比如说数据的更新操作，后台定期对DiskRowSet进行合并操作，删除历史数据和没有的数据，减少查询过程中的IO开销。
> 
>> 一个DiskRowSet包含1个`BloomFilter`，1个`Ad_hoc Index`，多个`UndoFile`、`RedoFile`、`BaseData`、`DeltaMem`
>>> BloomFile：根据一个DiskRowSet中的key生成一个bloom filter，用于快速模糊定位某个key是否在`DiskRowSet`中存在。
>> 
>>> Ad_hoc Index：是主键的索引，用于定位主键在DiskRowSet中的具体偏移位置。
>> 
>>> BaseData：是MemRowSet flush下来的数据，按列存储，按主键有序。
>>
>>> UndoFile：是基于BaseData之前时间（上次flush之前）的历史数据，可以获得历史数据，类似mysql中的回滚日志。
>>
>>> RedoFile：是基于BaseData之后时间（上次flush之后）的变更记录，可以获得新的数据，类似mysql中的重做日志。
>>
>>> DeltaMem：在内存中存储DiskRowSet中数据的更新，写满后flush到磁盘，形成deltafile文件。

> `注意`：
>> 这里有两个在内存中处理的数据集，区别如下：
>> 
>>> MemRowSet：存储新增的数据，对该内存数据集中还未flush的数据的更新；
>> 
>>> DeltaMem：对已flush到磁盘内的数据的更新；
>> 
>>![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/9.png)
> 
>> MemRowSets可以对比理解成HBase中的`MemStore`, 而DiskRowSets可理解成HBase中的`HFile`。MemRowSets中的数据按照`行视图进行存储`，数据结构为`B-Tree`。
> 
>> MemRowSets中的数据被Flush到磁盘之后，形成DiskRowSets。
> 
>> DiskRowSets中的数据，按照`32MB大小为单位`，按序划分为一个个的DiskRowSet。 DiskRowSet中的数据按照Column进行组织，与Parquet类似。

> 这是Kudu可支持一些分析性查询的基础。每一个Column的数据被存储在一个相邻的数据区域，而这个数据区域进一步被细分成一个个的小的Page单元，与HBase File中的Block类似，对每一个Column Page可采用一些Encoding算法，以及一些通用的Compression算法。 既然可对Column Page可采用Encoding以及Compression算法，那么，对单条记录的更改就会比较困难了。

> `前面提到了Kudu可支持单条记录级别的更新/删除，是如何做到的？`

> 与HBase类似，也是通过增加一条新的记录来描述这次更新/删除操作的。DiskRowSet是不可修改了，那么 KUDU 要如何应对数据的更新呢？在KUDU中，把DiskRowSet分为了两部分：
>> base data： 负责存储基础数据
> 
>> delta stores：delta stores负责存储 base data 中的变更数据.

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/10.png)

> 如上图所示，数据从 `MemRowSet` 刷到磁盘后就形成了一份 `DiskRowSet`（只包含 base data），每份 DiskRowSet 在内存中都会有一个对应的 DeltaMemStore，负责记录此 DiskRowSet 后续的数据变更（更新、删除）。DeltaMemStore 内部维护一个 B-树索引，映射到每个 row_offset 对应的数据变更。DeltaMemStore 数据增长到一定程度后转化成二进制文件存储到磁盘，形成一个 DeltaFile，随着 base data 对应数据的不断变更，DeltaFile 逐渐增长。

### 3.3 Kudu的读写原理

#### 3.3.1 工作模式

> Kudu的工作模式如下图，有些在上面的内容中已经介绍了，这里简单标注一下：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/11.png)

> 每个kudu table按照hash或range分区为多个tablet；

> 每个tablet中包含一个MemRowSet以及多个DiskRowSet；

> 每个DiskRowSet包含BaseData以及DeltaStores；

> DeltaStores由多个DeltaFile和一个DeltaMemStore组成；

> insert请求的新增数据以及对MemRowSet中数据的update操作（`新增的数据还没有来得及触发compaction操作再次进行更新操作的新数据`） 会先进入到MemRowSet；

> 当触发flush条件时将新增数据真正的持久化到磁盘的DiskRowSet内；

> 对老数据的update和delete操作是提交到内存中的DeltaMemStore；

> 当触发flush条件时会将更新和删除操作持久化到磁盘DIskRowSet中的DeltaFile内，此时老数据还在BaseData内（逻辑删除），新数据已在DeltaFile内；

> 当触发compaction条件时，将DeltaFile和BaseData进行合并，DiskRowSet进行合并，此时老数据才真正的从磁盘内消失掉（物理删除），只留下更新后的数据记录；

#### 3.3.2 Kudu的读流程

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/12.png)

> 客户端向Kudu Master请求tablet所在位置

> Kudu Master返回tablet所在位置

> 为了优化读取和写入，客户端将元数据进行缓存

> 根据主键范围过滤目标tablet，请求Tablet Follower

> 根据主键过滤scan范围，定位DataRowSets

> 加载BaseData，并与DeltaStores合并，得到老数据的最新结果

> 拼接第6步骤得到的老数据与MemRowSet数据 得到所需数据

> 将数据返回给客户端

#### 3.3.3 Kudu的写流程

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/13.png)

> 客户端向Kudu Master请求tablet所在位置；

> Kudu Master返回tablet所在位置；

> 为了优化读取和写入，客户端将元数据进行缓存；

> 根据分区策略，路由到对应Tablet，请求Tablet Leader；

> 根据RowSet记录的主键范围过滤掉不包含新增数据主键的RowSet；

> 根据RowSet 布隆过滤器再进行一次过滤，过滤掉不包含新数据主键的RowSet；

> 查询RowSet中的B树索引判断是否命中新数据主键，若命中则报错主键冲突，否则新数据写入MemRowSet；

> 返回响应给客户端；

#### 3.3.4 Kude的更新流程

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/14.png)

> 更新删除流程与写入流程类似，区别就是最后判断是否存在主键时候的操作，若存在才能更新，不存在才能插入新数据。

> 客户端向Kudu Master请求tablet所在位置

> Kudu Master返回tablet所在位置

> 为了优化读取和写入，客户端将元数据进行缓存

> 根据分区策略，路由到对应Tablet，请求Tablet Leader

> 根据RowSet记录的主键范围过滤掉不包含修改的数据主键的RowSet

> 根据RowSet 布隆过滤器再进行一次过滤，过滤掉不包含修改的数据主键的RowSet

> 查询RowSet中的B树索引判断是否命中修改的数据主键，若命中则修改至DeltaStores，否则报错数据不存在

> 返回响应给客户端

## 4 Kudu优化

### 4.1 Kudu关键配置

> TabletServer 在开始拒绝所有传入的写入之前可以消耗的最大内存量：memory_limit_hard_bytes=1073741824

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/15.png)

> 分配给 Kudu Tablet Server 块缓存的最大内存量：block_cache_capacity_mb=512

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Kudu/16.png)

### 4.2 Kudu的使用限制

#### 4.2.1 主键

> 创建表后，不能更改主键。必须删除并重新创建表以选择新的主键。

> 创建表的时候，主键必须放在最前边。

> 主键不能通过 update 更新，如果要修改主键就必须先删除行，然后重新插入。这种操作不是原子性的。（kudu的删除和插入操作无法事务）

> 不支持自动生成主键，可以通过内置的 uuid 函数表示为主键值。

> 联合主键由 kudu 编码后，大小不能超过 16KB。

#### 4.2.2 Cells

> 在编码或压缩之前，任何单个单元都不得大于 64KB。

> 在 Kudu 完成内部复合键编码之后，组成复合键的单元格总共限制为 16KB。

> 如果插入不符合这些限制的行时会报错误并返回给客户端。

#### 4.2.3 字段

> 默认情况下，Kudu 不允许创建超过 300 列的表。官方建议使用较少列的 Schema 设计以获得最佳性能。

> 不支持 CHAR、VARCHAR、DATE 和数组等复杂类型。

> 现有列的类型和是否允许为空，一旦设置后，是不可修改的。

> Decimal 类型的精度不可修改。也不允许通过更改表来更改 Decimal 列的精度和小数位数

> 删除列不会立即回收空间。首先必须运行压缩。

#### 4.2.4 表

> 表中的副本数必须为奇数，最多为 7

> 复制因子（在表创建时设置）不能更改

> 无法手动运行压缩，但是删除表将立即回收空间

#### 4.2.5 其他限制

> 不支持二级索引。

> 不支持多行事务。

> 不支持外键。

> 列名和表名之类的标识符仅限于有效的 UTF-8 字符串并且其最大长度为 256 个字符。

#### 4.2.6 分区限制

> 表必须根据一个主键 or 联合主键被预先切成 tablet，不支持自动切。表被创建后不支持修改分区字段，支持添加和删除 range 分区(意思分区表，分区字段需提前定义好，kudu 不会自动分)。

> 已经存在的表不支持自动重新分区，只能创建新表时指定。

> 丢失副本时，必须通过手动修复方式来恢复。

#### 4.2.7 扩展建议和限制

> 建议 TabletServer 最多为 100 台。

> 建议 Master 最多 3 台。

> 建议每个 TabletServer 最大数据为 8T(压缩后)。

> 建议每台 TabletServer 的 tablet 数为 1000，最多 2000。

> 创建表的时候，建议在每个 Tablet Server 上，每个表的 Tablet 数最大为 60，也就是 3 节点的话，3 副本，创表分区最大 60，这样每个单 TabletServer 上该表的 Tablets 也就为 60。

> 建议每个 Tablet 最大为 50GB，超出后可能导致压缩和启动有问题。

> 建议单 Tablet 的大小<10GB。

#### 4.2.8 守护进程

> 部署至少 4G 内存，理想情况下应超过 16GB。

> 预写日志（WAL）只能存储在一个磁盘上。

> 不能直接删除数据目录，必须使用重新格式化数据目录的方式来达到删除目的。

> TabletServer 不能修改 IP 和 PORT。

> Kudu 对 NTP 有严格要求，如果时间不同步时，Kudu 的 Master 和 TabletServer 会崩溃。

> Kudu 仅使用 NTP 进行了测试，不支持其他时间同步工具。

#### 4.2.9 集群管理限制

> 不支持滚动重启。

> 建议 Kudu 集群中的最大点对点延迟为 20 毫秒。推荐的最小点对点带宽是 10GB。

> 如果要使用位置感知功能将平板服务器放置在不同的位置，官方建议先测量服务器之间的带宽和延迟，以确保它们符合上述指导原则。

> 首次启动群集时，必须同时启动所有 Master 服务。

#### 4.2.10 复制和备份限制

> Kudu 当前不支持任何用于备份和还原的内置功能。鼓励用户根据需要使用 Spark 或 Impala之类的工具导出或导入表。

#### 4.2.11 Impala集成限制

> 创建 Kudu 表时，建表语句中的主键字段必须在最前面。

> Impala 无法更新主键列中的值。

> Impala 无法使用以下命令创建 Kudu 表 VARCHAR 或嵌套类型的列。

> 名称包含大写字母或非 ASCII 字符的 Kudu 表在 Impala 中用作外部表时，必须分配一个备用名称。

> 列名包含大写字母或非 ASCII 字符的 Kudu 表不能用作 Impala 中的外部表。可以在 Kudu 中重命名列以解决此问题。

> !=和 like 谓词不会下推到 Kudu，而是由 Impala 扫描节点评估。相对于其他类型的谓语，这会导致降低性能。

> 使用 Impala 进行更新，插入和删除是非事务性的。如果查询在部分途中失败，则其部分效果不会回滚。

> 单个查询的最大并行度受限于 Table 中 Tablet 的数量。为了获得良好的分析性能，每位主机目标为 10 片或更多 tablets。

> Impala 的关键字(PARTITIONED、LOCATION、ROWFORMAT)不适用于在创建 Kudu 表时使用。

#### 4.2.12 Spark集成限制

> 必须使用 JDK8，自 Kudu-1.5.0 起，Spark 2.2 是默认的依赖项版本。

> Kudu 表只能在 Spark SQL 中注册为临时表。

> 无法使用 HiveContext 查询 Kudu 表。


## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
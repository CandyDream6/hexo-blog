---
title: 记一次mysql连接数问题 
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(92).jpg 
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(92).jpg 
tags:
  - 大数据
  - Mysql 
category:
  - - 编程
    - 大数据 
date: 2021-08-15 17:15:40
---

最近在做离线采集的东西，在做采集的过程中，时不时遇到`mysql连接数满了`，于是排查了一下原因。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 事故现场

```shell
too many connections
```

> 背景是这样的，最近做的东西是`采集RDBMS`的数据到`HIVE`，`IMPALA`，`KUDU`，`HBASE`。目前我们用的RDBMS主要是mysql和oracle。

> 场景是通过`JDBC去访问HIVE`，创建HIVE和MYSQL的`StorageHandler外联表`，然后经常就遇到·mysql连接数满了·，而且很频繁。

> 加上这块代码是我撸的，心里顿时`慌的一批`，赶紧去看一下。

## 2 故障排查

### 2.1 代码排查

> 想都不用想，`连接数的问题`，直接找到JDBC连接的地方(下面代码我把所有业务逻辑全去了，只留主要的)

```shell
import org.apache.tomcat.jdbc.pool.DataSource;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class DataSourceTest {
    public static void main(String[] args) {
        DataSource dataSource = new DataSource();
        dataSource.setUrl("jdbc:mysql://127.0.0.1:3306");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        // 获得连接
        try {
            Connection connection = dataSource.getConnection();
            Statement statement = connection.createStatement();
            ResultSet resultSet = statement.executeQuery("show databases");
            // 关闭连接
            resultSet.close();
            statement.close();
            connection.close();
        } catch (SQLException exception) {
            exception.printStackTrace();
        }
        // 在这里打断点模拟web项目
        System.out.println("给爷停在这里，Fuck You!");
    }
}
```

> `pom`

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.26</version>
</dependency>
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-jdbc</artifactId>
    <version>10.1.0-M5</version>
</dependency>
```

> 一开始我想着，我这`结果集`、`执行语句`、`连接`全都关了呀，感觉没啥问题呀，有点`怀疑是不是我的问题`。

> 于是跑到数据库去观察。

### 2.2 数据库排查

> 随便搞个数据库测试下。

#### 2.2.1 查看最大连接数

```shell
mysql> SHOW VARIABLES LIKE '%max_connections%';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151   |
+-----------------+-------+
1 row in set (0.01 sec)
```

> 发现是`默认的151`

#### 2.2.2 查看当前连接数

```shell
mysql> SHOW STATUS LIKE 'Threads%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_cached    | 1     | 
| Threads_connected | 16    |       # 打开的连接数，这个到151就爆了
| Threads_created   | 19    |
| Threads_running   | 3     |       # 激活的连接数，一般远低于connected
+-------------------+-------+
4 rows in set (0.00 sec)


也可以用下面命令查看连接详细信息：这个数值和Threads_connected一致。


mysql> SHOW FULL PROCESSLIST;
+----+------+---------------------+---------------+-------------+------+---------------------------------------------------------------+-----------------------+
| Id | User | Host                | db            | Command     | Time | State                                                         | Info                  |
+----+------+---------------------+---------------+-------------+------+---------------------------------------------------------------+-----------------------+
|  2 | root | 172.18.17.22:10354  | canal_manager | Sleep       |   43 |                                                               | NULL                  |
|  3 | qiao | dbus-n3:39416       | NULL          | Binlog Dump |  567 | Master has sent all binlog to slave; waiting for more updates | NULL                  |
| 26 | root | 172.18.17.49:54266  | test_schema1  | Sleep       |   10 |                                                               | NULL                  |
| 27 | root | 172.18.17.49:54267  | test_schema1  | Sleep       |  495 |                                                               | NULL                  |
| 28 | root | 172.18.26.237:49224 | NULL          | Query       |    0 | starting                                                      | SHOW FULL PROCESSLIST |
| 29 | root | 172.18.17.49:62920  | NULL          | Binlog Dump |  379 | Master has sent all binlog to slave; waiting for more updates | NULL                  |
| 30 | root | 172.18.17.49:62921  | NULL          | Sleep       |  378 |                                                               | NULL                  |
| 31 | root | 172.18.17.49:62922  | NULL          | Sleep       |  378 |                                                               | NULL                  |
| 32 | root | 172.18.17.49:62923  | NULL          | Sleep       |  378 |                                                               | NULL                  |
| 33 | root | 172.18.17.49:62924  | NULL          | Sleep       |  378 |                                                               | NULL                  |
| 34 | root | 172.18.17.49:62925  | NULL          | Sleep       |  378 |                                                               | NULL                  |
| 35 | root | 172.18.17.49:62926  | NULL          | Sleep       |  378 |                                                               | NULL                  |
| 36 | root | 172.18.17.49:62927  | NULL          | Sleep       |  378 |                                                               | NULL                  |
| 37 | root | 172.18.17.49:62928  | NULL          | Sleep       |  378 |                                                               | NULL                  |
| 38 | root | 172.18.17.49:62929  | NULL          | Sleep       |  378 |                                                               | NULL                  |
| 39 | root | 172.18.17.49:62930  | NULL          | Sleep       |  376 |                                                               | NULL                  |
+----+------+---------------------+---------------+-------------+------+---------------------------------------------------------------+-----------------------+
16 rows in set (0.00 sec)
```

#### 2.2.3 场景还原

> 给我代码`最后一行打断点`，跑起来。

> 一跑问题就出来了，发现连接数`从16变成了26`，`多了10个`连接。

> 开`3个线程`试一下，一下子`多了30个连接`，`这他娘真是我代码问题了`。

> 回想了一下，以前写jdbc用的`DriverManager`，这次换成了`DataSource`，点进DataSource源码一看，我尼玛，原来是个`连接池`。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/记一次mysql连接数问题/1.png)

> 再看一下，连接池`默认大小是10`，这问题就`柳暗花明又一村`了。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/记一次mysql连接数问题/2.png)

> 问题找到了，那就开始`灭他满门`吧。

## 3 事故处理

> 总结下来有两个问题：
>> 一是我使用完`连接池后没有关闭`，这也是经常爆的原因。
>
>> 二是每次跑我`并不需要10个连接数`，可能只用到两三个，所以缩小连接池的大小。

> 修改后的代码如下(没有提取封装，只做demo测试)

```shell
import org.apache.tomcat.jdbc.pool.DataSource;
import org.apache.tomcat.jdbc.pool.PoolProperties;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class DataSourceTest {
    public static void main(String[] args) {
        // 连接池设置
        PoolProperties properties = new PoolProperties();
        properties.setInitialSize(3);
        // 获得连接池
        DataSource dataSource = new DataSource(properties);
        dataSource.setUrl("jdbc:mysql://127.0.0.1:3306");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        // 获得连接
        try {
            Connection connection = dataSource.getConnection();
            Statement statement = connection.createStatement();
            ResultSet resultSet = statement.executeQuery("show databases");
            // 关闭连接
            resultSet.close();
            statement.close();
            connection.close();
        } catch (SQLException exception) {
            exception.printStackTrace();
        } finally {
            // 关闭连接池
            dataSource.close();
        }
        // 在这里打断点模拟web项目
        System.out.println("给爷停在这里，Fuck You!");
    }
}
```

> 问题解决了，先干饭吧

## 4 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路

![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

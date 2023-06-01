---
title: Avro 数据序列化系统
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(56).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(56).jpg
tags:
  - 大数据
  - Avro
category:
  - - 编程
    - 大数据
 
date: 2020-02-19 18:41:50
---

在企业级大数据流处理项目中，往往在项目数据源处需要面临`实时海量数据的采集`。

采集数据的性能一般与网络带宽、机器硬件、数据量等因素有直接关系。

当其他因素是固定的，这里我们只考虑数据量的话，那么`数据量的传输和存储性能是我们首先需要面对和解决的`。

由此我们引入了Avro数据序列化框架，来`解决数据的传输性能问题`。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 Apache Avro 介绍

> 官网地址:https://avro.apache.org/

> Apache Avro（以下简称 Avro）是一个数据序列化系统，是一种与编程语言无关的序列化格式，是提供一种共享数据文件的方式。

> Avro是Hadoop中的一个子项目，Avro是一个`基于二进制数据传输高性能的中间件`。

> `Avro可以做到将数据进行序列化，适用于远程或本地大批量数据交互。在传输的过程中Avro对数据二进制序列化后节约数据存储空间和网络传输带宽`。

> 序列化就是将对象转换成二进制流，相应的反序列化就是将二进制流再转换成对应的对象。

> 因此，`Avro就是用来在传输数据之前，将对象转换成二进制流，然后此二进制流达到目标地址后，Avro再将二进制流转换成对象`。

> `Avro特点`：
>> 1.丰富的数据结构
> 
>> 2.一个紧凑的，快速的，二进制的数据格式
> 
>> 3.一个容器文件，来存储持久化数据
> 
>> 4.远程过程调用（RPC）
> 
>> 5.简单的动态语言集成。
> 
>> 6.Avro模式是使用JSON定义的。这有助于以已经具有JSON库的语言实现。

> 注意:是使用Json进行定义而不是传输,因为传输的话会出现冗余,定义只用一次<br/>
> 如:<br/>
> 下面是传输,会出现key冗余:</>
```
{"name":"jack","age",18}
{"name":"tom","age",18}
{"name":"lily","age",18}
```
> 下面是定义,只需要定义一次,然后将数据序列化传输,定义信息保存一份
```
{"filed":"name","tpye","String'} --json格式定义
{"filed":"age","tpye","int'}--json格式定义
{"jack",18} --二进制传输
{"tom",18} --二进制传输
{"lily",,18} --二进制传输
```
> `这样使用Avor对数据序列化之后, 就可以提升数据传输和存储的性能`

> JSON是一种轻量级的数据传输格式，对于大数据集，JSON数据会显示力不从心，因为JSON的格式是key：value型，每条记录都要附上key的名字，有的时候，光key消耗的空间甚至会超过value所占空间，这对空间的浪费十分严重，尤其是对大型数据集来说，因为它不仅不够紧凑，还要重复地加上key信息，不仅会造成存储空间上的浪费，更会增加了数据传输的压力，从而给集群增加负担，进而影响整个集群的吞吐量。而采用Avro数据序列化系统可以比较好的解决此问题，因为用Avro序列化后的文件由schema和真实内容组成，schema只是数据的元数据，相当于JSON数据的key信息，schema单独存放在一个JSON文件中，这样一来，数据的元数据只存了一次，相比JSON数据格式的文件，大大缩小了存储容量。从而使得Avro文件可以更加紧凑地组织数据。

## 2 Avro文件规范

> AVRO支持6种复杂类型，分别是：`records`, `enums`, `arrays`, `maps`, `unions`，`fixed`.

### 2.1 Records

> Records使用类型名称“record”，并且支持三个必选属性。

```
type: 必有属性。
name: 必有属性，是一个JSON string，提供了记录的名字。
namespace，也是一个JSON string，用来限定和修饰name属性。
doc: 可选属性，是一个JSON string，为使用这个Schema的用户提供文档。
aliases: 可选属性，是JSON的一个string数组，为这条记录提供别名。
fields: 必选属性，是一个JSON数组，数组中列举了所有的field。
```
> 每一个field都是一个JSON对象，并且具有如下属性：

```
（1）name: 必选属性，field的名字，是一个JSON string。例如：
"fields": [
                {"name": "name", "type": "string"},
                {"name": "age",  "type": ["int", "null"]},
                {"name": "address", "type": ["string", "null"]}
            ]

（2）doc: 可选属性，为使用此Schema的用户提供了描述此field的文档。

（3）type: 必选属性，定义Schema的一个JSON对象，或者是命名一条记录定义的JSON string。

（4）default: 可选属性，即field的默认值，当读到缺少这个field的实例时用到。默认值的允许的范围由这个field的Schama的类型决定，如下表所示。其中union fields的默认值对应于union中第一个Schema。Bytes和fixed的field的默认值都是JSON string，并且指向0-255的unicode都对应于无符号8位字节值0-255。
order: 可选属性，指定这个field如何影响record的排序。有效的可选值为“ascending”(默认),"descending"和"ignore"
alias: JSON的string数组，为这个field提供别名。
```

### 2.2 Enums

> Enums使用的名为“enum”的type并且支持如下的属性：

```
name: 必有属性，是一个JSON string，提供了enum的名字。
namespace，也是一个JSON string，用来限定和修饰name属性。
aliases: 可选属性，是JSON的一个string数组，为这个enum提供别名。
doc: 可选属性，是一个JSON string，为使用这个Schema的用户提供文档。
symbols: 必有属性，是一个JSON string数组，列举了所有的symbol，在enum中的所有symbol都必须是唯一的，不允许重复。比如下面的例子：
引用
{ 
    "type": "enum",
    "name": "Person",
    "symbols" : ["name", "age", "address", "sex"]
}
```

### 2.3 Arrays

> Array使用名为"array"的type，并且支持一个属性

```
items: array中元素的Schema
比如一个string的数组声明为：
引用
{"type": "array", "items": "string"}
```

### 2.4 Maps

> Map使用名为"map"的type，并且支持一个属性

```
values: 用来定义map的值的Schema。Maps的key都是string。比如一个key为string，value为long的maps定义为：
引用
{"type": "map", "values": "long"}
```

### 2.5 Unions

> Unions就像上面提到的，使用JSON的数组表示。比如

```
引用
["string", "null"]
声明了一个union的Schema，其元素即可以是string，也可以是null。
Unions不能包含多个相同类型的Schema，除非是命名的record类型、命名的fixed类型和命名的enum类型。比如，如果unions中包含两个array类型，或者包含两个map类型都不允许；但是两个具有不同name的相同类型却可以。由此可见，union是通过Schema的name来区分元素Schema的，因为array和map没有name属性，当然只能存在一个array或者map。（使用name作为解析的原因是这样做会使得读写unions更加高效）。unions不能紧接着包含其他的union。
```

### 2.6 Fixed

> Fixed类型使用"fixed"的type name，并且支持三个属性：

```
name: 必有属性，表示这个fixed的名称，JSON string。
namespace:也是一个JSON string，用来限定和修饰name属性。
aliases: 可选属性，同上
size: 必选属性，一个整数，志明每个值的字节数。
比如16字节的fixed可以声明为：
引用
{"type": "fixed", "size": 16, "name": "md5"}
```

> Record, enums 和 fixed都是命名的类型，这三种类型都各有一个全名，全名有两部分组成：名称和命名空间。名称的相等是定义在全名基础上的。

> 全名的名字部分和record的field名字必须：

```
以[A-Za-z_]开头
接下来的名字中只能包含[A-Za-z0-9_]
namespace是以点分隔的一系列名字。
在record、enum和fixed的定义中，全名由以下的任意一条决定：
同时指定name和namespace，比如使用 "name": "User", "namespace": "cn.itcast"来表示全名cn.itcast.User。
指定全名。如果name中包含点号，则认为是全名。比如用 "name": "org.foo.X" 表示全名org.foo.X。
仅仅指定name，name中没有点号。在这种情况下命名空间取自距离最近的父亲的Schema或者protocol。比如声明了"name": "X"， 这段声明在一条记录“org.foo.Y”的field中，那么X的全名就是org.foo.X。
原生类型没有命名空间，并且在不可以在命名空间中定义原生类型。如果多个全名定义相等的话，一个Schema可以包含多个全名定义。
```

## 3 Avro案例

### 3.1 pom

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.avro</groupId>
        <artifactId>avro</artifactId>
        <version>1.8.1</version>
    </dependency>
</dependencies>

<build>
<plugins>
    <!--maven编译插件-->
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>
    <!--Avro编译插件-->
    <plugin>
        <groupId>org.apache.avro</groupId>
        <artifactId>avro-maven-plugin</artifactId>
        <version>1.8.1</version>
        <executions>
            <execution>
                <phase>generate-sources</phase>
                <goals>
                    <goal>schema</goal>
                </goals>
                <configuration>
                    <!--Avro源文件-->
                    <sourceDirectory>${project.basedir}/src/main/avro/</sourceDirectory>
                    <!--Avro编译生成文件-->
                    <outputDirectory>${project.basedir}/src/main/java/</outputDirectory>
                </configuration>
            </execution>
        </executions>
    </plugin>
</plugins>
</build>
```

### 3.2 定义数据格式

> 参考官网定义如下的数据格式/AvroSchema约束文件并放入之前的文件夹下

```
{"namespace": "cn.itcast.avro",
 "type": "record",
 "name": "User",
 "fields": [
     {"name": "name", "type": "string"},
     {"name": "age",  "type": ["int", "null"]},
     {"name": "address", "type": ["string", "null"]}
 ]
}
```

### 3.3 使用maven编译

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Avro/1.png)

> 这时可以看到指定目录下生成了User类

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Avro/2.png)

#### 3.4 序列化和反序列化

```
import org.apache.avro.file.DataFileReader;
import org.apache.avro.file.DataFileWriter;
import org.apache.avro.io.DatumReader;
import org.apache.avro.io.DatumWriter;
import org.apache.avro.specific.SpecificDatumReader;
import org.apache.avro.specific.SpecificDatumWriter;

import java.io.File;
import java.io.IOException;

/**
 @desc ...
 @date 2021-02-05 18:21:57
 @author xiaoma
 */
public class TestAvroSerializeAndDeSerialize {
    public static void main(String[] args) throws IOException {
        //1.准备Java对象
        User user1 = new User("tom", 20, "shanghai");

        User user2 = new User();
        user2.setName("jack");
        user2.setAge(18);
        user2.setAddress("beijin");

        User user3 = User.newBuilder()
                .setName("tony")
                .setAge(19)
                .setAddress("shenzhen")
                .build();

        //2.序列化--将Java对象转为二进制数据存到users.avro文件中了
        //-创建写标准对象
        DatumWriter<User> userDatumWriter = new SpecificDatumWriter<User>(User.class);
        //-创建写文件对象
        DataFileWriter<User> dataFileWriter = new DataFileWriter<User>(userDatumWriter);
        //-设置文件路径和Schema
        dataFileWriter.create(user1.getSchema(), new File("users.avro"));
        //-设置要写哪些对象
        dataFileWriter.append(user1);
        dataFileWriter.append(user2);
        dataFileWriter.append(user3);
        dataFileWriter.close();

        //3.反序列化--将users.avro文件中的二进制数据反序列化为Java对象
        //-创建读标准对象
        DatumReader<User> userDatumReader = new SpecificDatumReader<User>(User.class);
        //-创建读文件对象
        DataFileReader<User> dataFileReader = new DataFileReader<User>(new File("users.avro"), userDatumReader);
        //-循环读
        User user = null;
        while (dataFileReader.hasNext()) {
            //传进入的是空的user,返回来的是反序列化之后的user
            user = dataFileReader.next(user);
            System.out.println("反序列化出来的Java对象信息为:"+user);
        }
    }
}
```

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
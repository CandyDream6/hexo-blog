---
title: Java Virtual Machine
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(73).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(73).jpg
tags:
  - Java
  - JVM
category:
  - - 编程
    - Java
  - - 编程
    - JVM
date: 2021-03-21 11:26:25
---

Java语言的一个非常重要的特点就是与`平台的无关性`。

而使用`Java虚拟机`是实现这一特点的关键。

一般的高级语言如果要`在不同的平台上运行`，至少`需要编译成不同的目标代码`。

而引入Java语言虚拟机后，`Java语言在不同平台上运行时不需要重新编译`。

Java语言使用模式Java虚拟机`屏蔽了与具体平台相关的信息`，使得Java语言编译程序只需生成在Java虚拟机上运行的`目标代码（字节码）`，就可以`在多种平台上不加修改地运行`。

Java虚拟机在执行字节码时，`把字节码解释成具体平台上的机器指令执行`。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 JVM的体系结构

> 我画的`visio原图`链接，有兴趣的同学可以在这个图上自己改

```
链接：https://pan.baidu.com/s/1ugOk3xFrK6iqdXKIeldplQ 
提取码：6666 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_JVM/1.png)

```
类装载器把程序类信息、静态变量、常量装载到方法区。
创建的对象数组等(例如new)放到堆里。
程序跑的时候去引用堆里的对象地址去跑。用到本地方法的去跑JNI
```

### 1.1 `JVM栈`

> `程序是在栈内存中运行的，所以栈内存解决的是程序运行时的问题`,程序运行永远都是在栈中进行的

> `存储的是基本数据类型和堆中数据的引用（引用地址）`

> `线程结束，栈内存也就是释放,对于栈来说,不存在垃圾回收问题`
>> 一旦线程结束，栈就Over!
> 
>> 栈内存中:8大基本类型+对象引用+实例的方法

> Java以栈帧为单位保存线程的运行状态，虚拟机只会对栈执行两种操作：以栈帧为单位的压栈或者出栈

> 一个线程独占一个Java栈（栈里的数据是线程私有的）

> 分为三个部分：基本类型变量区、执行环境上下文、操作指令区

> 异常：`java.lang.StackOverFlowError`

### 1.2 本地方法栈

> 和java栈的作用差不多，只不过是为JVM使用到的native方法(使用非Java语言实现的方法)服务的

> 例如线程的start方法底层调用了`start0`方法,这个start0方法就是个本地方法接口
```
new Thread(()->{},"xiaoma thread").start();
```

```
private native void start0();
```

> native :凡是带了native关键字的，说明java的作用范围达不到了，回去调用底层c语言的库!
> 会进入本地方法栈，调用本地方法本地接口 JNI (Java Native Interface)
> JNI作用:开拓Java的使用，融合不同的编程语言为Java所用!最初: C、C++
> Java诞生的时候C、C++横行，想要立足，必须要有调用C、C++的程序
> 它在内存区域中专门开辟了一块标记区域: Native Method Stack，登记native方法
> 在最终执行的时候，加载本地方法库中的方法通过JNI
> 例如：Java程序驱动打印机，管理系统，掌握即可，在企业级应用比较少

### 1.3 方法区

> 又称`静态区`

> 方法区是被所有线程共享,`所有字段和方法字节码，以及一些特殊方法，如构造函数,接口代码也在此定义`,简单说，`所有定义的方法的信息都保存在该区域`,此区域属于共享区间;

> `静态变量、常量、类信息(构造方法、接口定义)、运行时的常量池存在方法区中`，但是`实例变量存在堆内存中，和方法区无关`

> static、final、Class、常量池

### 1.4 `堆`

> `堆内存解决的是数据存储的问题`

> `存储的是对象和数组（对象本身）`

> 所有线程共享java堆

> 动态的分配内存（运行时分配），生命周期（不确定）不需要预先告诉编译器，Java的垃圾回收机制会自动收走不使用的数据

> 由于运行时动态分配内存，存储数据较慢

> 异常：`java.lang.OutOfMemoryError`

### 1.5 程序计数器

> `用于保存当前线程执行的内存地址`

> 由于JVM程序是多线程执行的（线程轮流切换），所以为了保证线程切换回来后，还能恢复到原先状态，就需要一个独立的计数器，记录之前中断的地方，可见程序计数器也是线程私有的

> 注意这个区域是唯一一个不抛出OutOfMemoryError的运行时数据区

### 1.6 元空间

> 对于Java8， HotSpots取消了永久代，那么是不是也就没有方法区了呢？当然不是，方法区是一个规范，规范没变，它就一直在。

> 那么`取代永久代的就是元空间`。它可永久代有什么不同的？
>> `存储位置不同`，永久代物理是是堆的一部分，和新生代，老年代地址是连续的，而元空间属于本地内存；
> 
>> `存储内容不同`，元空间存储类的元信息，静态变量和常量池等并入堆中。相当于永久代的数据被分到了堆和元空间中。

## 2 双亲委派机制

### 2.1 类加载器类型

> `AppClassLoader:应用程序加载器`
>> 当前应用程序中的类

> `ExtClassLoader:扩展类加载器`
>> 在`JRE\lib\ext\`目录中

> `BootStrapClassLoader:启动类(根)加载器`
>> 在`JRE\lib\rt.jar`中，如java.lang.String等。

> `自定义加载器`

### 2.2 双亲委派机制的流程

> 1.类加载器收到类加载的请求

> 2.将这个请求向上委托给父类加载器去完成，一直向上委托，直到启动类加载器

> 3.启动类加载器检查能否加载当前这个类，能够加载就结束了。不能加载的话，向下让子类加载器加载。

### 2.3 演示查看加载器

```
/**
 @desc ...
 @date 2021-03-22 18:43:57
 @author xiaoma
 */
public class Car {
    public static void main(String[] args) {
        Car car1 = new Car();

        // AppClassLoader  应用程序加载器
        System.out.println(car1.getClass().getClassLoader());
        // ExtClassLoader 扩展类加载器
        System.out.println(car1.getClass().getClassLoader().getParent());
        // root            启动类(根)加载器
        System.out.println(car1.getClass().getClassLoader().getParent().getParent());
    }
}

sun.misc.Launcher$AppClassLoader@18b4aac2
sun.misc.Launcher$ExtClassLoader@69222c14
null        这里为null是因为根加载器调用了底层C++语言的东西，java没法直接识别
```

### 2.4 演示双亲委派机制

> 自己在目录java.lang下写一个String类

```
package java.lang;

public class String {
    
    public String toString() {
        return "hello";
    }

    public static void main(String[] args) {
        String s = new String();
        s.toString();
    }
}
```

> 这里程序会报一个找不到main方法的错误。因为加载这个类的时候，`先找扩展类加载器`，`再找根加载器rt.jar`，根加载器里有String类，就用根加载器的。但是里面没有main方法。

> 所以程序会用rt.jar里的String类，而不用我们写的

### 2.5 双亲委派机制的作用

> `保证核心类不能被篡改。通过委托方式，不会去篡改核心.class`

> `防止内存中出现多份同样的字节码`
>> 比如两个类A和类B都要加载System类：
>
>> 如果不用委托而是自己加载自己的，那么类A就会加载一份System字节码，然后类B又会加载一份System字节码，这样内存中就出现了两份System字节码。
>
>> 如果使用委托机制，会递归的向父类查找，也就是首选用Bootstrap尝试加载，如果找不到再向下。这里的System就能在Bootstrap中找到然后加载，如果此时类B也要加载System，也从Bootstrap开始，此时Bootstrap发现已经加载过了System那么直接返回内存中的System即可而不需要重新加载，这样内存中就只有一份System的字节码了。

## 3 JProfiler排查OOM

> 项目中出现OOM的时候，可以使用JProfiler排查问题

### 3.1 安装JProfiler

> 安装包里有注册机，大家直接暗转就好

```
链接：https://pan.baidu.com/s/1CkcQ16lUWmOvpDxvZgPgDg 
提取码：6666 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

### 3.2 IDEA安装JProfiler插件

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_JVM/2.png)

> 配置JProfiler路径

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_JVM/3.png)

### 3.3 编写测试代码

```
import java.util.ArrayList;

/**
 @desc ...
 @date 2021-03-22 14:44:24
 @author xiaoma
 */
public class TestOOM {
    public static void main(String[] args) {
        ArrayList<TestOOM> list = new ArrayList<>();
        while (true){
            list.add(new TestOOM());
        }
    }
}

```

### 3.4 调整虚拟机参数，Dump文件

```
-Xms8m -Xmx8m -XX:+HeapDumpOnOutOfMemoryError
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_JVM/4.png)

### 3.5 测试

> 运行程序，发现报错

```
D:\application\JDK\bin\java.exe -Xms8m -Xmx8m -XX:+HeapDumpOnOutOfMemoryError "-javaagent:D:\application\IntelliJ IDEA 2020.2.3\lib\idea_rt.jar=3838:D:\application\IntelliJ IDEA 2020.2.3\bin" -Dfile.encoding=UTF-8 -classpath D:\application\JDK\jre\lib\charsets.jar;D:\application\JDK\jre\lib\deploy.jar;D:\application\JDK\jre\lib\ext\access-bridge-64.jar;D:\application\JDK\jre\lib\ext\cldrdata.jar;D:\application\JDK\jre\lib\ext\dnsns.jar;D:\application\JDK\jre\lib\ext\jaccess.jar;D:\application\JDK\jre\lib\ext\jfxrt.jar;D:\application\JDK\jre\lib\ext\localedata.jar;D:\application\JDK\jre\lib\ext\nashorn.jar;D:\application\JDK\jre\lib\ext\sunec.jar;D:\application\JDK\jre\lib\ext\sunjce_provider.jar;D:\application\JDK\jre\lib\ext\sunmscapi.jar;D:\application\JDK\jre\lib\ext\sunpkcs11.jar;D:\application\JDK\jre\lib\ext\zipfs.jar;D:\application\JDK\jre\lib\javaws.jar;D:\application\JDK\jre\lib\jce.jar;D:\application\JDK\jre\lib\jfr.jar;D:\application\JDK\jre\lib\jfxswt.jar;D:\application\JDK\jre\lib\jsse.jar;D:\application\JDK\jre\lib\management-agent.jar;D:\application\JDK\jre\lib\plugin.jar;D:\application\JDK\jre\lib\resources.jar;D:\application\JDK\jre\lib\rt.jar;E:\code_bigData\spark_study_42\target\classes;D:\soft\maven\repository\org\apache\spark\spark-core_2.11\2.4.5\spark-core_2.11-2.4.5.jar;D:\soft\maven\repository\com\thoughtworks\paranamer\paranamer\2.8\paranamer-2.8.jar;D:\soft\maven\repository\org\apache\avro\avro\1.8.2\avro-1.8.2.jar;D:\soft\maven\repository\org\codehaus\jackson\jackson-core-asl\1.9.13\jackson-core-asl-1.9.13.jar;D:\soft\maven\repository\org\apache\commons\commons-compress\1.8.1\commons-compress-1.8.1.jar;D:\soft\maven\repository\org\tukaani\xz\1.5\xz-1.5.jar;D:\soft\maven\repository\org\apache\avro\avro-mapred\1.8.2\avro-mapred-1.8.2-hadoop2.jar;D:\soft\maven\repository\org\apache\avro\avro-ipc\1.8.2\avro-ipc-1.8.2.jar;D:\soft\maven\repository\com\twitter\chill_2.11\0.9.3\chill_2.11-0.9.3.jar;D:\soft\maven\repository\com\esotericsoftware\kryo-shaded\4.0.2\kryo-shaded-4.0.2.jar;D:\soft\maven\repository\com\esotericsoftware\minlog\1.3.0\minlog-1.3.0.jar;D:\soft\maven\repository\org\objenesis\objenesis\2.5.1\objenesis-2.5.1.jar;D:\soft\maven\repository\com\twitter\chill-java\0.9.3\chill-java-0.9.3.jar;D:\soft\maven\repository\org\apache\xbean\xbean-asm6-shaded\4.8\xbean-asm6-shaded-4.8.jar;D:\soft\maven\repository\org\apache\spark\spark-launcher_2.11\2.4.5\spark-launcher_2.11-2.4.5.jar;D:\soft\maven\repository\org\apache\spark\spark-kvstore_2.11\2.4.5\spark-kvstore_2.11-2.4.5.jar;D:\soft\maven\repository\org\fusesource\leveldbjni\leveldbjni-all\1.8\leveldbjni-all-1.8.jar;D:\soft\maven\repository\com\fasterxml\jackson\core\jackson-core\2.6.7\jackson-core-2.6.7.jar;D:\soft\maven\repository\com\fasterxml\jackson\core\jackson-annotations\2.6.7\jackson-annotations-2.6.7.jar;D:\soft\maven\repository\org\apache\spark\spark-network-common_2.11\2.4.5\spark-network-common_2.11-2.4.5.jar;D:\soft\maven\repository\org\apache\spark\spark-network-shuffle_2.11\2.4.5\spark-network-shuffle_2.11-2.4.5.jar;D:\soft\maven\repository\org\apache\spark\spark-unsafe_2.11\2.4.5\spark-unsafe_2.11-2.4.5.jar;D:\soft\maven\repository\javax\activation\activation\1.1.1\activation-1.1.1.jar;D:\soft\maven\repository\org\apache\curator\curator-recipes\2.6.0\curator-recipes-2.6.0.jar;D:\soft\maven\repository\org\apache\curator\curator-framework\2.6.0\curator-framework-2.6.0.jar;D:\soft\maven\repository\com\google\guava\guava\16.0.1\guava-16.0.1.jar;D:\soft\maven\repository\org\apache\zookeeper\zookeeper\3.4.6\zookeeper-3.4.6.jar;D:\soft\maven\repository\javax\servlet\javax.servlet-api\3.1.0\javax.servlet-api-3.1.0.jar;D:\soft\maven\repository\org\apache\commons\commons-lang3\3.5\commons-lang3-3.5.jar;D:\soft\maven\repository\org\apache\commons\commons-math3\3.4.1\commons-math3-3.4.1.jar;D:\soft\maven\repository\com\google\code\findbugs\jsr305\1.3.9\jsr305-1.3.9.jar;D:\soft\maven\repository\org\slf4j\slf4j-api\1.7.16\slf4j-api-1.7.16.jar;D:\soft\maven\repository\org\slf4j\jul-to-slf4j\1.7.16\jul-to-slf4j-1.7.16.jar;D:\soft\maven\repository\org\slf4j\jcl-over-slf4j\1.7.16\jcl-over-slf4j-1.7.16.jar;D:\soft\maven\repository\log4j\log4j\1.2.17\log4j-1.2.17.jar;D:\soft\maven\repository\org\slf4j\slf4j-log4j12\1.7.16\slf4j-log4j12-1.7.16.jar;D:\soft\maven\repository\com\ning\compress-lzf\1.0.3\compress-lzf-1.0.3.jar;D:\soft\maven\repository\org\xerial\snappy\snappy-java\1.1.7.3\snappy-java-1.1.7.3.jar;D:\soft\maven\repository\org\lz4\lz4-java\1.4.0\lz4-java-1.4.0.jar;D:\soft\maven\repository\com\github\luben\zstd-jni\1.3.2-2\zstd-jni-1.3.2-2.jar;D:\soft\maven\repository\org\roaringbitmap\RoaringBitmap\0.7.45\RoaringBitmap-0.7.45.jar;D:\soft\maven\repository\org\roaringbitmap\shims\0.7.45\shims-0.7.45.jar;D:\soft\maven\repository\commons-net\commons-net\3.1\commons-net-3.1.jar;D:\soft\maven\repository\org\scala-lang\scala-library\2.11.12\scala-library-2.11.12.jar;D:\soft\maven\repository\org\json4s\json4s-jackson_2.11\3.5.3\json4s-jackson_2.11-3.5.3.jar;D:\soft\maven\repository\org\json4s\json4s-core_2.11\3.5.3\json4s-core_2.11-3.5.3.jar;D:\soft\maven\repository\org\json4s\json4s-ast_2.11\3.5.3\json4s-ast_2.11-3.5.3.jar;D:\soft\maven\repository\org\json4s\json4s-scalap_2.11\3.5.3\json4s-scalap_2.11-3.5.3.jar;D:\soft\maven\repository\org\scala-lang\modules\scala-xml_2.11\1.0.6\scala-xml_2.11-1.0.6.jar;D:\soft\maven\repository\org\glassfish\jersey\core\jersey-client\2.22.2\jersey-client-2.22.2.jar;D:\soft\maven\repository\javax\ws\rs\javax.ws.rs-api\2.0.1\javax.ws.rs-api-2.0.1.jar;D:\soft\maven\repository\org\glassfish\hk2\hk2-api\2.4.0-b34\hk2-api-2.4.0-b34.jar;D:\soft\maven\repository\org\glassfish\hk2\hk2-utils\2.4.0-b34\hk2-utils-2.4.0-b34.jar;D:\soft\maven\repository\org\glassfish\hk2\external\aopalliance-repackaged\2.4.0-b34\aopalliance-repackaged-2.4.0-b34.jar;D:\soft\maven\repository\org\glassfish\hk2\external\javax.inject\2.4.0-b34\javax.inject-2.4.0-b34.jar;D:\soft\maven\repository\org\glassfish\hk2\hk2-locator\2.4.0-b34\hk2-locator-2.4.0-b34.jar;D:\soft\maven\repository\org\javassist\javassist\3.18.1-GA\javassist-3.18.1-GA.jar;D:\soft\maven\repository\org\glassfish\jersey\core\jersey-common\2.22.2\jersey-common-2.22.2.jar;D:\soft\maven\repository\javax\annotation\javax.annotation-api\1.2\javax.annotation-api-1.2.jar;D:\soft\maven\repository\org\glassfish\jersey\bundles\repackaged\jersey-guava\2.22.2\jersey-guava-2.22.2.jar;D:\soft\maven\repository\org\glassfish\hk2\osgi-resource-locator\1.0.1\osgi-resource-locator-1.0.1.jar;D:\soft\maven\repository\org\glassfish\jersey\core\jersey-server\2.22.2\jersey-server-2.22.2.jar;D:\soft\maven\repository\org\glassfish\jersey\media\jersey-media-jaxb\2.22.2\jersey-media-jaxb-2.22.2.jar;D:\soft\maven\repository\javax\validation\validation-api\1.1.0.Final\validation-api-1.1.0.Final.jar;D:\soft\maven\repository\org\glassfish\jersey\containers\jersey-container-servlet\2.22.2\jersey-container-servlet-2.22.2.jar;D:\soft\maven\repository\org\glassfish\jersey\containers\jersey-container-servlet-core\2.22.2\jersey-container-servlet-core-2.22.2.jar;D:\soft\maven\repository\io\netty\netty-all\4.1.42.Final\netty-all-4.1.42.Final.jar;D:\soft\maven\repository\io\netty\netty\3.9.9.Final\netty-3.9.9.Final.jar;D:\soft\maven\repository\com\clearspring\analytics\stream\2.7.0\stream-2.7.0.jar;D:\soft\maven\repository\io\dropwizard\metrics\metrics-core\3.1.5\metrics-core-3.1.5.jar;D:\soft\maven\repository\io\dropwizard\metrics\metrics-jvm\3.1.5\metrics-jvm-3.1.5.jar;D:\soft\maven\repository\io\dropwizard\metrics\metrics-json\3.1.5\metrics-json-3.1.5.jar;D:\soft\maven\repository\io\dropwizard\metrics\metrics-graphite\3.1.5\metrics-graphite-3.1.5.jar;D:\soft\maven\repository\com\fasterxml\jackson\core\jackson-databind\2.6.7.3\jackson-databind-2.6.7.3.jar;D:\soft\maven\repository\com\fasterxml\jackson\module\jackson-module-scala_2.11\2.6.7.1\jackson-module-scala_2.11-2.6.7.1.jar;D:\soft\maven\repository\org\scala-lang\scala-reflect\2.11.8\scala-reflect-2.11.8.jar;D:\soft\maven\repository\com\fasterxml\jackson\module\jackson-module-paranamer\2.7.9\jackson-module-paranamer-2.7.9.jar;D:\soft\maven\repository\org\apache\ivy\ivy\2.4.0\ivy-2.4.0.jar;D:\soft\maven\repository\oro\oro\2.0.8\oro-2.0.8.jar;D:\soft\maven\repository\net\razorvine\pyrolite\4.13\pyrolite-4.13.jar;D:\soft\maven\repository\net\sf\py4j\py4j\0.10.7\py4j-0.10.7.jar;D:\soft\maven\repository\org\apache\spark\spark-tags_2.11\2.4.5\spark-tags_2.11-2.4.5.jar;D:\soft\maven\repository\org\apache\commons\commons-crypto\1.0.0\commons-crypto-1.0.0.jar;D:\soft\maven\repository\org\spark-project\spark\unused\1.0.0\unused-1.0.0.jar;D:\soft\maven\repository\org\apache\spark\spark-sql_2.11\2.4.5\spark-sql_2.11-2.4.5.jar;D:\soft\maven\repository\com\univocity\univocity-parsers\2.7.3\univocity-parsers-2.7.3.jar;D:\soft\maven\repository\org\apache\spark\spark-sketch_2.11\2.4.5\spark-sketch_2.11-2.4.5.jar;D:\soft\maven\repository\org\apache\spark\spark-catalyst_2.11\2.4.5\spark-catalyst_2.11-2.4.5.jar;D:\soft\maven\repository\org\codehaus\janino\janino\3.0.9\janino-3.0.9.jar;D:\soft\maven\repository\org\codehaus\janino\commons-compiler\3.0.9\commons-compiler-3.0.9.jar;D:\soft\maven\repository\org\antlr\antlr4-runtime\4.7\antlr4-runtime-4.7.jar;D:\soft\maven\repository\org\apache\orc\orc-core\1.5.5\orc-core-1.5.5-nohive.jar;D:\soft\maven\repository\org\apache\orc\orc-shims\1.5.5\orc-shims-1.5.5.jar;D:\soft\maven\repository\com\google\protobuf\protobuf-java\2.5.0\protobuf-java-2.5.0.jar;D:\soft\maven\repository\commons-lang\commons-lang\2.6\commons-lang-2.6.jar;D:\soft\maven\repository\io\airlift\aircompressor\0.10\aircompressor-0.10.jar;D:\soft\maven\repository\org\apache\orc\orc-mapreduce\1.5.5\orc-mapreduce-1.5.5-nohive.jar;D:\soft\maven\repository\org\apache\parquet\parquet-column\1.10.1\parquet-column-1.10.1.jar;D:\soft\maven\repository\org\apache\parquet\parquet-common\1.10.1\parquet-common-1.10.1.jar;D:\soft\maven\repository\org\apache\parquet\parquet-encoding\1.10.1\parquet-encoding-1.10.1.jar;D:\soft\maven\repository\org\apache\parquet\parquet-hadoop\1.10.1\parquet-hadoop-1.10.1.jar;D:\soft\maven\repository\org\apache\parquet\parquet-format\2.4.0\parquet-format-2.4.0.jar;D:\soft\maven\repository\org\apache\parquet\parquet-jackson\1.10.1\parquet-jackson-1.10.1.jar;D:\soft\maven\repository\org\apache\arrow\arrow-vector\0.10.0\arrow-vector-0.10.0.jar;D:\soft\maven\repository\org\apache\arrow\arrow-format\0.10.0\arrow-format-0.10.0.jar;D:\soft\maven\repository\org\apache\arrow\arrow-memory\0.10.0\arrow-memory-0.10.0.jar;D:\soft\maven\repository\com\carrotsearch\hppc\0.7.2\hppc-0.7.2.jar;D:\soft\maven\repository\com\vlkan\flatbuffers\1.2.0-3f79e055\flatbuffers-1.2.0-3f79e055.jar;D:\soft\maven\repository\org\apache\spark\spark-hive_2.11\2.4.5\spark-hive_2.11-2.4.5.jar;D:\soft\maven\repository\com\twitter\parquet-hadoop-bundle\1.6.0\parquet-hadoop-bundle-1.6.0.jar;D:\soft\maven\repository\org\spark-project\hive\hive-exec\1.2.1.spark2\hive-exec-1.2.1.spark2.jar;D:\soft\maven\repository\commons-io\commons-io\2.4\commons-io-2.4.jar;D:\soft\maven\repository\javolution\javolution\5.5.1\javolution-5.5.1.jar;D:\soft\maven\repository\log4j\apache-log4j-extras\1.2.17\apache-log4j-extras-1.2.17.jar;D:\soft\maven\repository\org\antlr\antlr-runtime\3.4\antlr-runtime-3.4.jar;D:\soft\maven\repository\org\antlr\stringtemplate\3.2.1\stringtemplate-3.2.1.jar;D:\soft\maven\repository\antlr\antlr\2.7.7\antlr-2.7.7.jar;D:\soft\maven\repository\org\antlr\ST4\4.0.4\ST4-4.0.4.jar;D:\soft\maven\repository\com\googlecode\javaewah\JavaEWAH\0.3.2\JavaEWAH-0.3.2.jar;D:\soft\maven\repository\org\iq80\snappy\snappy\0.2\snappy-0.2.jar;D:\soft\maven\repository\stax\stax-api\1.0.1\stax-api-1.0.1.jar;D:\soft\maven\repository\net\sf\opencsv\opencsv\2.3\opencsv-2.3.jar;D:\soft\maven\repository\org\spark-project\hive\hive-metastore\1.2.1.spark2\hive-metastore-1.2.1.spark2.jar;D:\soft\maven\repository\com\jolbox\bonecp\0.8.0.RELEASE\bonecp-0.8.0.RELEASE.jar;D:\soft\maven\repository\commons-cli\commons-cli\1.2\commons-cli-1.2.jar;D:\soft\maven\repository\commons-logging\commons-logging\1.1.3\commons-logging-1.1.3.jar;D:\soft\maven\repository\org\datanucleus\datanucleus-api-jdo\3.2.6\datanucleus-api-jdo-3.2.6.jar;D:\soft\maven\repository\org\datanucleus\datanucleus-rdbms\3.2.9\datanucleus-rdbms-3.2.9.jar;D:\soft\maven\repository\commons-pool\commons-pool\1.5.4\commons-pool-1.5.4.jar;D:\soft\maven\repository\commons-dbcp\commons-dbcp\1.4\commons-dbcp-1.4.jar;D:\soft\maven\repository\javax\jdo\jdo-api\3.0.1\jdo-api-3.0.1.jar;D:\soft\maven\repository\javax\transaction\jta\1.1\jta-1.1.jar;D:\soft\maven\repository\commons-httpclient\commons-httpclient\3.1\commons-httpclient-3.1.jar;D:\soft\maven\repository\org\apache\calcite\calcite-avatica\1.2.0-incubating\calcite-avatica-1.2.0-incubating.jar;D:\soft\maven\repository\org\apache\calcite\calcite-core\1.2.0-incubating\calcite-core-1.2.0-incubating.jar;D:\soft\maven\repository\org\apache\calcite\calcite-linq4j\1.2.0-incubating\calcite-linq4j-1.2.0-incubating.jar;D:\soft\maven\repository\net\hydromatic\eigenbase-properties\1.1.5\eigenbase-properties-1.1.5.jar;D:\soft\maven\repository\org\apache\httpcomponents\httpclient\4.5.6\httpclient-4.5.6.jar;D:\soft\maven\repository\org\apache\httpcomponents\httpcore\4.4.10\httpcore-4.4.10.jar;D:\soft\maven\repository\org\codehaus\jackson\jackson-mapper-asl\1.9.13\jackson-mapper-asl-1.9.13.jar;D:\soft\maven\repository\commons-codec\commons-codec\1.10\commons-codec-1.10.jar;D:\soft\maven\repository\joda-time\joda-time\2.9.3\joda-time-2.9.3.jar;D:\soft\maven\repository\org\jodd\jodd-core\3.5.2\jodd-core-3.5.2.jar;D:\soft\maven\repository\org\datanucleus\datanucleus-core\3.2.10\datanucleus-core-3.2.10.jar;D:\soft\maven\repository\org\apache\thrift\libthrift\0.9.3\libthrift-0.9.3.jar;D:\soft\maven\repository\org\apache\thrift\libfb303\0.9.3\libfb303-0.9.3.jar;D:\soft\maven\repository\org\apache\derby\derby\10.12.1.1\derby-10.12.1.1.jar;D:\soft\maven\repository\org\apache\spark\spark-hive-thriftserver_2.11\2.4.5\spark-hive-thriftserver_2.11-2.4.5.jar;D:\soft\maven\repository\org\spark-project\hive\hive-cli\1.2.1.spark2\hive-cli-1.2.1.spark2.jar;D:\soft\maven\repository\jline\jline\2.12\jline-2.12.jar;D:\soft\maven\repository\org\spark-project\hive\hive-jdbc\1.2.1.spark2\hive-jdbc-1.2.1.spark2.jar;D:\soft\maven\repository\org\spark-project\hive\hive-beeline\1.2.1.spark2\hive-beeline-1.2.1.spark2.jar;D:\soft\maven\repository\net\sf\supercsv\super-csv\2.2.0\super-csv-2.2.0.jar;D:\soft\maven\repository\net\sf\jpam\jpam\1.1\jpam-1.1.jar;D:\soft\maven\repository\org\apache\spark\spark-streaming_2.11\2.4.5\spark-streaming_2.11-2.4.5.jar;D:\soft\maven\repository\org\apache\spark\spark-mllib_2.11\2.4.5\spark-mllib_2.11-2.4.5.jar;D:\soft\maven\repository\org\scala-lang\modules\scala-parser-combinators_2.11\1.1.0\scala-parser-combinators_2.11-1.1.0.jar;D:\soft\maven\repository\org\apache\spark\spark-graphx_2.11\2.4.5\spark-graphx_2.11-2.4.5.jar;D:\soft\maven\repository\com\github\fommil\netlib\core\1.1.2\core-1.1.2.jar;D:\soft\maven\repository\net\sourceforge\f2j\arpack_combined_all\0.1\arpack_combined_all-0.1.jar;D:\soft\maven\repository\org\apache\spark\spark-mllib-local_2.11\2.4.5\spark-mllib-local_2.11-2.4.5.jar;D:\soft\maven\repository\org\scalanlp\breeze_2.11\0.13.2\breeze_2.11-0.13.2.jar;D:\soft\maven\repository\org\scalanlp\breeze-macros_2.11\0.13.2\breeze-macros_2.11-0.13.2.jar;D:\soft\maven\repository\com\github\rwl\jtransforms\2.4.0\jtransforms-2.4.0.jar;D:\soft\maven\repository\org\spire-math\spire_2.11\0.13.0\spire_2.11-0.13.0.jar;D:\soft\maven\repository\org\spire-math\spire-macros_2.11\0.13.0\spire-macros_2.11-0.13.0.jar;D:\soft\maven\repository\org\typelevel\machinist_2.11\0.6.1\machinist_2.11-0.6.1.jar;D:\soft\maven\repository\com\chuusai\shapeless_2.11\2.3.2\shapeless_2.11-2.3.2.jar;D:\soft\maven\repository\org\typelevel\macro-compat_2.11\1.1.1\macro-compat_2.11-1.1.1.jar;D:\soft\maven\repository\org\apache\spark\spark-streaming-kafka-0-10_2.11\2.4.5\spark-streaming-kafka-0-10_2.11-2.4.5.jar;D:\soft\maven\repository\org\apache\kafka\kafka-clients\2.0.0\kafka-clients-2.0.0.jar;D:\soft\maven\repository\org\apache\spark\spark-sql-kafka-0-10_2.11\2.4.5\spark-sql-kafka-0-10_2.11-2.4.5.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-client\2.7.5\hadoop-client-2.7.5.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-common\2.7.5\hadoop-common-2.7.5.jar;D:\soft\maven\repository\xmlenc\xmlenc\0.52\xmlenc-0.52.jar;D:\soft\maven\repository\commons-collections\commons-collections\3.2.2\commons-collections-3.2.2.jar;D:\soft\maven\repository\org\mortbay\jetty\jetty-sslengine\6.1.26\jetty-sslengine-6.1.26.jar;D:\soft\maven\repository\javax\servlet\jsp\jsp-api\2.1\jsp-api-2.1.jar;D:\soft\maven\repository\commons-configuration\commons-configuration\1.6\commons-configuration-1.6.jar;D:\soft\maven\repository\commons-digester\commons-digester\1.8\commons-digester-1.8.jar;D:\soft\maven\repository\commons-beanutils\commons-beanutils\1.7.0\commons-beanutils-1.7.0.jar;D:\soft\maven\repository\commons-beanutils\commons-beanutils-core\1.8.0\commons-beanutils-core-1.8.0.jar;D:\soft\maven\repository\com\google\code\gson\gson\2.2.4\gson-2.2.4.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-auth\2.7.5\hadoop-auth-2.7.5.jar;D:\soft\maven\repository\org\apache\directory\server\apacheds-kerberos-codec\2.0.0-M15\apacheds-kerberos-codec-2.0.0-M15.jar;D:\soft\maven\repository\org\apache\directory\server\apacheds-i18n\2.0.0-M15\apacheds-i18n-2.0.0-M15.jar;D:\soft\maven\repository\org\apache\directory\api\api-asn1-api\1.0.0-M20\api-asn1-api-1.0.0-M20.jar;D:\soft\maven\repository\org\apache\directory\api\api-util\1.0.0-M20\api-util-1.0.0-M20.jar;D:\soft\maven\repository\org\apache\curator\curator-client\2.7.1\curator-client-2.7.1.jar;D:\soft\maven\repository\org\apache\htrace\htrace-core\3.1.0-incubating\htrace-core-3.1.0-incubating.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-hdfs\2.7.5\hadoop-hdfs-2.7.5.jar;D:\soft\maven\repository\org\mortbay\jetty\jetty-util\6.1.26\jetty-util-6.1.26.jar;D:\soft\maven\repository\xerces\xercesImpl\2.9.1\xercesImpl-2.9.1.jar;D:\soft\maven\repository\xml-apis\xml-apis\1.3.04\xml-apis-1.3.04.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-mapreduce-client-app\2.7.5\hadoop-mapreduce-client-app-2.7.5.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-mapreduce-client-common\2.7.5\hadoop-mapreduce-client-common-2.7.5.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-yarn-client\2.7.5\hadoop-yarn-client-2.7.5.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-yarn-server-common\2.7.5\hadoop-yarn-server-common-2.7.5.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-mapreduce-client-shuffle\2.7.5\hadoop-mapreduce-client-shuffle-2.7.5.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-yarn-api\2.7.5\hadoop-yarn-api-2.7.5.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-mapreduce-client-core\2.7.5\hadoop-mapreduce-client-core-2.7.5.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-yarn-common\2.7.5\hadoop-yarn-common-2.7.5.jar;D:\soft\maven\repository\javax\xml\bind\jaxb-api\2.2.2\jaxb-api-2.2.2.jar;D:\soft\maven\repository\javax\xml\stream\stax-api\1.0-2\stax-api-1.0-2.jar;D:\soft\maven\repository\javax\servlet\servlet-api\2.5\servlet-api-2.5.jar;D:\soft\maven\repository\com\sun\jersey\jersey-core\1.9\jersey-core-1.9.jar;D:\soft\maven\repository\com\sun\jersey\jersey-client\1.9\jersey-client-1.9.jar;D:\soft\maven\repository\org\codehaus\jackson\jackson-jaxrs\1.9.13\jackson-jaxrs-1.9.13.jar;D:\soft\maven\repository\org\codehaus\jackson\jackson-xc\1.9.13\jackson-xc-1.9.13.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-mapreduce-client-jobclient\2.7.5\hadoop-mapreduce-client-jobclient-2.7.5.jar;D:\soft\maven\repository\org\apache\hadoop\hadoop-annotations\2.7.5\hadoop-annotations-2.7.5.jar;D:\soft\maven\repository\mysql\mysql-connector-java\5.1.38\mysql-connector-java-5.1.38.jar;D:\soft\maven\repository\com\alibaba\fastjson\1.2.47\fastjson-1.2.47.jar;D:\soft\maven\repository\com\hankcs\hanlp\portable-1.7.7\hanlp-portable-1.7.7.jar;D:\soft\maven\repository\redis\clients\jedis\2.9.0\jedis-2.9.0.jar;D:\soft\maven\repository\org\apache\commons\commons-pool2\2.4.2\commons-pool2-2.4.2.jar TestOOM
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid23136.hprof ...
Heap dump file created [10258550 bytes in 0.030 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:267)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:241)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:233)
	at java.util.ArrayList.add(ArrayList.java:464)
	at TestOOM.main(TestOOM.java:12)

Process finished with exit code 1
```

> 可以看到，生成了一个文件`java_pid23136.hprof`，这个文件在项目的`src`目录下，我们直接双击就可以用JProfiler打开

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_JVM/5.png)

> 我们发现，有一个对象，直接占用了77%的内存

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_JVM/6.png)

> 问题出在list的add方法，可以看到具体代码是在TestOOM类的main方法中的13行。

## 4 代码查看堆内存使用情况

```
public class TestJVM {
    public static void main(String[] args) {
        //返回虚拟机试图使用的最大内存
        long max = Runtime.getRuntime().maxMemory();
        //返回JVM的初始化总内存
        long total = Runtime.getRuntime().totalMemory();

        System.out.println("max="+max+"字节\t"+(max/(double)1024/1024)+"MB\t"+"默认分配的总内存一般占电脑总内存的1/4");
        System.out.println("total="+total+"字节\t"+(total/(double)1024/1024)+"MB\t"+"默认初始化的内存一般占电脑总内存的1/64");
        //max=7607943168字节	7255.5MB	默认分配的总内存一般占电脑总内存的1/4
        //total=514850816字节	491.0MB	默认初始化的内存一般占电脑总内存的1/64

        //VMOptions调整成这样，然后再执行：-Xms1024m -Xmx1024m -XX:+PrintGCDetails

        //max=1029177344字节	981.5MB	默认分配的总内存一般占电脑总内存的1/4
        //total=1029177344字节	981.5MB	默认初始化的内存一般占电脑总内存的1/64
        //Heap
        //  PSYoungGen      total 305664K, used 20971K [0x00000000eab00000, 0x0000000100000000, 0x0000000100000000)
        //      eden space 262144K, 8% used [0x00000000eab00000,0x00000000ebf7afb8,0x00000000fab00000)
        //      from space 43520K, 0% used [0x00000000fd580000,0x00000000fd580000,0x0000000100000000)
        //      to   space 43520K, 0% used [0x00000000fab00000,0x00000000fab00000,0x00000000fd580000)
        //  ParOldGen       total 699392K, used 0K [0x00000000c0000000, 0x00000000eab00000, 0x00000000eab00000)
        //      object space 699392K, 0% used [0x00000000c0000000,0x00000000c0000000,0x00000000eab00000)
        //  Metaspace       used 3358K, capacity 4496K, committed 4864K, reserved 1056768K
        //      class space    used 361K, capacity 388K, committed 512K, reserved 1048576K
    }
}
```

> 可以看到`新生代+老年代=总内存`  (305664+699392)/1024=981.5MB

> 也就是说：`元空间逻辑上在堆里，物理上它是不在堆里的`

## 5 JVM调优常用参数

> 后续再慢慢加

参数设置 | 描述 
---|---
-Xms4800m | 初始化堆空间大小
-Xmx4800m | 最大堆空间大小
-Xmn1800m | 年轻代的空间大小
-XX:+PrintGCDetails | 打印GC详情日志
-XX:+PrintGCDateStamps | 打印GC的耗时
-XX:+HeapDumpOnOutOfMemoryError | 当抛出OOM时进行HeapDump
-XX:HeapDumpPath=/home/admin/logs | 指定HeapDump的路径或目录

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)

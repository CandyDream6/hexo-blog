---
title: java多线程
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(99).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(99).jpg
tags:
  - 大数据
  - Java
category:
  - - 编程
    - Java
  - - 编程
    - 多线程
date: 2021-10-19 11:28:12
---

多线程相关内容

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 基础概念

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java多线程/1.png)

### 1.1 程序、进程、线程、多线程

> `程序是静态资源`。`程序加载到内存运行叫线程`。`线程中可以有多个进程`。

#### 1.1.1 程序

> 程序是`指令和数据的有序集合`，其本身没有任何运行的含义，是一个`静态的概念`。

#### 1.1.2 进程

> 进程是执`行程序的一次执行过程`，它是动态的概念。是系统资源分配的单位。

#### 1.1.3 线程

> 一个进程中可以包含若干个线程，当然一个进程至少有一个线程，不然没有存在的意义。`线程是CPU调度和执行的单位`。

#### 1.1.4 多线程

> `多线程就是多个线程同时运行或交替运行`。很多多线程是模拟出来的，真正的多线程是指有多个CPU，即多核，如服务器。如果是模拟出来的多线程，即在一个CPU的情况下，在同一时间点，cpu只能执行一个代码，因为切换的很快，所以有同时执行的错觉。

#### 1.1.5 杂记

##### 1.1.5.1 为什么多线程是必要的
``
> 个人觉得可以用一句话概括：开发高并发系统的基础，利用好多线程机制`可以大大提高系统整体的并发能力以及性能`。

##### 1.1.5.2 为什么提倡多线程而不是多进程

> `线程就是轻量级进程，是程序执行的最小单位`。使用多线程而不是用多进程去进行并发程序的设计，是因为`线程间的切换和调度的成本远远小于进程`。

##### 1.1.5.3 线程有什么优缺点

> `好处 `
>> 使用多线程可以`把程序中占据时间长的任务放到后台去处理`，如图片、视屏的下载。
> 
>> `发挥多核处理器的优势`，并发执行`让系统运行的更快、更流畅`，`用户体验更好`。 

> `坏处 `
>> 大量的线程`降低代码的可读性`。
> 
>> 更多的线程需要`更多的内存空间`。
> 
>> 当多个线程对同一个资源出现争夺时候要`注意线程安全的问题`。

### 1.2 其他概念

#### 1.2.1 同步和异步

> `同步和异步通常用来形容一次方法调`用。同步方法调用一旦开始，调用者必须等到方法调用返回后，才能继续后续的行为。异步方法调用更像一个消息传递，一旦开始，方法调用就会立即返回，调用者可以继续后续的操作。

#### 1.2.2 阻塞和非阻塞

> `非阻塞指在不能立刻得到结果之前，该函数不会阻塞当前线程，而会立刻返回，而阻塞与之相反`。

> 看着和同异步是一个概念，我们看《操作系统》中是怎么说的(就是一个概念)
>> 进程间的通信是通过 send() 和 receive() 两种基本操作完成的。具体如何实现这两种基础操作，存在着不同的设计。 `消息的传递有可能是阻塞的或非阻塞的 – 也被称为同步或异步的`. 

#### 1.2.3 并发和并行

> 并发(Concurrency)和并行(Parallelism) 并发和并行是两个非常容易被混淆的概念。它们都可以表示两个或者多个任务一起执行，但是偏重点有些不同。

> `并发偏重于多个任务交替执行`，而多个任务之间有可能还是串行的。而`并行是真正意义上的“同时执行`”。

#### 1.2.4 高并发

> 高并发（High Concurrency）是互联网分布式系统架构设计中必须考虑的因素之一，它通常是指，`通过设计保证系统能够同时并行处理很多请求`。

#### 1.2.5 临界区

> `临界区用来表示一种公共资源或者说是共享数据，可以被多个线程使用`。

> 但是每一次，只能有一个线程使用它，一旦临界区资源被占用，其他线程要想使用这个资源，就必须等待。

> `在并行程序中，临界区资源是保护的对象`。

## 2 创建线程的四种方式

### 2.1 继承Thread类

> 实现run方法

```java
package com.ma.thread;
Fluid 
public class ThreadTest {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println(Thread.currentThread());
    }
}

-------------------------------------------------------------------------------------
输出：
Thread[Thread-0,5,main]
```

### 2.2 实现Runnable接口

> 实现run方法

```java
package com.ma.runable;

public class RunnableTest {
    public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        Thread thread = new Thread(myRunnable, "我的Runnable测试线程");
        thread.start();
    }
}

class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread());
    }
}

-------------------------------------------------------------------------------------
输出：
Thread[我的Runnable测试线程,5,main]
```

### 2.3 实现Callable接口

> 实现call方法

```java
package com.ma.callable;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class CallableTest {
    public static void main(String[] args) {
        //1.创建Callable实现类的实例
        MyCallable myCallable = new MyCallable();
        //2.使用FutureTask类进行包装Callable对象，FutureTask对象封装了Callable对象的call()方法的返回值
        FutureTask<String> futureTask = new FutureTask<>(myCallable);

        System.out.println(Thread.currentThread() + "：=========futureTask线程启动之前=========");
        //3 启动futureTask线程
        new Thread(futureTask, "有返回值的线程FutureTask").start();
        System.out.println(Thread.currentThread() + "：=========futureTask线程启动之后=========");

        try {
            //4.调用FutureTask对象的get（）方法获取子线程执行结束后的返回值。
            //get()方法会阻塞，直到子线程执行结束才返回
            System.out.println(Thread.currentThread() + "：=========futureTask get 方法之前=========");
            System.out.println(Thread.currentThread() + "：子线程的返回值：" + futureTask.get());
            System.out.println(Thread.currentThread() + "：=========futureTask get 方法之后=========");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}

class MyCallable implements Callable<String> {
    @Override
    public String call() {
        System.out.println(Thread.currentThread());
        return "Fuck You";
    }
}

-------------------------------------------------------------------------------------
输出：
Thread[main,5,main]：=========futureTask线程启动之前=========
Thread[main,5,main]：=========futureTask线程启动之后=========
Thread[main,5,main]：=========futureTask get 方法之前=========
Thread[有返回值的线程FutureTask,5,main]
Thread[main,5,main]：子线程的返回值：Fuck You
Thread[main,5,main]：=========futureTask get 方法之后=========
```

### 2.4 线程池ThreadPoolExecutor

```java
package com.ma.threadpool;

import java.util.concurrent.*;

public class MyThreadPool {
    public static void main(String[] args) {
        ExecutorService executorService = new ThreadPoolExecutor(
                5,
                5,
                1L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(100),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy()
        );
        for (int i = 0; i < 5; i++) {
            RunnableTest thread = new RunnableTest();
            executorService.execute(thread);
        }
    }
}

class RunnableTest implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread());
    }
}

-------------------------------------------------------------------------------------
输出：
Thread[pool-1-thread-1,5,main]
Thread[pool-1-thread-3,5,main]
Thread[pool-1-thread-2,5,main]
Thread[pool-1-thread-4,5,main]
Thread[pool-1-thread-5,5,main]
```

## 8 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java多线程/1.png)
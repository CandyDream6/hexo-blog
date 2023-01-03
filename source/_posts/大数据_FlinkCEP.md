---
title: FlinkCEP Flink复杂事件处理
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(60).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(60).jpg
tags:
  - 大数据
  - Flink
category:
  - - 编程
    - 大数据
 
date: 2020-04-17 15:35:33
---

FlinkCEP是在Flink之上实现的复杂事件处理(CEP)库。

它允许您在无穷无尽的事件流中检测事件模式，使您有机会掌握数据中什么是重要的。

它可以用于处理实时数据并在事件流到达时从事件流中提取信息，并根据定义的规则来判断事件是否匹配，如果匹配则会触发新的事件做出响应。

除了支持单个事件的简单无状态的模式匹配（例如基于事件中的某个字段进行筛选过滤），也可以支持基于关联／聚合／时间窗口等多个事件的复杂有状态模式的匹配（例如判断用户下单事件后 30 分钟内是否有支付事件）。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 FlinkCEP概述

### 1.1 官网

https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/libs/cep.html

### 1.2 FlinkCEP是什么

> Complex Event Processing（CEP）是 Flink 提供的一个非常亮眼的功能，是Flink提供的复杂事件处理（CEP）库，使用它可以在无界的事件流中检测事件模式，让我们可以掌握数据中重要的事项。并允许指定要在流中检测的模式，然后检测匹配事件序列并对其进行操作。

> 复杂事件处理实际上就是基于事件流进行数据处理，把要分析的数据抽象成事件，然后将数据发送到CEP引擎，得到事件处理结果。

> 说到底，Flink 的 CEP 到底解决了什么样的问题呢？

> 在我们的实际生产中，随着数据的实时性要求越来越高，实时数据的量也在不断膨胀，在某些业务场景中需要根据连续的实时数据，发现其中有价值的那些事件。

> 比如，我们需要在大量的订单交易中发现那些虚假交易，在网站的访问日志中寻找那些使用脚本或者工具“爆破”登录的用户，或者在快递运输中发现那些滞留很久没有签收的包裹等。

> Flink 对 CEP 的支持非常友好，并且支持复杂度非常高的模式匹配，其吞吐和延迟都令人满意。

> CEP可以简单理解为:
>> 一个或多个由简单事件构成的事件流通过一定的规则匹配，然后输出用户想得到，满足规则的复杂事件/数据。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/1.png)

### 1.3 FlinkCEP使用场景

> Flink CEP应用于实时数据流的业务场景，可以应用于规则匹配，数据监控，实时预警、异常行为监测、风控等业务范围，具体有如下应用场景：

```
1.实时股票曲线预测
2.网站恶意攻击登陆行为
3.电子商务实时营销，对用户行为实时分析
4.滴滴打车异常行为检测
5.物流订单实时追踪
6.网络欺诈
7.故障检测
8.风险规避
9.智能营销等领域
```

> 下面是一些具体详细的例子

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/2.png)               

> `1.风险控制：`
>> 对用户异常行为模式进行实时检测，当一个用户发生了不该发生的行为，判定这个用户是不是有违规操作的嫌疑。
假设车辆维修的场景中，当一辆车出现故障时，这辆车会被送往维修点维修，然后被重新投放到市场运行。如果这辆车被投放到市场之后还未被使用就又被报障了，那么就有可能之前的维修是无效的。
对于电商来说，羊毛党是必不可少的，国内拼多多曾爆出 100 元的无门槛券随便领，当晚被人褥几百亿，对于这种情况肯定是没有做好及时的风控。另外还有就是商家上架商品时通过频繁修改商品的名称和滥用标题来提高搜索关键字的排名、批量注册一批机器账号快速刷单来提高商品的销售量等作弊行为，各种各样的作弊手法也是需要不断的去制定规则去匹配这种行为。

> `2.策略营销：`
>> 用预先定义好的规则对用户的行为轨迹进行实时跟踪，对行为轨迹匹配预定义规则的用户实时发送相应策略的推广。
假设打车的场景中，用户在 APP 上规划了一个行程订单，如果这个行程在下单之后超过一定的时间还没有被司机接单的话，那么就需要将这个订单输出到下游做相关的策略调整。
分析用户在手机 APP 的实时行为，统计用户的活动周期，通过为用户画像来给用户进行推荐。比如用户在登录 APP 后 1 分钟内只浏览了商品没有下单；用户在浏览一个商品后，3 分钟内又去查看其他同类的商品，进行比价行为；用户商品下单后 1 分钟内是否支付了该订单。如果这些数据都可以很好的利用起来，那么就可以给用户推荐浏览过的类似商品，这样可以大大提高购买率。

> `3.运维监控：`
>> 灵活配置多指标、多依赖来实现更复杂的监控模式。
通常运维会监控服务器的 CPU、网络 IO 等指标超过阈值时产生相应的告警。但是在实际使用中，后台服务的重启、网络抖动等情况都会造成瞬间的流量毛刺，对非关键链路可以忽略这些毛刺而只对频繁发生的异常进行告警以减少误报。

> `4.实时网络攻击检测`
>> 当下互联网安全形势仍然严峻，网络攻击屡见不鲜且花样众多，这里我们以 DDOS（分布式拒绝服务攻击）产生的流入流量来作为遭受攻击的判断依据。对网络遭受的潜在攻击进行实时检测并给出预警，云服务厂商的多个数据中心会定时向监控中心上报其瞬时流量，如果流量在预设的正常范围内则认为是正常现象，不做任何操作；如果某数据中心在 10 秒内连续 5 次上报的流量超过正常范围的阈值，则触发一条警告的事件；如果某数据中心 30 秒内连续出现 30 次上报的流量超过正常范围的阈值，则触发严重的告警。

### 1.4 FlinkCEP优缺点

> Flink的CEP是基于Flink Runtime构建的实时数据规则引擎，擅长解决跨事件的匹配问题, 是一套极具通用性、易于使用的实时流式事件处理方案。

> Flink CEP可以用于分析低延迟、频繁产生的不同来源的事件流。 CEP 可以帮助在复杂的、不相关的事件流中找出有意义的模式和复杂的关系，以接近实时或准实时的获得通知并阻止一些行为。

> Flink CEP支持在流 上进行模式匹配，根据模式的条件不同，分为连续的条件或不连续的条件；模式的条件允许有时间的限制，当在条件范围内没有达到满足的条件时，会导致模式匹配超时。

> `优势：`
>> 继承了 Flink 高吞吐的特点
> 
>> 查询是静态的，数据是动态的，满足实现和连续查询的需求
> 
>> 擅长解决跨事件的匹配
> 
>> API友好

> `劣势：`
>> 本身无法做的直接动态更新规则（痛点）,需要借助其他技术才可以动态注入或更新规则

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/3.png)

## 2 FlinkCEP原理

### 2.1 NFA 

> Apache Flink在实现CEP时借鉴了[Efficient Pattern Matching over Event Streams](https://people.cs.umass.edu/~yanlei/publications/sase-sigmod08.pdf)中NFA的模型

> 在这篇论文中，提到了NFA，也就是Non-determined Finite Automaton，叫做不确定的有限状态机，指的是状态有限，但是每个状态可能被转换成多个状态（不确定）。

### 2.2 状态和转换

> 先理解两个概念：
>> `状态`：状态分为三类，起始状态、中间状态和最终状态
>> `转换`：take/ignore/proceed都是转换的名称

> 在这NFA匹配规则里，本质上是一个状态转换的过程。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/4.png)

> Flink CEP 内部是用 NFA（非确定有限自动机）来实现的，由点和边组成的一个状态图，以一个初始状态作为起点，经过一系列的中间状态，达到终态。

> 点分为`起始状态`、`中间状态`、`最终状态`三种，

> 边分为 `take`、`ignore`、`proceed` 三种。
>> `take`：必须存在一个条件判断，当到来的消息满足 take 边条件判断时，把这个消息放入结果集，将状态转移到下一状态。
>> `ignore`：当消息到来时，可以忽略这个消息，将状态自旋在当前不变，是一个自己到自己的状态转移。
>> `proceed`：又叫做状态的空转移，当前状态可以不依赖于消息到来而直接转移到下一状态。举个例子，当用户购买商品时，如果购买前有一个咨询客服的行为，需要把咨询客服行为和购买行为两个消息一起放到结果集中向下游输出；如果购买前没有咨询客服的行为，只需把购买行为放到结果集中向下游输出就可以了。 也就是说，如果有咨询客服的行为，就存在咨询客服状态的上的消息保存，如果没有咨询客服的行为，就不存在咨询客服状态的上的消息保存，咨询客服状态是由一条 proceed 边和下游的购买状态相连。

### 2.3 CEP规则解析

> 我们以一个简单的CEP规则为例，看看在NFA中，这些事件之间是什么样的关系。

```
Pattern<Event, ?> pattern = Pattern.<Event>begin("start").where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) throws Exception {
        return value.getName().equals("a");
    }
}).followedBy("middle").optional().where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) throws Exception {
        return value.getName().equals("b");
    }
}).followedBy("end").where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) throws Exception {
        return value.getName().equals("c");
    }
});
```
> 上述代码描述的是start/middle/end之间的关系且每个事件满足的条件，其中，middle要在start之后，end要在middle之后；三者之间并不需要严格邻近，其中middle是可有可无的（optional），用NFA的结构来描述他们就是下面这张图：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/5.png)

> 现在让我们假设一条只有四个元素的数据流：

```
start -> xx -> middle -> end
```
> 收到start，满足条件，进行Take转换，当前状态转为middle。<br/>
> 收到xx，不满足条件，当前状态转为middle:0；因为有Proceed存在，当前状态转为end，但也不满足条件，所以丢弃这条转换。<br/>
> 收到middle，对于middle:0满足条件(Take)转换为end；对于end，不满足条件(Ignore)，转换为自身。<br/>
>收到end，满足条件(Take)，转换为$end$，结束匹配。

### 2.4 状态转换流程

> 下面以一个打车的例子来展示状态是如何流转的，规则见下图所示。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/6.png)

> 以乘客制定行程作为开始，匹配乘客的下单事件，如果这个订单超时还没有被司机接单的话，就把行程事件和下单事件作为结果集往下游输出。<br/>
> 假如消息到来顺序为：行程–>其他–>下单–>其他。<br/>
> 状态流转如下：

> 1.开始时状态处于行程状态，即等待用户制定行程。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/7.png)

> 2.当收到行程事件时，匹配行程状态的条件，把行程事件放到结果集中，通过 take 边将状态往下转移到下单状态。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/8.png)

> 3.由于下单状态上有一条 ignore 边，所以可以忽略收到的其他事件，直到收到下单事件时将其匹配，放入结果集中，并且将当前状态往下转移到超时未接单状态。<br/>
> 这时候结果集当中有两个事件：制定行程事件和下单事件。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/9.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/10.png)

> 4.超时未接单状态时，如果来了一些其他事件，同样可以被 ignore 边忽略，直到超时事件的触发，将状态往下转移到最终状态，这时候整个模式匹配成功，最终将结果集中的制定行程事件和下单事件输出到下游。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/11.png)

> 上面是一个匹配成功的例子，如果是不成功的例子会怎么样？
假如当状态处于超时未接单状态时，收到了一个接单事件，那么就不符合超时未被接单的触发条件，此时整个模式匹配失败，之前放入结果集中的行程事件和下单事件会被清理。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/12.png)

## 3 FlinkCEP案例

### 3.1 准备工作

#### 3.1.1 FlinkCEP在流处理中的位置

> CEP处于如下位置：
>> 1.目标：从有序的简单事件流中发现一些规则特征<br/>
>> 2.输入：一个或多个由简单事件构成的事件流<br/>
>> 3.处理：识别简单事件之间的内在联系，多个符合一定规则的简单事件构成复杂事件<br/>
>> 4.输出：满足规则的复杂事件

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/13.png)

#### 3.1.2 FlinkCEP编码步骤

https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/libs/cep.html

#### 3.1.3 FlinkCEP代码的完整构成

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/14.png)

> 上图中，蓝色方框代表的是一个个单独的模式；浅黄色的椭圆代表的是这个模式上可以添加的属性，包括模式可以发生的循环次数，或者这个模式是贪婪的还是可选的；橘色的椭圆代表的是模式间的关系，定义了多个模式之间是怎么样串联起来的。

> 通过定义模式，添加相应的属性，将多个模式串联起来三步，就可以构成了一个完整的 Flink CEP 程序。

> 总结

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_FlinkCEP/15.png)

### 3.2 案例1：量词

```
import cn.xiaoma.bean.Message;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternSelectFunction;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.api.windowing.time.Time;

import java.util.Arrays;
import java.util.List;
import java.util.Map;

/**
 * Author xiaoma
 * Date 2020/10/29 15:57
 * Desc
 * 需求
 * 识别恶意用户
 * 用户如果在10s内，输入了TMD 5次，就认为用户为恶意攻击，识别出该用户
 * 使用 Flink CEP量词模式
 * <p>
 * 开发步骤： FlinkCEP = 实时流数据  + 规则(模式) ==>匹配结果输出
 */
public class Demo1_MaliceUser {
    public static void main(String[] args) throws Exception {
        //开发步骤： FlinkCEP = 实时流数据  + 规则(模式) ==>匹配结果输出
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        //2.source
        SingleOutputStreamOperator<Message> source = env.fromCollection(Arrays.asList(
                new Message("1", "TMD", 1558430842000L),//2019-05-21 17:27:22
                new Message("1", "TMD", 1558430843000L),//2019-05-21 17:27:23
                new Message("1", "TMD", 1558430845000L),//2019-05-21 17:27:25
                new Message("1", "TMD", 1558430850000L),//2019-05-21 17:27:30
                new Message("1", "TMD", 1558430851000L),//2019-05-21 17:27:31
                new Message("2", "TMD", 1558430851000L),//2019-05-21 17:27:31
                new Message("3", "TMD", 1558430852000L)//2019-05-21 17:27:32
        )).assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<Message>(Time.seconds(0)) {
            @Override
            public long extractTimestamp(Message element) {
                return element.getEventTime();
            }
        });

        //3.transformation
        //识别恶意用户
        //用户如果在10s内，输入了TMD 5次，就认为用户为恶意攻击，识别出该用户
        //使用 Flink CEP量词模式
        //开发步骤： FlinkCEP = 实时流数据  + 规则(模式) ==>匹配结果输出
        //-1.定义模式规则
        Pattern<Message, Message> pattern = Pattern.<Message>begin("start").where(new SimpleCondition<Message>() {
            @Override
            public boolean filter(Message message) throws Exception {
                if (message.getMsg().equals("TMD")) {
                    return true;
                }
                return false;
            }
        }).times(5)
          .within(Time.seconds(10));

        //-2.将规则应用到数据流等到应用了规则的流patternDS
        PatternStream<Message> patternDS = CEP.pattern(source.keyBy(Message::getId), pattern);

        //-3.获取符合规则的数据
        SingleOutputStreamOperator<List<Message>> resultDS = patternDS.select(new PatternSelectFunction<Message, List<Message>>() {
            @Override
            public List<Message> select(Map<String, List<Message>> map) throws Exception {
                List<Message> resultMessage = map.get("start");//取出满足start规则的数据
                return resultMessage;
            }
        });
        //4.sink
        resultDS.print("被FlinkCEP规则检测到的恶意用户:");
        //5.execute
        env.execute();
    }
}
-------------------------------------------
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * @Date 2020/9/19
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Message {
    private String id;
    private String msg;
    private Long eventTime;
}
```

### 3.3 案例2：条件

```
import cn.xiaoma.bean.LoginEvent;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternSelectFunction;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.Arrays;
import java.util.List;
import java.util.Map;

/**
 * Author xiaoma
 * Date 2020/10/29 15:57
 * Desc
 * 需求
 * 识别出登录失败一定次数的用户
 * 查询匹配用户登陆状态是fail，且失败次数大于8的数据
 * 使用FlinkCEP条件模式
 */
public class Demo2_ConditionDemo {
    public static void main(String[] args) throws Exception {
        //开发步骤： FlinkCEP = 实时流数据  + 规则(模式) ==>匹配结果输出
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        //2.source
        DataStream<LoginEvent> source = env.fromCollection(Arrays.asList(
                new LoginEvent("1", "192.168.0.1", "fail", 8),
                new LoginEvent("1", "192.168.0.2", "fail", 9),
                new LoginEvent("1", "192.168.0.3", "fail", 10),
                new LoginEvent("1", "192.168.0.4", "fail", 10),
                new LoginEvent("2", "192.168.10.10", "success", -1),
                new LoginEvent("3", "192.168.10.10", "fail", 5),
                new LoginEvent("3", "192.168.10.11", "fail", 6),
                new LoginEvent("4", "192.168.10.10", "fail", 6),
                new LoginEvent("4", "192.168.10.11", "fail", 7),
                new LoginEvent("4", "192.168.10.12", "fail", 8),
                new LoginEvent("5", "192.168.10.13", "success", 8),
                new LoginEvent("5", "192.168.10.14", "success", 9),
                new LoginEvent("5", "192.168.10.15", "success", 10),
                new LoginEvent("6", "192.168.10.16", "fail", 6),
                new LoginEvent("6", "192.168.10.17", "fail", 8),
                new LoginEvent("7", "192.168.10.18", "fail", 5),
                new LoginEvent("6", "192.168.10.19", "fail", 10),
                new LoginEvent("6", "192.168.10.18", "fail", 9)
        ));
        //3.transformation
        //识别出登录失败一定次数的用户
        //查询匹配用户登陆状态是fail，且失败次数大于8的数据

        //开发步骤： FlinkCEP = 实时流数据  + 规则(模式) ==>匹配结果输出
        //-1.定义模式规则
        Pattern<LoginEvent, LoginEvent> pattern1 = Pattern.<LoginEvent>begin("start1").where(new SimpleCondition<LoginEvent>() {//简单条件
            @Override
            public boolean filter(LoginEvent loginEvent) throws Exception {
                //System.out.println("进入start1规则判断");
                if (loginEvent.getStatus().equals("fail")) {
                    //System.out.println("状态是fail，且count>8");
                    return true;
                }
                return false;
            }
        }).where(new SimpleCondition<LoginEvent>() {//简单条件
            @Override
            public boolean filter(LoginEvent loginEvent) throws Exception {
                //System.out.println("进入start1规则判断");
                if (loginEvent.getCount() > 8) {
                    //System.out.println("状态是fail，且count>8");
                    return true;
                }
                return false;
            }
        });

        /*Pattern<LoginEvent, LoginEvent> pattern1 = Pattern.<LoginEvent>begin("start1").where(new SimpleCondition<LoginEvent>() {//简单条件
            @Override
            public boolean filter(LoginEvent loginEvent) throws Exception {
                //System.out.println("进入start1规则判断");
                if (loginEvent.getStatus().equals("fail") && loginEvent.getCount() > 8) {
                    //System.out.println("状态是fail，且count>8");
                    return true;
                }
                return false;
            }
        });*/

        /*Pattern<LoginEvent, LoginEvent> pattern2 = Pattern.<LoginEvent>begin("start2").where(new IterativeCondition<LoginEvent>() {//迭代条件
            @Override
            public boolean filter(LoginEvent loginEvent, Context<LoginEvent> context) throws Exception {
                //迭代条件除了有数据流参数之外,还有context
                //context.getEventsForPattern("其他的pattern名称")
                if (loginEvent.getStatus().equals("fail") && loginEvent.getCount() > 8) {
                    return true;
                }
                return false;
            }
        });*/

        //-2.将规则应用到数据流等到应用了规则的流patternDS
        PatternStream<LoginEvent> patternDS = CEP.pattern(source.keyBy(LoginEvent::getId), pattern1);

        //-3.获取符合规则的数据
        SingleOutputStreamOperator<List<LoginEvent>> resultDS = patternDS.select(new PatternSelectFunction<LoginEvent, List<LoginEvent>>() {
            @Override
            public List<LoginEvent> select(Map<String, List<LoginEvent>> map) throws Exception {
                return map.get("start1");
            }
        });
        //4.sink
        resultDS.print();
        //5.execute
        env.execute();
    }
}

----------------------
@Data
@AllArgsConstructor
@NoArgsConstructor
public class LoginEvent {
    private String id;
    private String ip;
    private String status;
    private int count;
}
```

### 3.4 案例3：组合

```
import cn.xiaoma.bean.LoginUser;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternSelectFunction;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.api.windowing.time.Time;

import java.util.Arrays;
import java.util.List;
import java.util.Map;

/**
 * Author xiaoma
 * Date 2020/10/29 15:57
 * Desc
 * 需求：
 * 识别2秒内连续登录失败用户
 * 有一个业务系统，用户要使用该业务系统必须要先登陆
 * 过滤出来在2秒内连续登陆失败的用户
 */
public class Demo3_LoginFailDemo {
    public static void main(String[] args) throws Exception {
        //开发步骤： FlinkCEP = 实时流数据  + 规则(模式) ==>匹配结果输出
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        //2.source
        SingleOutputStreamOperator<LoginUser> source = env.fromCollection(Arrays.asList(
                new LoginUser(1, "192.168.0.1", "fail", 1558430842000L),    //2019-05-21 17:27:22
                new LoginUser(1, "192.168.0.2", "fail", 1558430843000L),    //2019-05-21 17:27:23
                new LoginUser(1, "192.168.0.3", "fail", 1558430843000L),    //2019-05-21 17:27:23
                new LoginUser(2, "192.168.10.10", "success", 1558430845000L)//2019-05-21 17:27:25
        )).assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<LoginUser>(Time.seconds(0)) {
            @Override
            public long extractTimestamp(LoginUser element) {
                return element.getEventTime();
            }
        });

        //3.transformation
        //识别2秒内连续登录失败用户
        //开发步骤： FlinkCEP = 实时流数据  + 规则(模式) ==>匹配结果输出
        //-1.定义模式规则
        Pattern<LoginUser, LoginUser> pattern = Pattern.<LoginUser>begin("start")
                .where(new SimpleCondition<LoginUser>() {
                    @Override
                    public boolean filter(LoginUser loginUser) throws Exception {
                        return loginUser.getStatus().equals("fail");
                    }
                }).next("next")
                .where(new SimpleCondition<LoginUser>() {
                    @Override
                    public boolean filter(LoginUser loginUser) throws Exception {
                        return loginUser.getStatus().equals("fail");
                    }
                }).within(Time.seconds(2));

        //-2.将规则应用到数据流等到应用了规则的流patternDS
        PatternStream<LoginUser> patternDS = CEP.pattern(source.keyBy(LoginUser::getUserId), pattern);

        //-3.获取符合规则的数据
        SingleOutputStreamOperator<Object> resultDS = patternDS.select(new PatternSelectFunction<LoginUser, Object>() {
            @Override
            public Object select(Map<String, List<LoginUser>> map) throws Exception {
                return map.get("next");
            }
        });
        //4.sink
        resultDS.print();
        //5.execute
        env.execute();
    }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
public class LoginUser {
    //new LoginUser (1, "192.168.0.1", "fail", 1558430842000L),//2019-05-21 17:27:22
    private int userId;
    private String ip;
    private String status;
    private Long eventTime;
}
```

### 3.5 案例4：连续和允许组合

```
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternSelectFunction;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.List;
import java.util.Map;

/**
 * Author xiaoma
 * Date 2020/10/29 15:57
 * Desc
 * 需求：
 * 从数据源中依次提取"c","a","b"元素
 */
public class Demo4_ConsecutiveDemo {
    public static void main(String[] args) throws Exception {
        //开发步骤： FlinkCEP = 实时流数据  + 规则(模式) ==>匹配结果输出
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        //env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        //2.source
        DataStreamSource<String> source = env.fromElements("c", "d", "a", "a", "a", "d", "a", "b");

        //3.transformation
        //从数据源中依次提取"c","a","b"元素
        //开发步骤： FlinkCEP = 实时流数据  + 规则(模式) ==>匹配结果输出
        Pattern<String, String> pattern = Pattern.<String>begin("begin")
                .where(new SimpleCondition<String>() {
                    @Override
                    public boolean filter(String value) throws Exception {
                        return value.equals("c");
                    }
                })
                .followedBy("middle")
                .where(new SimpleCondition<String>() {
                    @Override
                    public boolean filter(String value) throws Exception {
                        return value.equals("a");
                    }
                })
                .oneOrMore()
                //.consecutive()//连续匹配a
                /*
                1)使用consecutive()
                ([c],[a, a, a],[b])
                ([c],[a, a],[b])
                ([c],[a],[b])
                2)不使用consecutive()
                ([c],[a, a, a, a],[b])
                ([c],[a, a, a],[b])
                ([c],[a, a],[b])
                ([c],[a],[b])
                */
                .allowCombinations() //允许组合--了解
                /*
                1)使用allowCombinations()
                ([c],[a, a, a, a],[b])
                ([c],[a, a, a],[b])
                ([c],[a, a, a],[b])
                ([c],[a, a],[b])
                ([c],[a, a, a],[b])
                ([c],[a, a],[b])
                ([c],[a, a],[b])
                ([c],[a],[b])
                */
                .followedBy("end")
                .where(new SimpleCondition<String>() {
                    @Override
                    public boolean filter(String value) throws Exception {
                        return value.equals("b");
                    }
                });

        //-2.将规则应用到数据流
        PatternStream<String> cep = CEP.pattern(source, pattern);
        //-3.获取符合规则的数据Tuple3.of(begin, middle, end)
        SingleOutputStreamOperator<Object> resultDS = cep.select(new PatternSelectFunction<String, Object>() {
            @Override
            public Object select(Map<String, List<String>> pattern) throws Exception {
                //取出每一个模式下的匹配数据
                List<String> begin = pattern.get("begin");
                List<String> middle = pattern.get("middle");
                List<String> end = pattern.get("end");
                return Tuple3.of(begin, middle, end);
            }

        });
        //4.sink
        resultDS.print();
        //5.execute
        env.execute();
    }
}
```

### 3.6 案例5：恶意搜索用户胡别

```
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternSelectFunction;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.List;
import java.util.Map;

/**
 * Author xiaoma
 * Date 2020/10/14 22:39
 * Desc
 * 模拟电商网站用户搜索的数据来作为数据的输入源，
 * 然后查找其中重复搜索某一个商品的人2次，并且发送一条告警消息。
 */
public class Demo5_Search {
    public static void main(String[] args) throws Exception{
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        //2.source
        DataStreamSource source = env.fromElements(
                Tuple3.of("Marry", "外套", 1L),
                Tuple3.of("Marry", "帽子",1L),
                Tuple3.of("Marry", "帽子",2L),
                Tuple3.of("Marry", "帽子",3L),
                Tuple3.of("Ming", "衣服",1L),
                Tuple3.of("Marry", "鞋子",1L),
                Tuple3.of("Marry", "鞋子",2L),
                Tuple3.of("LiLei", "帽子",1L),
                Tuple3.of("LiLei", "帽子",2L),
                Tuple3.of("LiLei", "帽子",3L)
        );

        //3.transformation

        //-1.定义模式规则,寻找连续搜索帽子的用户
        Pattern<Tuple3<String, String, Long>, Tuple3<String, String, Long>> pattern = Pattern.<Tuple3<String, String, Long>>begin("start")
                .where(new SimpleCondition<Tuple3<String, String, Long>>() {
                    @Override
                    public boolean filter(Tuple3<String, String, Long> value) throws Exception {
                        return value.f1.equals("帽子");
                    }
                })
                //.timesOrMore(3);
                .next("next")
                .where(new SimpleCondition<Tuple3<String, String, Long>>() {
                    @Override
                    public boolean filter(Tuple3<String, String, Long> value) throws Exception {
                        return value.f1.equals("帽子");
                    }
                });

        //-2.将规则应用到数据流
        PatternStream patternStream = CEP.pattern(source.keyBy(0), pattern);
        //-3.获取符合规则的数据
        SingleOutputStreamOperator matchStream = patternStream.select(new PatternSelectFunction<Tuple3<String, String, Long>,Object>() {
            @Override
            public Object select(Map<String, List<Tuple3<String, String, Long>>> pattern) throws Exception {
                List<Tuple3<String, String, Long>> middle = pattern.get("next");
                return middle.get(0).f0 + ":" + middle.get(0).f2 + ":" + "连续搜索两次帽子!";
            }
        });
        //4.sink
        matchStream.printToErr();
        //5.execute
        env.execute();
    }
}
```

### 3.7 案例6：高频交易风险用户识别

```
import cn.xiaoma.bean.TransactionEvent;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternSelectFunction;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.api.windowing.time.Time;

import java.util.List;
import java.util.Map;

/**
 * 高频交易，找出活跃账户/交易活跃用户
 * 在这个场景中，我们模拟账户交易信息中，那些高频的转账支付信息，希望能发现其中的风险或者活跃的用户：
 * 需要找出那些 24 小时内至少 5 次有效交易的账户
 */
public class Demo6_HighFrequencyTrading {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        //2.source
        //[TransactionEvent(accout=100XX, amount=100.0, timeStamp=1597905235000),
        // TransactionEvent(accout=100XX, amount=200.0, timeStamp=1597905236000),
        // TransactionEvent(accout=100XX, amount=300.0, timeStamp=1597905237000),
        // TransactionEvent(accout=100XX, amount=400.0, timeStamp=1597905238000),
        // TransactionEvent(accout=100XX, amount=500.0, timeStamp=1597905239000)]
        DataStream<TransactionEvent> source = env.fromElements(
                new TransactionEvent("100XX", 0.0D, 1597905234000L),//2020-08-20 14:33:54
                new TransactionEvent("100XX", 100.0D, 1597905235000L),//2020-08-20 14:33:55
                new TransactionEvent("100XX", 200.0D, 1597905236000L),//2020-08-20 14:33:56
                new TransactionEvent("100XX", 300.0D, 1597905237000L),//2020-08-20 14:33:57
                new TransactionEvent("100XX", 400.0D, 1597905238000L),//2020-08-20 14:33:58
                new TransactionEvent("100XX", 500.0D, 1597905239000L),//2020-08-20 14:33:59
                new TransactionEvent("101XX", 0.0D, 1597905240000L),//2020-08-20 14:34:00
                new TransactionEvent("101XX", 100.0D, 1597905241000L)//2020-08-20 14:34:01
        ).assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<TransactionEvent>(Time.seconds(0)) {
            @Override
            public long extractTimestamp(TransactionEvent element) {
                return element.getTimeStamp();
            }
        });
        //3.transformation
        //需要找出那些 24 小时内至少 5 次有效交易的账户
        //-1.定义模式规则
        Pattern<TransactionEvent, TransactionEvent> pattern = Pattern.<TransactionEvent>begin("start").where(
                new SimpleCondition<TransactionEvent>() {
                    @Override
                    public boolean filter(TransactionEvent transactionEvent) {
                        return transactionEvent.getAmount() > 0;
                    }
                }
        ).timesOrMore(5)
         .within(Time.hours(24));
        //-2.将规则应用到数据流
        PatternStream<TransactionEvent> patternStream = CEP.pattern(source.keyBy(TransactionEvent::getAccout), pattern);
        //-3.获取符合规则的数据
        SingleOutputStreamOperator<Object> result = patternStream.select(new PatternSelectFunction<TransactionEvent, Object>() {
            @Override
            public Object select(Map<String, List<TransactionEvent>> match) throws Exception {
                List<TransactionEvent> start = match.get("start");
                return start;
            }
        });

        //4.sink
        result.print();

        //5.execute
        env.execute("execute cep");
    }
}

--------------------

@Data
@AllArgsConstructor
@NoArgsConstructor
public class TransactionEvent {
    private String accout;
    private Double amount;
    private Long timeStamp;
}

```

### 3.8 案例7：订单超时监控

```
import cn.xiaoma.bean.OrderEvent;
import org.apache.flink.api.common.typeinfo.TypeInformation;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternSelectFunction;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.PatternTimeoutFunction;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.OutputTag;

import java.util.Arrays;
import java.util.List;
import java.util.Map;

/**
 * Date 2020/9/21
 * 用户下单以后，应该设置订单失效时间，用来提高用户的支付意愿
 * 如果用户下单15分钟未支付，则输出监控信息
 */
public class Demo7_OrderTimeoutDemo {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        //2.source
        SingleOutputStreamOperator<OrderEvent> source = env.fromCollection(Arrays.asList(
                new OrderEvent("123", 1,"create", 1558430842000L),//2019-05-21 17:27:22
                new OrderEvent("456", 2,"create", 1558430843000L),//2019-05-21 17:27:23
                new OrderEvent("456", 2,"other", 1558430845000L), //2019-05-21 17:27:25
                new OrderEvent("456", 2,"pay", 1558430850000L)   //2019-05-21 17:27:30
        )).assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<OrderEvent>(Time.seconds(0)) {
            @Override
            public long extractTimestamp(OrderEvent element) {
                return element.getEventTime();
            }
        });
        //3.transformation
        //-1.定义模式规则
        //如果用户下单15分钟未支付，则输出监控信息
        Pattern<OrderEvent, OrderEvent> pattern = Pattern.<OrderEvent>begin("begin")
                .where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        //定义业务规则
                        return value.getStatus().equals("create");
                    }
                }).followedBy("end")
                .where(new SimpleCondition<OrderEvent>() {
                    @Override
                    public boolean filter(OrderEvent value) throws Exception {
                        return value.getStatus().equals("pay");
                    }
                }).within(Time.minutes(15));

        //-2.将规则应用到数据流
        PatternStream<OrderEvent> cep = CEP.pattern(source.keyBy(OrderEvent::getOrderId), pattern);
        //-3.设置侧输出流,用来封装超时数据
        OutputTag<OrderEvent> timeoutTag = new OutputTag<>("opt", TypeInformation.of(OrderEvent.class));

        //-4.获取超时数据和符合规则的数据
        SingleOutputStreamOperator<Object> result = cep.select(timeoutTag,//接收超时数据
                new PatternTimeoutFunction<OrderEvent, OrderEvent>() {//处理超时数据
                    @Override
                    //Map<规则名称, 符合规则的数据> map
                    public OrderEvent timeout(Map<String, List<OrderEvent>> map, long l) throws Exception {
                        return map.get("begin").get(0);
                    }
                }, new PatternSelectFunction<OrderEvent, Object>() {//处理正常数据
                    @Override
                    public Object select(Map<String, List<OrderEvent>> map) throws Exception {
                        List<OrderEvent> begin = map.get("begin");
                        List<OrderEvent> end = map.get("end");
                        begin.addAll(end);//把end集合中的所有元素添加到begin集合中,最后返回begin集合即可
                        return begin;
                    }
                });
        //4.sink
        result.print("正常数据:");
        DataStream<OrderEvent> timeoutDS = result.getSideOutput(timeoutTag);
        timeoutDS.print("超时数据");

        //5.execute
        env.execute();
    }
}

-------------------------------------

@Data
@NoArgsConstructor
@AllArgsConstructor
public class OrderEvent {
    private String orderId;
    private int userId;
    private String status;
    private Long eventTime;
}
```

### 3.9 案例8：监控市场价格

```
import cn.xiaoma.bean.Product;
import cn.xiaoma.util.RedisUtil;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternSelectFunction;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.api.windowing.time.Time;
import redis.clients.jedis.JedisCluster;

import java.util.Arrays;
import java.util.List;
import java.util.Map;

/**
 * Date 2020/9/19
 * 监控在1分钟之内有连续两次超过预定商品价格阀值的商品
 * 就是在Redis中有一个商品指导价(阈值),
 * 然后实时监控市场上的商品价格,出现1分钟2次超过指导价则触发告警!
 */
public class Demo8_CepMarkets {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        //2.source
        //注意:初始化所有state为false,表示默认都认为没有超过商品价格指导价/阈值
        SingleOutputStreamOperator<Product> source = env.fromCollection(Arrays.asList(
                new Product(100001L, 6.0, "apple", "苹果", 1558430843000L, false),
                new Product(100007L, 0.5, "mask", "口罩", 1558430844000L, false),
                new Product(100002L, 2.0, "rice", "大米", 1558430845000L, false),
                new Product(100003L, 2.0, "flour", "面粉", 1558430846000L, false),
                new Product(100004L, 12.0, "rice", "大米", 1558430847000L, false),
                new Product(100005L, 20.0, "apple", "苹果", 1558430848000L, false),
                new Product(100006L, 3.0, "banana", "香蕉", 1558430849000L, false),
                new Product(100007L, 10.0, "mask", "口罩", 1558430850000L, false),
                new Product(100001L, 16.0, "apple", "苹果", 1558430852000L, false),
                new Product(100007L, 15.0, "mask", "口罩", 1558430853000L, false),
                new Product(100002L, 12.0, "rice", "大米", 1558430854000L, false),
                new Product(100003L, 12.0, "flour", "面粉", 1558430855000L, false),
                new Product(100004L, 12.0, "rice", "大米", 1558430856000L, false),
                new Product(100005L, 20.0, "apple", "苹果", 1558430857000L, false),
                new Product(100006L, 13.0, "banana", "香蕉", 1558430858000L, false),
                new Product(100007L, 10.0, "mask", "口罩", 1558430859000L, false))
        ).assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<Product>(Time.seconds(0)) {
            @Override
            public long extractTimestamp(Product element) {
                return element.getOrderTime();
            }
        });
        //3.transformation
        //-0.根据redis中的阈值判商品是否超过了指导价/阈值,并修改state状态
        SingleOutputStreamOperator<Product> productDS = source.map(new RichMapFunction<Product, Product>() {
            JedisCluster jedis =null;

            @Override
            public void open(Configuration parameters) throws Exception {
                jedis = RedisUtil.getJedisCluster();
            }

            @Override
            public Product map(Product product) throws Exception {
                //获取价格阈值
                String threshold = jedis.hget("product", product.getGoodsName());
                if (product.getGoodsPrice() > Double.valueOf(threshold)) {
                    product.setStatus(true);//如果商品价格>指导价(阈值),则修改商品状态为true
                }
                return product;
            }
        });

        //-1.定义模式规则
        //实时监控市场上的商品价格,出现1分钟2次超过指导价则触发告警!
        Pattern<Product, Product> pattern = Pattern.<Product>begin("begin")
                .where(new SimpleCondition<Product>() {
                    @Override
                    public boolean filter(Product value) throws Exception {
                        return value.getStatus() == true;
                    }
                })
                //.next("next")
                .followedBy("next")
                .where(new SimpleCondition<Product>() {
                    @Override
                    public boolean filter(Product value) throws Exception {
                        return value.getStatus() == true;
                    }
                }).within(Time.minutes(1));

        //-2.将规则应用到数据流
        PatternStream<Product> cep = CEP.pattern(productDS.keyBy(Product::getGoodsId), pattern);
        //-3.获取符合规则的数据
        SingleOutputStreamOperator<List<Product>> result = cep.select(new PatternSelectFunction<Product, List<Product>>() {
            @Override
            public List<Product> select(Map<String, List<Product>> pattern) throws Exception {
                List<Product> next = pattern.get("next");
                return next;
            }
        });
        //4.sink
        result.print();
        //5.execute
        env.execute();
    }
}

-----------------------------------

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Product {
    private Long goodsId; //商品ID
    private Double goodsPrice; //商品价格
    private String goodsName; //商品名称
    private String alias; //中文名称
    private Long orderTime;//事件时间
    private Boolean status;//价格阀值状态，ture：超过阀值，false:未超过阀值
}
```

### 3.10 案例9：运维监控规则引擎

```
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternFlatSelectFunction;
import org.apache.flink.cep.PatternSelectFunction;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.RichParallelSourceFunction;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

import java.util.List;
import java.util.Map;
import java.util.concurrent.ThreadLocalRandom;

/**
 * Date 2020/9/21
 * 需求：机架温控预警
 * 预警规则1：警告：某机架在10秒内连续两次上报的温度超过阈值
 * 预警规则2：报警：某机架在20秒内连续两次匹配警告，并且第二次的警告温度超过了第一次的警告温度就报警
 */
public class Demo9_RackWarn {
    public static void main(String[] args) throws Exception {
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        //2.source
        SingleOutputStreamOperator<TemperatureMonitoringEvent> source = env.addSource(new MonitoringEventSource())
                .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<TemperatureMonitoringEvent>(Time.seconds(0)) {
                    @Override
                    public long extractTimestamp(TemperatureMonitoringEvent element) {
                        return element.getTimestamp();
                    }
                });

        //3.transformation
        //预警规则1：警告：某机架在10秒内连续两次上报的温度超过阈值
        //-1.定义模式规则
        Pattern<TemperatureMonitoringEvent, TemperatureEventTemperature> patternWarn = Pattern
                .<TemperatureMonitoringEvent>begin("begin").subtype(TemperatureEventTemperature.class)
                .where(new SimpleCondition<TemperatureEventTemperature>() {
                    @Override
                    public boolean filter(TemperatureEventTemperature value) throws Exception {
                        return value.getTemperature() > 100;
                    }
                }).next("next").subtype(TemperatureEventTemperature.class)
                .where(new SimpleCondition<TemperatureEventTemperature>() {
                    @Override
                    public boolean filter(TemperatureEventTemperature value) throws Exception {
                        return value.getTemperature() > 100;
                    }
                }).within(Time.seconds(10));

        //-2.将规则应用到数据流
        PatternStream<TemperatureMonitoringEvent> cepWarn = CEP.pattern(source.keyBy(TemperatureMonitoringEvent::getRackID), patternWarn);
        //-3.获取符合规则的数据返回TemperatureWarning警告对象
        SingleOutputStreamOperator<TemperatureWarning> warnData = cepWarn.select(new PatternSelectFunction<TemperatureMonitoringEvent, TemperatureWarning>() {
            @Override
            public TemperatureWarning select(Map<String, List<TemperatureMonitoringEvent>> pattern) throws Exception {
                TemperatureEventTemperature begin = (TemperatureEventTemperature) pattern.get("begin").get(0);
                TemperatureEventTemperature end = (TemperatureEventTemperature) pattern.get("next").get(0);
                //返回机架id和触发警告的平均温度
                return new TemperatureWarning(end.getRackID(), (begin.getTemperature() + end.getTemperature()) / 2);
            }
        });
        //4.sink
        warnData.print("规则1警告数据：");

        //预警规则2：报警：某机架在20秒内连续两次匹配警告，并且第二次的温度超过了第一次的温度就告警
        //-1.定义模式规则
        Pattern<TemperatureWarning, TemperatureWarning> alertPattern = Pattern
                //直接使用上面定义过的规则/模式
                .<TemperatureWarning>begin("begin").next("next").within(Time.seconds(20));
        //-2.将规则应用到数据流
        PatternStream<TemperatureWarning> cepAlert = CEP.pattern(warnData.keyBy(TemperatureWarning::getRackID), alertPattern);
        //-3.获取符合规则的数据返回TemperatureAlert报警对象
        SingleOutputStreamOperator<TemperatureAlert> result = cepAlert.flatSelect(new PatternFlatSelectFunction<TemperatureWarning, TemperatureAlert>() {
            @Override
            public void flatSelect(Map<String, List<TemperatureWarning>> pattern, Collector<TemperatureAlert> out) throws Exception {
                TemperatureWarning begin = pattern.get("begin").get(0);
                TemperatureWarning end = pattern.get("next").get(0);
                if (end.getAverageTemperature() > begin.getAverageTemperature()) {
                    out.collect(new TemperatureAlert(begin.getRackID()));
                }
            }
        });

        //4.sink
        result.print("规则2报警数据==>：");

        //5.execute
        env.execute();
    }

    /**
     * 自定义source模拟生成机架id、温度、电压等数据。
     */
    private static class MonitoringEventSource extends RichParallelSourceFunction<TemperatureMonitoringEvent> {
        private boolean flag = true;
        private final double temperatureRatio = 0.5;//温度阈值
        private final double powerStd = 100;//标准功率
        private final double powerMean = 10; //平均功率
        private final double temperatureStd = 80;//标准温度
        private final double temperatureMean = 20;//平均温度

        public void run(SourceContext<TemperatureMonitoringEvent> sourceContext) throws Exception {
            while (flag) {
                TemperatureMonitoringEvent temperatureMonitoringEvent;
                //生成随机数的对象
                final ThreadLocalRandom random = ThreadLocalRandom.current();
                int rackId = random.nextInt(2);
                //如果生成的随机温度大于温度阈值，那么就是过热
                if (random.nextDouble() >= temperatureRatio) {
                    //用Random类中的nextGaussian()方法，可以产生服从高斯分布的随机数，高斯分布即标准正态分布，均值为0，方差为1。
                    double power = random.nextGaussian() * powerStd + powerMean;
                    temperatureMonitoringEvent = new TemperaturePowerEventTemperature(rackId, power, System.currentTimeMillis());
                } else {
                    double temperature = random.nextGaussian() * temperatureStd + temperatureMean;
                    temperatureMonitoringEvent = new TemperatureEventTemperature(rackId, temperature, System.currentTimeMillis());
                }
                //System.out.println("随机生成的数据:"+ temperatureMonitoringEvent);
                sourceContext.collect(temperatureMonitoringEvent);
                //Thread.sleep(1000);
            }
        }
        public void cancel() {
            flag = false;
        }
    }
}
```

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
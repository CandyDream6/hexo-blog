---
title: Redis 内存数据结构存储
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(50).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(50).jpg
tags:
  - 缓存数据库
  - Redis
category:
  - - 编程
    - 大数据
 
date: 2019-12-15 11:27:14
---

Redis是一种`开放源代码`（BSD许可）的内存中数据结构存储，用作`数据库`，`缓存`和`消息代理`。

Redis提供数据结构，例如`字符串`，`哈希`，`列表`，`集合`，带范围查询的排序集合，`位图`，`超日志`，`地理空间索引`和`流`。

Redis具有内置的`复制`，`Lua脚本`，`LRU驱逐`，`事务`和`不同级别的磁盘持久性`，并通过`Redis Sentinel`和`Redis Cluster`自动分区提供了`高可用性`。

<!-- more -->

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/touBuYinDaoGuanZhu.gif)
## 1 安装

### 1.1 下载redis安装包

```
cd /export/software
wget http://download.redis.io/releases/redis-3.2.8.tar.gz
```

### 1.2 解压redis压缩包到指定目录

```
cd /export/software
#上传redis-3.2.8.tar.gz到linux此目录下
mkdir -p /export/server/
tar -zxvf redis-3.2.8.tar.gz -C ../server/
```

### 1.3 安装C程序运行环境

```
yum -y install gcc-c++
```

### 1.4 安装较新版本的tcl

#### 1.4.1 使用压缩包进行安装

```
cd /export/software
wget http://downloads.sourceforge.net/tcl/tcl8.6.1-src.tar.gz
解压tcl
tar -zxvf tcl8.6.1-src.tar.gz -C ../server/
进入指定目录
cd ../server/tcl8.6.1/unix/
./configure
make  && make  install
```

#### 1.4.2 在线安装tcl（推荐）

```
yum  -y  install  tcl
```

### 1.5 编译redis

```
cd /export/server/redis-3.2.8/
#或者使用命令  make  进行编译
make MALLOC=libc  
make test && make install PREFIX=/export/server/redis-3.2.8
```

#### 1.5.1 修改redis配置文件

```
cd /export/server/redis-3.2.8/
mkdir -p /export/server/redis-3.2.8/log
mkdir -p /export/server/redis-3.2.8/data

vim redis.conf
# 修改第61行，接收的访问地址
bind node1.itcast.cn
# 修改第128行，后台守护执行
daemonize yes
# 修改第163行，日志目录
logfile "/export/server/redis-3.2.8/log/redis.log"
# 修改第247行，数据持久化目录
dir /export/server/redis-3.2.8/data
```

#### 1.5.2 启动redis

```
cd  /export/server/redis-3.2.8/
bin/redis-server  redis.conf
```

#### 1.5.3 关闭redis

```
bin/redis-cli -h node1 shutdown
```

#### 1.5.4 连接redis客户端

```
cd /export/server/redis-3.2.8/
bin/redis-cli -h node1.itcast.cn
```
### 1.6 GUI工具

> [Another Redis DeskTop Manager](https://github.com/qishibo/AnotherRedisDesktopManager) 

```
https://github.com/qishibo/AnotherRedisDesktopManager
```

## 2 数据类型

> [命令查看手册](http://doc.redisfans.com/)

```
http://doc.redisfans.com/
```

### 2.1 String

> `一个键最大能存储512M`

序号 | 命令及描述 | 示例
---|---|---
1 | `SET key value`<br/>设置指定 key 的值 | SET hello world
2 | `GET key`<br/>获取指定 key 的值 | GET hello
3 | `GETSET key value`<br/>将给定 key 的值设为 value ，并返回 key 的旧值(old value) | GETSET hello world2
4 | `MGET key1 [key2..]`<br/>获取所有(一个或多个)给定 key 的值 | MGET hello world
5 | `SETEX key seconds value`<br/>将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位) | SETEX hello 10 world3
6 | `SETNX key value`<br/>只有在 key 不存在时设置 key 的值 | SETNX xiaoma redisvalue
7 | `STRLEN key`<br/>返回 key 所储存的字符串值的长度 | STRLEN xiaoma
8 | `MSET key value [key value ...]`<br/>同时设置一个或多个 key-value 对 | MSET xiaoma2 xiaomavalue2 xiaoma3 xiaomavalue3
9 | `MSETNX key value [key value ...]`<br/>同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在 | MSETNX xiaoma4 xiaomavalue4 xiaoma5 xiaomavalue5
10 | `PSETEX key milliseconds value`<br/>这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位 | PSETEX xiaoma6 6000 xiaoma6value
11 | `INCR key`<br/>将 key 中储存的数字值增一 | set xiaoma7 1<br/>INCR xiaoma7<br/>GET xiaoma7
12 | `INCRBY key increment`<br/>将 key 所储存的值加上给定的增量值（increment） | INCRBY xiaoma7 2<br/>get xiaoma7
13 | `INCRBYFLOAT key increment`<br/>将 key 所储存的值加上给定的浮点增量值（increment） | INCRBYFLOAT xiaoma7 0.8
14 | `DECR key`<br/>将 key 中储存的数字值减一 | set xiaoma8 1<br/>DECR xiaoma8<br/>GET xiaoma8
15 | `DECRBY key decrement`<br/>key 所储存的值减去给定的减量值（decrement） | DECRBY xiaoma8 3
16 | `APPEND key value`<br/>如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾 | APPEND xiaoma8 hello

### 2.2 Hash

> Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。

> Redis 中每个 hash 可以存储 2<sup>32</sup>- 1 键值对（40多亿）

序号 | 命令及描述 | 示例
---|---|---
1 | `HSET key field value`<br/>将哈希表 key 中的字段 field 的值设为 value | HSET key1 field1 value1
2 | `HSETNX key field value`<br/>只有在字段 field 不存在时，设置哈希表字段的值 | HSETNX key1 field2 value2
3 | `HMSET key field1 value1 [field2 value2 ]`<br/>同时将多个 field-value (域-值)对设置到哈希表 key 中 | HMSET key1 field3 value3 field4 value4
4 | `HEXISTS key field`<br/>查看哈希表 key 中，指定的字段是否存在 | HEXISTS key1 field4<br/>HEXISTS key1 field6
5 | `HGET key field`<br/>获取存储在哈希表中指定字段的值 | HGET key1 field4
6 | `HGETALL key`<br/>获取在哈希表中指定 key 的所有字段和值 | HGETALL key1
7 | `HKEYS key`<br/>获取所有哈希表中的字段 | HKEYS key1
8 | `HLEN key`<br/>获取哈希表中字段的数量 | HLEN key1
9 | `HMGET key field1 [field2]`<br/>获取所有给定字段的值 | HMGET key1 field3 field4
10 | `HINCRBY key field increment`<br/>为哈希表 key 中的指定字段的整数值加上增量 increment | HSET key2 field1 1<br/>HINCRBY key2 field1 1<br/>HGET key2 field1
11 | `HINCRBYFLOAT key field increment`<br/>为哈希表 key 中的指定字段的浮点数值加上增量 increment | HINCRBYFLOAT key2 field1 0.8
12 | `HVALS key`<br/>获取哈希表中所有值 | HVALS key1
13 | `HDEL key field1 [field2]`<br/>删除一个或多个哈希表字段 | HDEL key1 field3<br/>HVALS key1

### 2.3 List

> list列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

> 一个列表最多可以包含 2<sup>32</sup>- 1 个元素 (4294967295, 每个列表超过40亿个元素)。

序号 | 命令及描述 | 示例
---|---|---
1 | `LPUSH key value1 [value2]`<br/>将一个或多个值插入到列表头部 | LPUSH list1 value1 value2
2 | `LRANGE key start stop`<br/>查看list当中所有的数据 | LRANGE list1 0 -1
3 | `LPUSHX key value`<br/>将一个值插入到已存在的列表头部 | LPUSHX list1 value3
4 | `RPUSH key value1 [value2]`<br/>在列表中添加一个或多个值到尾部 | RPUSH list1 value4 value5<br/>LRANGE list1 0 -1
5 | `RPUSHX key value`<br/>为已存在的列表添加单个值到尾部 | RPUSHX list1 value6
6 | `LINSERT key BEFORE 或 AFTER pivot value`<br/>在列表的元素前或者后插入元素 | LINSERT list1 BEFORE value3 beforevalue3
7 | `LINDEX key index`<br/>通过索引获取列表中的元素 | LINDEX list1 0
8 | `LSET key index value`<br/>通过索引设置列表元素的值 | LSET list1 0 hello
9 | `LLEN key`<br/>获取列表长度 | LLEN list1
10 | `LPOP key`<br/>移出并获取列表的第一个元素 | LPOP list1
11 | `RPOP key`<br/>移除列表的最后一个元素，返回值为移除的元素 | RPOP list1
12 | `BLPOP key1 [key2 ] timeout`<br/>移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止 | BLPOP list1 2000
13 | `BRPOP key1 [key2 ] timeout`<br/>移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止 | BRPOP list1 2000
14 | `RPOPLPUSH source destination`<br/>移除列表的最后一个元素，并将该元素添加到另一个列表并返回 | RPOPLPUSH list1 list2
15 | `BRPOPLPUSH source destination timeout`<br/>从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止 | BRPOPLPUSH list1 list2 2000
16 | `LTRIM key start stop`<br/>对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除 | LTRIM list1 0 2
17 | `DEL key1 key2`<br/>删除指定key的列表 | DEL list2

### 2.4 Set

> Redis 的 Set 是 `String 类型的无序集合`。集合`成员是唯一的`，这就意味着集合中不能出现重复的数据

> Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)[最低时空复杂度，耗时与输入数据大小无关]。 

> 集合中最大的成员数为 2<sup>32</sup>- 1 (4294967295, 每个集合可存储40多亿个成员)。

序号 | 命令及描述 | 示例
---|---|---
1 | `SADD key member1 [member2]`<br/>向集合添加一个或多个成员 | SADD set1 setvalue1 setvalue2
2 | `SMEMBERS key`<br/>返回集合中的所有成员 | SMEMBERS set1
3 | `SCARD key`<br/>获取集合的成员数量 | SCARD set1
4 | `SDIFF key1 [key2]`<br/>返回给定所有集合的差集 | SADD set2 setvalue2 setvalue3<br/>SDIFF set1 set2
5 | `SDIFFSTORE destination key1 [key2]`<br/>返回给定所有集合的差集并存储在 destination 中 | SDIFFSTORE set3 set1 set2
6 | `SINTER key1 [key2]`<br/>返回给定所有集合的交集 | SINTER set1 set2
7 | `SINTERSTORE destination key1 [key2]`<br/>返回给定所有集合的交集并存储在 destination 中 | SINTERSTORE set4 set1 set2
8 | `SISMEMBER key member`<br/>判断 member 元素是否是集合 key 的成员 | SISMEMBER set1 setvalue1
9 | `SMOVE source destination member`<br/>将 member 元素从 source 集合移动到 destination 集合 | SMOVE set1 set2 setvalue1
10 | `SPOP key`<br/>移除并返回集合中的一个随机元素 | SPOP set2
11 | `SRANDMEMBER key [count]`<br/>返回集合中一个或多个随机数 | SRANDMEMBER set2 2
12 | `SREM key member1 [member2]`<br/>移除集合中一个或多个成员 | SREM set2 setvalue1
13 | `SUNION key1 [key2]`<br/>返回所有给定集合的并集 | SUNION set1 set2
14 | `SUNIONSTORE destination key1 [key2]`<br/>所有给定集合的并集存储在 destination 集合中 | SUNIONSTORE set5 set1 set2

### 2.5 ZSet

> Redis`有序集合`和集合一样也是string类型元素的集合,且`不允许重复`的成员

> 它用来`保存需要排序的数据`，例如排行榜，一个班的语文成绩，一个公司的员工工资，一个论坛的帖子等。

> 有序集合中，每个元素都带有score（权重），以此来对元素进行排序

> 它有三个元素：key、member和score。以语文成绩为例，key是考试名称（期中考试、期末考试等），member是学生名字，score是成绩。

序号 | 命令及描述 | 示例
---|---|---
1 | `ZADD key score1 member1 [score2 member2]`<br/>向有序集合添加一个或多个成员，或者更新已存在成员的分数 | 向ZSet中添加页面的PV值<br/>`ZADD pv_zset 120 page1.html 100 page2.html 140 page3.html`
2 | `ZCARD key`<br/>获取有序集合的成员数 | 获取所有的统计PV页面数量<br/>`ZCARD pv_zset`
3 | `ZCOUNT key min max`<br/>计算在有序集合中指定区间分数的成员数 | 获取PV在120-140在之间的页面数量<br/>`ZCOUNT pv_zset 120 140`
4 | `ZINCRBY key increment member`<br/>有序集合中对指定成员的分数加上增量 increment | 给page1.html的PV值+1<br/>`ZINCRBY pv_zset 1 page1.html`
5 | `ZINTERSTORE destination numkeys key [key ...]`<br/>计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中 | 创建两个保存PV的ZSET：<br/>`ZADD pv_zset1 10 page1.html 20 page2.html`<br/>`ZADD pv_zset2 5 page1.html 10 page2.html `<br/>`ZINTERSTORE pv_zset_result 2 pv_zset1 pv_zset2`
6 | `ZRANGE key start stop [WITHSCORES]`<br/>通过索引区间返回有序集合指定区间内的成员 | 获取所有的元素，并可以返回每个key对一个的score<br/>`ZRANGE pv_zset_result 0 -1 WITHSCORES`
7 | `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT]`<br/>通过分数返回有序集合指定区间内的成员 | 获取ZSET中120-140之间的所有元素<br/>`ZRANGEBYSCORE pv_zset 120 140`
8 | `ZRANK key member`<br/>返回有序集合中指定成员的索引 | 获取page1.html的pv排名（升序）<br/>`ZRANK pv_zset page3.html`
9 | `ZREM key member [member ...]`<br/>移除有序集合中的一个或多个成员 | 移除page1.html<br/>`ZREM pv_zset page1.html`
10 | `ZREVRANGE key start stop [WITHSCORES]`<br/>返回有序集中指定区间内的成员，通过索引，分数从高到低 | 按照PV降序获取页面<br/>`ZREVRANGE pv_zset 0 -1`
11 | `ZREVRANK key member`<br/>返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 | 获取page2.html的pv排名（降序）<br/>`ZREVRANK pv_zset page2.html`
12 | `ZSCORE key member`<br/>返回有序集中，成员的分数值 | 获取page3.html的分数值<br/>`ZSCORE pv_zset page3.html`


### 2.6 Key操作

序号 | 命令及描述 | 示例
---|---|---
1 | `DEL key`<br/>该命令用于在 key 存在时删除 key | del xiaoma5
2 | `DUMP key`<br/>序列化给定 key ，并返回被序列化的值 | DUMP key1
3 | `EXISTS key`<br/>检查给定 key 是否存在 | exists xiaoma
4 | `EXPIRE key seconds`<br/>为给定 key 设置过期时间，以秒计 | expire xiaoma 5
5 | `PEXPIRE key milliseconds`<br/>设置 key 的过期时间以毫秒计 | PEXPIRE set3 3000
6 | `KEYS pattern`<br/>查找所有符合给定模式( pattern)的 key | keys *
7 | `PERSIST key`<br/>移除 key 的过期时间，key 将持久保持 | persist set2
8 | `PTTL key`<br/>以毫秒为单位返回 key 的剩余的过期时间 | pttl  set2
9 | `TTL key`<br/>以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live) | ttl set2
10 | `RANDOMKEY`<br/>从当前数据库中随机返回一个 key | randomkey
11 | `RENAME key newkey`<br/>修改 key 的名称 | rename  set5 set8
12 | `RENAMENX key newkey`<br/>仅当 newkey 不存在时，将 key 改名为 newkey | renamenx  set8 set10
13 | `TYPE key`<br/>返回 key 所储存的值的类型 | type  set10

### 2.7 BitMaps位图

> 计算机最小的存储单位是位bit，Bitmaps是针对位的操作的，相较于String、Hash、Set等存储方式`更加节省空间`

> Bitmaps不是一种数据结构，操作是基于String结构的，一个String最大可以存储512M，那么一个Bitmaps则可以设置2^32个位

> Bitmaps单独提供了一套命令，所以在Redis中使用Bitmaps和使用字符串的方法不太相同。可以把Bitmaps想象成一个`存储0、1值的数组`，数组的每个单元值`只能存储0和1`，数组的`下标在Bitmaps中叫做偏移量`

![BitMaps位图](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/1.png)

> BitMaps 命令说明：将每个独立用户是否访问过网站存放在Bitmaps中， 将访问的用户记做1， 没有访问的用户记做0， 用偏移量作为用户的id 。

#### 2.7.1 设置值

```
SETBIT key offset value
```

> setbit命令设置的vlaue只能是`0`或`1`两个值

> 设置键的第offset个位的值（从0算起），假设现在有20个用户，uid=0，5，11，15，19的用户对网站进行了访问， 那么当前Bitmaps初始化结果如图所示

![设置值](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/2.png)

> 具体操作过程如下， unique:users:2016-04-05代表2016-04-05这天的独立访问用户的Bitmaps

```
setbit unique:users:2016-04-05 0 1
setbit unique:users:2016-04-05 5 1
setbit unique:users:2016-04-05 11 1
setbit unique:users:2016-04-05 15 1
setbit unique:users:2016-04-05 19 1
```

> 很多应用的用户id以一个指定数字（例如10000） 开头， 直接将用户id和Bitmaps的偏移量对应势必会造成一定的浪费， 通常的做法是每次做setbit操作时将用户id减去这个指定数字。

> 在第一次初始化Bitmaps时， 假如偏移量非常大， 那么整个初始化过程执行会比较慢， 可能会造成Redis的阻塞。

#### 2.7.2 获取值

```
GETBIT key offset
```

> 获取键的第offset位的值（从0开始算），例：下面操作获取id=8的用户是否在2016-04-05这天访问过， 返回0说明没有访问过

```
getbit unique:users:2016-04-05 8
```

#### 2.7.3 获取Bitmaps指定范围值为1的个数

```
BITCOUNT key [start end]
```

例：下面操作计算2016-04-05这天的独立访问用户数量：

```
bitcount unique:users:2016-04-05
```

#### 2.7.4 Bitmaps间的运算

```
BITOP operation destkey key [key, …]
```

> bitop是一个复合操作， 它可以做多个Bitmaps的and（交集） 、 or（并集） 、 not（非） 、 xor（异或） 操作并将结果保存在destkey中。 假设2016-04-04访问网站的userid=1， 2， 5， 9， 如图3-13所示：

![Bitmaps间的运算](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/998.png)

```
setbit unique:users:2016-04-04 1 1
setbit unique:users:2016-04-04 2 1
setbit unique:users:2016-04-04 5 1
setbit unique:users:2016-04-04 9 1
```


> 例1：下面操作计算出2016-04-04和2016-04-05两天都访问过网站的用户数量， 如下所示。

```
bitop and unique:users:and:2016-04-04_05 unique:users:2016-04-04 unique:users:2016-04-05
bitcount unique:users:2016-04-04_05
```

> 例2：如果想算出2016-04-04和2016-04-03任意一天都访问过网站的用户数量（例如月活跃就是类似这种） ， 可以使用or求并集， 具体命令如下：

```
bitop or unique:users:or:2016-04-04_05 unique:users:2016-04-04 unique:users:2016-04-05
bitcount unique:users:or:2016-04-04_05
```

![Bitmaps间的运算](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/999.png)

### 2.8 HyperLogLog结构

#### 2.8.1 应用场景

> HyperLogLog常用于`大数据量`的`去重`统计，比如页面访问量统计或者用户访问量统计。
> 要统计一个页面的访问量（PV），可以直接用redis计数器或者直接存数据库都可以实现，如果要统计一个页面的用户访问量（UV），一个用户一天内如果访问多次的话，也只能算一次，这样，我们可以使`用SET集合`来做，因为SET集合是有`去重`功能的，key存储页面对应的关键字，value存储对应的userid，这种方法是可行的。但如果访问量较多，假如有几千万的访问量，这就麻烦了。为了统计访问量，要频繁创建SET集合对象。

> Redis实现HyperLogLog算法，HyperLogLog 这个数据结构的发明人 是Philippe Flajolet（菲利普·弗拉若莱）教授。Redis 在 2.8.9 版本添加了 HyperLogLog 结构。

#### 2.8.2 UV计算示例

```
node1.xiaoma.cn:6379> help @hyperloglog

PFADD key element [element ...]
summary: Adds the specified elements to the specified HyperLogLog.
since: 2.8.9

PFCOUNT key [key ...]
summary: Return the approximated cardinality（基数） of the set(s) observed by the HyperLogLog at key(s).
since: 2.8.9

PFMERGE destkey sourcekey [sourcekey ...]
summary: Merge N different HyperLogLogs into a single one.
since: 2.8.9
```

> Redis集成的HyperLogLog使用语法主要有pfadd和pfcount，顾名思义，一个是来添加数据，一个是来统计的。为什么用`pf`？是因为HyperLogLog 这个数据结构的发明人 是Philippe Flajolet教授 ，所以用发明人的英文缩写，这样容易记住这个语法了。

> 下面我们通过一个示例，来演示如何计算uv。

```
node1.xiaoma.cn:6379> pfadd uv user1
(integer) 1
node1.xiaoma.cn:6379> keys *
1) "uv"
   node1.xiaoma.cn:6379> pfcount uv
   (integer) 1
   node1.xiaoma.cn:6379> pfadd uv user2
   (integer) 1
   node1.xiaoma.cn:6379> pfcount uv
   (integer) 2
   node1.xiaoma.cn:6379> pfadd uv user3
   (integer) 1
   node1.xiaoma.cn:6379> pfcount uv
   (integer) 3
   node1.xiaoma.cn:6379> pfadd uv user4
   (integer) 1
   node1.xiaoma.cn:6379> pfcount uv
   (integer) 4
   node1.xiaoma.cn:6379> pfadd uv user5 user6 user7 user8 user9 user10
   (integer) 1
   node1.xiaoma.cn:6379> pfcount uv
   (integer) 10
```
> HyperLogLog算法一开始就是为了大数据量的统计而发明的，所以很适合那种数据量很大，然后又没要求不能有一点误差的计算，HyperLogLog 提供不精确的去重计数方案，虽然不精确但是也不是非常不精确，标准误差是 0.81%，不过这对于页面用户访问量是没影响的，因为这种统计可能是访问量非常巨大，但是又没必要做到绝对准确，访问量对准确率要求没那么高，但是性能存储方面要求就比较高了，而HyperLogLog正好符合这种要求，不会占用太多存储空间，同时性能不错

> `pfadd`和`pfcount`常用于统计，需求：假如两个页面很相近，现在想统计这两个页面的用户访问量呢？这里就可以用`pfmerge`合并统计了，语法如例子：

```
node1.xiaoma.cn:6379> pfadd page1 user1 user2 user3 user4 user5
(integer) 1
node1.xiaoma.cn:6379> pfadd page2 user1 user2 user3 user6 user7
(integer) 1
node1.xiaoma.cn:6379> pfmerge page1+page2 page1 page2
OK
node1.xiaoma.cn:6379> pfcount page1+page2
(integer) 7
```

#### 2.8.3 HyperLogLog为什么适合做大量数据的统计

> Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。

> 在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

> 但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

什么是基数？

> 比如：数据集{1, 3, 5, 7, 5, 7, 8}，那么这个数据集的基数集{1, 3, 5, 7, 8}，基数（不重复元素）为5。基数估计就是在误差可接受的范围内，快速计算基数。

## 3 Redis的持久化

> 由于redis是一个内存数据库，所有的数据都是保存在内存当中的，内存当中的数据极易丢失，所以redis的数据持久化就显得尤为重要，在redis当中，提供了两种数据持久化的方式，分别为RDB以及AOF，且Redis默认开启的数据持久化方式为RDB方式。

### 3.1 RDB持久化方案

#### 3.1.1 介绍

> `Redis`会定期保存数据快照至一个`rbd`文件中，并在启动时自动加载`rdb`文件，恢复之前保存的数据。可以在配置文件中配置`Redis`进行快照保存的时机：

```
save [seconds] [changes]
```

> 意为在`seconds`秒内如果发生了`changes`次数据修改，则进行一次`RDB`快照保存，例如

```
save 60 100
```

> 会让`Redis`每`60秒`检查一次数据变更情况，如果发生了`100次`或以上的数据变更，则进行`RDB`快照保存。可以配置多条`save`指令，让`Redis`执行多级的快照保存策略。`Redis`默认开启`RDB`快照。也可以通过`SAVE`或者`BGSAVE`命令手动触发RDB快照保存。`SAVE`和`BGSAVE`两个命令都会调用`rdbSave`函数，但它们调用的方式各有不同：

> `SAVE`直接调用`rdbSave`，阻塞`Redis`主进程，直到保存完成为止。在主进程阻塞期间，服务器不能处理客户端的任何请求。

> `BGSAVE`则`fork`出一个子进程，子进程负责调用`rdbSave`，并在保存完成之后向主进程发送信号，通知保存已完成。`Redis`服务器在`BGSAVE`执行期间仍然可以继续处理客户端的请求。


#### 3.1.2 RDB方案优点

> 对性能影响最小。如前文所述，Redis在保存RDB快照时会fork出子进程进行，几乎不影响Redis处理客户端请求的效率。

> 每次快照会生成一个完整的数据快照文件，所以可以辅以其他手段保存多个时间点的快照（例如把每天0点的快照备份至其他存储媒介中），作为非常可靠的灾难恢复手段。

> 使用RDB文件进行数据恢复比使用AOF要快很多

#### 3.1.3 RBD方案缺点

> 快照是定期生成的，所以在Redis crash时或多或少会丢失一部分数据

> 如果数据集非常大且CPU不够强（比如单核CPU），Redis在fork子进程时可能会消耗相对较长的时间，影响Redis对外提供服务的能力

#### 3.1.4 RDB配置

> 修改redis的配置文件

```
cd /export/server/redis-3.2.8/
vim redis.conf
# 第202行
save 900 1
save 300 10
save 60 10000
save 5 1
```

> 这三个选项是redis的配置文件默认自带的存储机制。表示每隔多少秒，有多少个key发生变化就生成一份dump.rdb文件，作为redis的快照文件

例如：save  60  10000 表示在60秒内，有10000个key发生变化，就会生成一份redis的快照

> 重新启动redis服务

每次生成新的dump.rdb都会覆盖掉之前的老的快照

```
ps -ef | grep redis
bin/redis-cli -h node1.xiaoma.cn shutdown
bin/redis-server redis.conf
```

### 3.2 AOF持久化方案

#### 3.2.1 介绍

> 采用`AOF`持久方式时，`Redis`会把每一个写请求都记录在一个日志文件里。在`Redis`重启时，会把`AOF`文件中记录的所有写操作顺序执行一遍，确保数据恢复到最新。

#### 3.2.2 开启AOF

> AOF默认是关闭的，如要开启，进行如下配置：

```
# 第594行
appendonly yes

```

#### 3.2.3 配置AOF

> AOF提供了三种fsync配置：always/everysec/no，通过配置项[appendfsync]指定：

> 1.`appendfsync no`：不进行fsync，将flush文件的时机交给OS决定，速度最快<br/>
2.`appendfsync always`：每写入一条日志就进行一次fsync操作，数据安全性最高，但速度最慢<br/>
3.`appendfsync everysec`：折中的做法，交由后台线程每秒fsync一次

#### 3.2.4 AOF rewrite

> 随着AOF不断地记录写操作日志，因为所有的写操作都会记录，所以必定会出现一些无用的日志。大量无用的日志会让AOF文件过大，也会让数据恢复的时间过长。不过Redis提供了AOF rewrite功能，可以重写AOF文件，只保留能够把数据恢复到最新状态的最小写操作集。

> AOF rewrite可以通过BGREWRITEAOF命令触发，也可以配置Redis定期自动进行：

```
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

> Redis在每次AOF rewrite时，会记录完成rewrite后的AOF日志大小，当AOF日志大小在该基础上增长了100%后，自动进行AOF rewrite

> auto-aof-rewrite-min-size最开始的AOF文件必须要触发这个文件才触发，后面的每次重写就不会根据这个变量了。该变量仅初始化启动Redis有效。

#### 3.2.5 AOF优点

> 最安全，在启用appendfsync为always时，任何已写入的数据都不会丢失，使用在启用appendfsync everysec也至多只会丢失1秒的数据

> AOF文件在发生断电等问题时也不会损坏，即使出现了某条日志只写入了一半的情况，也可以使用redis-check-aof工具轻松修复

> AOF文件易读，可修改，在进行某些错误的数据清除操作后，只要AOF文件没有rewrite，就可以把AOF文件备份出来，把错误的命令删除，然后恢复数据。

#### 3.2.6 AOF缺点

> 1.AOF文件通常比RDB文件更大<br/>
2.性能消耗比RDB高<br/>
3.数据恢复速度比RDB慢

> Redis的数据持久化工作本身就会带来延迟，需要根据数据的安全级别和性能要求制定合理的持久化策略：

>`AOF + fsync always`的设置虽然能够绝对确保数据安全，但每个操作都会触发一次fsync，会对Redis的性能有比较明显的影响<br/>
`AOF + fsync every second`是比较好的折中方案，每秒fsync一次<br/>
`AOF + fsync never`会提供AOF持久化方案下的最优性能<br/>
使用RDB持久化通常会提供比使用AOF更高的性能，但需要注意RDB的策略配置

### 3.3 RDB or AOF

> 每一次RDB快照和AOF Rewrite都需要Redis主进程进行fork操作。fork操作本身可能会产生较高的耗时，与CPU和Redis占用的内存大小有关。根据具体的情况合理配置RDB快照和AOF Rewrite时机，避免过于频繁的fork带来的延迟

> Redis在fork子进程时需要将内存分页表拷贝至子进程，以占用了24GB内存的Redis实例为例，共需要拷贝48MB的数据。在使用单Xeon 2.27Ghz的物理机上，这一fork操作耗时216ms。

## 4 Redis 高级使用

### 4.1 Redis事务

#### 4.1.1 Redis事务简介

> Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

**`总结说：Redis事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令`**

- Redis事务没有隔离级别的概念
> 批量操作在发送 EXEC 命令前被放入队列缓存，并不会被实际执行，也就不存在事务内的查询要看到事务里的更新，事务外查询不能看到。

- Redis不保证原子性
> Redis中，单条命令是原子性执行的，但事务不保证原子性，且没有回滚。事务中任意命令执行失败，其余的命令仍会被执行。
一个事务从开始到执行会经历以下三个阶段：

- 第一阶段：开始事务
- 第二阶段：命令入队
- 第三阶段、执行事务。

Redis事务相关命令：
```
MULTI
开启事务，redis会将后续的命令逐个放入队列中，然后使用EXEC命令来原子化执行这个命令队列

EXEC
执行事务中的所有操作命令

DISCARD
取消事务，放弃执行事务块中的所有命令

WATCH
监视一个或多个key，如果事务在执行前，这个key（或多个key）被其他命令修改，则事务被中断，不会执行事务中的任何命令

UNWATCH
取消WATCH对所有key的监视
```

#### 4.1.2 Redis事务演示

> MULTI开始一个事务：给k1、k2分别赋值，在事务中修改k1、k2，执行事务后，查看k1、k2值都被修改。

```
node1.xiaoma.cn:6379> set k1 v1
OK
node1.xiaoma.cn:6379> set k2 v2
OK
node1.xiaoma.cn:6379> multi
OK
node1.xiaoma.cn:6379> set k1 11
QUEUED
node1.xiaoma.cn:6379> set k2 22
QUEUED
node1.xiaoma.cn:6379> exec
1) OK
2) OK
node1.xiaoma.cn:6379> get k1
"11"
node1.xiaoma.cn:6379> get k2
"22"
```

> 事务失败处理：语法错误（编译器错误），在开启事务后，修改k1值为11，k2值为22，但k2语法错误，最终导致事务提交失败，k1、k2保留原值。

```
node1.xiaoma.cn:6379> flushdb
OK
node1.xiaoma.cn:6379> keys *
(empty list or set)
node1.xiaoma.cn:6379> set k1 v1
OK
node1.xiaoma.cn:6379> set k2 v2
OK
node1.xiaoma.cn:6379> multi
OK
node1.xiaoma.cn:6379> set k1 11
QUEUED
node1.xiaoma.cn:6379> sets k2 22
(error) ERR unknown command 'sets'
node1.xiaoma.cn:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
node1.xiaoma.cn:6379> get k1
"v1"
node1.xiaoma.cn:6379> get k2
"v2"
```

> Redis类型错误（运行时错误），在开启事务后，修改k1值为11，k2值为22，但将k2的类型作为List，在运行时检测类型错误，最终导致事务提交失败，此时事务并没有回滚，而是跳过错误命令继续执行， 结果k1值改变、k2保留原值。

```
node1.xiaoma.cn:6379> flushdb
OK
node1.xiaoma.cn:6379> keys *
(empty list or set)
node1.xiaoma.cn:6379> set k1 v1
OK
node1.xiaoma.cn:6379> set k2 v2
OK
node1.xiaoma.cn:6379> multi
OK
node1.xiaoma.cn:6379> set k1 11
QUEUED
node1.xiaoma.cn:6379> lpush k2 22
QUEUED
node1.xiaoma.cn:6379> exec
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
node1.xiaoma.cn:6379> get k1
"11"
node1.xiaoma.cn:6379> get k2
"v2"
```

> DISCARD取消事务

```
node1.xiaoma.cn:6379> multi
OK
node1.xiaoma.cn:6379> set k6 v6
QUEUED
node1.xiaoma.cn:6379> set k7 v7
QUEUED
node1.xiaoma.cn:6379> discard
OK
node1.xiaoma.cn:6379> get k6
(nil)
node1.xiaoma.cn:6379> get k7
(nil)
```

#### 4.1.3 为什么Rdis不支持事务回滚

> 多数事务失败是由语法错误或者数据结构类型错误导致的，语法错误说明在命令入队前就进行检测的，而类型错误是在执行时检测的，Redis为提升性能而采用这种简单的事务，这是不同于关系型数据库的，特别要注意区分。Redis之所以保持这样简易的事务，完全是为了保证高并发下的核心问题——`性能`。

### 4.2 Redis过期策略

> Redis是key-value数据库，可以设置Redis中缓存的key的过期时间。Redis的过期策略就是指当Redis中缓存的key过期了，Redis如何处理。
过期策略通常有以下三种：

- `定时过期`

> 每个设置过期时间的key都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，`对内存很友好`；但是会`占用大量的CPU资源`去处理过期的数据，从而影响缓存的响应时间和吞吐量。

- `惰性过期`

> 只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以`最大化地节省CPU资源，却对内存非常不友好`。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。

- `定期过期`

> 每隔一定的时间，会扫描一定数量的数据库的expires字典中一定数量的key，并清除其中已过期的key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得CPU和内存资源达到最优的平衡效果。

### 4.3 内存淘汰策略

> Redis的内存淘汰策略是指在Redis的用于缓存的内存不足时，怎么处理需要新写入且需要申请额外空间的数据，在Redis的配置文件中描述如下：

```
# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
#最大内存策略：当到达最大使用内存时，你可以在下面5种行为中选择，Redis如何选择淘汰数据库键
#当内存不足以容纳新写入数据时

# volatile-lru -> remove the key with an expire set using an LRU algorithm
# volatile-lru ：在设置了过期时间的键空间中，移除最近最少使用的key。这种情况一般是把 redis 既当缓存，又做持久化存储的时候才用。

# allkeys-lru -> remove any key according to the LRU algorithm
# allkeys-lru ： 移除最近最少使用的key （推荐）

# volatile-random -> remove a random key with an expire set
# volatile-random ： 在设置了过期时间的键空间中，随机移除一个键，不推荐

# allkeys-random -> remove a random key, any key
# allkeys-random ： 直接在键空间中随机移除一个键，弄啥叻

# volatile-ttl -> remove the key with the nearest expire time (minor TTL)
# volatile-ttl ： 在设置了过期时间的键空间中，有更早过期时间的key优先移除 不推荐

# noeviction -> don't expire at all, just return an error on write operations
# noeviction ： 不做过键处理，只返回一个写操作错误。 不推荐

# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are no suitable keys for eviction.
# 上面所有的策略下，在没有合适的淘汰删除的键时，执行写操作时，Redis 会返回一个错误。下面是写入命令：
#       At the date of writing these commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort

# 过期策略默认是：
# The default is:
# maxmemory-policy noeviction
```

> 实际项目中设置内存淘汰策略：maxmemory-policy allkeys-lru，移除最近最少使用的key。

## 5 Redis的主从复制架构

### 5.1 简介 

> 主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为`主节点(master)`，后者称为`从节点(slave)`，数据的复制是单向的，只能由主节点到从节点。

![简介](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/3.png)

> 默认情况下，每台Redis服务器都是主节点；且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。

#### 5.1.1 一主一从

> 如下图所示左边是Master节点，右边是slave节点，即主节点和从节点。从节点也是可以对外提供服务的，主节点是有数据的，从节点可以通过复制操作将主节点的数据同步过来，并且随着主节点数据不断写入，从节点数据也会做同步的更新。

![一主一从](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/4.png)

> 从节点起到的就是数据备份的效果。

#### 5.1.2 一主多从

> 除了一主一从模型之外，Redis还提供了一主多从的模型，也就是一个master可以有多个slave，也就相当于有了多份的数据副本。

![一主多从](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/5.png)

> 可以做一个更加高可用的选择，例如一个master和一个slave挂掉了，还能有其他的slave数据备份。

### 5.2 主从复制原理

1. 当从数据库启动后，会向主数据库发送SYNC命令
2. 主数据库接收到SYNC命令后开始在后台保存快照（RDB持久化），并将保存快照期间接收到的命令缓存再来
3. 快照完成后，Redis（Master）将快照文件和所有缓存的命令发送给从数据库
4. Redis（Slave）接收到RDB和缓存命令时，会开始载入快照文件并执行接收到的缓存的命令
5. 后续，每当主数据库接收到写命令时，就会将命令同步给从数据库。所以3和4只会在初始化的时候执行

### 5.3 主从复制的应用的场景

#### 5.3.1 读写分离

1. 通过主从复制可以实现读写分离，以提高服务器的负载能力
2. 在常见的场景中（例如：电商网站），读的频率大于写
3. 当单机Redis无法应付大量的读请求时（尤其是消耗资源的请求），就可以通过主从复制功能来建立多个从数据库节点，主数据库只进行写操作，从数据库负责读操作
4. 这种主从复制，比较适合用来处理读多写少的场景，而当单个主数据库不能满足需求时，就需要使用Redis 3.0后推出的集群功能

#### 5.3.2 从数据库持久化

1. Redis中相对耗时的操作就是持久化，为了提高性能，可以通过主从复制创建一个或多个从数据库，并在从数据库中启用持久化，同时在主数据库中禁用持久化（例如：禁用AOF）
2. 当从数据库崩溃重启后主数据库会自动将数据同步过来，无需担心数据丢失
3. 而当主数据库崩溃时，后续我们可以通过哨兵（Sentinel）来解决

### 5.4 另外两台服务器安装Redis

#### 5.4.1 安装Redis依赖环境

#### 5.4.2 上传Redis压缩包

#### 5.4.3 服务器安装tcl

#### 5.4.4 编译redis

#### 5.4.5 修改redis配置文件

##### 5.4.5.1 node2.itast.cn服务器修改配置文件

> 执行以下命令修改Redis配置文件

```
cd /export/server/redis-3.2.8/
mkdir -p /export/server/redis-3.2.8/log
mkdir -p /export/server/redis-3.2.8/data

vim redis.conf
# 修改第61行
bind node2.xiaoma.cn
# 修改第128行
daemonize yes
# 修改第163行
logfile "/export/server/redis-3.2.8/log/redis.log"
# 修改第247行
dir /export/server/redis-3.2.8/data
# 修改第266行，配置node2.xiaoma.cn为第一台服务器的slave节点
slaveof node1.xiaoma.cn 6379
```

##### 5.4.5.2 node3.xiaoma.cn服务器修改配置文件

> 执行以下命令修改redis配置文件

```
cd /export/server/redis-3.2.8/
mkdir -p /export/server/redis-3.2.8/log
mkdir -p /export/server/redis-3.2.8/data

vim redis.conf
# 修改第61行
bind node3.xiaoma.cn
# 修改第128行
daemonize yes
# 修改第163行
logfile "/export/server/redis-3.2.8/log/redis.log"
# 修改第247行
dir /export/server/redis-3.2.8/data
# 修改第266行，配置node2.xiaoma.cn为第一台服务器的slave节点
slaveof node1.xiaoma.cn 6379
```

### 5.5 启动Redis服务

> node2.xiaoma.cn执行以下命令启动Redis服务

```
cd  /export/server/redis-3.2.8/
bin/redis-server redis.conf
```

> node3.xiaoma.cn执行以下命令启动Redis服务

```
cd  /export/server/redis-3.2.8/
bin/redis-server redis.conf
```

> 启动成功便可以实现redis的主从复制，node1.xiaoma.cn可以读写操作，node2.xiaoma.cn与node3.xiaoma.cn只支持读取操作。

## 6 Redis中的Sentinel架构

### 6.1 Sentinel介绍

> Sentinel（哨兵）是Redis的高可用性解决方案：由一个或多个Sentinel实例 组成的Sentinel系统可以监视任意多个主服务器，以及这些主服务器属下的所有从服务器，并在被监视的主服务器进入下线状态时，自动将下线主服务器属下的某个从服务器升级为新的主服务器。

例如：

![Sentinel介绍](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/6.png)

在Server1 掉线后：

![Server1 掉线后](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/7.png)

升级Server2 为新的主服务器：

![升级Server2 为新的主服务器](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/8.png)

### 6.2 配置哨兵

![配置哨兵](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/9.png)

#### 6.2.1 三台机器修改哨兵配置文件

> 三台机器执行以下命令修改redis的哨兵配置文件

```
cd /export/server/redis-3.2.8
vim sentinel.conf
```

**配置监听的主服务器**

> 1.修改node1.xiaoma.cn的sentinel.conf文件

```
#修改第15行， bind配置，每台机器修改为自己对应的主机名
bind node1.xiaoma.cn  
# 在下方添加配置，让sentinel服务后台运行
daemonize yes
#修改第71行，三台机器监控的主节点，现在主节点是node1.xiaoma.cn服务器
sentinel monitor mymaster node1.xiaoma.cn 6379 2
```

参数说明
- sentinel monitor代表监控
- mymaster代表服务器的名称，可以自定义
- node1.xiaoma.cn代表监控的主服务器，6379代表端口
- 2代表只有两个或两个以上的哨兵认为主服务器不可用的时候，才会进行failover操作。

如果Redis是有密码的，需要指定密码

```
# sentinel author-pass定义服务的密码，mymaster是服务名称，123456是Redis服务器密码
# sentinel auth-pass <master-name> <password>
```

> 2.分发到node2.xiaoma.cn和node3.xiaoma.cn

```
scp sentinel.conf node2.xiaoma.cn:$PWD
scp sentinel.conf node3.xiaoma.cn:$PWD
```

> 3.分别修改配置中bind的服务器主机名

node2.xiaoma.cn
```
cd /export/server/redis-3.2.8
vim sentinel.conf
# 修改第18行
bind node2.xiaoma.cn
```

node3.xiaoma.cn

```
cd /export/server/redis-3.2.8
vim sentinel.conf
# 修改第18行
bind node3.xiaoma.cn
```

#### 6.2.2 三台机器启动哨兵服务

```
cd /export/server/redis-3.2.8
bin/redis-sentinel sentinel.conf
```
三台服务器的进程信息：
```
node1.xiaoma.cn
[root@node1 redis-3.2.8]# ps aux | grep redis
root      18911  0.0  0.0 136920  2456 ?        Ssl  08:58   0:04 bin/redis-server node1.xiaoma.cn:6379
root      19112  0.0  0.0 149232  5152 pts/1    S+   09:16   0:00 vim redis.conf
root      20544  0.1  0.0 135728  2328 ?        Ssl  10:48   0:00 bin/redis-sentinel node1.xiaoma.cn:26379 [sentinel]
root      20548  0.0  0.0 112712   960 pts/3    S+   10:48   0:00 grep --color=auto redis

node2.xiaoma.cn
[root@node2 redis-3.2.8]# ps aux | grep redis
root      26260  0.0  0.1 139200  4456 ?        Ssl  10:34   0:00 bin/redis-server node2.xiaoma.cn:6379
root      26421  0.1  0.0 139204  2440 ?        Ssl  10:48   0:00 bin/redis-sentinel node2.xiaoma.cn:26379 [sentinel]
root      26438  0.0  0.0 112812   972 pts/1    S+   10:49   0:00 grep --color=auto redis

node3.xiaoma.cn
[root@node3 redis-3.2.8]# ps aux | grep redis
root      22325  0.0  0.0 135992  2376 ?        Ssl  10:34   0:00 bin/redis-server node3.xiaoma.cn:6379
root      22463  0.1  0.0 135836  2384 ?        Ssl  10:48   0:00 bin/redis-sentinel node3.xiaoma.cn:26379 [sentinel]
root      22475  0.0  0.0 112812   972 pts/1    S+   10:49   0:00 grep --color=auto redis
```

#### 6.2.3 node1服务器杀死redis服务进程

```
查看Sentinel master的状态
bin/redis-cli -h node2.xiaoma.cn -p 26379
使用ping命令检查哨兵是否工作，如果正常会返回PONG
node2.xiaoma.cn:26379> ping
PONG
node2.xiaoma.cn:26379> info
... ... ...

# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.88.100:6379,slaves=2,sentinels=3
```


使用kill -9命令杀死redis服务进程，模拟redis故障宕机情况
过一段时间之后，就会在node2.xiaoma.cn与node3.xiaoma.cn服务器选择一台服务器来切换为主节点

```
node2.xiaoma.cn:26379> info
... ... ...
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.88.102:6379,slaves=2,sentinels=3
```

### 6.3 Redis的sentinel模式代码开发连接

通过哨兵连接，要指定哨兵的地址，并使用JedisSentinelPool来创建连接池。

实现步骤：
1.在 cn.xiaoma.redis.api_test 包下创建一个新的类 ReidsSentinelTest
2.构建JedisPoolConfig配置对象
3.创建一个HashSet，用来保存哨兵节点配置信息（记得一定要写端口号）
4.构建JedisSentinelPool连接池
5.使用sentinelPool连接池获取连接

```
public class ReidsSentinelTest {
private JedisPoolConfig jedisPoolConfig;
private JedisSentinelPool jedisSentinelPool;

    @BeforeTest
    public void beforeTest() {
        // 1. 构建JedisPoolConfig配置对象
        jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(50);
        jedisPoolConfig.setMaxIdle(10);
        jedisPoolConfig.setMinIdle(5);
        jedisPoolConfig.setMaxWaitMillis(10000);

        // 2. 创建一个HashSet，用来保存哨兵节点配置信息
        HashSet<String> sentinelNodeSet = new HashSet<>(Arrays.asList("node1.xiaoma.cn:26379", "node2.xiaoma.cn:26379", "node3.xiaoma.cn:26379"));

        // 3. 构建JedisSentinelPool连接池
        jedisSentinelPool = new JedisSentinelPool("mymaster", sentinelNodeSet, jedisPoolConfig);

    }

    @Test
    public void keysOpTest() {
        // 使用sentinelPool连接池获取连接
        Jedis connection = jedisSentinelPool.getResource();
        Set<String> keySet = connection.keys("*");

        for (String key : keySet) {
            System.out.println(key);
        }
    }

    @AfterTest
    public void afterTest() {
        jedisSentinelPool.close();
    }
}
```

## 7 Redis 集群

> Redis最开始使用`主从模式做集群`，若master宕机需要手动配置slave转为master；后来为了`高可用提出来哨兵模式`，该模式下有一个哨兵监视master和slave，若master宕机可自动将slave转为master，但它也有一个问题，就是`不能动态扩充`；所以在Redis 3.x提出`cluster集群`模式。

### 7.1 引言

Redis Cluster是Redis官方提供的Redis集群功能，为什么要实现Redis Cluster？

1. 主从复制不能实现高可用
2. 随着公司发展，用户数量增多，并发越来越多，业务需要更高的QPS，而主从复制中单机的QPS可能无法满足业务需求；
3. 数据量的考虑，现有服务器内存不能满足业务数据的需要时，单纯向服务器添加内存不能达到要求，此时需要考虑分布式需求，把数据分布到不同服务器上；
4. 网络流量需求，业务的流量已经超过服务器的网卡的上限值，可考虑使用分布式来进行分流；
5. 离线计算，需要中间环节缓冲等其他需求；
 
在存储引擎框架（MySQL、HDFS、HBase、Redis、Elasticsearch等）中，只要数据量很大时，单机无法承受压力，最好的方式就是：数据分布进行存储管理。

**`对Redis 内存数据库来说：全量数据，单机Redis节点无法满足要求，按照分区规则把数据分到若干个子集当中。`**

![引言](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/10.png)

> Redis集群数据分布方式：

`虚拟槽分区`：虚拟槽分区是Redis Cluster采用的分区方式，预设虚拟槽，每个槽就相当于一个数字，有一定范围。每个槽映射一个数据子集，一般比节点数大。Redis Cluster中预设虚拟槽的范围为0到16383。

![集群数据分布方式](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/11.png)

### 7.2 Redis Cluster 设计

> Redis Cluster是分布式架构，有多个节点，每个节点都负责进行数据读写操作，每个节点之间会进行通信。Redis Cluster采用无中心结构，每个节点保存数据和整个集群状态，每个节点都和其他所有节点连接。

![Redis Cluster 设计](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/12.png)

结构特点：

- 所有的redis节点彼此互联(PING-PONG机制)，内部使用二进制协议优化传输速度和带宽；
- 节点的fail是通过集群中超过半数的节点检测失效时才生效；
- 客户端与redis节点直连，不需要中间proxy层，客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可；
- redis-cluster 把所有的物理节点映射到[0-16383]slot上（不一定是平均分配）,cluster 负责维护node<->slot<->value；
- Redis集群预分好16384个桶（Slot），当需要在 Redis 集群中放置一个 key-value 时，根据 CRC16(key) & 16384的值，决定将一个key放到哪个桶中；

![结构特点](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/13.png)

Redis 集群的优势：
- 缓存永不宕机：启动集群，永远让集群的一部分起作用。主节点失效了子节点能迅速改变角色成为主节点，整个集群的部分节点失败或者不可达的情况下能够继续处理命令；
- 迅速恢复数据：持久化数据，能在宕机后迅速解决数据丢失的问题；
- Redis可以使用所有机器的内存，变相扩展性能；
- 使Redis的计算能力通过简单地增加服务器得到成倍提升，Redis的网络带宽也会随着计算机和网卡的增加而成倍增长；
- Redis集群没有中心节点，不会因为某个节点成为整个集群的性能瓶颈;
- 异步处理数据，实现快速读写；

Redis 3.0以后，节点之间通过去中心化的方式提供了完整的sharding(数据分片)、replication(复制机制、Cluster具备感知准备的能力)、failover解决方案。

![集群的优势](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/14.png)

### 7.3 Redis Cluster 搭建

> Redis3.0及以上版本实现，集群中至少应该有奇数个节点，所以至少有三个节点，官方推荐`三主三从`的配置方式。Redis 3.x和Redis4.x 搭建集群是需要手动安装`ruby`组件的，比较麻烦。

> 2018年十月 Redis 发布了稳定版本的 5.0 版本，推出了各种新特性，其中一点是放弃 Ruby的集群方式，改为 使用 C语言编写的redis-cli的方式，是集群的构建方式复杂度大大降低。`Redis cluster tutorial`：https://redis.io/topics/cluster-tutorial 。

![Redis Cluster 搭建](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/15.png)

> 基于Redis-5.0.8版本，在三台机器上搭建6个节点的Redis集群：三主三从架构。

#### 7.3.1 环境准备

关闭以前Redis主从复制和哨兵模式监控的所有服务，备注：如果以前没有安装过Redis服务，不用执行此步骤操作。

```
# ============= node1.xiaoma.cn、node2.xiaoma.cn和node3.xiaoma.cn =============
# 关闭哨兵服务SentinelServer
ps -ef|grep redis
kill -9 哨兵的进程ID

# 关闭Redis服务
redis-cli -h node1.xiaoma.cn -p 6379 SHUTDOWN
redis-cli -h node2.xiaoma.cn -p 6379 SHUTDOWN
redis-cli -h node3.xiaoma.cn -p 6379 SHUTDOWN
```

安装Redis编译环境：GCC和TCL。

```
yum -y install gcc-c++ tcl
```

#### 7.3.2 上传和解压

> 将Redis-5.0.8软件安装包上传至 /export/software 目录，并解压与安装。

```
# node01, 上传安装包至/export/softwares
cd /export/software
rz

# 解压
cd /export/software
chmod u+x redis-5.0.8.tar.gz
tar -zxvf redis-5.0.8.tar.gz -C /export/server/
```

#### 7.3.3 编译安装

> 编译Redis 源码，并安装至【/export/server/redis-5.0.8-bin】目录。

```
# node01, 编译、安装、创建软连接
# 进入源码目录
cd /export/server/redis-5.0.8
# 编译
make
# 安装至指定目录
make PREFIX=/export/server/redis-5.0.8-bin install

# 创建安装目录软连接
cd /export/server
ln -s redis-5.0.8-bin redis
```

配置环境变量（如果以前安装过Redis，配置过环境变量，就不用配置）。

```
# 配置环境变量
vim /etc/profile
# ======================== 添加如下内容 ========================
    # REDIS HOME
export REDIS_HOME=/export/server/redis
export PATH=:$PATH:$REDIS_HOME/bin
# 执行生效
source /etc/profile
```

#### 7.3.4 拷贝配置文件

> 从Redis-5.0.8源码目录下拷贝配置文件：redis.conf至Redis 安装目录。

```
# ====================== node01 上操作 ======================
# 拷贝配置文件
cd /export/server/redis-5.0.8
cp redis.conf /export/server/redis
```

#### 7.3.5 修改配置文件

> 每台机器上启动2个Redis服务，一个主节点服务：7001，一个从节点服务：7002，如下图所示：

![修改配置文件](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/16.png)

> 在Redis安装目录下创建7001和7002目录，分别存储Redis服务配置文件、日志及数据文件。

```
# 创建目录：7001和7002
cd /export/server/redis
mkdir -p 7001 7002
```

> 拷贝配置文件：redis.conf至7001目录，并重命名为redis_7001.conf。

```
cd /export/server/redis
cp redis.conf 7001/redis_7001.conf
```

> 编辑配置文件：redis_7001.conf，内容如下：

```
cd /export/server/redis/7001
vim redis_7001.conf
## =========================== 修改内容说明如下 ===========================
## 69行，配置redis服务器接受链接的网卡
bind 0.0.0.0
## 88行，关闭保护模式
protected-mode no
## 92行，设置端口号
port 7001
## 136行，redis后台运行
daemonize yes
## 158行，Redis服务进程PID存储文件名称
pidfile /var/run/redis_7001.pid

## 171行，设置redis服务日志存储路径
logfile "/export/server/redis-5.0.8-bin/7001/log/redis.log"
## 263行，设置redis持久化数据存储目录
dir /export/server/redis-5.0.8-bin/7001/data/

## 699行，启动AOF方式持久化
appendonly yes

## 832行，启动Redis Cluster
cluster-enabled yes
## 840行，Redis服务配置保存文件名称
cluster-config-file nodes-7001.conf
## 847行，超时时间
cluster-node-timeout 15000
```

创建日志目录和数据目录：

```
mkdir -p /export/server/redis/7001/log
mkdir -p /export/server/redis/7001/data
```

> 配置7002端口号启动Redis服务，操作命令如下：

```
## 拷贝配置文件
cd /export/server/redis
cp 7001/redis_7001.conf 7002/redis_7002.conf

## 修改配置文件：redis_7002.conf
cd /export/server/redis/7002
vim redis_7002.conf
# 进入vim编辑之后，执行以下代码将7001全部替换成7002
:%s/7001/7002/g   # 表示:%s/old/new/g  g表示全部替换

# 创建目录
mkdir -p /export/server/redis/7002/log
mkdir -p /export/server/redis/7002/data
```

#### 7.3.6 发送安装包

> 将node1.xiaoma.cn上配置好的Redis安装包，发送至node2.xiaoma.cn和node3.xiaoma.cn，每台机器运行2个Redis服务，端口号分别为7001和7002，具体命令如下：

```
# 发送安装包
cd /export/server
scp -r redis-5.0.8-bin root@node2.xiaoma.cn:$PWD
scp -r redis-5.0.8-bin root@node3.xiaoma.cn:$PWD

# 在node2和node3创建软连接
cd /export/server
ln -s redis-5.0.8-bin redis

# 配置环境变量
vim /etc/profile
# ======================== 添加如下内容 ========================
    # REDIS HOME
export REDIS_HOME=/export/server/redis
export PATH=:$PATH:$REDIS_HOME/bin
# 执行生效
source /etc/profile
```

### 7.4 启动Redis服务

> 在三台机器node01、node02和node03，分别启动6个Redis服务，命令如下：

```
# 启动7001端口Redis服务
/export/server/redis/bin/redis-server /export/server/redis/7001/redis_7001.conf

# 启动7002端口Redis服务
/export/server/redis/bin/redis-server /export/server/redis/7002/redis_7002.conf
```

> Redis服务启动完成以后，查看如下：

![edis服务启动完成以后](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/17.png)

![edis服务启动完成以后](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/18.png)

![edis服务启动完成以后](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/19.png)

#### 7.4.1 启动集群

> Redis5.x版本之后，通过redis-cli客户端命令来进行创建集群，注意：Redis对主机名解析不友好，使用IP地址。

```
# 任意选择一台机器执行如下命令，创建集群
/export/server/redis/bin/redis-cli --cluster create 192.168.88.100:7001 192.168.88.101:7002 192.168.88.102:7001 192.168.88.100:7002 192.168.88.101:7001 192.168.88.102:7002 --cluster-replicas 1
```

> 启动集群日志信息如下：

```
>>> Performing hash slots allocation on 6 nodes...
## ====== 进行Slot槽范文划分 ======
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 – 16383
## ====== 主从分配，一主一从 ======
Adding replica 192.168.88.101:7001 to 192.168.88.100:7001
Adding replica 192.168.88.102:7002 to 192.168.88.101:7002
Adding replica 192.168.88.100:7002 to 192.168.88.102:7001
## ====== 主从节点配对信息 ======
M: 97e1d381d59561e075ac813e2df7fed00114687e 192.168.88.100:7001
slots:[0-5460] (5461 slots) master
M: 6e0353e92377a71d691a853152673a8774d11dc2 192.168.88.101:7002
slots:[5461-10922] (5462 slots) master
M: d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 192.168.88.102:7001
slots:[10923-16383] (5461 slots) master
S: b679ac2df0df7509ffd3a1d3b460cb9e13f9dbfa 192.168.88.100:7002
replicates d9bf2ac8eec5637ed7ec50061419ff6b951eef0b
S: b8a76df88aafb19ce38232d0b4c1daf12370f257 192.168.88.101:7001
replicates 97e1d381d59561e075ac813e2df7fed00114687e
S: c038b9e271f5b5dba47d6ef81c4f49548a9f3698 192.168.88.102:7002
replicates 6e0353e92377a71d691a853152673a8774d11dc2
## ====== 是否同意上述划分，一般都是yes ======
Can I set the above configuration? (type 'yes' to accept): yes
## ====== 节点配置更新，进入集群，主从节点，高可用 ======
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 192.168.88.100:7001)
M: 97e1d381d59561e075ac813e2df7fed00114687e 192.168.88.100:7001
slots:[0-5460] (5461 slots) master
1 additional replica(s)
S: c038b9e271f5b5dba47d6ef81c4f49548a9f3698 192.168.88.102:7002
slots: (0 slots) slave
replicates 6e0353e92377a71d691a853152673a8774d11dc2
S: b8a76df88aafb19ce38232d0b4c1daf12370f257 192.168.88.101:7001
slots: (0 slots) slave
replicates 97e1d381d59561e075ac813e2df7fed00114687e
S: b679ac2df0df7509ffd3a1d3b460cb9e13f9dbfa 192.168.88.100:7002
slots: (0 slots) slave
replicates d9bf2ac8eec5637ed7ec50061419ff6b951eef0b
M: 6e0353e92377a71d691a853152673a8774d11dc2 192.168.88.101:7002
slots:[5461-10922] (5462 slots) master
1 additional replica(s)
M: d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 192.168.88.102:7001
slots:[10923-16383] (5461 slots) master
1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

#### 7.4.2 启动关闭集群

> 编写脚本，方便启动和关闭Redis集群：redis-cluster-start.sh和redis-cluster-stop.sh。
- 进入Redis安装目录中bin目录，创建脚本文件

```
cd /export/server/redis-5.0.8-bin/bin/

touch redis-cluster-start.sh
touch redis-cluster-stop.sh

# 给以执行权限
chmod u+x redis-cluster-start.sh
chmod u+x redis-cluster-stop.sh
```

- 启动集群：redis-cluster-start.sh

```
vim redis-cluster-start.sh
#!/bin/bash

REDIS_HOME=/export/server/redis
# Start Server
## node1.xiaoma.cn
ssh node1.xiaoma.cn "${REDIS_HOME}/bin/redis-server /export/server/redis/7001/redis_7001.conf"
ssh node1.xiaoma.cn "${REDIS_HOME}/bin/redis-server /export/server/redis/7002/redis_7002.conf"
## node02
ssh node2.xiaoma.cn "${REDIS_HOME}/bin/redis-server /export/server/redis/7001/redis_7001.conf"
ssh node2.xiaoma.cn "${REDIS_HOME}/bin/redis-server /export/server/redis/7002/redis_7002.conf"
## node03
ssh node3.xiaoma.cn "${REDIS_HOME}/bin/redis-server /export/server/redis/7001/redis_7001.conf"
ssh node3.xiaoma.cn "${REDIS_HOME}/bin/redis-server /export/server/redis/7002/redis_7002.conf"
```

- 关闭集群：redis-cluster-stop.sh

```
vim redis-cluster-stop.sh
#!/bin/bash

REDIS_HOME=/export/server/redis
# Stop Server
## node01
${REDIS_HOME}/bin/redis-cli -h node1.xiaoma.cn -p 7001 SHUTDOWN
${REDIS_HOME}/bin/redis-cli -h node1.xiaoma.cn -p 7002 SHUTDOWN
## node02
${REDIS_HOME}/bin/redis-cli -h node2.xiaoma.cn -p 7001 SHUTDOWN
${REDIS_HOME}/bin/redis-cli -h node2.xiaoma.cn -p 7002 SHUTDOWN
## node03
${REDIS_HOME}/bin/redis-cli -h node3.xiaoma.cn -p 7001 SHUTDOWN
${REDIS_HOME}/bin/redis-cli -h node3.xiaoma.cn -p 7002 SHUTDOWN
```

#### 7.4.3 测试集群

> 在任意一台机器，使用redis-cli客户端命令连接Redis服务：

```
redis-cli -c -p 7001
```

> 输入命令：cluster nodes（查看集群信息）和info replication（主从信息）：

```
127.0.0.1:7001> cluster nodes
c038b9e271f5b5dba47d6ef81c4f49548a9f3698 192.168.88.102:7002@17002 slave 6e0353e92377a71d691a853152673a8774d11dc2 0 1598239340178 6 connected
b8a76df88aafb19ce38232d0b4c1daf12370f257 192.168.88.101:7001@17001 slave 97e1d381d59561e075ac813e2df7fed00114687e 0 1598239338161 5 connected
b679ac2df0df7509ffd3a1d3b460cb9e13f9dbfa 192.168.88.100:7002@17002 slave d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 0 1598239337000 4 connected
6e0353e92377a71d691a853152673a8774d11dc2 192.168.88.101:7002@17002 master - 0 1598239338000 2 connected 5461-10922
d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 192.168.88.102:7001@17001 master - 0 1598239339170 3 connected 10923-16383
97e1d381d59561e075ac813e2df7fed00114687e 192.168.88.100:7001@17001 myself,master - 0 1598239338000 1 connected 0-5460

127.0.0.1:7001> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.88.101,port=7001,state=online,offset=840,lag=1
master_replid:e669abc55325a3ea3f4273aeed92c3370568fb34
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:854
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:854
```

> 测试数据，设置Key值和查询Key的值。

```
# node1(192.168.88.153:7001)，redis-cli登录
127.0.0.1:7001> KEYS *
(empty list or set)
127.0.0.1:7001> set k1 v1
-> Redirected to slot [12706] located at 192.168.88.102:7001      # 自动定向到node03上主服务
OK
192.168.88.102:7001> set k2 v2
-> -> Redirected to slot [449] located at 192.168.88.100:7001		  # 自动定向到node02上主服务
OK
192.168.88.100:7001> set k3 v3
OK
192.168.88.100:7001> get k1
-> Redirected to slot [12706] located at 192.168.88.102:7001	 # 自动定向到node03上主服务
"v1"
192.168.88.102:7001> get k2
-> Redirected to slot [449] located at 192.168.88.100:7001		# 自动定向到node03上主服务
"v2"
192.168.88.102:7001> get k3
"v3"
192.168.88.100:7001> KEYS *
1) "k3"
2) "k2"
```

#### 7.4.4 主从切换

> 测试Redis Cluster中主从服务切换，首先查看集群各个服务状态：

![主从切换](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/20.png)

在node3上将7001端口Redis 服务关掉：SHUTDOWN
../redis-cli -h node3.xiaoma.cn -p 7001 SHUTDOWN

```
[root@node3 server]# ps aux | grep redis
root     112830  0.1  0.3 161164 14084 ?        Ssl  11:32   0:00 /export/server/redis/bin/redis-server 0.0.0.0:7002 [cluster]
root     112993  0.0  0.0 112812   972 pts/1    S+   11:36   0:00 grep --color=auto redis
```

再次查看集群状态信息：

![再次查看集群状态信息](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/21.png)

重新启动node03上7001端口Redis服务，查看集群状态信息

```
127.0.0.1:7001> cluster nodes
c038b9e271f5b5dba47d6ef81c4f49548a9f3698 192.168.88.102:7002@17002 slave 6e0353e92377a71d691a853152673a8774d11dc2 0 1598240006108 6 connected
6e0353e92377a71d691a853152673a8774d11dc2 192.168.88.101:7002@17002 master - 0 1598240004000 2 connected 5461-10922
d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 192.168.88.102:7001@17001 master - 0 1598240007115 3 connected 10923-16383
97e1d381d59561e075ac813e2df7fed00114687e 192.168.88.100:7001@17001 myself,master - 0 1598240004000 1 connected 0-5460
b8a76df88aafb19ce38232d0b4c1daf12370f257 192.168.88.101:7001@17001 slave 97e1d381d59561e075ac813e2df7fed00114687e 0 1598240006000 5 connected
b679ac2df0df7509ffd3a1d3b460cb9e13f9dbfa 192.168.88.100:7002@17002 slave d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 0 1598240004093 4 connected
127.0.0.1:7001> redis-cli –h node3.xiaoma.cn –p 7001 SHUTDOWN

127.0.0.1:7001> cluster nodes
c038b9e271f5b5dba47d6ef81c4f49548a9f3698 192.168.88.102:7002@17002 slave 6e0353e92377a71d691a853152673a8774d11dc2 0 1598240223000 6 connected
6e0353e92377a71d691a853152673a8774d11dc2 192.168.88.101:7002@17002 master - 0 1598240224000 2 connected 5461-10922
d9bf2ac8eec5637ed7ec50061419ff6b951eef0b 192.168.88.102:7001@17001 master,fail - 1598240171708 1598240170299 3 disconnected
97e1d381d59561e075ac813e2df7fed00114687e 192.168.88.100:7001@17001 myself,master - 0 1598240224000 1 connected 0-5460
b8a76df88aafb19ce38232d0b4c1daf12370f257 192.168.88.101:7001@17001 slave 97e1d381d59561e075ac813e2df7fed00114687e 0 1598240224734 5 connected
b679ac2df0df7509ffd3a1d3b460cb9e13f9dbfa 192.168.88.100:7002@17002 master - 0 1598240223000 7 connected 10923-16383
```

### 7.5 Redis Cluster 管理

> redis-cli集群命令帮助：

```
[root@node01 ~]# redis-cli --cluster help
Cluster Manager Commands:
  create         host1:port1 ... hostN:portN
                 --cluster-replicas <arg>
  check          host:port
                 --cluster-search-multiple-owners
  info           host:port
  fix            host:port
                 --cluster-search-multiple-owners
  reshard        host:port
                 --cluster-from <arg>
                 --cluster-to <arg>
                 --cluster-slots <arg>
                 --cluster-yes
                 --cluster-timeout <arg>
                 --cluster-pipeline <arg>
                 --cluster-replace
  rebalance      host:port
                 --cluster-weight <node1=w1...nodeN=wN>
                 --cluster-use-empty-masters
                 --cluster-timeout <arg>
                 --cluster-simulate
                 --cluster-pipeline <arg>
                 --cluster-threshold <arg>
                 --cluster-replace
  add-node       new_host:new_port existing_host:existing_port
                 --cluster-slave
                 --cluster-master-id <arg>
  del-node       host:port node_id
  call           host:port command arg arg .. arg
  set-timeout    host:port milliseconds
  import         host:port
                 --cluster-from <arg>
                 --cluster-copy
                 --cluster-replace
  help           

For check, fix, reshard, del-node, set-timeout you can specify the host and port of any working node in the cluster.
```

> 在实际项目中可能由于Redis Cluster中节点宕机或者增加新节点，需要操作命令管理，主要操作如下。

![操作命令管理](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/22.png)

### 7.6 JavaAPI操作redis集群

> 连接Redis集群，需要使用JedisCluster来获取Redis连接。

> 实现步骤：
1. 在cn.xiaoma.redis.api_test包下创建一个新的类：RedisClusterTest
2. 创建一个HashSet<HostAndPort>，用于保存集群中所有节点的机器名和端口号
3. 创建JedisPoolConfig对象，用于配置Redis连接池配置
4. 创建JedisCluster对象
5. 使用JedisCluster对象设置一个key，然后获取key对应的值

```
public class RedisClusterTest {
    private JedisPoolConfig jedisPoolConfig;
    private JedisCluster jedisCluster;

    @BeforeTest
    public void beforeTest() {
        // 1. 创建一个HashSet<HostAndPort>，用于保存集群中所有节点的机器名和端口号
        HashSet<HostAndPort> hostAndPortSet = new HashSet<>();
        hostAndPortSet.add(new HostAndPort("node1.xiaoma.cn", 7001));
        hostAndPortSet.add(new HostAndPort("node1.xiaoma.cn", 7002));
        hostAndPortSet.add(new HostAndPort("node2.xiaoma.cn", 7001));
        hostAndPortSet.add(new HostAndPort("node2.xiaoma.cn", 7002));
        hostAndPortSet.add(new HostAndPort("node3.xiaoma.cn", 7001));
        hostAndPortSet.add(new HostAndPort("node3.xiaoma.cn", 7002));

        // 2. 创建JedisPoolConfig对象，用于配置Redis连接池配置
        jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxIdle(10);
        jedisPoolConfig.setMinIdle(5);
        jedisPoolConfig.setMaxWaitMillis(5000);
        jedisPoolConfig.setMaxTotal(50);

        // 3. 创建JedisCluster对象
        jedisCluster = new JedisCluster(hostAndPortSet);
    }

    @Test
    public void clusterOpTest() {
        // 设置一个key
        jedisCluster.set("pv", "1");

        // 获取key
        System.out.println(jedisCluster.get("pv"));
    }

    @AfterTest
    public void afterTest() {
        try {
            jedisCluster.close();
        } catch (IOException e) {
            System.out.println("关闭Cluster集群连接失败！");
            e.printStackTrace();
        }
    }
}
```

## 8 Redis面试题

> 在应用程序和MySQL数据库中建立一个中间层：Redis缓存，通过Redis缓存可以有效减少查询数据库的时间消耗，但是引入redis又有可能出现`缓存穿透`、`缓存击穿`、`缓存雪崩`等问题。

![Redis面试题](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/23.png)

### 8.1 缓存穿透

`缓存穿透`：key对应的数据在`数据源并不存在`，每次针对此key的请求从`缓存获取不到`，请求都会到数据源，从而可能压垮数据源。
`一言以蔽之：查询Key，缓存和数据源都没有，频繁查询数据源`

比如用一个不存在的用户id获取用户信息，无论论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

- 如何解决缓存穿透：当查询不存在时，也将结果保存在缓存中。[PS：布隆过滤器虽快，但不能准确判断key值是否已存在，不推荐]

### 8.2 缓存击穿

`缓存击穿`：key对应的`数据库存在`，但在`redis中过期`，此时若有`大量并发请求`过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

`一言以蔽之：查询Key，缓存过期，大量并发，频繁查询数据源`

业界比较常用的做法：`使用互斥锁`。简单地来说，就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db（查询数据库），而是先使用Redis的SETNX操作去set一个mutex key【此key作为互斥锁，在指定的 key 不存在时，为key设置指定的值，返回1；`key存在时返回0`】，只让一个线程构建缓存，其他线程等待构建缓存的线程执行完，重新从缓存获取数据。

```
String get(String key) {  
   String value = redis.get(key);  
   if (value  == null) {  
// 如果key不存在，则设置为1
    if (redis.setnx(key_mutex, "1")) {  
        // 设置key的过期时间为3分钟  
        redis.expire(key_mutex, 3 * 60)  
// 从db中加载数据，但注意：只有一个线程能进入到这里，其他线程访问的时候已有课key_mutex
        value = db.get(key);  
// 从数据库中加载成功，则设置对应的数据
        redis.set(key, value);  
        redis.delete(key_mutex);  
    } else {  
        //其他线程休息50毫秒后重试  
        Thread.sleep(50);  
        get(key);  
    }  
  }  
}
```

### 8.3 缓存雪崩

`缓存雪崩`：当`缓存服务器重启`或者大量缓存集中在`某一个时间段失效`，这样在失效的时候，也会给后端系统(比如DB)带来很大压力。

`一言以蔽之：缓存不可用（服务器重启或缓存失效），频繁查询数据源`

与缓存击穿的区别在于这里针对很多key缓存，前者则是某一个key。<br/>
缓存正常从Redis中获取，示意图如下：

![缓存雪崩](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/24.png)

缓存失效瞬间示意图如下：

![缓存雪崩](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/25.png)

> 缓存失效时的雪崩效应对底层系统的冲击非常可怕！

> 大多数系统设计者考虑用`加锁或者队列`的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。

> 还有一个简单方案就时`将缓存失效时间分散开`，比如可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

### 8.4 Redis的命名规范是？

> 使用统一的命名规范

- 一般使用业务名(或数据库名)为前缀，用冒号分隔，例如，业务名:表名:id。
- 例如：shop:usr:msg_code（电商:用户:验证码）

> 控制key名称的长度，不要使用过长的key
 
- 在保证语义清晰的情况下，尽量减少Key的长度。有些常用单词可使用缩写，例如，user缩写为u，messages缩写为msg。

> 名称中不要包含特殊字符

- 包含空格、单双引号以及其他转义字符

### 8.5 集群

> 问题一：Redis的多数据库机制，了解多少

![Redis的多数据库机制](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/26.png)

> 问题二：懂Redis的批量操作么？

![Redis的批量操作](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/27.png)

> 问题三：Redis集群机制中，你觉得有什么不足的地方吗？

![Redis集群机制中的不足](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/28.png)

> 问题四：在Redis集群模式下，如何进行批量操作？

![在Redis集群模式下批量操作](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/29.png)

> 问题五：懂Redis事务么？

![懂Redis事务么](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/大数据_Redis/30.png)

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)

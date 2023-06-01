---
title: HashMap
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(74).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(74).jpg
tags:
  - Java
  - HashMap
category:
  - - 编程
    - Java
date: 2021-03-23 12:10:11
---

`哈希表`（hash table） 也叫`散列表`，是一种非常重要的数据结构。

应用场景及其丰富，许多`缓存技术`（比如memcached）的核心其实就是在内存中维护一张大的哈希表，

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

> 我画的`visio原图`链接，有兴趣的同学可以在这个图上自己改

```
链接：https://pan.baidu.com/s/12Rx9pp2kBje2E46eayuRfw 
提取码：6666 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

## 1 基础

### 1.1 数组、链表和散列表

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_HashMap/1.png)

#### 1.1.1 数组

> `优势`：内存连续，查询效率高

> `劣势`：插入效率低。大小固定，不可动态存储。

#### 1.1.2 链表

> `优势`：插入效率高

> `劣势`：查询效率低

#### 1.1.3 散列表

> `散列表：数组+链表`

### 1.2 什么是哈希

> Hash也称散列、哈希。`基本原理是把任意长度的输入，通过Hash算法变成固定长度的输出.`
>> 这个映射的规则就是`哈希算法`，原始数据映射后的`二进制串`就是`哈希值`.

#### 1.2.1 哈希的特点

> 1.从hash值`不可以反向推出`原始数据

> 2.输入数据的`微小变化`会得到完全不同的hash值，相同的数据会得到相同的hash值。

> 3.hash算法的执行效率要`高效`，长的文本也要快速算出哈希值。

> 4.hash算法的`冲突概率`要小。

## 2 HashMap原理分析

### 2.1 HashMap存储原理

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_HashMap/2.png)

### 2.2 PutVal方法原理

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_HashMap/3.png)

### 2.3 Resize方法原理

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_HashMap/4.png)

## 3 手撕源码

### 3.1 常量

```
// 缺省数组容量
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

// 最大数组容量
static final int MAXIMUM_CAPACITY = 1 << 30;

// 缺省负载因子:(数组元素个数达到 容量*负载因子 时扩容)
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 树化阈值
static final int TREEIFY_THRESHOLD = 8;

// 链化阈值
static final int UNTREEIFY_THRESHOLD = 6;

// HashMap中的个数达到64，并且链表长度达到8，才进行树化。(两个条件缺一不可)
static final int MIN_TREEIFY_CAPACITY = 64;
```

### 3.2 变量

```
// 整个哈希表，第一次put的时候才初始化(懒加载)
transient Node<K,V>[] table;

transient Set<Map.Entry<K,V>> entrySet;

//当前哈希表中的元素个数
transient int size;

//结构修改次数(插入、删除会引发结构变化，修改不会)
transient int modCount;

// 扩容阈值(哈希表中的元素超过这个时触发扩容，(capacity * load factor))
int threshold;

// 负载因子
final float loadFactor;
```

### 3.3 put方法

#### 3.3.1 put方法

```
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

#### 3.3.2 hash方法

> 可以看到key为null的值都放在数组索引0的位置，也就是`HashMap可以存key为null的数据`，但是这个数据只有一个。

```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

#### 3.3.3 putVal方法

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    // tab:引用当前hashMap的散列表
    // p:当前散列表的元素
    // n:散列表数组的长度
    // i:路由寻址结果
    Node<K,V>[] tab; Node<K,V> p; int n, i;   
    // 延迟初始化逻辑，第一次调用putVal时会初始化hashMap
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 最简单的一种情况，寻址找到的桶位，刚好是null，这个时候，直接将k-v=>node 扔进去就可以了
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        // e: 不为null的话，找到了一个与当前要插入的key一致的元素
        // k: 表示临时的一个key
        Node<K,V> e; K k;
        // 桶位中的该元素与要插入的元素的key完全一直，后续直接进行替换值的操作
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;        
        // 桶位已经树化
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);        
        // 桶位还是链表
        else {
            // 遍历这个链表
            for (int binCount = 0; ; ++binCount) {            
                // 遍历的这个元素已经是链表的最后一个元素，也没有找到与要插入的元素key相同的node
                if ((e = p.next) == null) {                
                    //将元素插入到链表最后面
                    p.next = newNode(hash, key, value, null);                    
                    //元素个数达到树化阈值，进行树化
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st                    
                        // 这里面会判断整个哈希表元素个数时候达到64(前面变量里的)，所以树化条件有两个。默认链表8个，哈希表64个，缺一不可
                        treeifyBin(tab, hash);
                    break;
                }                
                // 找到了一个与插入的元素的key相同的node
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }        
        // e不等于null，说明找到了与插入元素key相同的元素，替换值
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    
    ++modCount;
    //插入新元素，size自增，如果自增后的值，大于扩容阈值，则触发扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

### 3.4 resize扩容方法

```
final Node<K,V>[] resize() {
    // oldTab:引用扩容前的哈希表
    Node<K,V>[] oldTab = table;
    // oldCap:扩容之前table数组的长度
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // oldThr:扩容之前的扩容阈值，触发本次扩容的阈值
    int oldThr = threshold;
    // newCap:扩容之后数组的大小
    // newThr:扩容之后，下次再次触发扩容的阈值
    int newCap, newThr = 0;
    // 条件如果成立，说明hashMap中的散列表已经初始化过了，是一次正常扩容
    if (oldCap > 0) {
        // 扩容之前的table数组大小已经达到最大阈值后，则不扩容，且设置扩容条件为int的最大值。
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 这里oldCap<<1很关键，是说扩容后的容量是扩容前的2倍， newCap小于数组最大值限制 且 扩容之前的阈值>=16
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 新的扩容阈值翻倍
            newThr = oldThr << 1; // double threshold
    }
    // oldCap == 0，说明hashMap中的散列表还没有初始化，是null
    // 1.new HashMap(initCap,loadFactor);
    // 2.new HashMap(initCap);
    // 3.new HashMap(map);//并且这个map有数据
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    // oldCap == 0，说明hashMap中的散列表还没有初始化，是null
    //new HashMap();
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // newThr为0时，通过newCap和loadFactor计算出一个newThr
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    //创建出一个更大，更长的数组
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 说明HashMap本次扩容之前，table不为null
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            // 当前遍历的Node节点
            Node<K,V> e;
            // 说明数组的中头节点不为空，至于里面是单个数据还是链表还是红黑树并不知道
            if ((e = oldTab[j]) != null) {
                // 置空，方便JVM GC时回收内存
                oldTab[j] = null;
                // 第一种情况：说明数组的这个桶中是单个元素，没有碰撞，这种情况直接计算出当前元素在新数组中的位置，扔进去就好了
                if (e.next == null)
                    // 寻址put数据即可
                    newTab[e.hash & (newCap - 1)] = e;
                // 第二种情况：说明这个桶位是个红黑树
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // 第三种情况：说明这个桶位是个链表
                else { // preserve order
                    // 低位链表：存放在扩容之后的数组的下标位置，与当前数组的下标一致
                    Node<K,V> loHead = null, loTail = null;
                    // 高位链表：存放在扩容之后的数组的下标位置，当前数组下标位置 + 扩容之前数组的长度
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    //遍历这个链表，直到某个元素的next指向null(最后一个元素)
                    do {
                        next = e.next;
                        // 以15桶为例： 
                        //      hash -> ....1 1111
                        //      hash -> ....0 1111
                        
                        // 成立说明放在低位链中
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 否则放在高位链中
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 低位链表有数据
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 高位链有数据
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

## 4 为什么选用红黑树

> 红黑树是一种近乎平衡的二叉查询树

> 二叉查询树最差的时间复杂度是0(n)，也就是数据是一条链的时候，这个肯定不适合
> 二叉平衡树的话，查询性能很好 0(logn)，但是几乎每次插入数据，都会破坏平衡性，从而产生自平衡操作，很浪费性能。
> 红黑树是二叉平衡树的变体，保证了近乎平衡的同时，插入数据不会频繁影响平衡性，不会频繁的进行自平衡操作，不像二叉平衡树那样。

## 5 HashMap 为什么是线程不安全的

### 5.1 插入数据的时候

> AB线程同时插入，两个计算除了相同的哈希值，对应数组的相同位置，第一个线程写了数据后，第二个线程再写，不就覆盖了。

### 5.2 扩容的时候

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_HashMap/5.png)

> 第一步：线程2已经将1和2元素弄好了，这时候挂起

> 第二步：线程1将弄好。

> 第三步：线程2找到2节点的next，发现是1，然后把1插进去，就是1指向2，如图中红色箭头，死循环了。

### 5.3 ......

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

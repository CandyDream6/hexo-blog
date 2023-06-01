---
title: 树先生
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(75).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(75).jpg
tags:
  - Java
  - Tree
category:
  - - 编程
    - 数据结构
date: 2021-03-23 18:51:27
---

树先生是一类数据存储结构，应用十分广泛，慢慢总结下。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

> `慢慢更新，一点一点来..........`

> 树可视化网站推荐，美的一批

```
https://www.cs.usfca.edu/~galles/visualization/Algorithms.html
```

> 我画的`visio原图`链接，有兴趣的同学可以在这个图上自己改

```
链接：https://pan.baidu.com/s/1FgXyi1PR7kCkXZWuNvCVuQ 
提取码：6666 
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

## 1 二叉树

> `满二叉树`：除最后一层无任何子节点外，每一层上的所有结点都有两个子结点。

> `完全二叉树`：若设二叉树的深度为h，除第 h 层外，其它各层 (1～(h-1)层) 的结点数都达到最大个数，第h层所有的结点都连续集中在最左边，这就是完全二叉树。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_Tree/1.png)

## 2 二叉查找树(二叉排序树)

> 左节点/左子树小于父节点，右节点/右子树大于父节点

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_Tree/2.png)

## 3 平衡二叉树

### 3.1 平衡二叉树之AVL树

> `具有二叉查找树的所有特性`
> `每个节点的左子树和右子树的高度至多等于1`

> 好处：保证了不会出现数据`倾斜`

> `为什么出现了AVL树还要出现红黑树`
>> 因为AVL树要求严格(左右子树高度差<=1)，导致在插入或者删除的时候，几乎每次都会破坏树的平衡，进而需要进行左旋和右旋操作使之成为一个合格的AVL树。
>> 在那种插入、删除很频繁的场景中，AVL树需要频繁左旋右旋调整，这样性能就大打折扣，为了解决这个问题，于是有了红黑树。

### 3.2 平衡二叉树之红黑树

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_Tree/3.png)

#### 3.2.1 红黑树性质

> 1.每个节点要么是`黑色`，要么是`红色`

> 2.根节点是`黑色`

> 3.每个叶子节点(NIL)是`黑色`

> 4.每个`红色`节点的两个子节点一定是`黑色`(不能有两个红色节点相连)

> 5.任意一节点到每个叶子节点的路径都包含`数量相同`的`黑节点`，俗称：`黑高`。所以红黑树这种平衡称为`黑色完美平衡`

> `注意：插入的数据肯定是红色，因为如果是黑色，那么绝对破坏了红黑树的黑高原则。`

#### 3.2.2 红黑树自平衡：左旋、右旋、变色

> 1.`变色`：颜色由黑变红，由红变黑

> 2.`左旋`：以某个节点作为支点(旋转节点)，其右子节点变为旋转节点的父节点，右子节点的左子节点变为旋转节点的右子节点，左子节点保持不变

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_Tree/左旋.gif)

> 3.`右旋`：以某个节点作为支点(旋转节点)，其左子节点变为旋转节点的父节点，左子节点的右子节点变为旋转节点的左子节点，右子节点保持不变

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Java_Tree/右旋.gif)

## 4 B树

## 5 B+树

## 6 B*树

## 7 Trie树

## 8 时间复杂度

> `二叉查找树`
>> 最好：O(logn)
> 
>> 最坏：O(n)   （极端情况：就和一个链表一样）
>
> `二叉平衡树`
>> O(logn)
> 
> `红黑树`
>> O(logn)

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

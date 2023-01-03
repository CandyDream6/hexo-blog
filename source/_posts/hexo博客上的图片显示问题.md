---
title: hexo博客上的图片显示问题
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(2).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(2).jpg
tags:
  - 博客
  - hexo
  - 图片
category:
    - [博客, 技术]
 
date: 2018-08-03 10:17:35
---

在搭建博客的过程中，遇到的一个贴别蛋疼的事情就是图片加载不出来。

## 解决过程中踩坑

当时花了我一个下午都没有搞出来，网上去搜的话，搜出来99篇文章，98篇连错别字都是一样的。

这些人挺气人的。最后发现是网上所有人提供的包版本有问题。

下面我就给大家说下走得通的路。

## 解决方式

1. 在_config.yml配置文件中将post_asset_folder设置为true，默认是false

``` bash
$ post_asset_folder: true
```

2. 执行下面第一个命令，大家几乎全在这一步死掉，是因为你从网上看到的是下面第二个命令，这个包是有点问题的

``` bash
$ npm install https://github.com/CodeFalling/hexo-asset-image    安装这个
$ 
$ npm install hexo-asset-image --save                            不要安装这个(坑死个人哩)
```

3. 当你用下面命令新建一个笔记时

``` bash
$ hexo new post myFirstBlog
```

会在source->_posts文件夹下，创建myFirstBlog.md文件，同时会生产一个同名文件夹。

我们把文章里需要用到的图片放到这个文件夹下。

4. 图片引用

在你的笔记里这样引入图片，比如我传了一个hotGirl.png到myFirstBlog文件夹下。

``` bash
![可以为空] (./myFirstBlog/hotGirl.png)
```

我也试过比如myFirstBlog/hotGirl.png等等，但是经常就不起作用，所以还是老老实实用上面这一种就好。

## 完工

所有的坑其实就是那个包的问题，你有没有被这个问题搞过的经历呢。

---

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)
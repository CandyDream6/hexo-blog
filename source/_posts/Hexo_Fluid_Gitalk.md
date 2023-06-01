---
title: Hexo_Fluid_Gitalk 评论系统添加
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(49).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(49).jpg
tags:
    - 博客
    - hexo
    - Gitalk
    - 评论
category:
    - [博客, 技术]
 
date: 2019-12-03 12:25:51
---

最近有朋友让出个使用`Gitalk存储博客评论`的教程，就写一下。

博客的`评论系统`有很多，种种尝试后，我最终还是选择采用`Gitalk`。

Gitalk原理是用`Github`的仓库`Issues`来实现`评论存储功能`。

下面，就让我们`一起开始`吧！

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 创建评论仓库

> 这个仓库不创建也可以，我是为了把博客和评论分离开来，创建一个仓库专门用来`存储博客中的评论`。他是用Github仓库的`Issues`来实现评论存储的。

> 注意：这个仓库`必须是公开的`。

## 2 申请Github应用

### 2.1 [申请地址](https://github.com/settings/applications/new)

```
https://github.com/settings/applications/new
```
### 2.2 申请应用

![申请应用](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Hexo_Fluid_Gitalk/1.png)

> 申请完成后看到下面页面，把`Client id`和`Client secrets`填到下面配置中

![申请应用成功界面](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Hexo_Fluid_Gitalk/2.png)

> 如果不小心关了这个界面，没关系，去[下面地址](https://github.com/settings/developers)找到自己刚创建的app，重新生成一下secrets就好了

```
https://github.com/settings/developers
```

![重新生成Client secrets](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Hexo_Fluid_Gitalk/3.png)  

## 3 主题开启Gitalk评论

```
post:
  comments:
    enable: true
    # 使用Gitalk作为评论系统
    type: gitalk   
gitalk:
  clientID: '********************'                           # 创建应用后的 Client id
  clientSecret: '********************'                       # 创建应用后的 Client secrets
  repo: 'hexoBlogGitalk'                                     # 评论用的仓库名称
  owner: 'xiaoma55'                                          # 填自己github账户名字就好
  admin: ['xiaoma55']                                        # 填自己github账户名字就好
  language: zh-CN
  labels: ['Gitalk']
  perPage: 10
  pagerDirection: first
  distractionFreeMode: true
  createIssueManually: true
  proxy: 'https://cors-anywhere.azm.workers.dev/https://github.com/login/oauth/access_token'
```

## 4 完工

![图片](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Hexo_Fluid_Gitalk/4.png)

`开启你的评论之旅吧！！！`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
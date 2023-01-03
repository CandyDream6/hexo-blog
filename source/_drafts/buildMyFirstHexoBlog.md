---
title: 搭建一个属于自己的博客
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(1).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(1).jpg
tags:
  - 博客
  - hexo
category:
    - [博客, 技术] 
comment: 'on'
date: 2020-11-13 13:56:54
---
博客，是一种灵魂，记录灵感，记录生活，记录青春，记录我这苦难而又平凡的一生。

搭建个人博客的方式有很多，比如wordPress，hexo，hugo等等，我这里用的是hexo(我们公司架构师推荐的)

## 博客搭建

[hexo官网文档](https://hexo.io/zh-cn/docs/index.html)
> 快速、简洁且高效的博客框架

按着官网文档操作基本就OK了，就是安装git，node，hexo这三步骤。

安装好之后就是初始化了

``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```
完成后目录结构基本如下

``` bash
.
├── _config.yml        是很重要的配置文件，很多东西都在这配置(比如博客站点名，主题，发布仓库等等)
├── package.json    
├── scaffolds          这个文件夹是模板文件夹，就是用命令新建笔记的时候的模板，下面说怎么用命令新建笔记
|   ├── _draft.md      草稿模板，用这个模板写的笔记，在source->draft中，发布后不会显示
|   └── _post.md       文章模板，用这个模板写的笔记，在source->post中，发布后不会显示
|   └── _page.md       页面模板，很少用
├── source
|   ├── _drafts        这个就是对应上面的draft，里面放的是草稿，也就是发布后不会显示的
|   └── _posts         这个就是对应上面的post，里面放的是文章，也就是发布后会显示的
└── themes             主题，默认有一个主题，也可以自己从网上下载自己喜欢的主题，拷贝刀这里就可以

``` 

这个时候执行下面命令任一个，访问 http://localhost:4000/ 就可以看到自己的博客了。
``` bash
$ hexo s
$ hexo server
```

## 写作
``` bash
$ hexo new [layout] <title>

如:  hexo new post 我的第一个博客
     hexo new draft 我的第一个草稿
```

## 部署

上面的步骤在hexo官网都可以看到，我就不多写了，主要说一下部署这块需要注意点的东西

### 仓库设置
注意deploy下面的repo填仓库的git地址：

``` bash
deploy:
  type: git
  repo: https://github.com/xiaoma55/xiaoma55.github.io.git
  branch: master
```

这个时候访问是通过 <你的 GitHub 用户名>.github.io 去访问的，因为仓库名设置成了github账户名

我们可以绑定一个自己的域名

## 绑定域名

### 购买域名
自己去购买一个域名，完成认证

### 域名解析
接下下面这两个地址

![域名解析](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/jieXiYuMing.png)

## github仓库设置

![仓库设置](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/githubSetting.png)

## 大功告成

![大功告成](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/boKeZhuYeTu.png)


## 后续维护

以后写了写笔记，执行下面命令就可以发布成功
``` bash
$ hexo clean
$ hexo g   
$ hexo d
```
要是本地预览
``` bash
$ hexo s
```
---

## 关注博主不迷路
![联系博主](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/icon/wechatFindMeNew.png)


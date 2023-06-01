---
title: vercel部署node版本问题解决
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(123).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(123).jpg
tags:
  - 大数据
  - vercel
category:
  - - 编程
    - vercel

date: 2022-10-26 16:21:48
---

有段时间没写博客了，在部署的时候遇到了一个问题，就是`node`版本问题，今天就来介绍一下`vercel`部署`node`版本问题的解决方法，方便大家去参考一下。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 事故现场

vercel在同步github代码部署的时候，部署失败，日志如下：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/vercel部署node版本问题/1.png)

## 2 解决方案

> 先查看本地node版本，保证在16以上，不是的话更新一下。

> 然后在项目package.json中配置一下node版本指定，如下：

```shell
{
  "engines": {
    "node": ">=16"
  }
}
```

## 3 撒花

问题不大，但很烦人，其他人应该也会碰到，是vercel版本的更新导致的，希望能帮到大家。

## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
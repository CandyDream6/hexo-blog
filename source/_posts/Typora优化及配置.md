---
title: Typora优化及配置
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(120).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(120).jpg
tags:
  - 编程
  - 工具
category:
  - - 编程
    - 工具
  - - 编程
    - Markdown
date: 2022-07-21 09:33:49
---

我一直认为：不是markdown的笔记，都不能叫做笔记。

所以，今天来记录下我自己关于Typora的一些配置及优化。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)


## 1 主题配置

> 以前我一直用的是vue那款主题，比较清新。

> 然后最近发现了一款个人非常喜欢的主题，叫`Forset(森林主题)`。

> 喜欢的原因主题是他采用了Jetbrains的经典字体`Jetbrains Mono`，以及整体风格比较舒适。

> 下面是他的官网地址，自己看着去配置就行。

https://github.com/sylviaxgj/typora-forest-theme

## 2 配置图片双击放大

> 我们都知道Typora中，`图片是不能放大查看的`，这就导致某些场景下`图片看不清楚`。接下来就配置一下

### 2.1 下载LightBox

https://github.com/lokesh/lightbox2

### 2.2 拷贝文件

> 将dist目录中的文件拷贝到Typora的安装目录的相应位置下。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/1.png)

> 在安装目录的D:\soft\Typora\Typora\resources文件夹下，新建了extend文件夹，用来存放扩展插件的，dist目录下的文件也放到了这。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/2.png)

### 2.3 修改window.html文件

> 位置：D:\soft\Typora\Typora\resources\window.html（这个文件实际就是Typora的主界面）

#### 2.3.1 引入css文件

> 可以搜索`</head>`，在它的前面添加：

```html
<link rel="stylesheet" href="./extend/lightbox2/css/lightbox.min.css" crossorigin="anonymous">
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/3.png)

#### 2.3.2 引入js文件

> 可以搜索`</script><script src="./appsrc/window/frame.js" defer="defer"></script>`，然后在它的后面添加：

```html
<script type="text/javascript" src="./extend/lightbox2/js/lightbox.js" defer="defer"></script>
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/4.png)

### 2.4 修改lightbox.js文件

> 因为lightbox需要有 a 标签包围着 img 标签，相应的点击事件是绑定在 a 标签上的，且需要 href 、 data-lightbox 、 data-alt 等属性，而Typora中的图片只有 img 标签，父级没有 a 标签，也没有相应要求的属性。因此需要改造一下 lightbox.js 文件。
> 
> D:\soft\Typora\Typora\resources\extend\lightbox2\js\lightbox.js

#### 2.4.1 为img标签绑定双击事件

> 修改enable函数，搜索`Lightbox.prototype.enable = function()`，添加代码：

```javascript
//为img绑定双击事件，但要排除本就是双击放大展示的图片
$('body').on('dblclick', "img:not([class='lb-image'])" ,function(event){
    self.start($(event.currentTarget)) ;
    return false;
}) ;
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/5.png)

#### 2.4.2 修改属性值获取

> lightbox用到三个属性：alt、href 、title，这三个属性在img标签中均有对应的。

> 修改start函数，搜索`Lightbox.prototype.start = function($link)`，换成img对应属性：

```javascript
function addToAlbum($link) {
  self.album.push({
    alt: $link.attr('data-alt')|| $link.attr('alt'),
    link: $link.attr('href')|| $link.attr('src'),
    title: $link.attr('data-title') || $link.attr('title')
  });
}
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/6.png)

#### 2.4.3 注释data-lightbox

> 还有另外一个属性data-lightbox，img没有该属性，因此这里不使用，lightbox中有一段判断是否有该属性的代码，修改不存在该属性时的逻辑代码
> 
> 搜索`if (dataLightboxValue)`（注释掉原有的，添加addToAlbum($link);）：

```javascript
addToAlbum($link);
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/7.png)

### 2.5 重启Typora，验证

> typora里的图片可以双击放大了。

## 3 配置腾讯云COS图床

> markdown里默认是不能插入图片的，只能引用本地图片，那我们给别人发的markdown文件就是多个文件或者一个压缩包了，今天配置一下腾讯云COS作为图床。
> 
> 目标效果是：往typora里粘贴或者上传一张图片，会自动上传到cos中。

### 3.1 安装PicGo-Core插件

> 文件->偏好设置->图像->PicGo-Core(command line)->下载或更新。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/8.png)

### 3.2 创建腾讯云对象存储COS

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/9.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/10.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/11.png)

### 3.3 配置API秘钥

#### 3.3.1 获取API密钥

> 秘钥API获取链接： https://console.cloud.tencent.com/cam/capi

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/12.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/13.png)

#### 3.3.2 配置API密钥

> 文件->偏好设置->图像-打开配置文件。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/14.png)

> 在腾讯云找到对应信息填入

```json
{
	"picBed": {
		"current": "tcyun",
		"tcyun": {
			"secretId": "填入上面获取的secretId",
			"secretKey": "填入上面获取的secretKey",
			"bucket": "填入创建的存储桶名称",
			"appId": "",
			"area": "填写创建的存储桶的地域",
			"path": "自定义存储路径，比如 img/ 或者 /img/,如果不填写则会默认存储在根目录",
			"customUrl": "这里填写访问域名，即url地址",
			"version": "v5"
		}
	},
	"picgoPlugins": {}
}
```

> `secretId`，`secretKey`，`appId`在上面的密钥界面
>
> `bucket`，`area`，`customUrl`在存储桶概览页面

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/15.png)

> 填好的示例如下：

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/16.png)

### 3.4 验证图床是否配置成功

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/17.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/18.png)

### 3.5 配置插入图片自动上传

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Typora优化及配置/19.png)

### 3.6 完工

> 这时候再去typora粘贴或者上传图片，就会自动上传到我们刚刚配置的cos存储桶中了。

## 4 Typora基础配置

```
开启自动保存：文件->偏好设置->通用->保存&恢复(勾选)
侧边大纲可折叠：文件->偏好设置->外观->侧边栏的大纲视图允许折叠和展开(勾选)，然后【视图】->【大纲】。
缩进改为4个字符：文件->偏好设置->编辑器->【默认缩进】改为4。
开启代码块不换行： 文件->偏好设置->Markdown->代码块->代码块自动换行(取消勾选)。
显示代码块行号：文件->偏好设置->Markdown->代码块->显示行号(勾选)
待更细......
```

## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
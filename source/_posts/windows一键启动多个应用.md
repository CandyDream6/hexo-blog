---
title: windows一键启动多个应用
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(121).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(121).jpg
tags:
  - 编程
  - 软件
category:
  - - 编程
    - 软件
  - - 编程
    - windows

date: 2022-07-27 17:16:25
---

由于每天工作电脑都得`开很多软件`，我粗略算了下，维持最基本的工作，我得打开`20多个软件`。

加上公司`强制要求每天下班电脑关机`，于是乎每天上班开软件就成了`噩梦`。

于是搞了下`一键打开多个软件`，记录一下。`

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 编写脚本

> 新建脚本`***.bat`, 写入如下内容，其中@Rem表示注释

```
@Rem 1 打开wox
start "" "C:\Users\V\AppData\Local\Wox\Wox.exe"
@Rem 2 打开everything
start "" "C:\Program Files\Everything\Everything.exe"
@Rem 3 打开chrome
start "" "C:\Program Files\Google\Chrome\Application\chrome.exe"
@Rem 4 打开Panda
start "" "D:\soft\Panda\Panda.exe"
@Rem 5 打开微信
start "" "D:\soft\WeChat\WeChat.exe"
@Rem 6 打开钉钉
start "" "D:\soft\DingDing\DingtalkLauncher.exe"
@Rem 7 打开网易邮箱大师
start "" "D:\soft\wangYiMail\MailMaster\Application\mailmaster.exe"
@Rem 8 打开网易有道词典
start "" "D:\soft\YouDaoCiDian\Dict\YoudaoDict.exe"
@Rem 9 打开FSCapture
start "" "D:\soft\FSCapture\FSCapture.exe"
@Rem 10 打开Snipaste
start "" "D:\soft\Snipaste\Snipaste-1.11.0-x86\Snipaste.exe"
@Rem 11 打开TrafficMonitor
start "" "D:\soft\TrafficMonitor\TrafficMonitor.exe"
@Rem 12 打开JetBrains Toolbox
start "" "C:\Users\V\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\JetBrains Toolbox\JetBrains Toolbox.lnk"
@Rem 13 打开IntelliJ IDEA
start "" "C:\Users\V\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\JetBrains Toolbox\IntelliJ IDEA Ultimate.lnk"
@Rem 14 打开DataGrip
start "" "C:\Users\V\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\JetBrains Toolbox\DataGrip.lnk"
@Rem 15 打开PyCharm
start "" "C:\Users\V\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\JetBrains Toolbox\PyCharm Professional.lnk"
@Rem 16 打开WebStorm
start "" "C:\Users\V\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\JetBrains Toolbox\WebStorm.lnk"
@Rem 17 打开HHKB键盘热键
start "" "D:\soft\HHKB-HotKey\HHKB-autoHotKey.ahk"
@Rem 18 打开Telegram
start "" "C:\Users\V\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Telegram Desktop\Telegram.lnk"
@Rem 19 打开VMWare
start "" "D:\application\VMware\vmware.exe"
@Rem 20 打开Xshell
start "" "D:\application\xshell\Xshell.exe"
@Rem 21 打开XFtp
start "" "D:\application\xftp\Xftp.exe"
```

## 2 配置脚本到启动页

> 由于我平时用的wox，所以桌面上什么也没有，于是把这个bat文件想办法搞到开始屏幕，一切就都完美了。

### 2.1 创建快捷方式

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/windows一键启动多个应用/1.png)

```shell
cmd /c "/***/***/****.bat"

cmd /c "批处理文件路径": 此命令运行完批处理后窗口自动关闭;
cmd /k "批处理文件路径""：此命令运行完批处理后窗口不会自动关闭;
```

### 2.2 开启屏幕启动

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/windows一键启动多个应用/2.png)

> 以后就可以一键启动多个应用了，每天早上点一下，在他启动的过程中，去接杯水，或者冲个咖啡，然后开始，写bug。

## Finally 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
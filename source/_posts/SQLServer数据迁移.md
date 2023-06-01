---
title: 'SQLServer数据迁移'
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(85).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(85).jpg
tags:
  - 大数据
  - 数据迁移
category:
  - - 编程
    - 数据迁移
    
date: 2021-05-31 18:10:57
---

最近公司`云服务器磁盘吃紧`，所以决定把云上的`SQLServer数据库迁移到内网`。

`SQLServer` 完全没用过。于是开始查资料，`谷歌`，`百度`，双爸爸齐上手。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 方案了解

> 了解了一通之后，发现SQLServer的数据迁移一般有三种方案。
>> `保存脚本法`
> 
>> `备份法`
> 
>> `物理文件备份法`

## 2 方案对比

### 2.1 方案选择

> 综合对比之后，我们选择`方案二`进行迁移，方案对比往下看。

### 2.2 方案一：保存脚本法

> 特点是`（大数据量时，耗时长，导出的文件大小较大）`

> `数据库`-->`右键`-->`任务`-->`生成脚本`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/SQLServer数据迁移/1.png)

> `完整数据库迁移`选第一个，`部分迁移`（自选）选第二个，点击下一步

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/SQLServer数据迁移/2.png)

> 点`高级`
>> ①：按照需要进行选择，默认为只迁移表结构，此次我选“`择架构和数据`”；
>
>> ②：点击“...”选择存放路径及文件名称，默认存到“文档\script.sql”（忘记修改去文档中找），设置好后点击下一步
>
>> ③：此处根据需要选择，一般数据库小的选择第一个，数据库大的选择第二个。否则导出文件过大，恢复数据时比较麻烦。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/SQLServer数据迁移/3.png)

> 核对下要迁移的数据是否都包含，包含“下一步”，不包含“上一步”回去添加；

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/SQLServer数据迁移/4.png)

> 等待进度完成后点击“完成”。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/SQLServer数据迁移/5.png)

### 2.3 方案二：备份法

> 特点是`大数据量时，耗时短，导出的文件大小与数据库大小相似`

> `数据库`-->`右键`-->`任务`-->`备份`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/SQLServer数据迁移/6.png)

> 方框处是设置`完整备份`或`增量备份`，按需要设置即可；
>
> 点击“`添加`”
>
>> 选择输出地址“...”
>
>> 然后确定就好了。

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/SQLServer数据迁移/7.png)

### 2.4 方案三：物理文件备份法

> `备份时不能使用数据库`

> `数据库`-->`右键`-->`属性`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/SQLServer数据迁移/8.png)

> 点击左上角“`文件`”，查看路径，即为`物理文件mdf存放路径`；

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/SQLServer数据迁移/9.png)

> 打开文件目录，找到物理文件（`注意此时复制并不是正常的，由于数据库物理文件被占用`）；

> 选中要迁移的数据库，右键选择“`任务`-->`分离`”；
>> 由于需要脱机，我这不方便，就不尝试了。
> 
>> 可以脱机的场景下，脱机之后选择mdf备份就可以了。 

## 3 备份定时脚本

> 数据库不单要`备份数据`，还有一些`定时脚本也需要备份`过来。

> 右键一个个太麻烦了，直接用我的脚本吧,把数据库名一个个换掉就好了。

```shell
BACKUP DATABASE [b3d] TO  DISK = N'D:\lankr_powerbi\sqlserver\MSSQL11.MSSQLSERVER\MSSQL\Backup\b3d.bak' WITH NOFORMAT, NOINIT,  NAME = N'b3d-完整 数据库 备份', SKIP, NOREWIND, NOUNLOAD,  STATS = 10
GO

BACKUP DATABASE [baiji_ctong] TO  DISK = N'D:\lankr_powerbi\sqlserver\MSSQL11.MSSQLSERVER\MSSQL\Backup\baiji_ctong.bak' WITH NOFORMAT, NOINIT,  NAME = N'baiji_ctong-完整 数据库 备份', SKIP, NOREWIND, NOUNLOAD,  STATS = 10
GO

BACKUP DATABASE [baiji_pami] TO  DISK = N'D:\lankr_powerbi\sqlserver\MSSQL11.MSSQLSERVER\MSSQL\Backup\baiji_pami.bak' WITH NOFORMAT, NOINIT,  NAME = N'baiji_pami-完整 数据库 备份', SKIP, NOREWIND, NOUNLOAD,  STATS = 10
GO

.....
```

## 4 恢复数据

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/SQLServer数据迁移/10.png)

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/SQLServer数据迁移/11.png)

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

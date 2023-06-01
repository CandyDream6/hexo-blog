---
title: Hive之UDF的编写与部署
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(97).jpg
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(97).jpg
tags:
  - 大数据
  - Hive
category:
  - - 编程
    - Hive
  - - 编程
    - UDF
date: 2021-10-11 19:48:42
---

公司最近在`载数到Hive`时，要做一些`数据处理`，比如`脱敏`，`加解密`什么的，这时候就需要编写`UDF函数`了。

`UDF`(`User-Defined Functions`)：`用户自定义函数`。

Hive 自带了一些函数，比如：max/min等，但是数量有限，自己可以通过`自定义UDF`来方便的扩展。

当Hive提供的`内置函数无法满足`你的业务处理需要时，此时就可以考虑使用`自定义UDF`。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 编写UDF函数

> 1:`继承UDF类`。
> 
> 2:`重写evaluate方法`

```java
package com.hex.hive.udf;

import java.util.UUID;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.hive.ql.exec.UDF;

/**
 * @Description: 自定义UDF函数，获取32位唯一uuid
 * @Package: com.hex.hive.udf
 * @ClassName: UuidUDF
 *
 * @Author: xiaoma
 * @Date: 2021/10/12 9:20
 * @Version: v1.0
 **/
public final class UuidUDF extends UDF {
	private final transient Text result;

	public UuidUDF() {
		this.result = new Text();
	}

	public Text evaluate() {
		String uuid = UUID.randomUUID().toString().replaceAll("-", "");
		this.result.set(uuid);
		return result;
	}
}

```

## 2 打包上传到服务器和HDFS

### 2.1 上传到服务器

> 将刚才的jar包上传到服务器，任意位置(我是上传到`/bigdata/hex-hive-ext/libs`)
>> 如果`开启了用户权限验证`，那么必须传到hive的jar配置目录：Hive集群配置的【`Hive 辅助 JAR 目录`】，英文叫【`Hive Auxiliary JARs Directory`】

### 2.2 上传到HDFS

> 编写shell脚本：`put_hive_udf.sh`,我放在`/bigdata/hex-hive-ext`目录下

```shell
#!/bin/bash
# author: Junjie.M

BIN_HOME=`dirname $0`
cd $BIN_HOME
BIN_HOME=`pwd`

# UDF刚才上传的jar包目录
LOCAL_HIVE_UDF_DIR=/bigdata/hex-hive-ext/libs
# UDF上传到HDFS的目录
HDFS_HIVE_UDF_DIR=/user/hive/lib
# UDF的jar包名称
HIVE_UDF_JAR_NAME=hex-hive-udf-1.0-SNAPSHOT.jar
# UDF的jar包在服务器的全路径
LOCAL_HIVE_UDF_JAR_PATH=$LOCAL_HIVE_UDF_DIR/$HIVE_UDF_JAR_NAME
# UDF的jar包在HDFS的全路径
HDFS_HIVE_UDF_JAR_PATH=$HDFS_HIVE_UDF_DIR/$HIVE_UDF_JAR_NAME


# ==========================在Linux中执行如下语句==========================
# 创建HDFS目录
sudo -u hdfs hadoop fs -mkdir -p $HDFS_HIVE_UDF_DIR
# 删除HDFS中原来的UDF的jar包
sudo -u hdfs hadoop fs -rm $HDFS_HIVE_UDF_JAR_PATH
# 上传新的UDF的jar包
sudo -u hdfs hadoop fs -put $LOCAL_HIVE_UDF_JAR_PATH $HDFS_HIVE_UDF_JAR_PATH
if [ $? -ne 0 ]; then
    echo "===>上传【失败】！"
    echo "===>请检查$LOCAL_HIVE_UDF_JAR_PATH是否存在！！！"
    exit 1
fi
# 添加权限，hive可以访问
sudo -u hdfs hadoop fs -chown -R hive:hive $HDFS_HIVE_UDF_DIR

echo "===>上传【成功】！"
echo "===>如果开启了用户权限验证，需要重启Hive服务，然后才能执行register_hive_udf.sh！！！"

exit 0
```

> 执行shell脚本

```shell
sh /bigdata/hex-hive-ext/put_hive_udf.sh
```

## 3 准备注册sql脚本

> 编写脚本 `register_hive_udf.sql`，我放在`/bigdata/hex-hive-ext`目录下

```shell
show functions 'default.*';

-- sys_uuid() returns string
DROP FUNCTION IF EXISTS default.sys_uuid;
CREATE FUNCTION default.sys_uuid AS 'com.hex.hive.udf.UuidUDF'
USING JAR 'hdfs://hdfsIP:8020/user/hive/lib/hex-hive-udf-1.0-SNAPSHOT.jar';

-- -- sys_id_card(string) returns string
-- DROP FUNCTION IF EXISTS default.sys_id_card;
-- CREATE FUNCTION default.sys_id_card AS 'com.hex.hive.udf.ConvertIDCardUDF'
-- USING JAR 'hdfs://hdfsIP:8020/user/hive/lib/hex-hive-udf-1.0-SNAPSHOT.jar';

-- ......

show functions 'default.*';
```

## 4 准备注册脚本执行sql

> 编写脚本 `register_hive_udf.sh`，我放在`/bigdata/hex-hive-ext`目录下

```shell
#!/bin/bash
# author: xiaoma

BIN_HOME=`dirname $0`
cd $BIN_HOME
BIN_HOME=`pwd`

HIVE_HOST=localhost
if [ "$1" != "" ];then
    HIVE_HOST=$1
fi
HIVE_PORT=25005
if [ "$2" != "" ];then
    HIVE_PORT=$2
fi
IMPALA_HOST=localhost
IMPALA_PORT=25004
USERNAME=edw
if [ "$3" != "" ];then
    USERNAME=$3
fi
PASSWORD=000000
if [ "$4" != "" ];then
    PASSWORD=$4
fi
HIVE_UDF_FILE=$BIN_HOME/register_hive_udf.sql

# ==========================在Hive中执行如下语句==========================
beeline -u jdbc:hive2://$HIVE_HOST:$HIVE_PORT -n $USERNAME -p $PASSWORD -f $HIVE_UDF_FILE
if [ $? -ne 0 ]; then
    echo "===>Hive和Impala永久注册自定义函数【失败】！"
    echo "===>请检查注册函数的SQL语句！！！"
    exit 1
fi

# ==========================在Impala中执行如下语句==========================
impala-shell -l -i $IMPALA_HOST:$IMPALA_PORT -u $USERNAME --ldap_password_cmd="echo -n $PASSWORD" \
--auth_creds_ok_in_clear -q "
invalidate metadata;
show functions;
"
if [ $? -ne 0 ];then
    echo "===>Impala中刷新元信息失败！"
    echo "===>请手动登录Impala执行invalidate metadata！！！"
fi

echo "===>Hive和Impala永久注册自定义函数【成功】！"

exit 0
```

> 执行注册脚本

```shell
sh /bigdata/hex-hive-ext/register_hive_udf.sh hiveIP 10000
```

## 5 过程整理

> 1：先打包为`hex_hive_udf.jar`，将其拷贝到`hive主节点`的`/bigdata/hex-hive-ext/libs`下；

> 2：将`put_hive_udf.sh`拷贝到`hive主节点`的`/bigdata/hex-hive-ext`下；

> 3：执行脚本`上传jar包到HDFS`；

```shell
sh /bigdata/hex-hive-ext/put_hive_udf.sh
```

> 4：在`register_hive_udf.sql`中配置好`注册函数的SQL语句`；

> 5:将`register_hive_udf.sql`、`register_hive_udf.sh`拷贝到`hive主节点`的`/bigdata/hex-hive-ext`下；

> 6：执行`脚本注册sql`，等待注册完成。

```shell
sh /bigdata/hex-hive-ext/register_hive_udf.sh hiveIP 10000
```

## 6 完工校验

> 在hive中执行sql`校验是否成功`

```shell
SELECT default.sys_uuid();
```

## 7 总结

> 1:如果`不使用impala`，那么把`UDF的jar传到任意目录`，然后`注册函数即可`(注册`不使用USING JAR`语句).
> 
> 2：如果`使用impala`，那么必须把`UDF的jar传到HDFS`，因为`Impala只会共享Hive中指定HDFS URL的注册函数`(注册必须使用`USING JAR`语句)，hive注册后在impala中执行`invalidate metadata`.
> 
> 3：如果`hive开启了权限验证`，那么必须把`UDF的jar传到hive辅助JAR目录`，并`重启hive`.(Hive集群配置的【`Hive 辅助 JAR 目录`】，英文叫【`Hive Auxiliary JARs Directory`】)
> 
> 4：本次注册的都是`永久注册`，如果要`临时注册`，那么只需要把`UDF的jar放到任意目录`，然后注册下就可以(添加`temporary`关键字)，和上面hive不开启权限验证且不使用impala是一样的。

```shell
----------------------- hive临时注册函数 -----------------------
-- 添加临时本地jar包
add jar /bigdata/tools/hive_udf/hex_hive_udf.jar;

-- 添加临时函数
-- 字符串转MD5，如：sys_md5(string str)
create temporary function sys_md5 as 'com.hex.hive.udf.Md5UDF';
```

## 8 demo

```shell
链接：https://pan.baidu.com/s/1uwtnEpcwrYlQRWFA6FsFAg 
提取码：6666
```

## 9 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路
![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

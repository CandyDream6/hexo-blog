---
title: Kerberos安装和使用 
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(105).jpg 
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(105).jpg 
tags:
  - 大数据
  - Kerberos 
category:
  - - 编程
    - 大数据
  - - 编程
    - Kerberos 
date: 2021-12-01 16:46:23
---

最近项目的`CDH集群`要加`Kerberos认证`。

做完了之后我把`过程记录一下`，方便以后`有用的话来看看`。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 安装Kerberos

### 1.1 server节点安装KDC服务

```shell
[root@node1]# yum -y install krb5-server krb5-libs krb5-auth-dialog krb5-workstation
```

### 2.2 client界面安装

```
[root@node2/3]# yum install -y krb5-workstation krb5-libs
```

### 2.3 在Cloudera Manager Server服务器上安装额外的包

```
[root@node1]# yum -y install openldap-clients
```

## 2 配置Kerberos

> 需要配置的文件有两个为`kdc.conf`和`krb5.conf `, 配置只需要修改`Server`服务节点配置，然后将`krb5.conf`发往各个`client`节点

### 2.1 修改KDC配置：kdc.conf

```
[root@node1]# vim /var/kerberos/krb5kdc/kdc.conf

[kdcdefaults]
kdc_ports = 88
kdc_tcp_ports = 88

[realms]
BIGDATA.HEX.COM = {                                ************
 #master_key_type = aes256-cts
 max_renewable_life = 7d 0h 0m 0s                  ************
 acl_file = /var/kerberos/krb5kdc/kadm5.acl
 dict_file = /usr/share/dict/words
 admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
 supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hma  ************
c:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:norm  ************
al des-cbc-crc:normal
}
```

> `后面带*号的是需要手动修改的`
>
> > `BIGDATA.HEX.COM`：realm名称，Kerberos支持多个realm，一般全用大写。
>
> > `acl_file`:admin的用户权。
>
> > `admin_keytab`:KDC进行校验的keytab。
>
> > `supported_enctypes`:支持的校验方式，注意把`camellia256-cts:normal` `camellia128-cts:normal`删除，因为camellia128-cts-cmac,camellia256-cts-cmac这种加密方式不能在6.x操作系统上正确解密
>
> > 注：java通过**keytab**访问kerberos不受redhat版本的影响

### 2.2 修改krb5.conf

```
[root@node1]# vim /etc/krb5.conf

[logging]
 default = FILE:/software/log/krb5libs.log
 kdc = FILE:/software/log/krb5kdc.log
 admin_server = FILE:/software/log/kadmind.log

[libdefaults]
 default_realm = BIGDATA.HEX.COM                   *******
 dns_lookup_realm = false
 dns_lookup_kdc = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 #default_ccache_name = KEYRING:persistent:%{uid}   *******
 udp_preference_limit = 1                           *******

[realms]
 BIGDATA.HEX.COM = {                                *******
  kdc = 192.168.1.61                                *******
  admin_server = 192.168.1.61                       *******
 }

[domain_realm]
 .bigdata.hex.com = BIGDATA.HEX.COM                  *******
 bigdata.hex.com = BIGDATA.HEX.COM                   *******
```

> 修改完成后将此文件`同步到其他client节点`

```
[root@node1]# scp /etc/krb5.conf ${hostname}:/etc/
```

> `后面带*号的是需要手动修改的`
>
> > `default_realm`：默认的realm，设置 Kerberos 应用程序的默认领域，必须跟要配置的realm的名称一致。
>
> > `ticket_lifetime`：表明凭证生效的时限，一般为24小时。
>
> > `renew_lifetime` ： 表明凭证最长可以被延期的时限，一般为一个礼拜。当凭证过期之后，对安全认证的服务的后续访问则会失败。
>
> > `udp_preference_limit`= 1：禁止使用 udp，可以防止一个 Hadoop 中的错误。
>
> > `realms`：配置使用的 realm，如果有多个领域，只需向 [realms] 节添加其他的语句。
>
> > `domain_realm`：集群域名与Kerberos realm的映射关系，单个realm的情况下，可忽略。

### 2.3 赋予Kerberos管理员所有权限

```
[root@node1]# vim /var/kerberos/krb5kdc/kadm5.acl

*/admin@BIGDATA.HEX.COM
```

> `*/admin`：admin实例的全部主体
>
> `@BIGDATA.HEX.COM`：realm
>
> `*`：全部权限
>
> 这个授权的意思：就是授予admin实例的全部主体对应BIGDATA.HEX.COM领域的全部权限。也就是创建Kerberos主体的时候如果实例为admin,就具有BIGDATA.HEX.COM领域的全部权限，比如创建如下的主体user1/admin就拥有全部的BIGDATA.HEX.COM领域的权限。

### 2.4 生成Kerberos数据库

> 在server节点执行下面操作，此处需要输入Kerberos数据库的密码，如：`yourPassword`

```
[root@node1]# kdb5_util create -r BIGDATA.HEX.COM -s
```

> 创建完成后`/var/kerberos/krb5kdc`目录下会生成对应的文件

```
[root@node1]# ls /var/kerberos/krb5kdc/
kadm5.acl  kdc.conf  principal  principal.kadm5  principal.kadm5.lock  principal.ok
```

### 2.5 添加到自启动

```
[root@node1]# service krb5kdc start
[root@node1]# service kadmin start
[root@node1]# chkconfig krb5kdc on
[root@node1]# chkconfig kadmin on
```

> 注意：启动失败时可以通过`/var/log/krb5kdc.log`和`/var/log/kadmind.log`来查看。

## 3 Kerberos数据库操作

### 3.1 登录Kerberos数据库

> 本地登录（无需认证）

```
[root@node1]# kadmin.local 
```

> 远程登录（需进行主体认证）

```
[root@node2]# kadmin
```

> 退出登录

```
kadmin:  exit
```

### 3.2 创建Kerberos主体

> `addprinc`, `add_principal`, `ank

```
[root@node1]# kadmin.local -q "addprinc -pw yourPassword admin/admin@BIGDATA.HEX.COM"
```

### 3.3 查看Kerberos所有主体

> `listprincs`, `list_principals`, `getprincs`, `get_principals`

```
[root@node1]# kadmin.local -q "listprincs"
```

### 3.4 删除Kerberos主体

> `delprinc`, `delete_principal`

```
[root@node1]# kadmin.local -q "delprinc admin/admin@BIGDATA.HEX.COM"
```

### 3.5 修改Kerberos主体密码

> `cpw`, `change_password`

```
[root@node1]# kadmin.local -q "cpw yourPassword admin/admin@BIGDATA.HEX.COM"
```

## 4 Kerberos主体认证

### 4.1 认证方式

#### 4.1.1 密码认证

```
[root@node1]# kinit admin/admin@BIGDATA.HEX.COM
```

#### 4.1.2 keytab密钥文件认证

> 先生成keytab文件，再进行认证

```
[root@node1]# kadmin.local -q "ktadd -k /usr/local/share/admin.keytab -norandkey admin/admin@BIGDATA.HEX.COM"
```

```
[root@node1]# kinit -kt /root/admin.keytab admin/admin@BIGDATA.HEX.COM
```

### 4.2 查看当前认证凭证

```
[root@node1]# klist
```

### 4.3 销毁当前认证凭证

```
[root@node1]# kdestroy
```

## 5  CDH启用Kerberos安全认证

### 5.1 为CM创建管理员主体

```
[root@node1]# kadmin.local -q "addprinc -pw yourPassword cloudera-scm/admin@BIGDATA.HEX.COM"
```

### 5.2 启用Kerberos

![1](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/1.png)

![2](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/2.png)

![3](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/3.png)

![4](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/4.png)

![5](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/5.png)

![6](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/6.png)

![7](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/7.png)

![8](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/8.png)

### 5.3 Kafka开启Kerberos

> `在ClouderaManager找到如下配置更改并重启`

```shell
ssl.client.auth 改为 none
kerberos.auth.enable 勾选
authenticate.zookeeper.connection 勾选
security.inter.broker.protocol 改为 SASL_PLAINTEXT
```

### 5.4 Hue开启Kerberos

> 添加Kerberos后`Hue死活起不来`，执行下面语句再启动

```shell
modprinc -maxrenewlife 90day krbtgt/BIGDATA.HEX.COM
```

## 6 各种方式访问开启Kerberos的CDH集群

### 6.1 Java_JDBC访问Hive,Impala,Kudu

> 不同的组件的keytab用自己组件对应的Kerberos用户生成的keytab

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.security.UserGroupInformation;

// 登录Kerberos账号
System.setProperty("java.security.krb5.conf", "C:\\Users\\V\\Desktop\\kerberos\\krb5.conf");
// 进行认证
Configuration configuration = new Configuration();
configuration.set("hadoop.security.authentication", "Kerberos");
UserGroupInformation.setConfiguration(configuration);
UserGroupInformation.loginUserFromKeytab("hive/node1.company.cn@BIGDATA.HEX.COM", "C:\\Users\\V\\Desktop\\kerberos\\hive.keytab");
```

#### 6.1.1 JDBC_URL和驱动名称

##### 6.1.1.1 Hive

```shell
url: jdbc:hive2://172.18.25.246:10000/db;principal=hive/node1.company.cn@BIGDATA.HEX.COM   (不写db也可以：jdbc:hive2://172.18.25.246:10000/;principal=hive/node1.company.cn@BIGDATA.HEX.COM)
driverClassName: org.apache.hive.jdbc.HiveDriver
```

##### 6.1.1.2 Impala

> 之前Impala驱动用的是`com.cloudera.impala.jdbc41.Driver`,官网推荐这种方式，另一种支持的方式是用Hive的驱动
>
> 然后`加了Kerberos后`，发现`Impala只能用Hive的驱动去连接`。

```shell
url: jdbc:hive2://172.18.25.246:21050/db;principal=impala/node1.zhiyu.cn@BIGDATA.HEX.COM   (db写不写都可以)
driverClassName: org.apache.hive.jdbc.HiveDriver
```

##### 6.1.1.3 Kudu

> kudu我们是通过Impala去操作的，所以`和上面Impala一样`，`用户和KeyTab文件也用Impala的`

### 6.2 Java_API访问HBase

```shell
import  org.apache.hadoop.conf.Configuration;
import  org.apache.hadoop.security.UserGroupInformation;

Configuration configuration = HBaseConfiguration.create();
System.setProperty("java.security.krb5.conf", "C:\\Users\\V\\Desktop\\kerberos\\krb5.conf");
configuration.set("hbase.zookeeper.quorum", "172.18.26.132:2181,172.18.25.246:2181,172.18.25.241:2181");
configuration.set("hadoop.security.authentication", "Kerberos");
configuration.set("hbase.security.authentication", "Kerberos");
configuration.set("hbase.master.kerberos.principal", "hbase/node2.company.cn@BIGDATA.HEX.COM");

UserGroupInformation.setConfiguration(configuration);
UserGroupInformation.loginUserFromKeytab("hbase/node1.zhiyu.cn@BIGDATA.HEX.COM", "C:\\Users\\V\\Desktop\\kerberos\\hbase.keytab");
```

### 6.3 Java_API访问Kafka

```shell
Properties props = new Properties();
System.setProperty("java.security.krb5.conf", "C:\\Users\\V\\Desktop\\kerberos\\krb5.conf");
System.setProperty("java.security.auth.login.config", "C:\\Users\\V\\Desktop\\kerberos\\jaas-cache.conf");
props.put("bootstrap.servers", "172.18.25.246:9092,172.18.25.241:9092,172.18.26.132:9092");
props.put("group.id", "myGroup");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("security.protocol", "SASL_PLAINTEXT");
props.put("sasl.kerberos.service.name", "kafka");
```

> jaas-cache.conf文件如下

```shell
KafkaClient {
com.sun.security.auth.module.Krb5LoginModule required
useKeyTab=true
keyTab="C:/Users/V/Desktop/kerberos/kafka.keytab"
principal="kafka/node1.company.cn@BIGDATA.HEX.COM";
};
```

### 6.4 Kafka Tool访问Kafka

#### 6.4.1 Kafka Tool快捷方式配置

> 在快捷方式后配置krb5文件路径

```shell
-J-Djava.security.krb5.conf=C:/Users/V/Desktop/kerberos/krb5.conf
```

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/10.png)

#### 6.4.2 Kafka Tool参数配置

![1](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/11.png)

![2](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/12.png)

```shell
GSSAPI
```

![3](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/13.png)

```shell
com.sun.security.auth.module.Krb5LoginModule required
serviceName="kafka"
refreshKrb5Config=true
useKeyTab=true
keyTab="C:/Users/V/Desktop/kerberos/kafka.keytab"
principal="kafka/node1.zhiyu.cn@BIGDATA.HEX.COM";
```

![4](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/14.png)

#### 6.4.3 结果展示

![4](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/15.png)

### 6.5 Beeline_Hive访问

> 先认证再操作

```shell
kinit -k -t /usr/local/hive.keytab hive/node1.company.cn@BIGDATA.HEX.COM && beeline --verbose=true -u "jdbc:hive2://172.18.25.111:10000/hive_uat;principal=hive/node1.company.cn@BIGDATA.HEX.COM" -d org.apache.hive.jdbc.HiveDriver -e "show databases;"
```

### 6.6 Impala-shell访问

> `注意：这里impala-shell里的-i参数要用主机名称，不能用ip`

```shell
kinit -k -t /usr/local/impala.keytab impala/node1.company.cn@BIGDATA.HEX.COM  &&  impala-shell -i "node1.company.cn:21000" -q "show databases"
```

## 7 遇到问题

### 7.1 使用hive/hue操作HBaseStorageHandler的HBase,报用户没权限异常

> 原因，`没有使用sentry`，`但是HBase开启了授权校验`，关了就好了

```shell
hbase.security.authorization  不勾选
```

### 7.2 hue操作hive用到mapperReduce时,报用户hue没有加入白名单

> 解决方案，`将用户hue加入到yarn的白名单`

![](https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleContent/Kerberos安装和使用/9.png)

## 8 躺板板

`红伞伞白杆杆，吃完一起躺板板`

`躺板板埋山山，亲朋都来吃饭饭`

`饭饭里有红伞伞，吃完全村埋山山，来年长满红伞伞`

## 关注博主不迷路

![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
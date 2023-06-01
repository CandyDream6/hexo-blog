---
title: SpringBoot参数校验 
index_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(86).jpg 
banner_img: https://blog-1305951218.cos.ap-shanghai.myqcloud.com/blog/image/articleBg/1(86).jpg

tags:
  - Java
  - 参数校验 
category:
  - - 编程
    - Java 
date: 2021-07-26 14:16:12
---

最近项目刚起，`SpringBoot`的，开始做了个`参数校验`，这里就记录以下。

<!-- more -->

# `关注博主不迷路，获取更多干货资源`

![](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)

## 1 POM

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
    <version>2.5.3</version>
</dependency>
```

## 2 Controller

```java
@GetMapping(value = "allDatabases", consumes = MediaType.APPLICATION_JSON_VALUE, produces = MediaType.APPLICATION_JSON_VALUE)
public ResponseInfo getAllDatabases(@Validated(SourceInfo.GetAllDatabases.class) @RequestBody SourceInfo sourceInfo) {
    return (ResponseInfo) sourceMetadataService.getAllDatabases(sourceInfo);
}
```

### 2.1 参数解释

> `@Validated`:这里使用`@Validated`而不是`@Valid`，因为实体`SourceInfo`里需要做`分组校验`，@Valid做不了。

> `SourceInfo.GetAllDatabases.class`:这个是`级联校验的分组`，后面实体里看一眼就知道了。

## 3 实体

```java
@Data
@Accessors(chain = true)
public class SourceInfo {
    /** 数据源编码 */
    private String sourceCode;
    /** 数据源名称 */
    private String sourceName;
    /** 数据源类型 */
    @NotBlank(message = "sourceType cannot be blank")
    private String sourceType;
    /** 数据源描述 */
    private String sourceDes;

    /** 数据源配置信息 */
    //做级联校验，需要在属性上加@Valid
    @Valid
    private List<SourceConfInfo> sourceConfInfos;
    
    /** 数据源操作信息 */
    @Valid
    private SourceOptInfo sourceOptInfo;

    /** 参数校验分组:创建表 */
    /*
    * 分组信息，继承Default是因为一个校验不指定分组时，默认是Default分组，例如上面的sourceType
    * 在controller里:
    *   如果不指定分组，那么走Defalut校验
    *   如果指定了分组，那么走指定分组的校验(如果不继承Default，那么不会走sourceType的校验)
    *  
    */
    public interface CreateTable extends Default {
    }

    /** 参数校验分组:删除表 */
    public interface DropTable extends Default {
    }

    /** 参数校验分组:执行sql语句 */
    public interface ExecuteSql extends Default {
    }

    /** 参数校验分组:获取所有数据库 */
    public interface GetAllDatabases extends Default {
    }

    /** 参数校验分组:获取目标库所有表 */
    public interface GetTargetDatabaseAllTables extends Default {
    }

    /** 参数校验分组:获取目标表所有字段 */
    public interface GetTargetTableFields extends Default {
    }

}
```

```java
@Data
@Accessors(chain = true)
public class SourceConfInfo {
    /** 配置名 */
    @NotBlank(message = "confName cannot be blank")
    private String confName;
    /** 配置值 */
    @NotBlank(message = "confValue cannot be blank")
    private String confValue;
    /** 配置描述 */
    private String confDes;
}
```

```java
@Data
@Accessors(chain = true)
public class SourceOptInfo {

    /** 数据库名 */
    @NotEmpty(message = "databases cannot be empty",
            groups = {SourceInfo.CreateTable.class, SourceInfo.DropTable.class,
                    SourceInfo.GetTargetDatabaseAllTables.class, SourceInfo.GetTargetTableFields.class, SourceInfo.ExecuteSql.class})
    private List<String> databases;

    /** sql集合 */
    @NotEmpty(message = "databases cannot be empty",
            groups = {SourceInfo.CreateTable.class, SourceInfo.ExecuteSql.class})
    private List<String> sqls;

    /** 表集合：删除表时传递 */
    @NotEmpty(message = "databases cannot be empty",
            groups = {SourceInfo.DropTable.class, SourceInfo.GetTargetTableFields.class})
    private List<String> tables;
}
```

## 4 支持的注解

> 我只简单列一下，具体的打开源码去看吧。

注解 | 解释
----|---
 `@AssertFalse` | 带注释的元素必须为 false。 支持的类型是boolean和Boolean 。<br/><br/> null元素被认为是有效的。
 `@AssertTrue` | 带注释元素必须为true。 支持的类型是boolean和Boolean 。<br/><br/>null元素被认为是有效的。
 `@DecimalMax` | 带注释的元素必须是一个数字，其值必须小于或等于指定的最大值。
 `@DecimalMin` | 带注释的元素必须是一个数字，其值必须大于或等于指定的最小值。
 `@Digits` | 带注释元素必须是可接受范围内的数字。
 `@Email` | 该字符串必须是格式正确的电子邮件地址。 构成有效电子邮件地址的确切语义留给 Jakarta Bean 验证提供程序。 接受CharSequence 。<br/><br/>null元素被认为是有效的。
 `@Future` | 带注释元素必须是未来的某个时刻、日期或时间。
 `@FutureOrPresent` | 带注释元素必须是现在或将来的瞬间、日期或时间。
 `@Max` | 带注释的元素必须是一个数字，其值必须小于或等于指定的最大值。
 `@Min` | 带注释的元素必须是一个数字，其值必须大于或等于指定的最小值。
 `@Negative` | 带注释的元素必须是严格的负数（即 0 被视为无效值）
 `@NegativeOrZero` | 带注释的元素必须是负数或 0。
 `@NotBlank` | 带注释的元素不能为null并且必须至少包含一个非空白字符。(一般字符串用这个)
 `@NotEmpty` | 带注释的元素不得为null或为空。(一般数组用这个)
 `@NotNull` | 带注释的元素不能为null 。 接受任何类型。
 `@Null` | 带注释的元素必须为null 。 接受任何类型。
 `@Past` | 带注释的元素必须是过去的某个时刻、日期或时间。
 `@PastOrPresent` | 带注释元素必须是过去或现在的瞬间、日期或时间。
 `@Pattern` | 带注释的CharSequence必须匹配指定的正则表达式。
 `@Positive` | 带注释元素必须是严格的正数（即 0 被视为无效值）。
 `@PositiveOrZero` | 带注释元素必须是正数或 0。
 `@Size` | 带注释的元素大小必须在指定的边界（包括）之间。

## 5 完工

> `吃饭饭`，`躺板板`。

## 关注博主不迷路

![联系博主](https://github-edu-student-id-card-basic-1305951218.cos.ap-shanghai.myqcloud.com/shouhou.jpg)
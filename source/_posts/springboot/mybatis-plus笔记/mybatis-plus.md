---
title: Mybatis Plus的使用
typora-copy-images-to: ./mybatis-plus
typora-root-url: ./mybatis-plus
date: 2020-03-17 13:48:28
updated:
tags:
  - Spring Boot
  - Mybatis Plus
categories:
  - Java
  - Mybatis
---

# Mybatis Plus

mybatis plus 在 mybatis 基础上没有改变其本身，并做了增强。

[官方提供在Github的mybatis-plus-samples](https://github.com/baomidou/mybatis-plus-samples)

[参考：MP官网](https://mp.baomidou.com/)

![官方宣传图](/relationship-with-mybatis.png)

<!--more-->

## 基本结构：

其中core是mybatis，

![](/mybatis-plus-framework.jpg)

## 配置基本：

### 导入依赖：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>版本</version>
</dependency>
```

### application.yml

```yaml
spring:
  datasource:
    username: root
    password: lcl123
    url: jdbc:mysql://localhost:3306/mybatis?useSSl=false&serverTimeZone=UTC&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
```

### 添加扫描路径：

1. 在 Spring Boot 启动类中添加 `@MapperScan` 注解，扫描 Mapper 文件夹：

```java
@SpringBootApplication
@MapperScan("com.baomidou.mybatisplus.samples.quickstart.mapper")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(QuickStartApplication.class, args);
    }

}
```

2. 或者在每个Mapper类上加上`@Mapper`注解

## 基本方法：

### 用户实体：

编写实体类 `User.java`（此处使用了 [Lombok](https://www.projectlombok.org/) 简化代码）

```java
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

### 编写Mapper类，并集成BaseMapper

```java
public interface UserMapper extends BaseMapper<User> {

}
```

## 常用注解

### @TableName

- 描述：表名注解

| 属性             | 类型    | 必须指定 | 默认值 | 描述                                                         |
| ---------------- | ------- | -------- | ------ | ------------------------------------------------------------ |
| value            | String  | 否       | ""     | 表名                                                         |
| schema           | String  | 否       | ""     | schema                                                       |
| keepGlobalPrefix | boolean | 否       | false  | 是否保持使用全局的 tablePrefix 的值(如果设置了全局 tablePrefix 且自行设置了 value 的值) |
| resultMap        | String  | 否       | ""     | xml 中 resultMap 的 id                                       |
| autoResultMap    | boolean | 否       | false  | 是否自动构建 resultMap 并使用(如果设置 resultMap 则不会进行 resultMap 的自动构建并注入) |

### @TableId

- 描述：主键注解

| 属性  | 类型   | 必须指定 | 默认值      | 描述       |
| ----- | ------ | -------- | ----------- | ---------- |
| value | String | 否       | ""          | 主键字段名 |
| type  | Enum   | 否       | IdType.NONE | 主键类型   |

#### IdType

| 值            | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| AUTO          | 数据库ID自增                                                 |
| NONE          | 无状态,该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT) |
| INPUT         | insert前自行set主键值                                        |
| ASSIGN_ID     | 分配ID(主键类型为Number(Long和Integer)或String)(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextId`(默认实现类为`DefaultIdentifierGenerator`雪花算法) |
| ASSIGN_UUID   | 分配UUID,主键类型为String(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextUUID`(默认default方法) |
| ID_WORKER     | 分布式全局唯一ID 长整型类型(please use `ASSIGN_ID`)          |
| UUID          | 32位UUID字符串(please use `ASSIGN_UUID`)                     |
| ID_WORKER_STR | 分布式全局唯一ID 字符串类型(please use `ASSIGN_ID`)          |



### @TableField

- 描述：字段注解(非主键)

| 属性             | 类型                         | 必须指定 | 默认值                   | 描述                                                         |
| ---------------- | ---------------------------- | -------- | ------------------------ | ------------------------------------------------------------ |
| value            | String                       | 否       | ""                       | 字段名                                                       |
| el               | String                       | 否       | ""                       | 映射为原生 `#{ ... }` 逻辑,相当于写在 xml 里的 `#{ ... }` 部分 |
| exist            | boolean                      | 否       | true                     | 是否为数据库表字段                                           |
| condition        | String                       | 否       | ""                       | 字段 `where` 实体查询比较条件,有值设置则按设置的值为准,没有则为默认全局的 `%s=#{\%s}` |
| update           | String                       | 否       | ""                       | 字段 `update set` 部分注入, 例如：update="%s+1"：表示更新时会set version=version+1(该属性优先级高于 `el` 属性) |
| insertStrategy   | Enum                         | N        | DEFAULT                  | 举例：NOT_NULL: `insert into table_a(<if test="columnProperty != null">column</if>) values (<if test="columnProperty != null">#{columnProperty}</if>)` |
| updateStrategy   | Enum                         | N        | DEFAULT                  | 举例：IGNORED: `update table_a set column=#{columnProperty}` |
| whereStrategy    | Enum                         | N        | DEFAULT                  | 举例：NOT_EMPTY: `where <if test="columnProperty != null and columnProperty!=''">column=#{columnProperty}</if>` |
| fill             | Enum                         | 否       | FieldFill.DEFAULT        | 字段自动填充策略                                             |
| select           | boolean                      | 否       | true                     | 是否进行 select 查询                                         |
| keepGlobalFormat | boolean                      | 否       | false                    | 是否保持使用全局的 format 进行处理                           |
| jdbcType         | JdbcType                     | 否       | JdbcType.UNDEFINED       | JDBC类型 (该默认值不代表会按照该值生效)                      |
| typeHandler      | Class<? extends TypeHandler> | 否       | UnknownTypeHandler.class | 类型处理器 (该默认值不代表会按照该值生效)                    |
| numericScale     | String                       | 否       | ""                       | 指定小数点后保留的位数                                       |

关于`jdbcType`和`typeHandler`以及`numericScale`的说明:

`numericScale`只生效于 update 的sql. `jdbcType`和`typeHandler`如果不配合`@TableName#autoResultMap = true`一起使用,也只生效于 update 的sql. 对于`typeHandler`如果你的字段类型和set进去的类型为`equals`关系,则只需要让你的`typeHandler`让Mybatis加载到即可,不需要使用注解

#### FieldStrategy

| 值        | 描述                                                      |
| --------- | --------------------------------------------------------- |
| IGNORED   | 忽略判断                                                  |
| NOT_NULL  | 非NULL判断                                                |
| NOT_EMPTY | 非空判断(只对字符串类型字段,其他类型字段依然为非NULL判断) |
| DEFAULT   | 追随全局配置                                              |

#### FieldFill

| 值            | 描述                 |
| ------------- | -------------------- |
| DEFAULT       | 默认不处理           |
| INSERT        | 插入时填充字段       |
| UPDATE        | 更新时填充字段       |
| INSERT_UPDATE | 插入和更新时填充字段 |

在更新时候遇到一个小发现

用Lambda方法进行更新时，用set()某个值不会进行自动填充，必须new一个新的对象才会进行自动填充update
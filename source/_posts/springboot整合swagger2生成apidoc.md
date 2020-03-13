---
title: springboot整合swagger2
date: 2020-02-29 01:41:22
tags:
  - Spring Boot
categories:
  - Java
  - Spring Boot
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
---

# swagger介绍

swagger 提供了一套优秀的api生成，可以在springboot中进行简单配置后，利用响应的注解自动生成apidoc，并且还具有接口测试功能

<!-- more -->

# swagger整合进springboot

## 1.新建springboot项目

这里新建一个springboot项目，勾选springboot-web即可

```bash
.
|____main
| |____resources
| | |____application-prod.yml
| | |____application-dev.yml
| | |____application.yml
| |____java
| | |____com
| | | |____sblearning
| | | | |____swagger2
| | | | | |____Swagger2Application.java
| | | | | |____config
| | | | | | |____SwaggerConfig.java
| | | | | |____entity
| | | | | | |____User.java
| | | | | |____controller
| | | | | | |____HelloController.java
```

并在建立相应文件，结构如上所示

## 2.添加swagger依赖并注入

需要添加两个依赖，目前的稳定版本是2.9.2，可以在maven仓库中查找

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

在`SwaggerConfig`中加上如下注释

```java
package com.sblearning.swagger2.config;

import org.springframework.context.annotation.Configuration;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

}
```

重启后即可发现，swagger已经启动在了`http://localhost:8080/swagger-ui.html`之中，现在还是默认配置

# swagger配置

## 1.配置swagger基本信息

这的配置主要是将`Docket`对象注入到springboot中

修改`SwaggerConfig`文件如下：

```java
package com.sblearning.swagger2.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;

@Configuration
@EnableSwagger2
public class SwaggerConfig {



    Contact myContact = new Contact("","","");

    /**
     * 配置首页信息
     * @return
     */
    private ApiInfo myApiInfo() {
        return new ApiInfo("Api Documentation",
                "Api Documentation",
                "1.0",
                "urn:tos",
                myContact,
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList());
    }

    /**
     * 配置 Swagger 的 Docket 的 bean 实例
     *
     * @return
     */
    @Bean
    public Docket docket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(myApiInfo());
    }


}
```

## 2.配置扫描接口

修改`Swagger`文件下的`docket()`，并且设置lcl分组只有在sprintboot在dev环境下才可用，代码如下：

```java
@Value("${spring.profiles.active}")
private String environment;

/**
 * 配置 Swagger 的 Docket 的 bean 实例
 *
 * @return
 */
@Bean
public Docket docket() {
    return new Docket(DocumentationType.SWAGGER_2)
        .apiInfo(myApiInfo())
        // enable: 是否能访问swagger，可以通过flag使swagger只在生产环境下使用
        .enable(this.ifDevEnvironment())
        .groupName("lcl")
        .select()
        // RequestHandlerSelectors 选择扫描方式
        // basePackage() 配置要扫描的包
        .apis(RequestHandlerSelectors.basePackage("com.sblearning.swagger2"))
        // path() 配置要扫描的路径
        .build();
}

public Boolean ifDevEnvironment() {
    return this.environment.equals("dev");
}
```

## 3.配置api文档分组

在上面`Docket`对象的配置中，`groupName`就是当前分组名，如果为springboot注入多个`Docket`对象，那么将会有多个分组，这样做的好处是在多人共同开发时，可以按照各自的工作分组，对于各自的工作也都能清晰的看到了。

# 常用注解

## 1.在entity中使用

首先我们要知道的是，只要我们的controller中映射的接口返回了一个实体类，他都会在Models中出现

```java
package com.sblearning.swagger2.entity;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

@ApiModel("用户")
public class User {

    @ApiModelProperty("用户名")
    public String username;
    @ApiModelProperty("密码")
    public String password;

    ...省略constructor，get，set
}
```

主要就是用到`@ApiModel`和`@ApiModelProperty`，分别是对实体类和字段进行说明。

## 2.在controller中使用

### @Api

对一个java类进行解释说明

```java
@Api(description = "API接口")
@RestController
public class HelloController {

}
```

### @ApiImplicitParam

对于这个注解在get请求中有两种用法

- 通过url传参时用`@ApiImplicitParam(value = "名字", name = "te2")`，其中name要和`@RequestParam`的参数`name` 一致，后者默认为函数的形参名

  ```java
  @ApiOperation(value = "你好2")
  @GetMapping(value = "/hello2")
  @ApiImplicitParam(value = "名字", name = "te2")
  public User hello2(@RequestParam String te2) {
      return new User(te2, "123");
  }
  ```

- 通过REST形式, 这里的`id`参数名要一致

  ```java
  @ApiOperation(value = "你好3")
  @GetMapping(value = "/hello3/{id}")
  @ApiImplicitParam(value = "用户id", name = "id")
  public User hello3(@PathVariable("id") String parameter) {
      return new User(parameter, "123");
  }
  ```

- 如果多个参数可以用@ApiImplicitParams

  ```java
  @ApiImplicitParams({
      @ApiImplicitParam(),
      @ApiImplicitParam(),
      ......
  })
  ```


### @ApiParam

该注解的使用直接放到参数前面就可以，相对比较简单

```java
@ApiOperation(value = "你好2")
@GetMapping(value = "/hello2")
public User hello2(@ApiParam("名字") @RequestParam String te2) {
return new User(te2, "123");
}
```

而且对于post请求，相对比较友好，因为在测试时会直接生成json字段

```java
@PostMapping("/hellopost")
public User hellopost(@ApiParam("用户实体") @RequestBody User u1) {
    return u1;
}
```

### 上两种方法对比

个人感觉`@ApiParam`注解对于测试api更加灵活好用。
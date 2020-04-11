---
title: Java中传递函数(行为)的两种方式
date: 2020-04-11 15:06:17
typora-copy-images-to:
typora-root-url:
tags:
  - Java
categories:
  - Java
---

## 开门见山：

1. 接收一个接口，传参接口的实现
2. 使用jdk8的Function类进行接收

在spring boot的项目中，为了对前端传递的参数进行判断，一个比较好的方法对每一个接受的json对应一个接受的实体类，判断传递的各个字段是否完整可以通过对实体类添加`@NotNull`注解。

```java
@Data
public class SysRole implements Serializable {

    @TableId(type = IdType.AUTO)
    private Integer id;

    @NotNull(message = "角色名不能为空")
    private String roleName;

    private static final long serialVersionUID = 1L;
}
```

然后在controller的方法中中添加如下参数：

```java
@PostMapping("/add")
public Result add(@RequestBody @Valid SysRole role, BindingResult result) {
    ...
}
```

<!--more-->

其中第二个参数就是spring boot验证后的结果，可以通过

```java
if (result.hasErrors()) {
    return ResultUtil.error(
        ResultEnum.MISS_FIELD.getCode(),
        result.getFieldError().getDefaultMessage());
} 
return serviceMethod.get();
```

获取到验证字段为空时自己设置的`message`。

## 优化出工具方法

### jdk8的Function类

在写代码时，我发现，如果有很多接口都需要验证参数，难道要写很多遍上面的代码吗，这时一个好的办法是写一个util类写一个验证字段的通用方法。在这个需求中，就会出现一个问题，`BindingResult result`这个参数是一个对象肯定要传入，我们要传递的第二个参数应该是一个动作，一个方法，比如在验证成功之后要怎么做。如果熟悉js的同学一定想到了将函数作为参数传递，jdk8也加入了面向函数编程这一写法，提供了非常多的Function类，这里我使用了`Supplier`类，它的英文意思是生产者，顾名思义，它对应的函数特点是无参有返回值。

```java
/**
 * 此方法是springboot valid的字段验证
 * @param bindingResult springboot的验证结果
 * @param serviceMethod 验证成功后执行的业务逻辑
 * @return 验证失败的处理结果或验证成功的 successResult
 */
public static Result vaildFieldError(BindingResult bindingResult, Supplier<Result> serviceMethod) {
    if (bindingResult.hasErrors()) {
        return ResultUtil.error(
            ResultEnum.MISS_FIELD.getCode(),
            bindingResult.getFieldError().getDefaultMessage());
    } else {
        return serviceMethod.get();
    }
}
```

`Supplier`类的`get()`方法就是相当于执行这个函数获得返回值了。

这就是在开头提到的使用jdk8的Function类进行接收。

### jdk8之前的方法

对于之前的方法，我也是试着实现了一下，后来删除了使用jdk8的方法，毕竟面向函数这是一个现代语言的趋势。

在这里大致说一下实现过程。

1. 新建一个接口，里面写一个方法，这个方法就对应上面`serviceMethod`。
2. 在工具类中接收接口为参数
	```java
	public static Result vaildFieldError(BindingResult bindingResult, MyInterface interface)
	```
3. 在使用此工具类的时候，传参需要写一个匿名接口实现类，这里也可以使用jdk8的Lambda函数，如果不用jdk8那么就用传统的匿名实现类，重写接口里的那个方法就行了。
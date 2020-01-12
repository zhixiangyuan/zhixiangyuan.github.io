---
title: "Spring AOP 完全使用指北"
date: 2020-01-09T16:09:34+08:00
keywords: []
description: ""
tags: [
    "spring"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
draft: true
---

> 本文仅介绍基于注解的方式

# 概念简述

Aspect：

Join Point：可以被切的地方，比如将要执行的方法

Advice：切入到切点上面的逻辑都放在这里面

Pointcut：切面放到哪个切点上，就是条件的意思

Introduction：

Target object：

AOP proxy：

Weaving：

# 1 开启 AOP 功能

在标有 @Configuration 的类上面增加 @EnableAspectJAutoProxy 注解即可

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```

需要注意的是，当我们使用注解方式的时候，那么我们必然需要用到 @Aspect 注解来定义切面，如果我们是 SpringBoot 项目，那么直接引入 `spring-boot-starter-aop` 即可，但是如果我们不是 SpringBoot 的项目，那么便需要自己引入 AspectJ Weaver 项目，[在 pom 中引入 aspectjweaver 的依赖](https://mvnrepository.com/artifact/org.aspectj/aspectjweaver) @AspectJ 的注解。

# 2 定义切面

在类上面加上 @Aspect 便定义该类为切面，但是需要注意的是，如果不在上面加上 @Component，Spring 是不会将该类作为 Bean 放到容器中去的。

```java
@Aspect
@Component
public class OneAspect {

}
```

# 声明切点

定义切点的方法，返回值必须是 void，权限修饰符使用  皆可，至于使用哪种权限修饰符就需要看具体的业务场景了。这里需要注意的是，

```java
    @Pointcut("execution(* transfer(..))") // the pointcut expression
    private void anyOldTransfer() {}
```

## 切点表达式

Spring AOP 并不支持 AspectJ 的所有切点表达式，目前仅支持以下几种切点表达式

| Pointcut Designators | Mean                                                   |
| -------------------- | ------------------------------------------------------ |
| execution            | 具体执行哪些方法                                       |
| within               | 某个包内                                               |
| this                 |
| target               |
| args                 |
| @target              |
| @args                |
| @within              |
| @annotation          | 标注了某个注解                                         |
| bean                 | 这是 Spring 定义的关键字，用于表示哪个 bean 需要被代理 |

### execution

```
以下是 execution 表达式的格式:

execution([$权限修饰符] <$返回类型> [$类全名]<$方法名>(<$参数列表>) [$抛出的异常])

[$权限修饰符]：可填入 protected 或者 public，同时可以省略，省略时表示匹配除了 private 的所有权限
需要注意的是，权限修饰符无法匹配 private 这是由于 AOP 的代理机制引起的，如果是 JDK 的代理方式，那
么这是一种

<$返回类型>：必填，如果要匹配所有类型则填入 * 即可

[$类全名]：可省略，

<$方法名>：

<$参数列表>：

[$抛出的异常]：
```




# 参考文章

1. [官方文档](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-annotation-config)
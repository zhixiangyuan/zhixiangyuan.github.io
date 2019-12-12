---
title: "JDK8 中新增的 TYPE_USE 和 TYPE_PARAMETER 可以用在什么场景"
date: 2019-12-12T13:48:56+08:00
keywords: []
description: ""
tags: [
    "java"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

本篇直接用示例代码展示该注解可以用在什么场景

# 1 TYPE_USE

```java
// 先定义一个测试注解
@Target(ElementType.TYPE_USE)
public @interface TestTypeUse {
}

    // 1. new 对象的时候可以使用
    Object testStr = new @TestTypeUse String();

    // 2. 类型转换的时候可以使用
    Integer testInt = (@TestTypeUse Integer) testStr;

    // 3. 声明变量的时候可以使用
    @TestTypeUse Object obj;

    // 4. 使用范型的时候可以使用
    List<@TestTypeUse String> list;

    // 5. 也可以标在类上面
    @TestTypeUse
    public class Main {...}

    // 6. 也可以标在方法上，不过要注意的是，方法的返回值不能是 void
    @TestTypeUse
    public int testMethod() {...}

    // 7. 标在 implements 后面
    public class Main implements @TestTypeUse Serializable {...}

    // 8. 标在 extends 后面
    public class Main extends @TestTypeUse Object {...}

    // 9. 标在方法的参数上
    public static void main(@TestTypeUse String[] args) {...}

    // 10. 类上面声明范型的时候可以使用
    public class Main<@TestTypeUse T> {...}
```

# 2 TYPE_PARAMETER

```java
// 先定义一个测试注解
@Target(ElementType.TYPE_PARAMETER)
public @interface TestTypeParameter {
}
    // 1. 类上面声明范型的时候可以使用
    public class Main<@TestTypeParameter T> {...}

    // 2. 方法上面声明范型的时候也可以使用
    public static <@TestTypeParameter T> void testMethod() {...}
```
---
title: "Method 的 getParameterTypes 与 getGenericParameterTypes 的区别"
date: 2019-12-17T18:03:24+08:00
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

getParameterTypes 与 getGenericParameterTypes 区别在于，对于没有泛型的方法参数，两者返回的均是相同的 Class，对于带泛型的方法参数 getGenericParameterTypes 返回的是带泛型的类型，getParameterTypes 返回的是不带泛型的类型。

下面是一个例子

```java
class GenericMethod {
    public void method(List list) { }

    public void genericMethod(List<Integer> list) { }
}

public class MethodTest {

    public static void main(String[] args) throws NoSuchMethodException {
        Method method = GenericMethod.class.getMethod("method", List.class);
        System.out.println(method.getParameterTypes()[0]);        // interface java.util.List
        System.out.println(method.getGenericParameterTypes()[0]); // interface java.util.List
        Method genericMethod = GenericMethod.class.getMethod("genericMethod", List.class);
        System.out.println(genericMethod.getParameterTypes()[0]);         // interface java.util.List
        // 打印出带泛型的类型
        System.out.println(genericMethod.getGenericParameterTypes()[0]);  // java.util.List<java.lang.Integer>
    }
}
```
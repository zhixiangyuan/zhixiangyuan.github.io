---
title: "Constructor 的 getParameterTypes 与 getGenericParameterTypes 的区别"
date: 2019-12-17T18:11:08+08:00
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
    public GenericMethod(List<String> list) { }
    public GenericMethod(Set list) { }
}

public class MethodTest {
    public static void main(String[] args) throws NoSuchMethodException {
        Constructor setConstructor = GenericMethod.class.getConstructor(Set.class);
        System.out.println(setConstructor.getParameterTypes()[0]);         // interface java.util.Set
        System.out.println(setConstructor.getGenericParameterTypes()[0]);  // interface java.util.Set
        Constructor listConstructor = GenericMethod.class.getConstructor(List.class);
        System.out.println(listConstructor.getParameterTypes()[0]);        // interface java.util.List
        // 打印出带泛型的类型
        System.out.println(listConstructor.getGenericParameterTypes()[0]); // java.util.List<java.lang.String>
    }
}
```
---
title: "Class 的 getSuperclass 与 getGenericSuperclass 区别"
date: 2019-12-17T17:29:41+08:00
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

getSuperclass 与 getGenericSuperclass 如果继承的父类没有泛型，那么返回的结果是相同的，如果继承的父类包含泛型，则会返回带泛型的父类类型（sun.reflect.generics.reflectiveObjects.ParameterizedTypeImpl）

```java
class A<T> { }

class B extends A { }

class C extends A<String> { }

public class Main {
    public static void main(String[] args) {
        System.out.println(B.class.getSuperclass());        // class me.yuanzx.test.A
        System.out.println(B.class.getGenericSuperclass()); // class me.yuanzx.test.A
        System.out.println(C.class.getSuperclass());        // class me.yuanzx.test.A
        // 打印出带泛型的类型
        System.out.println(C.class.getGenericSuperclass()); // me.yuanzx.test.A<java.lang.String>
    }
}
```
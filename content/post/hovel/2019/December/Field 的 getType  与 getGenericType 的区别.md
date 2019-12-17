---
title: "Field 的 getType  与 getGenericType 的区别"
date: 2019-12-17T17:49:26+08:00
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

getType 与 getGenericType 区别在于，对于没有泛型的字段类型，两者返回的均是相同的 Class，对于带泛型的字段 getGenericType 返回的是带泛型的类型，getType 返回的是不带泛型的类型。

下面是一个例子

```java
class GenericTest<T> {
    public List list;
    public List<CharSequence> genericList;
}

public class FieldTest {
    public static void main(String[] args) throws NoSuchFieldException {
        Field strField = GenericTest.class.getField("list");
        System.out.println(strField.getType());           // interface java.util.List
        System.out.println(strField.getGenericType());    // interface java.util.List

        Field charSequenceListField = GenericTest.class.getField("genericList");
        System.out.println(charSequenceListField.getType());        // interface java.util.List
        // 打印出带泛型的类型
        System.out.println(charSequenceListField.getGenericType()); // java.util.List<java.lang.CharSequence>
    }
}
```
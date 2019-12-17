---
title: "了解 JDK 中的 GenericArrayType 接口"
date: 2019-12-17T16:08:41+08:00
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

# 1 GenericArrayType 接口的继承关系

![](/hub/2019/December/26.png)

这个接口表示泛型类型的数组，直接这样说可能不是很清晰，看下面的讲解

# 2 GenericArrayType 中的方法含义

```java
public interface GenericArrayType extends Type {
    /** 获得数组元素类型，其中包含泛型相关的信息 */
    Type getGenericComponentType();
}
```

# 3 举个例子

```java
public class Main {
    private Map<String, Integer>[] mapArray;
    @Test
    public void testGetGenericComponentType() throws NoSuchFieldException {
        Field mapArrayField = Main.class.getDeclaredField("mapArray");
        Type mapArrayType = mapArrayField.getGenericType();
        GenericArrayType genericArrayType = (GenericArrayType) mapArrayType;
    }
}
```

下图为断点查看的结果，可以很清晰的看到 genericArrayType 中存储了 mapArray 的泛型信息和原来的类型

![](/hub/2019/December/27.png)

# 参考文章

1. [我眼中的 Java-Type 体系 (2)](https://www.jianshu.com/p/e8eeff12c306)
---
title: "了解 JDK 中的 ParameterizedType 接口"
date: 2019-12-17T13:08:36+08:00
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

# 1 ParameterizedType 接口的继承关系

![](/hub/2019/December/23.png)

Type 是 Java 中所有类型的通用超级接口，继承就完事了。

# 2 ParameterizedType 中的方法含义

```java
/** 下面将 <> 中存储的内容称为泛型列表 */
public interface ParameterizedType extends Type {
    /**  获取范型列表中的实际类型，泛型列表中可能存放了多个泛型信息，所以返回的是数组类型的 Type[] */
    Type[] getActualTypeArguments();

    /** 获取泛型列表前面的实际类型 */
    Type getRawType();

    /** 获取内部类父类的类信息 */
    Type getOwnerType();
}
```

# 3 举个例子

```java
public class Main<T> {

    private Map.Entry<String, Integer> mapEntry = null;

    public static void  main(String[] args) throws NoSuchFieldException {
        // 找 Main 中的 mapEntry 的 Field 信息
        Field fieldMap = Main.class.getDeclaredField("mapEntry");
        // 获取到 ParameterizedType
        Type typeMap = fieldMap.getGenericType();
        ParameterizedType parameterizedTypeMap = (ParameterizedType) typeMap;
        // 下面获取出来的 type 都是 Class
        // 获取泛型列表中的类型
        Type[] types = parameterizedTypeMap.getActualTypeArguments();
        System.out.println(types[0]); // class java.lang.String
        System.out.println(types[1]); // class java.lang.Integer
        // 获取泛型列表前面的信息
        Type rawType = parameterizedTypeMap.getRawType();
        System.out.println(rawType); // interface java.util.Map$Entry
        // 获取内部类父类的类信息
        Type ownerType = parameterizedTypeMap.getOwnerType();
        System.out.println(ownerType); // interface java.util.Map
    }
}
```

# 参考文章

1. [我眼中的 Java-Type 体系 (2)](https://www.jianshu.com/p/e8eeff12c306)
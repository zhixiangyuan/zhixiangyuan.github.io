---
title: "了解 JDK 中的 WildcardType 接口"
date: 2019-12-17T16:42:34+08:00
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

# 1 WildcardType 接口的继承关系

![](/hub/2019/December/28.png)

# 2 WildcardType 中的方法含义

```java
public interface WildcardType extends Type {
    /** 获取泛型变量的上边界 */
    Type[] getUpperBounds();

    /** 获取泛型变量的下边界 */
    Type[] getLowerBounds();
}
```

# 3 举个例子

```java
public class WildcardTypeTest {
    /** 看下面的代码是怎么拿到 Number 的 */
    private List<? extends Number> listNum;
    /** 看下面的代码是怎么拿到 String 的 */
    private List<? super String> listStr;

    public static void main(String[] args) throws NoSuchFieldException {
        // 先拿到 listNum 字段
        Field listNumField = WildcardTypeTest.class.getDeclaredField("listNum");
        // 获取字段的范型类型
        Type listNumType = listNumField.getGenericType();
        // 强转成相应的类型
        ParameterizedType  listNumParameterizedType = (ParameterizedType) listNumType;
        // 取出真实的参数类型
        Type[] listNumTypeArray = listNumParameterizedType.getActualTypeArguments();
        // 强转成特殊的 WildcardType 类型
        WildcardType listNumWildcardType = (WildcardType) listNumTypeArray[0];
        // 取出上界
        Type[] upperBounds = listNumWildcardType.getUpperBounds();
        System.out.println(upperBounds[0]); // class java.lang.Number

        // 先拿到 listStr 字段
        Field listStrField = WildcardTypeTest.class.getDeclaredField("listStr");
        // 获取字段的范型类型
        Type listStrType = listStrField.getGenericType();
        // 强转成相应的类型
        ParameterizedType  listStrParameterizedType = (ParameterizedType) listStrType;
        // 取出真实的参数类型
        Type[] listStrTypeArray = listStrParameterizedType.getActualTypeArguments();
        // 强转成特殊的 WildcardType 类型
        WildcardType listStrWildcardType = (WildcardType) listStrTypeArray[0];
        // 取出下界
        Type[] lowerBounds = listStrWildcardType.getLowerBounds();
        System.out.println(lowerBounds[0]); // class java.lang.String
    }
}
```

# 参考文章

1. [我眼中的 Java-Type 体系 (2)](https://www.jianshu.com/p/e8eeff12c306)
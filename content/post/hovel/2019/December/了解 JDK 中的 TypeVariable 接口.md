---
title: "了解 JDK 中的 TypeVariable 接口"
date: 2019-12-17T14:29:00+08:00
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
markup: mmark
mathjax: false
---

JDK 中的 TypeVariable 翻译过来就是类型变量，其实就是指泛型。

# 1 TypeVariable 接口的继承关系

![](/hub/2019/December/24.png)

GenericDeclaration 这个接口的含义是决定哪些地方可以定义泛型

![](/hub/2019/December/25.png)

通过上图可以看到 Class、Constructor、Method 三个类实现了 GenericDeclaration，说明这三个类上都可以声明泛型

# 2 TypeVariable 中的方法含义

```java
public interface TypeVariable<D extends GenericDeclaration> extends Type, AnnotatedElement {
    /** 获取泛型的上边界 */
    Type[] getBounds();

    /** 获取声明该类型变量实体（即获得类、方法或构造器名） */
    D getGenericDeclaration(); // 

    /** 获取定义泛型时定义的名称，如 K、V */
    String getName(); 

    /** 如果这个这个泛型参数类型的上界用注解标记了，我们可以通过它拿到相应的注解 */
     AnnotatedType[] getAnnotatedBounds();
}
```

# 3 举个例子

```java
/** 声明一个注解 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE_USE)
@interface Custom {}

/** 类型变量的上限可以为多个，必须使用 & 符号相连接，例如 List<T extends Number & Serializable>；其中，& 后必须为接口 */
public class TypeVariableTest<T extends @Custom Number & @Custom Serializable & Comparable> {

    private T t;

    @Test
    public void testGetBounds() throws NoSuchFieldException {
        Field tField = TypeVariableTest.class.getDeclaredField("t");
        TypeVariable typeVariable = (TypeVariable) tField.getGenericType();
        // 获取类型变量 t 的上边界
        Type[] bounds = typeVariable.getBounds();
        for (Type bound : bounds) {
            System.out.println(bound);
        }
        // 输出：
        //     class java.lang.Number
        //     interface java.io.Serializable
        //     interface java.lang.Comparable
    }

    @Test
    public void testGetGenericDeclaration() throws NoSuchFieldException {
        Field tField = TypeVariableTest.class.getDeclaredField("t");
        TypeVariable typeVariable = (TypeVariable) tField.getGenericType();
        // 获取声明该类型变量（T）的实体
        // 比如说这个地方是 TypeVariableTest 这个类声明的，所以获取到的就 TypeVariableTest.class 的信息
        GenericDeclaration genericDeclaration = typeVariable.getGenericDeclaration();
        System.out.println(genericDeclaration); // class me.yuanzx.test.TypeVariableTest
    }

    @Test
    public void testGetName() throws NoSuchFieldException {
        Field tField = TypeVariableTest.class.getDeclaredField("t");
        TypeVariable typeVariable = (TypeVariable) tField.getGenericType();
        // 获取范型在源码中定义时写的名字，比如这里定义的是 T
        String name = typeVariable.getName();
        System.out.println(name); // T
    }

    @Test
    public void testGetAnnotatedBounds() throws NoSuchFieldException {
        Field tField = TypeVariableTest.class.getDeclaredField("t");
        TypeVariable typeVariable = (TypeVariable) tField.getGenericType();
        // 可以获取到泛型上面标的注解
        AnnotatedType[] annotatedBounds = typeVariable.getAnnotatedBounds();
        for (AnnotatedType annotatedBound : annotatedBounds) {
            System.out.print(annotatedBound.getType().getTypeName() + ": ");
            if (annotatedBound.getAnnotations().length != 0) {
                System.out.println(annotatedBound.getAnnotations()[0]);
            } else {
                System.out.println();
            }
        }
        // 输出：
        //     java.lang.Number: @me.yuanzx.test.Custom()
        //     java.io.Serializable: @me.yuanzx.test.Custom()
        //     java.lang.Comparable: 
    }
}
```

# 参考文章

1. [秒懂 Java 类型（Type）系统](https://zhuanlan.zhihu.com/p/64584427)
2. [我眼中的 Java-Type 体系 (2)](https://www.jianshu.com/p/e8eeff12c306)
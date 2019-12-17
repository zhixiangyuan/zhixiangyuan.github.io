---
title: "范型中的 extends 与 super"
date: 2019-12-17T10:44:08+08:00
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
draft: true
---

# 1 范型的缺点

之前的博客中数组的协变与范型的协变提到过这个缺点，范型是不支持协变的，下面再通过一个例子来回顾一下。

```java
/** 先定义一个水果盘子 */
public class FruitPlate<T> {
    private T item;

    public FruitPlate(T t) { item = t; }

    public void set(T t) { item = t; }

    public T get() { return item; }
}
/** 定义一个食物基类 */ 
public class Food { }
/** 然后定义一个水果基类 */
public class Fruit extends Food { }
/** 定义两个水果 */
public class Apple extends Fruit { }
public class Orange extends Fruit { }

public class Main {
    public static void main(String[] args) {
        // 这样写的话是不会报错的，这是由于 ? extends Fruit 没有完全生效，
        // 这里我理解的可能也有问题，不过构造函数确实很特殊，在构造函数上
        // 传入 Fruit 子类不会报错
        FruitPlate<? extends Fruit> p = new FruitPlate<>(new Apple());
        // 好了，我们要说的重点不是上面这个问题，只是顺带一提，上面的情况我也不是很懂为什么能用，不过这里记录一下，将来看 JVM 实现可能能找到答案
        // 我们要说的是下面这种情况
        

    }
}
```

# 参考文章

1. [【Java】泛型中 extends 和 super 的区别？](https://itimetraveler.github.io/2016/12/27/%E3%80%90Java%E3%80%91%E6%B3%9B%E5%9E%8B%E4%B8%AD%20extends%20%E5%92%8C%20super%20%E7%9A%84%E5%8C%BA%E5%88%AB%EF%BC%9F/)
---
title: "谈谈 CAS"
date: 2019-12-09T15:04:35+08:00
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

CAS 其实指的就是 sun.misc.Unsafe 类中的一系列 compareAndSwap 方法

例如 compareAndSwapInt 方法

```java
public final native boolean compareAndSwapInt(
    Object o, long offset, int expected, int x
);
```

这种方法提供四个参数，分别是需要修改值的对象 o，需要改的值在对象中的偏移量 offset，期望原值 expected，修改之后的值 x

实现的效果就是内存（这里的内存应该是堆内存）中的值如果是指定的 expected，那么就将值修改为 x，而这个过程是通过 CPU 的 cmpxchg 指令（比较并交换）实现的，是一个原子操作。

将来深挖 openjdk 中的相关实现时，再补上 native 内部的实现。

# 1 CAS 的问题

## 1.1 ABA 问题

CAS 需要在操作值的时候检查内存值是否发生变化，没有发生变化才会更新内存值。但是如果内存值原来是 A，后来变成了 B，然后又变成了 A，那么 CAS 进行检查时会发现值没有发生变化，但是实际上是有变化的。ABA 问题的解决思路就是在变量前面添加版本号，每次变量更新的时候都把版本号加一，这样变化过程就从 “A－B－A” 变成了 “1A－2B－3A”。

## 1.2 循环时间长开销大

通常我们使用 CAS 的时候都换在 CAS 修改失败之后进行重试（也就是自旋），所以 CAS 操作如果长时间不成功，会导致其一直自旋，给 CPU 带来非常大的开销。

## 1.3 只能保证一个共享变量的原子操作

可以看到 CAS 的方法只能指定一个对象加上一个值在对象中的偏移量，所以 CAS 方法只对单个对象起作用。

# 参考文章

1. [【基本功】不可不说的 Java “锁” 事](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651749434&idx=3&sn=5ffa63ad47fe166f2f1a9f604ed10091&chksm=bd12a5778a652c61509d9e718ab086ff27ad8768586ea9b38c3dcf9e017a8e49bcae3df9bcc8&scene=38#wechat_redirect)
   - 深度好文，强烈推荐
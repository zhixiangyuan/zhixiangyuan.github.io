---
title: "Java 中的 WeakReference 和 SoftReference"
date: 2019-11-28T16:48:50+08:00
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

# 1 WeakReference

当一个对象没有被强引用时，只有 WeakReference，那么在下一次 GC 时会被回收。

```java
    public static void main(String[] args) {
        Object referent = new Object();
        WeakReference<Object> weakReference = new WeakReference<>(referent);
        referent = null;
        System.gc();
        System.out.println(weakReference.get() == null ? "删除了" : "没删除");
    }

OUTPUT:
删除了
```

# 2 SoftReference

SoftReference 于 WeakReference 的特性基本一致， 最大的区别在于 SoftReference 会尽可能长的保留引用直到 JVM 内存不足时才会被回收 (虚拟机保证), 这一特性使得 SoftReference 非常适合缓存应用

```java
    public static void main(String[] args) {
        Object referent = new Object();
        SoftReference<Object> softReference = new SoftReference<>(referent);
        referent = null;
        System.gc();
        System.out.println(softReference.get() == null ? "删除了" : "没删除");
    }

OUTPUT:
没删除
```

# 参考文章

1. [理解 Java 的 GC 与 幽灵引用](/https://www.iteye.com/topic/401478)
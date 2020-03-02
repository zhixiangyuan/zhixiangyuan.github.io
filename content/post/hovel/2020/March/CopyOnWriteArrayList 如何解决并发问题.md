---
title: "CopyOnWriteArrayList 如何解决并发问题"
date: 2020-03-02T09:57:46+08:00
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

CopyOnWriteArrayList 解决并发问题的关键在于其 add 的方法，解决方式便是写时复制，直接看源码。

```java
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        // 上锁
        lock.lock();
        try {
            // 获取老数组
            Object[] elements = getArray();
            int len = elements.length;
            // 拷贝出一个 len + 1 的新数组
            Object[] newElements = Arrays.copyOf(elements, len + 1); 
            // 像新数组中添加元素
            newElements[len] = e;
            // 将引用指向新数组
            setArray(newElements); 
            return true;
        } finally {
            // 解锁
            lock.unlock();
        }
    }
```

从源码中可以清晰的看到，解决方式就是在写的时候上锁，然后复制出一个新数组。
---
title: "CompletableFuture 小记"
date: 2020-03-01T15:54:03+08:00
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

# 前言

本文主要讲解使用，同时使用了 junit 框架


```xml
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
<!--            <scope>test</scope>-->
        </dependency>
    </dependencies>
```

# 1 completedFuture

```java
    static void completedFutureExample() {
        // 生成一个返回结果为 message 的 CompletableFuture
        CompletableFuture<String> cf = CompletableFuture.completedFuture("message");
        // 方法已经执行完了
        assertTrue(cf.isDone());
        // 返回值为 message
        assertEquals("message", cf.getNow(null));
    }
```


# 参考文章

1. [20 Examples of Using Java’s CompletableFuture](https://mahmoudanouti.wordpress.com/2018/01/26/20-examples-of-using-javas-completablefuture/)
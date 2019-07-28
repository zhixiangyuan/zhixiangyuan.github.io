---
title: "多线程中会出现 Bug 的代码"
date: 2019-07-27T17:29:51+08:00
keywords: []
description: ""
tags: [

]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
draft: true
---

# 序列号生成器

```java
public class UnsafeSequence {
    private int value;

    /** 返回一个唯一值 */
    public int getNext() {
        return value++;
    }
}
```

// todo 
//      1. 在多线程中会出现的问题
//      2. 修复的方法
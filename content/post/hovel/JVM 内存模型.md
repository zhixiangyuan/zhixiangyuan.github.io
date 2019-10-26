---
title: "JVM 内存模型"
date: 2019-10-26T21:42:18+08:00
keywords: []
description: ""
tags: [
    "JVM"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

JVM 的内存模型分为主内存和线程的工作内存两大块，这属于一个抽象的内存模型，用于解决并发问题，与 JVM 中的内存结构如堆、栈没什么关系，不要弄混。

![](/hub/2019/october/8.png)


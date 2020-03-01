---
title: "ForkJoinPool 小记"
date: 2020-03-01T18:46:31+08:00
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

阅读本文最少需要知道 `ThreadPoolExecutor` 的使用方式，本文介绍 ForkJoinPool 的使用方式和大概的工作原理

# 正文

对于 `ForkJoinPool`，先看一下它的 uml 图

![](/hub/2020/March/2.png)

由上图可以发现，`ForkJoinPool` 可以被当作是一个线程池来使用。它的特殊之处，在于不仅可以当作正常的线程池来使用，同时支持提交 `ForkJoinTask` 的任务。


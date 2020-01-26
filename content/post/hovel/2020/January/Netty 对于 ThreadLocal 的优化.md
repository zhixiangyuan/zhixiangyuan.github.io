---
title: "Netty 对于 ThreadLocal 的优化"
date: 2020-01-27T20:58:11+08:00
keywords: []
description: ""
tags: [
    "netty"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

对于 ThreadLocal 这个使用起来很方便的线程级别的全局变量，其本身有两点是需要注意的

第一，它在线程池的环境中使用时是可能会导致内存泄漏的，这一点如果大家不了解可以去查一下相关的资料。

第二，它内部的实现是通过数组存放相应的数据的，如果出现 hash 冲突会使用线性探测法解决 hash 冲突。

Netty 中的优化便是围绕这两点进行展开的

# 1 如何解决内存泄漏问题


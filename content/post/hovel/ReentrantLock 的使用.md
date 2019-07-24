---
title: "ReentrantLock 的使用"
date: 2019-07-24T21:31:26+08:00
keywords: []
description: ""
tags: [
    "Java 并发编程"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
draft: true
---

# 1 ReentrantLock 的作用

Java 中已经有了 synchronized 的来进行隐式的加锁和解锁，那还为什么还要引入 ReentrantLock 这把锁呢。这主要是 synchronized 的加锁和解锁操作并不灵活，
---
title: "Effective Java"
date: 2019-07-29T14:18:11+08:00
keywords: []
description: ""
tags: [

]
categories: [
    "杂货铺"
]
autoCollapseTocv: false
author: "yuanzx"
draft: true
---

# 1 考虑使用静态工厂方法替代构造器

这里所说的静态工厂并不等同于设计模式中的静态工厂

## 1.1 静态工厂方法的优点

1. 静态工厂方法有自己的名字，在阅读代码的时候就可以起到见名知义的效果。
2. 通过静态工厂不需要每次都创建一个新对象，可以仅在必要的时候创建对象。
3. 可以根据输入参数的不同返回不同的子类。

## 1.2 静态工厂方法的缺点
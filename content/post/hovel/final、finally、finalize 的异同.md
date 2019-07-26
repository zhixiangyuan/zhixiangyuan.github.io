---
title: "final、finally、finalize 的异同"
date: 2018-10-17T16:40:23+08:00
keywords: []
description: ""
tags: [

]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 final 

final 可以用来修饰类、方法、变量

## 1.1 final 修饰类

final 修饰 class 代表类不可以被继承

## 1.2 final 修饰变量

final 修饰的变量在初始化之后不能被再次赋值

## 1.3 final 修饰方法

final 修饰方法的方法不能被子类重写

# 2 finally

finally 是 Java 保证代码一定被执行的一种机制，一般在 finally 中进行关闭连接、解锁的操作。

# 3 finalize 

finalize 是 java.lang.Object 中的一个方法，它的设计目的是保证对象在被垃圾手机前完成特定资源的回收，finalize 函数现在已经不推荐使用了

## 3.1 finalize 方法的缺点

1. 不能保证被执行
    - 程序被强制终结的时候，不会调用 finalize 方法
    - finalize 方法执行时抛出异常？？？
2. 不能保证被即使执行
    - 对时间敏感的任务不能在 finalize 中执行，譬如文件资源回收
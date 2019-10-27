---
title: "Exception 与 Error 的异同"
date: 2018-10-16T11:26:50+08:00
keywords: []
description: ""
tags: [
    "Java"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

Exception 和 Error 都是继承了 Throwable 类，在 Java 中只有继承了 Throwable 的类才可以被 throw 或者 catch，它是异常处理机制的基本组成类型。Exception 是程序正常运行中，可以预料的意外情况，应该被捕获并进行相应的处理。Error 是在正常情况不应该出现的，如果出现了 Error 很可能会导致程序比如 JVM 处于非正常的、不可恢复的状态，所以 Error 不方便也不需要捕获。
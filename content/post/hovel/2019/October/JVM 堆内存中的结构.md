---
title: "JVM 堆内存中的结构"
date: 2019-10-26T20:43:53+08:00
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

# 1 堆内存结构

堆内存中分为三块，分别是年轻代、老年代和方法区，Java 堆不需要连续的内存，可以动态增加内存，增加失败则抛出 OutOfMemoryError 异常

![堆内存中的结构](/hub/2019/october/3.png)

# 1.1 年轻代与老年代

年轻代中又分为三块 edan、survivor0、survivor1，这里面使用了标记整理算法，每次只使用 edan 与一块 survivor，当 edan 中满了之后便触发一次 GC 将 edan 与 survivor0 中存活的对象整理到 survivor1 中，然后将 edan 和 survivor0 清理干净，之后便使用 edan 和 survivor1，同时在整理的时候，这些被整理的对象年龄加一，当年龄达到一定岁数时便转移到老年代。

![1](/hub/2019/october/4.png)
![2](/hub/2019/october/5.png)
![3](/hub/2019/october/6.png)

在年轻代中发生的 GC 称为 Minor GC，在老年代中发生的 GC 称为 Major GC

![4](/hub/2019/october/7.png)

# 2 方法区

方法区用于存放已经被加载的类的信息、常量、静态变量、即时编译器编译后的代码等数据。堆这块区域进行垃圾回收的主要目标是对常量池的回收和对类的卸载，HotSpot 虚拟机将它当作永久代来进行垃圾回收。方法区逻辑上属于堆的一部分，但是为了与堆进行区分，通常由叫做“非堆”。
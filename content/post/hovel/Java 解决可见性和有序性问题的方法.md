---
title: "Java 解决可见性和有序性问题的方法"
date: 2019-07-16T17:24:13+08:00
keywords: []
description: ""
tags: [
    "并发编程"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1. Happens-Before 规则

Java 通过提出 Happens-Before 规则来解决可见性和有序性的问题，Happens-Before 的意思是前面一个操作的结果对后续操作是可见的。

## 1.1 程序的顺序性规则

程序按照代码顺序执行，前面的代码 Happens-Before 于后面的代码。

## 1.2 volatile 变量规则

对一个 volatile 变量的写，Happens-Before 于任意后续对这个 volatile 的读

## 1.3 传递性规则

如果 A Happens-Before B，且 B Happens-Before C，那么 A Happens-Before C

结合上面三个规则，可以看如下代码

```java
class VolatileExample {
  private int x = 0;
  private volatile boolean v = false;
  public void writer() {
    x = 42; // 1
    v = true; // 2
  }
  public int reader() {
    if (v == true) { // 3
      return x; // 4
    }
  }
}
```

如果线程 A 调用 writer() 执行到了 2，线程 B 调用 reader() 执行到了 4，那么线程 B return 的值是多少呢？

1. 程序的顺序性规则：1 Happens-Before 2，3 Happens-Before 4
2. volatile 变量规则：2 Happens-Before 3
3. 传递性规则：1 Happens-Before 4

得到的结果就是 return 42

volatile 的底层实现就是强制所修饰的变量及它前边的变量刷新至内存，并且 volatile 禁止了指令的重排序

## 1.4 管程中锁的规则

对一个监视器的解锁，happens-before 于随后对这个监视器的加锁

```java
// 也就是线程 A 执行了以下代码释放锁之后，
// 线程 B 执行如下代码的时候能观察到 x 的变化
synchronized (this) {
  if (this.x < 12) {
    this.x = 12; 
  }  
}
```

## 1.5 线程 start () 规则

如果线程 A 调用了线程 B 的 start() 方法，线程 A 在调用 start() 方法之前的修改线程 B 都能看到

## 1.6 线程 join () 规则

如果线程 A 调用了线程 B 的 join() 方法，在线程 A 调用 join() 方法之后会等待线程 B 结束，在线程 B 结束之后，线程 A 能够看到线程 B 的所有改动。

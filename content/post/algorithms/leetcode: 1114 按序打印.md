---
title: "Leetcode: 1114 按序打印"
date: 2019-08-13T11:05:30+08:00
keywords: []
description: ""
tags: [
    "leetcode"
]
categories: [
    "数据结构与算法"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 题目

[按序打印](https://leetcode-cn.com/problems/print-in-order/)

# 2 解

## 2.1 我的解

```java

class Foo {
    // 题目确保使用单例模式
	public Foo() {
            
    }  
    private boolean firstRun = false;
    private boolean secondRun = false;

    public synchronized void first(Runnable printFirst) throws InterruptedException {
        // printFirst.run() outputs "first". Do not change or remove this line.
        printFirst.run();
        firstRun = true;
        // 通知等待的方法运行
        this.notifyAll();
    }

    public synchronized void second(Runnable printSecond) throws InterruptedException {
        while(!firstRun) {
            // 如果 first 方法没有运行则等待
            this.wait();
        }
        // printSecond.run() outputs "second". Do not change or remove this line.
        printSecond.run();
        secondRun = true;
        // 通知等待的方法运行
        this.notifyAll();
    }

    public synchronized void third(Runnable printThird) throws InterruptedException {
        while(!secondRun) {
            // 如果 second 方法没有运行则等待
            this.wait();           
        }
        // printThird.run() outputs "third". Do not change or remove this line.
        printThird.run();
    }
}
```

## 2.2 别人的解

通过倒计时的方式实现

```java
import java.util.concurrent.CountDownLatch;
class Foo {

    CountDownLatch latch1;
    CountDownLatch latch2;
    public Foo() {
        latch1 = new CountDownLatch(1);
        latch2 = new CountDownLatch(2);
    }

    public void first(Runnable printFirst) throws InterruptedException {
        
        // printFirst.run() outputs "first". Do not change or remove this line.
        printFirst.run();
        latch1.countDown();
        latch2.countDown();
    }

    public void second(Runnable printSecond) throws InterruptedException {
        latch1.await();
        // printSecond.run() outputs "second". Do not change or remove this line.
        printSecond.run();
        latch2.countDown();
    }

    public void third(Runnable printThird) throws InterruptedException {
        latch2.await();
        // printThird.run() outputs "third". Do not change or remove this line.
        printThird.run();
    }
}
```
---
title: "Leetcode: 1115 交替打印 FooBar"
date: 2019-08-14T20:54:44+08:00
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

[交替打印 FooBar](https://leetcode-cn.com/problems/print-foobar-alternately/)

# 2 解

## 2.1 我的解

思路，先打印 Foo，然后通知另一个线程打印 Bar

```java
class FooBar {
    private int n;

    public FooBar(int n) {
        this.n = n;
    }
    
    private boolean isPrintFoo = false;

    public synchronized void foo(Runnable printFoo) throws InterruptedException {
        
        for (int i = 0; i < n; i++) {
        	// printFoo.run() outputs "foo". Do not change or remove this line.
        	printFoo.run();
            isPrintFoo = true;
            this.notify();
            this.wait();
        }
    }

    public synchronized void bar(Runnable printBar) throws InterruptedException {
        
        for (int i = 0; i < n; i++) {
            if (!isPrintFoo) {
                this.wait();
            }
            // printBar.run() outputs "bar". Do not change or remove this line.
        	printBar.run();
            isPrintFoo = false;
            this.notify();
        }
    }
}
```

## 2.2 别人的解

用信号量来实现

```java
import java.util.concurrent.Semaphore;

/*
semaphore 与 semaphore2 在下面四种状态转换

semaphore semaphore2 代码位置
1         0          准备打印 Foo
0         0          打印 Foo
0         1          准备打印 Bar
0         0          打印 Bar
*/

class FooBar {
    private int n;
    private Semaphore semaphore = new Semaphore(1);
    private Semaphore semaphore2 = new Semaphore(0);
    public FooBar(int n) {
        this.n = n;
    }

    public void foo(Runnable printFoo) throws InterruptedException {
        
        for (int i = 0; i < n; i++) {
            semaphore.acquire();
        	// printFoo.run() outputs "foo". Do not change or remove this line.
        	printFoo.run();
        	semaphore2.release();
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        
        for (int i = 0; i < n; i++) {
            semaphore2.acquire();
            // printBar.run() outputs "bar". Do not change or remove this line.
        	printBar.run();
            semaphore.release();
        }
    }
}
```
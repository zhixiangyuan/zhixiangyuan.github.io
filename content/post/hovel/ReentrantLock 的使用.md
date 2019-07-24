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
---

# 1 ReentrantLock 的作用

Java 中已经有了 synchronized 的来进行隐式的加锁和解锁，那还为什么还要引入 ReentrantLock 这把锁呢。这主要是 synchronized 的加锁和解锁操作并不灵活，ReentrantLock 是一把可重入的锁，它提供了可中断、可限时、公平锁的特点。

# 2 ReentrantLock 的特性

## 2.1 可中断、可限时

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    ...
}
```

从 ReentrantLock 的源码中，我们能看到 ReentrantLock 实现了 Lock，下面我们看一下 Lock 的方法

```java
public interface Lock {

    /**
     * 加锁，没抢到锁则等待
     */
    void lock();

    /**
     * 加锁，没抢到锁则等待，但是可以感知到 Interrupted
     *
     * @throws InterruptedException 接收到 Interrupted 会抛出 InterruptedException 异常
     */
    void lockInterruptibly() throws InterruptedException;

    /**
     * 尝试加锁
     * 
     * @return true 加锁成功 / false 加锁失败
     */
    boolean tryLock();

    /**
     * 尝试加锁
     *
     * @param time 等待锁的时间
     * @param unit 时间单位
     * @return true 加锁成功 / false 加锁失败
     *
     * @throws InterruptedException 接收到 Interrupted 会抛出 InterruptedException 异常
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    /**
     * 释放锁
     */
    void unlock();
    ...
}
```

可以看到 Lock 中提供了加锁、解锁、感知中断的加锁、感知超时的加锁方法，非常的灵活。

## 2.2 公平锁

开启公平锁功能意味着先来的线程可以先获取到锁，防止某些线程点背，永远获取不到锁。

开启公平锁的功能也很简单，调用 ReentrantLock 中提供的构造方法，传入 true 既使用公平锁，false 既非公平锁。

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    ...
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
    ...
}
```

# 参考资料  

1. [ReentrantLock 的使用](https://www.jianshu.com/p/155260c8af6c)
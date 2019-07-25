---
title: "ReentrantLock 详解"
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

Java 中已经有了 synchronized 的来进行隐式的加锁和解锁，那还为什么还要引入 ReentrantLock 这把锁呢。这主要是 synchronized 的加锁和解锁操作并不灵活，ReentrantLock 是一把可重入的锁，它提供了非阻塞的获取锁、获取锁时可中断、可限时、公平锁的特点。

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
     * 非阻塞的获取锁
     * 
     * @return true 加锁成功 / false 加锁失败
     */
    boolean tryLock();

    /**
     * 非阻塞的获取锁
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

# 3 ReentrantLock 的使用

以下是一个编程范式，先创建一个锁，然后在 try 外边加锁，在 finally 中解锁，防止锁无法释放。这里有一点需要注意，不要将 `lock.lock();` 写在 try 里面，这可能会导致在没有加锁的时候抛出异常，然后触发 `lock.unlock();`，由于这时候没有加锁就释放锁，会导致 `lock.unlock();` 无故抛出 `java.lang.IllegalMonitorStateException` 的异常。

```java
    Lock lock = new ReentrantLock();
    lock.lock();
    try {
        ...
    } finally {
        lock.unlock();
    }
```

# 4 ReentrantLock 源码解析

## 4.1 ReentrantLock 中的锁结构

ReentrantLock 中实现了两把锁， 一把是公平锁，另一把是非公平锁。Sync 继承 AbstractQueuedSynchronizer 

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    /**
     * 默认构造函数使用非公平锁
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    /**
     * 这个构造函数提供选择公平锁与非公平锁的方式
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
    /**
     * 对 AbstractQueuedSynchronizer 的抽象实现
     */
    abstract static class Sync extends AbstractQueuedSynchronizer {
        ...
    }
    /**
     * 非公平锁
     */
    static final class NonfairSync extends Sync {
        ...
    }
    /**
     * 公平锁
     */
     static final class FairSync extends Sync {
         ...
     }
     ...
}
```

## 4.2 lock() 的实现方式

### 4.2.1 非公平锁

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    /**
     * 非公平锁
     */
    static final class NonfairSync extends Sync {
        final void lock() {
            // 比较然后交换，如果当前锁的 state 为 0，则将其置为 1
            if (compareAndSetState(0, 1))
                // 将当前线程设置为占有锁的线程
                setExclusiveOwnerThread(Thread.currentThread());
            else
                // 如果设置 state 失败，则将其放入队列
                // acquire 方法的实现在 AbstractOwnableSynchronizer 中
                acquire(1);
        }
        ...
    }
    ...
}
```

### 4.2.2 公平锁

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    /**
     * 公平锁
     */
     static final class FairSync extends Sync {
        final void lock() {
            acquire(1);
        }
        protected final boolean tryAcquire(int acquires) {
            // 获取当前线程
            final Thread current = Thread.currentThread();
            // 获取当前 state
            int c = getState();
            if (c == 0) {
                // 如果当前阻塞队列上没有先来的线程在等待
                if (!hasQueuedPredecessors() &&
                    // 比较然后交换，如果当前锁的 state 为 0，则将其置为 acquires
                    compareAndSetState(0, acquires)) {
                    // 将当前线程设置为占有锁的线程
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                // 如果当前线程就是获取到锁的线程，这里
                // 将 state 加 1
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                // 则将 nextc 设置给 state
                setState(nextc);
                return true;
            }
            // 如果不满足上述状态，则获取锁失败
            // 后续这个线程会被加入阻塞队列
            return false;
        }
     }
    ...
}
```




# 参考资料  

1. [ReentrantLock 的使用](https://www.jianshu.com/p/155260c8af6c)
2. [AbstractQueuedSynchronizer 超详细原理解析](https://juejin.im/post/5c3ac10351882524bb0b337f)
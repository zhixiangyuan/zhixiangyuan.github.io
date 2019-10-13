---
title: "ReentrantLock 详解"
date: 2019-07-24T21:31:26+08:00
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

>本文基于 JDK 1.8

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

以下分析的源码为公平锁源码

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    /**
     * 公平锁
     */
     static final class FairSync extends Sync {
        final void lock() {
            // acquire 的实现在 AbstractQueuedSynchronizer 中
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
                    // 执行 CAS 操作，如果当前锁的 state 为 0，则将其置为 acquires
                    // 可能会有多个线程执行这里，所以使用 CAS 防止出现并发问题
                    compareAndSetState(0, acquires)) {
                    // 将当前线程设置为占有锁的线程
                    // 这句代码只有一个线程能够抵达
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
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    // 判断当前线程前面是否有别的线程
    public final boolean hasQueuedPredecessors() {
        Node t = tail; 
        Node h = head;
        Node s;
        return h != t &&
        // 主要看 s.thread != Thread.currentThread()，如果 head 的 next 线程
        // 不是当前线程，则说明当前线程前面还有别的线程
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
    public final void acquire(int arg) {
        // 尝试获取锁，获取失败则将线程加入到等待队列
        if (!tryAcquire(arg) &&
            // addWaiter 方法将线程加入到等待队列
            // acquireQueued
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
    // --- 下面看一下 addWaiter 的实现
    // 则将线程加入到等待队列
    private Node addWaiter(Node mode) {
        // 创建节点
        Node node = new Node(Thread.currentThread(), mode);
        // 获取队尾
        Node pred = tail;
        if (pred != null) {
            // 先用快速入队法试一下
            // 当前线程节点前置节点指向 tail
            node.prev = pred;
            // CAS 将队尾指针指向当前线程节点
            if (compareAndSetTail(pred, node)) {
                // 指向成功则将旧队尾的 next 指向新的 node
                pred.next = node;
                return node;
            }
        }
        // 快速入队法入队失败，使用复杂一点的算法入队
        enq(node);
        return node;
    }
    // 复杂一点的算法入队
    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { 
                // 初始化的时候才会走到这里
                // 队列初始化一个空队头
                if (compareAndSetHead(new Node()))
                    // CAS 初始化成功，tail 置成 head
                    // 需要注意的是 head 是一个哨兵的作用，并不代表某个要获取锁的线程节点
                    tail = head;
            } else {
                // 和 addWaiter 中的算法一致，不过有了外侧的无限循环，不断的尝试，
                // 形成自旋锁
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
    // --- 下面看一下 acquireQueued 的实现
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            // 一直执行，直到获取锁，返回.
            for (;;) {
                // 获取 node 节点的前一个节点
                final Node p = node.predecessor();
                // node 的前驱节点是 head，就说明 node 是将要获取锁的下一个节点，
                // 那么调用 tryAcquire 方法尝试获取锁 
                if (p == head && tryAcquire(arg)) {
                    // 成功获取到锁，则将字节设置为 head
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    // 此时，还没有进入阻塞状态，所以直接返回 false，表示不需要
                    // 调用 selfInterrupt 函数
                    return interrupted;
                }
                // shouldParkAfterFailedAcquire 判断是否要进入阻塞状态
                if (shouldParkAfterFailedAcquire(p, node) &&
                    // 调用 parkAndCheckInterrupt 进行阻塞
                    // 返回值为是否中断
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
    // 将 node 设置为头节点
    private void setHead(Node node) {
        head = node;
        node.thread = null;
        node.prev = null;
    }
    // 检查失败获取锁的节点是否需要阻塞
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        // 获取前驱节点的状态
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            // 前驱节点在等待锁释放的通知，所以当前节点可以阻塞
            return true;
        if (ws > 0) {
            // 前驱节点处于取消锁的状态，所以可以跳过去
            do {
                // 将当前节点的前驱节点设置为前驱节点的前驱节点
                node.prev = pred = pred.prev;
                // 判断前驱节点的状态，如果前驱节点也处于等待锁释放的通知，
                // 那么继续跳过去
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            // 将上一个节点的状态设置为 Node.SIGNAL
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        // 返回 false 不阻塞
        return false;
    }
    // 阻塞调用线程，同时检查是否中断
    private final boolean parkAndCheckInterrupt() {
        // 将 AQS 对象自己传入，进入阻塞
        LockSupport.park(this);
        return Thread.interrupted();
    }
    // todo 这个方法还没解析
    private void cancelAcquire(Node node) {
        // Ignore if node doesn't exist
        if (node == null)
            return;
        // node 节点内的线程置为空
        node.thread = null;

        // 跳过取消的节点，找到该节点的前驱节点
        Node pred = node.prev;
        // 找到 pred 节点前面
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;

        // predNext is the apparent node to unsplice. CASes below will
        // fail if not, in which case, we lost race vs another cancel
        // or signal, so no further action is necessary.
        Node predNext = pred.next;

        // Can use unconditional write instead of CAS here.
        // After this atomic step, other Nodes can skip past us.
        // Before, we are free of interference from other threads.
        node.waitStatus = Node.CANCELLED;

        // If we are the tail, remove ourselves.
        if (node == tail && compareAndSetTail(node, pred)) {
            compareAndSetNext(pred, predNext, null);
        } else {
            // If successor needs signal, try to set pred's next-link
            // so it will get one. Otherwise wake it up to propagate.
            int ws;
            if (pred != head &&
                ((ws = pred.waitStatus) == Node.SIGNAL ||
                 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                pred.thread != null) {
                Node next = node.next;
                if (next != null && next.waitStatus <= 0)
                    compareAndSetNext(pred, predNext, next);
            } else {
                unparkSuccessor(node);
            }

            node.next = node; // help GC
        }
    }
    static final class Node {
        // 获取当前节点的前驱节点
        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }
        ...
    }
    ...
}
```

## 4.3 unlock() 的实现方式

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    public void unlock() {
        // 释放锁
        sync.release(1);
    }
    public final boolean release(int arg) {
        // 尝试释放锁，tryRelease 这个方法在 Sync 中重写了
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                // 唤醒 head 的后继节点
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
    abstract static class Sync extends AbstractQueuedSynchronizer {
        protected final boolean tryRelease(int releases) {
            // state - releases
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            // 当 c 等于 0 时，表示锁已经被释放了，否则表示当前线程
            // 有多次 lock 操作
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            // 只有上了锁的线程可以走到这里，所以不需要使用 CAS 操作
            setState(c);
            return free;
        }
        ...
    }
    private void unparkSuccessor(Node node) {
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        // 获取节点的后继节点
        Node s = node.next;
        // 判断后继节点是否为空或者后继节点的状态是否为 CANCELLED
        if (s == null || s.waitStatus > 0) {
            // 如果为空或已取消，将后继节点置为 null
            s = null;
            // 从尾节点从后向前遍历直到节点为空或者抵达当前节点为止
            for (Node t = tail; t != null && t != node; t = t.prev)
                // 如果此时节点的状态小于等于 0 
                if (t.waitStatus <= 0)
                    // 将此节点赋给 s，后面将唤醒这个节点中的线程
                    s = t;
        }
        if (s != null)
            // 节点不为空，唤醒节点中的线程
            LockSupport.unpark(s.thread);
    }
    ...
}
```



# 参考资料  

1. [ReentrantLock 的使用](https://www.jianshu.com/p/155260c8af6c)
2. [AbstractQueuedSynchronizer 超详细原理解析](https://juejin.im/post/5c3ac10351882524bb0b337f)
3. [AQS 源码分析 - 独占模式](https://mingshan.fun/2019/01/25/aqs-exclusive/)

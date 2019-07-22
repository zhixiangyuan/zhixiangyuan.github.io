---
title: "Java 线程的生命周期"
date: 2019-07-21T20:09:56+08:00
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

Java 语言中的线程本质上就是操作系统的线程，Java 创建线程同样是调用操作系统的 API 来创建线程。所以，了解 Java 线程的生命周期的第一步是了解操作系统线程的生命周期。

# 1 操作系统线程的生命周期

了解生命周期总共分为两个步骤，一，了解生命周期有哪些状态，二，了解各个状态之间的转换。

线程的生命周期总共拥有五个状态，分别是：初始状态、可运行状态、运行状态、休眠状态和终止状态。

![线程状态转换图 —— 五态模型](/media/hovel/20.png)

1. 初始状态：这个状态指的是线程被创建，但是还不允许分配 CPU 执行。这个状态属于编程语言特有的，这里所说的被创建，仅仅是在编程语言层面被创建，而在操作系统层面，真正的线程还没有创建。
2. 可运行状态：指的是线程可以分配 CPU 执行。在这种状态下，真正的操作系统线程已经被成功创建，可以分配 CPU 执行。
3. 运行状态：线程处于 CPU 当中正在运行。
4. 休眠状态：运行状态的线程如果调用一个阻塞的 API （例如以阻塞方式读文件）或者等待某个事件（例如条件变量），那么线程的状态就会转换到休眠状态，同时释放 CPU 的使用权，休眠状态的线程永远没有机会获得 CPU 使用权。
5. 终止状态：线程执行完或者出现异常就会进入终止状态，终止状态的线程不会切换到其他的任何状态，进入终止状态也就意味着线程的生命周期结束了。

# 2 Java 中线程的生命周期

Java 语言将可运行状态和运行状态合并了，这两个状态在操作系统调度层面游泳，而 JVM 层面不关心这两个状态，因为 JVM 把线程调度交给操作系统处理了。同时，Java 语言细化了休眠状态，Java 将休眠状态细化为了阻塞状态、无限时等待、有限时等待。

在 Java 语言当中线程总共有六种状态，分别是：

1. NEW：初始状态
2. RUNNABLE：可运行/运行状态
3. BLOCKED：阻塞状态
4. WAITING：无限时等待
5. TIMED_WAITING：有限时等待
6. TERMINATED：终止状态

BLOCKED、WAITING、TIMED_WAITING 在操作系统层面对应休眠状态，也就是说只要 Java 线程处于这三种状态之一，那么这个线程就永远没有 CPU 的使用权。

这六种状态可以在 `java.lang.Thread.State` 中明确定义了出来。

![Java 中的线程状态转换图](/media/hovel/21.png)

## 2.1 RUNNABLE 与 BLOCKED 的状态转换

只有一种场景会触发这种转换，就是线程等待 synchronized 的隐式锁。synchronized 修饰的方法、代码块同一时间只允许一个线程执行，其他线程只能等待，等待的线程就会从 RUNNABLE 转换到 BLOCKED。当线程获取到 synchronized 隐式锁的时候，就会从 BLOCKED 转换到 RUNNABLE 状态。

```java
public class Main {
    public static void main(String[] args) {
        // 创建两个线程，一个抢到锁然后休眠
        // 另一个等待锁，然后我们使用 jstack 
        // 命令查看该线程所处的状态
        new Thread(()-> BLOCKED(), "This thread get lock.").start();
        new Thread(()-> BLOCKED(), "This thread wait lock.").start();
    }

    public static synchronized void BLOCKED() {
        try {
            Thread.sleep(1000_000_000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

运行上述代码，然后我们使用 `jstack [pid]` 来查看线程的状态

![通过 jstack 查看到的线程状态](/media/hovel/22.png)

可以看到 `This thread wait lock.` 这个线程由于没有抢到锁，所以线程进入了 BLOCKED 的状态。

## 2.2 RUNNABLE 与 WAITING 的状态转换

总体来说，有三种场景会触发这种转换

### 2.2.1 争抢 synchronized 隐式锁

第一种场景，获得 synchronized 隐式锁的线程，然后调用无参数的 `Object.wait()` 方法。

```java
public class Main {
    public static void main(String[] args) {
        new Thread(() -> {
            Object object = new Object();
            synchronized (object) {
                try {
                    object.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "This thread invoke wait method.").start();
    }
}
```

执行上述代码，然后我们使用 `jstack [pid]` 来查看线程的状态

![通过 jstack 查看到的线程状态](/media/hovel/23.png)

### 2.2.2 调用 Thread.join() 方法

第二种场景，调用无参数的 `Thread.join()` 方法。调用线程会从 RUNNABLE 转换到 WAITING.

```java
/*
    当 main 所在的线程调用 th 的 join 方法时，main 所在的线程会
    等待 th 线程运行结束，这个时候 main 所在的线程会从 RUNNABLE 
    转换到 WAITING，当 th 线程运行结束之后 main 线程会从 WAITING 
    转换到 RUNNABLE 状态继续运行。
*/
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread th = new Thread(() -> {
            try {
                Thread.sleep(1000_000_000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        th.start();
        th.join();
    }
}
```

执行上述代码，然后我们使用 `jstack [pid]` 来查看线程的状态

![通过 jstack 查看到的线程状态](/media/hovel/24.png)

### 2.2.3 调用 LockSupport.park() 方法

第三种场景，调用 LockSupport.park() 方法，调用 LockSupport.park() 方法，当前线程会阻塞，线程的状态会从 RUNNABLE 转换到 WAITING。调用 LockSupport.unpark(Thread thread) 可唤醒目标线程，目标线程也会从 WAITING 状态转换到 RUNNABLE 状态。

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> LockSupport.park(), "Invoke lock support part method.");
        thread.start();
    }
}
```

执行上述代码，然后我们使用 `jstack [pid]` 来查看线程的状态，线程进入等待状态。

![通过 jstack 查看到的线程状态](/media/hovel/25.png)

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> LockSupport.park(), "Invoke lock support part method.");
        thread.start();
        // 通过 unpark 即可让线程脱离 WAITING 状态
        LockSupport.unpark(thread);
    }
}
```

## 2.3 RUNNABLE 与 TIMED_WAITING 的状态转换

TIMED_WAITING 与 WAITING 的区别，仅仅是 TIMED_WAITING 带有超时参数

有五种场景会触发这种转换：

1. 调用带超时参数的 `Thread.sleep(long millis)` 方法
2. 获得 synchronized 隐式锁的线程，调用带超时参数的 `Object.wait(long timeout)` 方法
3. 调用带超时参数的 `Thread.join(long millis) 方法`
4. 调用带超时参数的 `LockSupport.parkNanos(Object blocker, long deadline)` 方法
5. 调用带超时参数的 `LockSupport.parkUntil(long deadline)` 方法

## 2.4 从 NEW 到 RUNNABLE 状态

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {}, "This thread not start.");
        LockSupport.park();
    }
}
```

执行 `new Thread(() -> {}, "This thread not start.");`  之后，在堆内存中就已经存在一个线程对象，这个时候使用 `jstack` 并不能看到线程，但此时的线程就处于 NEW 的状态，线程执行 `start()` 方法之后，就变成 RUNNABLE 状态。

## 2.5 从 RUNNABLE 到 TERMINATED 状态

当线程执行完 `run()` 方法之后，或者执行 `stop()` 方法都可以使线程从 RUNNABLE 状态进入到 TERMINATED 状态。

### 2.5.1 stop() 方法与 interrupt()

`stop()` 方法在 1.8 当中已经标注为废弃，这个方法比较危险，它会让被调用的线程直接死亡，并且抢占的锁也不会释放，那么如果一个线程抢占了锁，然后被调用了这个方法，那么后续来抢这个锁的线程就全部都会进入阻塞，并且永远也醒不过来，非常危险。

推荐的方式是使用 `interrupt()`，这个方法仅仅是通知线程，线程依然可以有一些后续的操作。线程接收这个通知有两种方式，一种是监测到 `InterruptedException` 这个异常，另一种是主动检测，也就是通过 `Thread.currentThread().isInterrupted()` 这个方法判断是否被打断，如果被打断，那么可以做一些操作。

# 参考资料

1. 极客时间：JAVA 并发编程实战：Java 线程的生命周期
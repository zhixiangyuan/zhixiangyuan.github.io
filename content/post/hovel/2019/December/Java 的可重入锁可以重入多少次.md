---
title: "Java 的可重入锁可以重入多少次"
date: 2019-12-10T15:01:38+08:00
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

Java 中有两个常用的可重入锁，一个是 synchronized，另一个是 ReentrantLock，我们先测试 ReentrantLock 的可重入次数

# 1 ReentrantLock 的可重入次数上限

```java
    public static void main(String[] args) {
        ReentrantLock reentrantLock = new ReentrantLock();
        long count = 0;
        try {
            for (long i = 0; i < Long.MAX_VALUE; i++) {
                reentrantLock.lock();
                count++;
            }
        } finally {
            System.out.println(String.format("重入次数：[%s].", count));
        }
    }

OUTPUT：
重入次数：[2147483647].
Exception in thread "main" java.lang.Error: Maximum lock count exceeded
	at java.util.concurrent.locks.ReentrantLock$Sync.nonfairTryAcquire(ReentrantLock.java:141)
	at java.util.concurrent.locks.ReentrantLock$NonfairSync.tryAcquire(ReentrantLock.java:213)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:1198)
	at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLock.java:209)
	at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)
	at me.yuanzx.file.Main.main(Main.java:20)
```

2147483647 这个数字就是 Integer.MAX_VALUE，我们查看一下抛出异常的源码，后面的分析直接打在代码上

```java
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            // 可以看到这里取出了 state，下面是 state 的声明
            // private volatile int state;
            // 可以看到 state 是一个 int
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            // 这里判断是否是可重入的线程
            else if (current == getExclusiveOwnerThread()) {
                // 如果是可重入的则对 c + acquires
                // 其实我们这里的 acquires 就是 1，可重入次数加一
                // 当达到 int 最大值之后变成负数抛出异常
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    // 这里抛出的异常
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

# 2 synchronized 关键字的可重入次数

这个关键字用递归去重入的话会出现栈内存溢出，所以我写了下面这段程序去生成一个 Java 文件，不通过递归去调用

```java
    public static void main(String[] args) throws IOException {
        String filePath = "/Users/zhixiangyuan/workspace/personal/yuanzx-tool/src/main/java/me/yuanzx/file/Main.java";
        File file = new File(filePath);
        if (!file.exists()) {
            file.createNewFile();
        }
        FileOutputStream fileOutputStream = new FileOutputStream(file);
        fileOutputStream.write("package me.yuanzx.file;".getBytes());
        fileOutputStream.write(System.lineSeparator().getBytes());
        fileOutputStream.write("public class Main {".getBytes());
        fileOutputStream.write(System.lineSeparator().getBytes());
        fileOutputStream.write("    public static void main(String[] args) {".getBytes());
        fileOutputStream.write(System.lineSeparator().getBytes());
        fileOutputStream.write("        long count = 0;".getBytes());
        fileOutputStream.write(System.lineSeparator().getBytes());
        fileOutputStream.write("        try {".getBytes());
        fileOutputStream.write(System.lineSeparator().getBytes());
        long count = Integer.MAX_VALUE;
        for (long i = 0; i < count; i++) {
            fileOutputStream.write("synchronized (Main.class) { count++; ".getBytes());
        }
        for (long i = 0; i < count; i++) {
            fileOutputStream.write("}".getBytes());
        }
        fileOutputStream.write(System.lineSeparator().getBytes());
        fileOutputStream.write("        } finally {".getBytes());
        fileOutputStream.write(System.lineSeparator().getBytes());
        fileOutputStream.write("            System.out.println(String.format(\"重入次数：[%s].\", count));".getBytes());
        fileOutputStream.write(System.lineSeparator().getBytes());
        fileOutputStream.write("        }".getBytes());
        fileOutputStream.write(System.lineSeparator().getBytes());
        fileOutputStream.write("    }".getBytes());
        fileOutputStream.write(System.lineSeparator().getBytes());
        fileOutputStream.write("}".getBytes());
    }
```

但是又出现了另一个问题，写出来的文件很大，没有运行完都已经几个 G 了，所以这个方法也毙掉了，具体能重入多少次等以后看 openjdk 的时候看一下用什么数据类型存的就知道了。
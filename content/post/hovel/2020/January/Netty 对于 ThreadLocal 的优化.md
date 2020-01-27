---
title: "Netty 对于 ThreadLocal 的优化"
date: 2020-01-27T12:58:11+08:00
keywords: []
description: ""
tags: [
    "netty"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

对于 `ThreadLocal` 这个使用起来很方便的线程级别的全局变量，其本身有两点是需要注意的

第一，它在线程池的环境中使用时是可能会导致内存泄漏的，这一点如果大家不了解可以去查一下相关的资料。

第二，它内部的实现是通过数组存放相应的数据的，如果出现 hash 冲突会使用线性探测法解决 hash 冲突。

Netty 中的优化便是围绕这两点进行展开的

# 1 如何解决内存泄漏问题

Netty 封装的 `FastThreadLocalRunnable` 会在 `run` 方法运行完之后清理掉 `FastThreadLocal`

```java
final class FastThreadLocalRunnable implements Runnable {
    private final Runnable runnable;

    private FastThreadLocalRunnable(Runnable runnable) {
        this.runnable = ObjectUtil.checkNotNull(runnable, "runnable");
    }

    @Override
    public void run() {
        try {
            runnable.run();
        } finally {
            // 在 run 方法运行完之后清理掉 ThreadLocal
            FastThreadLocal.removeAll();
        }
    }

    static Runnable wrap(Runnable runnable) {
        return runnable instanceof FastThreadLocalRunnable ? runnable : new FastThreadLocalRunnable(runnable);
    }
}
```

但是这里的设计似乎有问题，一般内存泄漏是出现在线程池里面，也就是线程一直在运行的情况，如果线程死亡了，那么资源也就可以被 gc 回收了。而 Netty 所做的优化也是在线程执行完毕之后才去清理掉所有的 `FastThreadLocal`，这个时候清理看上去也不是很必要了，反正 gc 也能够清理。

# 2 如何实现对于 hash 冲突的优化

其实 Netty 内部实现看上去是通过空间来换时间，`Thread` 内部通过数组来保存数据。（这里面需要注意的是，它里面写的是 `InternalThreadLocalMap`，虽然命名为 `Map`，但是它里面的实现是通过数组实现的，这里可以理解为 `key` 为数组下标，`value` 为存储的数据）每一个 `FastThreadLocal` 都有一个自己的 `index`，这个 `index` 是在创建 `FastThreadLocal` 时申请到的一个唯一的 `index`，同时这个 `index` 会用于 `InternalThreadLocalMap` 中存取数据。

解决 hash 冲突的关键便在于这个 `index` `index` 表示数组中的下标，并且这个 `index` 是绝对唯一的，那么在实际访问的时候便可以通过该 `index` 直接进行存取。

但是这里面也有个问题，它申请的 `index` 是从 0 开始增加的，也就是说如果别的线程申请了 1000 个 `FastThreadLocal`，之后另一个线程只申请一个，那么它内部的存储数据的数组大小却会是 1024，这便造成了严重的浪费。但是可能 netty 觉得外人用这个 `FastThreadLocal` 的时候会注意到这个问题，所以不会滥用。
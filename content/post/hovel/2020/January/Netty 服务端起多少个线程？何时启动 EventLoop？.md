---
title: "Netty 服务端起多少个线程？何时启动 EventLoop？"
date: 2020-01-23T22:53:44+08:00
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

# 1 Netty 服务端起多少个线程

一般我们都是直接去 `new NioEventLoopGroup`，通过跟踪 NioEventLoopGroup 的构造方法能够发现以下的代码

```java
public abstract class MultithreadEventLoopGroup extends MultithreadEventExecutorGroup implements EventLoopGroup {
    private static final int DEFAULT_EVENT_LOOP_THREADS;

    static {
        // 默认为两倍的 CPU 核数
        DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(
                "io.netty.eventLoopThreads", NettyRuntime.availableProcessors() * 2));

        if (logger.isDebugEnabled()) {
            logger.debug("-Dio.netty.eventLoopThreads: {}", DEFAULT_EVENT_LOOP_THREADS);
        }
    }
    
    protected MultithreadEventLoopGroup(int nThreads, Executor executor, Object... args) {
        super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, executor, args);
    }
}
```

可以看到当我们指定的线程数为 0 时，默认启动两倍的 CPU 核数的线程数。

# 2 何时启动 EventLoop

通过追踪 `ServerBootstrap` 的 `bind()` 方法，能够找到 `AbstractBootstrap` 的 `doBind0()` 方法，在其中通过 `eventLoop()` 去执行 `execute()` 的时候会在内部判断，如果是外部线程调用的方法则说明没有启动，会调用 `startThread()` 方法启动 `EventLoop`。（这里判断的方法是 `inEventLoop()`，内部判断逻辑是通过判断当前的线程是否是 `EventLoop` 内部的线程来实现）

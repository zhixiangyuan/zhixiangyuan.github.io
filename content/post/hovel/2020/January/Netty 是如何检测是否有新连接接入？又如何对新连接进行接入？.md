---
title: "Netty 是如何检测是否有新连接接入？又如何对新连接进行接入？"
date: 2020-01-24T15:04:33+08:00
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

# 1 如何检测是否有新连接接入

一般对于服务端，我们会 new 两个 `NioEventLoopGroup`，一个我们称为 bossGroup，另一个称为 workerGroup。bossGroup 的作用便是对检测是否有新连接，如果有新连接便将新连接接入到 workerGroup 中，所以对于 bossGroup，我们一般就在其中初始化一个 `NioEventLoop`，一个 `NioEventLoop` 相当于一个可以执行任务的线程，不过这个线程的功能更加强大，它还有等待任务队列和定时任务队列。

在 bossGroup 中的 `NioEventLoop` 启动之后，便开始不断的执行 select 操作去检查当前是否有新连接需要接入，对应的代码入口在 `NioEventLoop` 的 `protected void run()` 方法中的 `select(wakenUp.getAndSet(false));` 处。

需要注意的是，这里 `ServerSocketChannel` 在之前注册时候只设置了关心 `OP_ACCEPT` 的事件。

# 2 如何接入新连接

在监测到新连接之后，关于新连接的处理入口在 `NioEventLoop` 的 `protected void run()` 方法中的 `processSelectedKeys();`。在这里面对于连接的处理处于 `unsafe.read();` 处，由于我们这里是新连接的接入，所以对于 `unsafe` 的实现是 `NioMessageUnsafe`，这是 `AbstractNioMessageChannel` 的内部类。

实际的代码便不贴了，这里简述以下逻辑，在 `unsafe` 接入的时候会使用 Jdk 的 `ServerSocketChannel` 去获取接入的客户端的 `SocketChannel`，然后将 `SocketChannel` 包装称为 Netty 自己实现的 `NioSocketChannel`。在包装之后对该 Channel 进行初始化设置，然后为其在 workerGroup 中分配一个 `NioEventLoop` 然后注册进入，如果在注册进去的时候该 `NioEventLoop` 没有启动便启动起来，如果启动起来了便由该 `NioEventLoop` 接管后续的数据读取操作。
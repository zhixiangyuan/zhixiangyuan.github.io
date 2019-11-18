---
title: "TCP 中的 SO_LINGER 选项"
date: 2019-11-19T21:04:04+08:00
keywords: []
description: ""
tags: [
    "网络协议"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

SO_LINGER 参数的作用就是改变 close 的默认行为，SO_LINGER 参数在 Linux 系统中就是一个 linger 结构体，代码如下

```c
struct linger {
    int l_onoff;    /* linger active */
    int l_linger;   /* how many seconds to linger for */
};
```

第一个字段 l_onoff 用来表示是否启用 linger 特性，非 0 为启用，0 为禁用 ，linux 内核默认为禁用。这种情况下 close 函数立即返回，操作系统负责把缓冲队列中的数据全部发送至对端

第二个参数 l_linger 在 l_onoff 为非 0 （即启用特性）时才会生效。

- 如果 l_linger 的值为 0，那么调用 close，close 函数会立即返回，同时丢弃缓冲区内所有数据并立即发送 RST 包重置连接
- 如果 l_linger 的值为非 0，那么此时 close 函数在阻塞直到 l_linger 时间超时或者数据发送完毕，发送队列在超时时间段内继续尝试发送，如果发送完成则皆大欢喜，超时则直接丢弃缓冲区内容 并 RST 掉连接。

以下是在 Java 中的使用

```java
   public static void main(String[] args) throws Exception {
        Socket socket = new Socket();
        // 默认设置
        // 关闭 SO_LINGER
        socket.setSoLinger(false, 0);
        // 开启 SO_LINGER，调用 close 时立刻关闭 socket
        socket.setSoLinger(true, 0);
        // 开启 SO_LINGER，调用 close 时等待 2s，如果
        // 在 2s 内将缓冲区内的内容全部发送到对端则完
        // 美，如果超过两秒则会在两秒到达的时候发
        // 送 RST 包关闭连接
        socket.setSoLinger(true, 2);
   }
```

# 参考文章

1. [深入理解 TCP 协议：从原理到实战](https://juejin.im/book/5c70dbbe51882562046911bc?referrer=5aa21ad15188255585072268)
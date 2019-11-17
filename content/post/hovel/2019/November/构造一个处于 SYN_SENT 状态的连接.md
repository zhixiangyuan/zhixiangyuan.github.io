---
title: "构造一个处于 SYN_SENT 状态的连接"
date: 2019-11-14T21:29:27+08:00
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

下图是一个三次握手的变化过程

![](/hub/2019/November/43.png)

在客户端发出 SYN 包之后，便进入 SYN-SENT 的状态，如果没有收到服务端的 ACK，那么便会一直维持在这个状态，多次重发 SYN 数据包，当重发达到一定次数时便停止重发，反馈连接失败，Java 中便会爆出 `java.net.ConnectException: Connection timed out` 的异常。这个重发的次数对于 linux 系统可以查看这个文件 `/proc/sys/net/ipv4/tcp_syn_retries`，每一次发出 SYN 包后下一次发出的时间便会翻倍，消耗的时间为 65s = 1s+2s+4s+8s+16s+32s

新建一个脚本

```
// 新建一个 server socket
+0 socket(..., SOCK_STREAM, IPPROTO_TCP) = 3
// 客户端 connect
+0 connect(3, ..., ...) = -1
```

首先使用 tcpdump 开始抓包 `tcpdump -i any port 8080 -nn`，然后使用 packetdrill 运行这个脚本，此时运行 `netstat -atnp | grep -i 8080` 就能看到发送端处于 SYN_SENT 的状态

```shell
$> netstat -atnp | grep -i 8080
tcp        0      1 192.168.143.236:57780   192.0.2.1:8080          SYN_SENT    4804/packetdrill
```


# 参考资料

1. [深入理解 TCP 协议：从原理到实战](https://juejin.im/book/5c70dbbe51882562046911bc?referrer=5aa21ad15188255585072268)
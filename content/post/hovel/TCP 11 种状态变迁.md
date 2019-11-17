---
title: "TCP 11 种状态变迁"
date: 2019-11-17T15:56:38+08:00
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

![](/hub/2019/November/53.png)

# 1 CLOSED

CLOSED 状态是一个假想的状态，它处于 TCP 连接还未开始建立或者已经释放掉的状态。因此无法通过 netstat 或者 lsof 看到。

从 CLOSED 状态转换为其他状态有两种可能

1. 一种是服务端监听特定端口，即进入 LISTEN 状态
2. 另一种是客户端发送一个 SYN 包准备三次握手则进入 SYN_SENT

# 2 LISTEN

通常是服务端调用 bind、listen 系统调用监听特定端口是进入 LISTEN 状态，等待客户端发送 SYN 报文进行三次握手建立连接。

在 Java 中通过 `ServerSocket serverSocket = new ServerSocket(9999);` 即可开启一个处于 LISTEN 状态的 socket，通过 `netstat -tnpa | grep -i 9999` 即可查看到该连接的状态。

```
$> netstat -tnpa | grep -i 9999
tcp6       0      0 :::9999     :::*                    LISTEN      20096/java  
```

# 3 SYN_SENT

一端发送 SYN 报文后进入 SYN_SENT 状态，等待另一端的 ACK。同时会开启一个定时器，如果超时还没有收到 ACK 会重发 SYN。

# 4 SYN_RCVD

一端收到 SYN 报文以后会回复 SYN+ACK，然后进入 SYN_RCVD 状态，等待另一端的 ACK

# 5 ESTABLISHED

// todo 从这里开始改图

进入 ESTABLISHED 状态后，后面有两种状态的变化方式

1. 调用 close 系统调用主动关闭连接，这个时候会发送 FIN 包给对端，同时自己进入 FIN_WAIT_1 状态
2. 收到对端的 FIN 包，执行被动关闭，收到 FIN 包以后回复 ACK，同时自己进入 CLOSE_WAIT 状态

# 6 FIN_WAIT_1

主动关闭的一端发送了 FIN 包，进入 FIN_WAIT_1 状态等待对端的 ACK，后面的状态变化有三种可能

1. 收到 FIN+ACK，向对端回复 ACK，FIN_WAIT_1 状态直接转换到 TIME_WAIT 状态，跳过 FIN_WAIT_2 状态
2. 当收到 ACK 以后，FIN_WAIT_1 转变为 FIN_WAIT_2 状态，等待对端的 FIN 包
3. 当收到 FIN 以后，回复对端 ACK，然后进入到 TIME_WAIT 状态

# 7 FIN_WAIT_2

这个状态在等待对端的 FIN 包，当收到对端的 FIN 包之后回复 ACK 并进入 TIME_WAIT 状态。

# 8 CLOSING

CLOSING 状态在同时关闭的情况下才会出现，这里的同时指的是在发出 FIN 后还未收到 ACK 就收到了对端的 FIN。

# 9 TIME-WAIT

TIME-WAIT 可能是所有状态中面试问的最频繁的一种状态，这个状态是收到了被动关闭方的 FIN 包，发送确认 ACK 给对端，开启 2MSL 定时器，定时器到期后进入 CLOSED 状态，连接释放。

# 10 CLOSE_WAIT

处于 ESTABLISHED 状态的时候收到 FIN 包后向对端发送 ACK 后进入 CLOSE_WAIT 状态，当自己这边数据发送完毕之后再发送 FIN 包然后进入 LAST_ACK 状态

# 11 LAST_ACK

处于 CLOSE_WAIT 时发送 FIN 包后进入 LAST_ACK 状态，当收到 FIN 的 ACK 后进入 CLOSED 状态

# 参考资料

1. [深入理解 TCP 协议：从原理到实战](https://juejin.im/book/5c70dbbe51882562046911bc?referrer=5aa21ad15188255585072268)
2. [tcp 状态图使用这个 web 服务绘制](https://www.overleaf.com/)

---
title: "TCP 三次握手详解"
date: 2019-11-17T14:40:12+08:00
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

# 1 三次握手数据包交互过程

![TCP 连接的建立](/media/hovel/11.jpg)

下面用 client 表述客户端，server 表述服务端，这样看起来更清晰。

1. client 发出 SYN 数据包给服务端，同时初始化一个序列号 X，发出数据包后 client 进入 SYN-SENT 状态
2. server 收到 client 的 SYN 数据包返回一个 ACK 包，ACK 的序列号为 x + 1（表示 x+1 序列号之前的数据全部接受了），同时初始化一个自己的序列号 y，然后 server 端进入 SYN-RCVD 状态
3. client 收到服务端的 SYN+ACK 数据包后返回给 server 一个关于 SYN 的 ACK 包，ACK 的序列号为 y + 1，然后进入 ESTABLISHED 状态，server 在收到 ACK 后也进入 ESTABLISHED 状态。

## 1.1 为什么要建立连接

连接建立时会初始化第一个发送字节的初始化序列号，建立连接之后，发送的每一串二进制数据都要以初始序列号为远点进行编号，需要对方来确认每一个字节编号都已经成功接收。这即保证了逻辑上不产生丢包，同时又保证了包的顺序。同时在建立连接的时候还会初始化一些别的参数，比如窗口大小等。

## 1.2 为什么序号不能从 1 开始

如果 A 和 B 建立了连接，发送了三个包，序号为 2，3，4，其中 4 由于等到超时发了两次，简单交流之后结束连接，结束之后又重新建立了连接，又从 1 开始，发了两个包 2，3，结果之前 4 那个包因为网络延迟问题，又到了，B 以为这是一个跟在 3 后面的包，就收进来了，这时候数据就错了，所以序号不能从 1 开始。

# 2 握手可以变成四次吗？

握手时只需要将 SYN+ACK 包拆成 SYN 包和 ACK 包就变成了四次握手，不过此时还没有建立通信，这样做没有意义。

# 3 客户端与服务端同时发出 SYN 打开连接

![](/hub/2019/November/52.png)

1. 最初的状态是 CLOSED
2. client 发起主动打开，发送 SYN 给 server，然后进入 SYN-SENT 状态
3. client 还在等待 server 回复的 ACK 的过程中，收到了 server 发过来的 SYN，what are you 弄啥咧，client 没有办法，只能硬着头皮回复 SYN+ACK，随后进入 SYN-RCVD
client 依旧死等 server 的 ACK
4. 好不容易等到了 server 的 ACK，对于 client 来说连接建立成功

# 4 伪造 IP 攻击

如果对于 client 来说能够猜测到 server 的 ISN 的话，那么随便一台 client 都可以冒充别的 client 与 server 建立连接。

client 首先修改自己的 IP 为别的 client 的 IP，然后发送一个 SYN 包，发送之后由于能猜测到 ISN，那么就再发送一个 server 回复的 SYN+ACK 的 ACK 成功与 server 建立连接，这条连接建立完成之后，由于真正的 client 无法知道序列号，所以无法与 server 建立连接，这样就完成了攻击。


# 参考资料

1. [深入理解 TCP 协议：从原理到实战](https://juejin.im/book/5c70dbbe51882562046911bc?referrer=5aa21ad15188255585072268)
2. [基于 TCP/IP 协议的常见攻击方法](https://www.jianshu.com/p/b6466db30160)
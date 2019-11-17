---
title: "TCP 四次挥手详解"
date: 2019-11-17T11:00:11+08:00
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

# 1 四次挥手数据包的交互过程

![](/hub/2019/November/44.png)

下面用 client 表述客户端，server 表述服务端，这样看起来更清晰。

1. client 发送 FIN 包给 server，发送后 client 进入 FIN-WAIT-1 状态
2. server 收到 client 的 FIN 包，然后 server 回复 client 关于 FIN 的 ACK 包并进入 CLOSE_WAIT 状态
3. client 收到 server 发送的关于 FIN 的 ACK 包则进入 FIN-WAIT-2 状态，此时 client 的 FIN 包已经确认完成
4. server 发送 FIN 包给 client，然后进入 LAST-ACK 状态
5. client 收到 server 的 FIN 包，然后回复关于 FIN 包的 ACK 之后进入 TIME-WAIT 状态，同时在这个状态等待 2*MSL 的时间，防止 ACK 包中途丢失，如果 ACK 包中途丢失，那么 server 在等待 ACK 超时之后会再次重发 FIN 包
6. server 收到 client 关于 FIN 的 ACK 包，进入 CLOSED 状态
7. client 在等待 2*MSL 的时间之后也进入 CLOSED 状态

# 2 为什么 FIN 报文要消耗一个序列号

如果 FIN 不消耗一个序列号，那么如果 FIN 包之前有数据未被确认，那么当发出 FIN 包之后收到 ACK 便无法确认到底这个 ACK 是之前数据的还是 FIN 包的。

如下是一个例子，client 先发送一个 hello，然后进行四次挥手

下图为 client 发出一个 hello 的数据包，sequence number 为 1，data 有 6 个字节，所以 next sequence number 为 7，ack 也要 ack 7

![](/hub/2019/November/45.png)

下图为 server 给的 ack 数据包，其中 ack 为 7

![](/hub/2019/November/46.png)

下图为 client 发送的 FIN 数据包，sequence number 为 7，虽然没有携带数据，但是 next sequence number 却要为 8（wireshark 显示为 7 不知道是不是 bug）

![](/hub/2019/November/47.png)

下图为 server 回复 client 关于 FIN 的 ACK，可以看到 ack 为 8，说明消耗了一个序列号

![](/hub/2019/November/48.png)

下图为 server 发给 client 的 FIN 数据包，sequence number 1 为 ，虽然没有携带数据，但是 next sequence number 却要为 2（wireshark 显示为 2 不知道是不是 bug）

![](/hub/2019/November/49.png)

下图为 client 回复 server 关于 FIN 的 ACK，可以看到 ack 为 2，说明消耗了一个序列号

![](/hub/2019/November/50.png)

# 3 为什么挥手要四次，变成三次可以吗？

完全可以，当 server 收到 client 的 FIN 包后，如果 server 没有数据发给 client，那么可以将 server 端的 FIN 和 ACK 包合并发给 client，这样就变成了三次挥手。如果在 server 端收到 FIN 包之后还有数据要发给 client，那么就不能完成三次挥手，而先回复 FIN 的 ACK，然后将剩下需要发送的数据发送给 client，然后再发送 ACK 包。

# 4 握手可以变成四次吗？

握手时只需要将 SYN+ACK 包拆成 SYN 包和 ACK 包就变成了四次握手，不过此时还没有建立通信，这样做没有意义。

# 5 服务端与客户端同时发起 FIN 关闭连接

![](/hub/2019/November/51.png)

以客户端为例：

1. 最初 client 和 server 都处于 ESTABLISHED 状态
2. client 发送 FIN 包，等待 server 发送 ACK 包，同时进入 FIN-WAIT-1 状态
3. 处于 FIN-WAIT-1 状态的 client 还没等到 ACK，就收到了 server 发送过来的 FIN 包
4. 收到 FIN 包以后 client 会发送这个 FIN 包的确认 ACK 包，同时自己进入 CLOSING 状态，继续等待自己的 FIN 包的 ACK
5. 处于 CLOSING 状态的 client 终于等到了 ACK，随后进入 TIME-WAIT 状态
6. 在 TIME-WAIT 状态持续 2*MSL 最终进入 CLOSED 状态

# 参考资料

1. [深入理解 TCP 协议：从原理到实战](https://juejin.im/book/5c70dbbe51882562046911bc?referrer=5aa21ad15188255585072268)
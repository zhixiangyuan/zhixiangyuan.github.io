---
title: "关于 TCP 半连接队列与全连接队列"
date: 2019-11-18T14:51:35+08:00
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

以下使用 client 代替客户端，server 代替服务端，看起来更清晰

半连接队列与全连接队列与服务端相关，在 server 调用 `int listen(int sockfd, int backlog);` 开启 LISTEN 监听时，与此同时内核创建两个队列

- 半连接队列（Incomplete connection queue），又称 SYN 队列
- 全连接队列（Completed connection queue），又称 accept 队列

![](/hub/2019/November/54.png)

当 client 发起 SYN 到 server，server 收到后会回复 SYN+ACK，这是服务端这边的 TCP 从 LISTEN 变成 SYN_RCVD，此时会将这个连接信息放入半连接队列。当 client 收到 server 的 SYN+ACK 后会回复 ACK，当 server 收到 client 的 ACK 后会尝试将这个连接放入全连接队列。

全连接队列中包含了服务端所有完成了三次握手，但是还未被应用取走的连接队列。此时的 socket 处于 ESTABLISHED 状态。每次应用调用 accept() 函数会移除队列头的连接。如果队列为空，accept() 通常会阻塞。

# 1 全连接队列大小

全连接队列大小由 `int listen(int sockfd, int backlog)` 中的 backlog 参数指定（Nginx 和 Redis 默认的 backlog 值等于 511，Linux 默认的 backlog 为 128，Java 默认的 backlog 等于 50）。

如果全连接队列满了，此时只有 accept 操作才能从全连接队列中移除连接，那么半连接队列中的连接在收到 ACK 后就无法将连接放入全连接队列，此时对于收到 ACK 的连接有两种处理方式。处理方式看 `/proc/sys/net/ipv4/tcp_abort_on_overflow` 文件，文件中的值可能为 0 或 1。

- 0: 丢弃收到 ACK 并重传 SYN+ACK
- 1: 直接给 client 发送 RST 包断开连接

# 2 半连接队列大小

半连接队列可以通过系统中 `/proc/sys/net/ipv4/tcp_max_syn_backlog` 文件查看到大小

# 3 相关的命令

```shell
# 可以通过下面这个命令观察全连接队列的使用情况
$> ss -lnt
# Recv-Q 为当前全连接队列使用了多少
# Send-Q 为全连接队列最大值
Recv-Q Send-Q   Local Address:Port Peer Address:Port
0       100                 *:8080            *:*
```

# 4 一种针对半连接队列的 Dos 攻击

随便一台主机冒充不同的 IP 地址不停的向受害者主机发送 SYN 请求，很快就能把受害者主机的半连接队列填满，这时候受害者主机没法处理正常用户的请求，便出现了停止服务的情况。

# 参考文章

1. [深入理解 TCP 协议：从原理到实战](https://juejin.im/book/5c70dbbe51882562046911bc?referrer=5aa21ad15188255585072268)
2. [关于 TCP 半连接队列和全连接队列](http://jm.taobao.org/2017/05/25/525-1/)
   - 其中包含一个实战案例
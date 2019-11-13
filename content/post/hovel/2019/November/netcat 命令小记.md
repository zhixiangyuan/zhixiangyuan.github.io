---
title: "netcat 命令小记"
date: 2019-11-13T16:21:11+08:00
keywords: []
description: ""
tags: [
    "Shell"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

netcat 因为功能强大，被称为网络工具中的瑞士军刀，nc 是 netcat 的简称。

![](/hub/2019/November/40.png)

```shell
# -l 参数表示 nc 将监听某个端口
# -p 表示监听的端口号为 $port
$> nc -l -p <$port>

# 与 [$ip]:[$port] 的服务器建立 tcp 连接
$> nc [$ip] [$port]

# 查看远程端口是否打开
# 其中 -z 参数表示不发送任何数据包，
# tcp 三次握手完后自动退出进程。有
# 了 -v 参数则会输出更多详细信息（verbose）
$> nc -zv [$ip] [$port]
```

# 参考资料

1. [深入理解 TCP 协议：从原理到实战](https://juejin.im/book/5c70dbbe51882562046911bc?referrer=5aa21ad15188255585072268)
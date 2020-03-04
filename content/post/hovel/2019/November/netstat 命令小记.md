---
title: "netstat 命令小记"
date: 2019-11-13T16:42:22+08:00
keywords: []
description: ""
tags: [
    "linux"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

![](/hub/2019/November/39.png)

```shell
# 只列出 TCP 套接字
# -t 选项可以只列出 TCP 的套接字，也可也用 --tcp
$> netstat -at

# 只列出 UDP 连接
# -u 选项用来指定显示 UDP 的连接，也可也用 --udp
$> netstat -au

# 只列出处于监听状态的连接
$> netstat -l

# 只列出处于监听状态的 tcp 连接
$> netstat -lt

# 常用端口会被映射成名字，比如 22 端口输出为 ssh，
# 可以通过 -n 禁用该选项
$> netstat -ltn

# 显示进程
# 使用 -p 命令可以显示连接归属的进程信息，在查看端口
# 被哪个进程占用时非常有用
$> netstat -ltnp

# 显示所有的网卡信息
$> netstat -i

# 显示 8080 端口所有处于 ESTABLISHED 状态的连接
$> netstat -atnp | grep ":8080" | grep ESTABLISHED

# 统计处于各个状态的连接个数
$> netstat -ant | awk '{print $6}' | sort | uniq -c | sort -n
```

# 参考资料

1. [深入理解 TCP 协议：从原理到实战](https://juejin.im/book/5c70dbbe51882562046911bc?referrer=5aa21ad15188255585072268)
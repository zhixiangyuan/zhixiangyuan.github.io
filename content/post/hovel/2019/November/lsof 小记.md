---
title: "lsof 小记"
date: 2019-11-13T21:32:47+08:00
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

![](/hub/2019/November/41.png)

```shell
# 查看端口被什么进程占用
# -n 表示不将 IP 转换为 hostname
# -P 表示不将 port number 转换为 service name
$> lsof -n -P -i:<$port>

# 查看进程监听了哪些端口号
$> lsof -n -P -p <$pid> | grep TCP
```
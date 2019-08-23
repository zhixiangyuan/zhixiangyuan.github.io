---
title: "理解 CAP"
date: 2019-08-23T14:08:22+08:00
keywords: []
description: ""
tags: [
    "分布式系统"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
draft: true
---

CAP 理论是 1998 年，加州大学的计算机科学家 EricBrewer 提出，其中包含三个指标

1. Consistency
2. Availability
3. Partition tolerance

# 1 Partition tolerance

该指标为分区容错，它的意思就是只要程序需要经过通信交换数据，那么就有可能发生超时等异常情况。

这在分布式系统里面无法避免，所有的服务都通过 rpc 通信，需要通过网络传输，那么就可能存在各种各样的异常情况了，而这便是指的分区容错。

# 2 Consistency

该指标为一致性，它的意思是说在用户获取数据的时候，当用户在同一时刻获取不同服务器上的数据时，可能会出现不一致的情况。

# 3 Availability

该指标为可用性，意思是服务器收到请求必须立刻给出回应

# 4 Consistency 与 Availability 存在的矛盾

当用户发出修改数据的指令到 A 服务器上，然后又像 B 服务器发出查询指令，那么就可能出现数据不一致的情况，如果要避免这种情况，那么当 A 修改数据的时候需要锁定 B 服务器读写该数据的能力，如果出现锁定那么就无法满足可用性，也就出现了矛盾。

# 参考资料

1. [CAP 定理的含义](http://www.ruanyifeng.com/blog/2018/07/cap.html)
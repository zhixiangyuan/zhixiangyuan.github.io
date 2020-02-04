---
title: "CAP 简述"
date: 2019-08-23T14:08:22+08:00
keywords: []
description: ""
tags: [
    "分布式事务"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

CAP 理论是 1998 年，加州大学的计算机科学家 EricBrewer 提出，其中包含三个指标

1. Consistency
2. Availability
3. Partition tolerance

# 1 Consistency 一致性

指的是所有数据在同一时刻看到的数据是一致的

>all nodes see the same data at the same time

# 2 Availability 可用性

指的是服务在正常响应时间内一直可用

>Reads and writes always succeed

# 3 Partition tolerance 分区容错性

指的是分布式系统在遇到某节点或网络分区故障的时候，仍然能够对外提供满足一致性或可用性的服务

>the system continues to operate despite arbitrary message loss or failure of part of the system

# 4 Consistency 与 Availability 存在的矛盾

当用户发出修改数据的指令到 A 服务器上，然后又像 B 服务器发出查询指令，那么就可能出现数据不一致的情况，如果要避免这种情况，那么当 A 修改数据的时候需要锁定 B 服务器读写该数据的能力，如果出现锁定那么就无法满足可用性，也就出现了矛盾。

# 5 关于分布式系统

对于分布式系统而言，其本身就是为了获取分区容错性，所以在实现分区容错性后，便只能在一致性和可用性之间做妥协，由此便引出了强一致性，弱一致性和最终一致性。在互联网领域的绝大多数的场景中，都需要牺牲强一致性来换取系统的高可用性，系统往往只需要保证最终一致性，只要这个最终时间是在用户可以接受的范围内即可。

# 参考资料

1. [CAP 定理的含义](http://www.ruanyifeng.com/blog/2018/07/cap.html)
2. 非常推荐：[谈谈分布式系统的 CAP 理论](https://zhuanlan.zhihu.com/p/33999708)
---
title: "Redis 中的 Pipeline"
date: 2019-10-10T10:45:25+08:00
keywords: []
description: ""
tags: [
    "redis"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
markup: mmark
mathjax: true 
---


# 1 为什么要有 pipeline 

要了解这个问题首先需要了解 redis 性能瓶颈在哪里，其实，redis 性能问题很多时候在网络传输上，我来计算一下为什么会这样。首先 redis 客户端执行一条命令分为如下四个过程：

1. 发送命令
2. 命令排队
3. 命令执行
4. 返回结果

其中 1 和 4  合起来称为 RTT （Round Trip Time 往返时间），如果 redis 客户端在北京，redis 服务端在上海，两地直线距离约为 1300 公里，那么$$一次 RTT 时间 = 1300 * 2 / （ 300000 * 2/3 ) = 13 ms$$（光在真空中的传输速度为 30 万公里/秒，这里假设光纤中的光速为 2/3），那么客户端在 1 秒内大约只能执行 80 次命令左右，这和 redis 高并发高吞吐的特性背道而驰。

通过 pipeline 可以改善这个问题，我们可以将多个命令一次性发过去，然后将结果一次性传回来，这样就可以有效提高吞吐量。

# 2 对 pipeline 进行基准测试

如下是 10000 条 set 在非 pipeline 和 pipeline 执行时间的比较

| 网络       | 延迟   | 非 pipeline | pipeline |
| ---------- | ------ | ----------- | -------- |
| 本机       | 0.17ms | 573ms       | 134ms    |
| 内网服务器 | 0.41ms | 1610ms      | 240ms    |
| 异地服务器 | 7ms    | 78499ms     | 1104ms   |

# 3 原生批量命令与 pipeline 对比

使用 pipeline 可以模拟出批量操作的效果，不过这与原生批量命令有几点区别；

1. 原生批量命令是原子的，pipeline 是非原子的
2. 原生批量命令是一个命令对应多个 key，pipeline 是多个命令对应多个 key
3. 原生批量命令是 redis 服务端支持实现的，而 pipeline 需要服务端与客户端共同实现

# 4 最佳实践

pipeline 虽然好用，但是每次 pipeline 组装的命令个数不能没有节制，否则一次组装 pipeline 数据量过大，一方面会增加客户端的等待时间，另一方面会造成一定的网络阻塞，可以将一次包含大量命令的 pipeline 拆分成多次较小的 pipeline 来完成。

# 参考资料

1.[Redis 开发与运维](https://gitee.com/zhixiangyuan/bookStorage/raw/master/%E7%BC%96%E7%A8%8B/Redis%20%E5%BC%80%E5%8F%91%E4%B8%8E%E8%BF%90%E7%BB%B4.pdf)
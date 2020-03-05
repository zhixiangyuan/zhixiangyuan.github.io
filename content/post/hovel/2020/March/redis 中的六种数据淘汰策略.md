---
title: "redis 中的六种数据淘汰策略"
date: 2020-03-03T17:08:23+08:00
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
---


```
1. volatile-lru   ：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的 key
2. volatile-ttl   ：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的 key 优先移除
3. volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个 key
4. allkeys-lru    ：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key
5. allkeys-random ：当内存不足以容纳新写入数据时，在键空间中，随机移除某个 key
6. no-enviction   ：当内存不足以容纳新写入数据时，新写入操作会报错
```

# 参考文章

1. [Redis](https://juejin.im/post/5ad6e4066fb9a028d82c4b66)
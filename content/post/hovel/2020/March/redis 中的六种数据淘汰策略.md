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
1. volatile-lru   ：从已设置过期时间的数据集中挑选最近最少使用的数据淘汰
2. volatile-ttl   ：从已设置过期时间的数据集中挑选将要过期的数据淘汰
3. volatile-random：从已设置过期时间的数据集中任意选择数据淘汰
4. allkeys-lru    ：从数据集中挑选最近最少使用的数据淘汰
5. allkeys-random ：从数据集中任意选择数据淘汰
6. no-enviction   ：禁止驱逐数据
```

# 参考文章

1. [Redis](https://juejin.im/post/5ad6e4066fb9a028d82c4b66)
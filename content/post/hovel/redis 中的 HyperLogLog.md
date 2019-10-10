---
title: "Redis 中的 HyperLogLog"
date: 2019-10-10T14:22:00+08:00
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

HyperLogLog 并不是数据结构而是一种基数算法，通过 HyperLogLog 可以利用极小的内存空间完成独立总数的统计，数据集可以是 IP、Email、ID 等。HyperLogLog 提供了 3 个命令：pfadd、pfcount、pfmerge。

# 1 命令

## 1.1 pfadd 添加

`pfadd [key] [element...]`

该命令用于想 HyperLogLog 中添加元素，如果添加成功则返回 1

```shell
$redis-cli> pfadd 2016_03_06:unique:ids "uuid-1" "uuid-2" "uuid-3" "uuid-4"
```

## 1.2 pfcount 用于 key 中存储的 value 的数量

`pfcount [key...]`

```shell
# 向 HyperLogLog 中添加数据
$redis-cli> pfadd test 1 2 3
# 统计数量
$redis-cli> pfcount test
3
```

使用该命令统计多个 key 时似乎是只显示出最大值

## 1.3 pfmerge 合并

`pfmerge [destkey] [sourcekey...]`

将 sourcekey 合并，并将结果放入 destkey

# 2 注意

使用 HyperLogLog 时需要考虑是否满足以下两条：

1. 只为了计算独立总数，不需要获取单条数据
2. 可以容忍一定误差率，毕竟 HyperLogLog 在内存的占用量上有很大的优势
    - redis 官方给出的误差率是 0.81%

# 参考资料

1.[Redis 开发与运维](https://gitee.com/zhixiangyuan/bookStorage/raw/master/%E7%BC%96%E7%A8%8B/Redis%20%E5%BC%80%E5%8F%91%E4%B8%8E%E8%BF%90%E7%BB%B4.pdf)
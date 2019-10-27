---
title: "Redis 中的数据结构"
date: 2019-10-27T13:06:22+08:00
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
mathjax: false
---

# 1 Redis 中的数据结构

redis 中的数据结构是 key value 形式的，key 是字符串，value 有五种数据结构

1. string
2. list
3. hash
4. set
5. sorted set

# 2 Redis 中底层的数据结构

1. 简单动态字符串（SDS）
2. 链表
3. 字典
4. 跳跃表
5. 整数集合
6. 压缩列表
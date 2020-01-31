---
title: "Netty 已实现的解码器"
date: 2020-01-31T12:52:40+08:00
keywords: []
description: ""
tags: [
    "netty"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
draft: true
---


# 1 固定长度解码器

这个解码器非常简单它只需要输入一个参数就是每一帧的长度，然后内部就会按照固定的长度进行切分。比如说输入数据为 `123456` 那么在解析后向下传递的数据便是 `123`、`456`。对应的实现类是 `FixedLengthFrameDecoder`。

```java
    // 该参数在 FixedLengthFrameDecoder 构造函数中进行设置
    private final int frameLength;
```


# 2 基于行的解码器
---
title: "MySQL 数值处理函数"
date: 2019-10-29T10:04:43+08:00
keywords: []
description: ""
tags: [
    "MySQL"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
draft: true
# markup: mmark
# mathjax: true
---

# 1 数值处理函数

| 函数            | 说明                       |
| --------------- | -------------------------- |
| `Abs(<number>)` | 返回一个数的绝对值         |
| `Cos(<number>)` | 返回一个角度的余弦         |
| `Exp(<number>)` | 返回一个数的指数值，等同于 |
| Mod()           | 返回除操作的余数           |
| Pi()            | 返回圆周率                 |
| Rand()          | 返回一个随机数             |
| Sin()           | 返回一个角度的正弦         |
| Sqrt()          | 返回一个数的平方根         |
| Tan()           | 返回一个角度的正切         |

下图是三角函数值的表，可以与 MySQL 计算的值对照一下

![三角函数值对照表](/hub/2019/october/13.png)

## 1.1 `Abs(<number>)`

```shell
mysql> SELECT Abs(-1);
+---------+
| Abs(-1) |
+---------+
|       1 |
+---------+
```

## 1.2 `Cos(<number>)` 

```shell
mysql> SELECT Cos(0);
+--------+
| Cos(0) |
+--------+
|      1 |
+--------+

mysql> SELECT Cos(PI());
+-----------+
| Cos(PI()) |
+-----------+
|        -1 |
+-----------+
```



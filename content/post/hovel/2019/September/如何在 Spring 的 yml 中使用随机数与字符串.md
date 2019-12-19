---
title: "如何在 Spring 的 Yml 中使用随机数与字符串"
date: 2019-09-19T17:31:13+08:00
keywords: []
description: ""
tags: [
    "spring"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---


| 占位符                        | 描述                     |
| ----------------------------- | ------------------------ |
| `${random.value}`             | 取得随机字符串           |
| `${random.int}`               | 取得随机 int 型数据      |
| `${random.long}`              | 取得随机 long 型数据     |
| `${random.int(10)}`           | 取得 10 以内的随机数     |
| `${random.int[10,20]}`        | 取得 10~20 的随机数      |
| `${自定义占位符 or 环境变量}` | 自定义占位符 or 环境变量 |
---
title: "Object 中 hashcode() 与 equals() 的关系"
date: 2019-09-17T17:14:02+08:00
keywords: []
description: ""
tags: [

]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

情况分为以下四种：

1. hashcode 相等，equals 可能相等
2. hashcode 不相等，equals 一定不相等
3. equals 相等，hashcode 一定相等
   - equals 相等说明是同一个对象，所以 hashcode 一定相等
4. equals 不相等，hashcode 可能相等
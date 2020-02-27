---
title: "include 中使用 <> 和 \"\" 的区别"
date: 2020-02-27T17:37:45+08:00
keywords: []
description: ""
tags: [
    "C 语言"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---


```c
// 使用 <> 表示先从系统环境变量下面寻找，如果没找到再去当前目录寻找
#include <stdio.h>
// 使用 "" 表示先从当前目录下寻找，如果没找到再去环境变量下寻找
#include "stdio.h"

// 一般对于系统的库使用 <>，对于用户自定义的库使用 ""
```

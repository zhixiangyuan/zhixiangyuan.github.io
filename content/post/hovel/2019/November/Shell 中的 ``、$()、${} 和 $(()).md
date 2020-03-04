---
title: "Shell 中的 ``、$()、${} 和 $(())"
date: 2019-11-17T21:30:01+08:00
keywords: []
description: ""
tags: [
    "linux"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 ${<content>}

这个符号用于变量替换

# 2 `` 和 $(<content>)

这两个都用于命令替换，意思就是他们两个里面写的东西会当成命令执行。

`` 所有的系统都支持，$() 可能会有系统不支持

# 3 $((<content>))

这个符号中间放的内容用于数学运算

# 参考资料

1. [shell 中 $(())、$( )、`` 与 ${ } 的区别](https://www.jianshu.com/p/7e711c297c02)


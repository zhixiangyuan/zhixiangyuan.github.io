---
title: "Export 命令"
date: 2019-11-06T10:50:16+08:00
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

# 1 语法

`export [-fnp] [{$变量名称}={$变量的值}]`

# 2 命令作用

设置环境变量，使用 export 设置的环境变量仅改变当前进程的环境，当前进程销毁则设置的环境变量失效。

# 3 参数选项

1. `-f` 代表 $变量名称 为函数名称
2. `-n` 删除指定的变量，实际上并未删除，只是不会输出到后续指令的执行环境中。
3. `-p` 列出所有 shell 赋予程序的环境变量

# 参考资料

1. 《Linux Shell 编程从入门到精通》

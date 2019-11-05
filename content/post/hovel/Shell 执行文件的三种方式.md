---
title: "Shell 执行文件的三种方式"
date: 2019-11-05T23:07:50+08:00
keywords: []
description: ""
tags: [
    "Shell"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 Shell 执行文件的三种方式

1. 使文件具有可执行权限，直接运行文件
2. 直接调用命令解释器执行文件，如 sh、bash
3. 使用 source 执行文件

## 1.1 前两种h执行文件的方式

前两种方式执行脚本的时候，系统发现不是内建命令，于是父进程创建一个和自己一摸一样的子进程，此时父进程阻塞，同时执行子进程，当子进程执行完命令后退出子进程，父进程退出阻塞接着执行。

这种方式脚本无法修改父进程的环境变量，如 $PWD

## 1.3 使用 source 执行文件

使用 source 命令执行脚本则不会开辟子进程，而是在当前进程中执行脚本，所以脚本中修改的环境变量会影响到当前进程中的环境变量。

### 1.3.1 source 命令的语法

```shell
source <$script>
. <$script>
```

# 参考资料

1. 《Linux Shell 编程从入门到精通》
---
title: "Shell 中的 #!"
date: 2019-11-05T23:22:09+08:00
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

# 1 #! 的作用

它的作用便是高所 shell 使用什么进程来执行此脚本

# 2 例子

## 2.1 echo Hello world

```shell
#!/bin/bash
echo "Hello world!"
```

## 2.2 python 的 Hello world

```shell
#!/usr/bin/python
print "Hello world!"
```

## 2.3 只会删除自己的脚本

```shell
#!/bin/rm
# 这个 Hello world 打印不出来
echo "Hello world!"
```

## 2.4 给 README 文件加上 #!/bin/more

```shell
#!/bin/more
....README
```

# 参考资料

1. 《Linux Shell 编程从入门到精通》
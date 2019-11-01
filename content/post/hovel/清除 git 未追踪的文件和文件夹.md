---
title: "清除 Git 未追踪的文件和文件夹"
date: 2019-11-01T11:08:07+08:00
keywords: []
description: ""
tags: [
    "Git"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

清除未追踪的文件使用 git clean 命令，git clean 有以下几个常用参数

```shell
-f # f 应该是 file 的缩写，表示删除未追踪的文件
-d # d 应该是 directory 的缩写，表示删除未追踪的文件和文件夹
-n # 与 f 和 d 组合使用，查看要删除哪些文件，并不真的删除
```

```shell
# 删除未追踪的文件
$> git clean -f 

# 查看即将删除的文件
$> git clean -nf

# 删除未追踪的文件和文件夹
# 这里需要注意的是，如果你的文件夹内没有文件
# 那么执行这个命令同样会删除该文件夹
# df 需要一起使用，仅使用 d 会报错
$> git clean -df

# 查看即将删除的文件和文件夹
$> git clean -ndf
```
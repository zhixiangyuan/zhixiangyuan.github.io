---
title: "Unset 命令"
date: 2019-11-06T11:59:46+08:00
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

```shell
# 删除某个变量
$> unset <$变量名>
# 或
$> unset -v <$变量名>

# 删除某个函数
$> unset -f <$函数名>
```

案例：

```shell
# 设置 money 变量
$> money=1billion
# 查看 money 变量
$> echo $money
1billion
# 删除 money 变量
$> unset money
# 再次查看 money
$> echo $money
# 删么也没有
```
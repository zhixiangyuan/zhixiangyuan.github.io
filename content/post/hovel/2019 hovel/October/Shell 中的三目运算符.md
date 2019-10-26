---
title: "Shell 中的三目运算符"
date: 2019-10-13T19:01:27+08:00
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

shell 中的三目运算符有四种，分别是 :-、:+、:=、:?，下面分别举例

# 1 :- 

`${var:-string}` 对于这种表达式表示若 $var 为空则使用 string 替换 `${var:-string}`，否则则使用 $var

# 2 := 

`${var:=string}` 这个与 :- 的不同之处在于它不仅替换而且将 string 赋值给 var

# 3 :+

`${var:+string}` 与 `${var:-string}` 相反，当 $var 不为空才使用 string 替换，$var 为空则使用空值替换

# 4 :?

`${var:?string}` 与 `${var:-string}` 相似，当 $var 不为空时，使用 $var 的值，如果 $var 为空，则将 string 输出到标准错误中，并从脚本中退出。我们可以利用此特性来检查是否设置了变量的值。

# 参考资料

1. [Shell 基石：模式匹配](https://www.jianshu.com/p/776fccbef083)


---
title: "Shell 中的变量"
date: 2019-11-05T23:42:07+08:00
keywords: []
description: ""
tags: [
    "Linux"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 变量的名称

shell 变量的名称以字母或下划线开始，后面可以接任意长度的字母、数字或下划线。

# 2 获取变量值

使用 $ 来获取变量值，如 $var，这种表示方式是 ${var} 的缩写

# 3 变量的赋值

Shell 中全是字符串类型，如果字符串中存储的是整数，那么则可以进行整数运算（+、-、*、/）。

赋值时使用 = 来进行赋值，两边都不能有空格

使用 `"<$var>"` 扩起来的字符串，中间可以使用变量替换，使用 `'<$var>'` 扩起来的字符串，中间不能进行变量替换

变量名和用来赋值的字符串长度没有限制

未赋值的变量值是 NULL，对于数字而言就是 0

# 4 全局变量与局部变量

局部变量必须明确以 local 声明，否则即使在代码块中，它也是全局可见的。

环境变量是局部变量的一种

```shell
#!/bin/bash

num=123
func1 () {
    num=456
    echo 'func1: '$num
}
func2 () {
    local num=789
    echo 'func2: '$num
}
echo 'before func1: '$num
func1
echo 'after func1: '$num
echo 'before func2: '$num
func2
echo 'after func2: '$num

# 以下是输出
before func1: 123
func1: 456
after func1: 456
before func2: 456
func2: 789
after func2: 456
```


# 参考资料

1. 《Linux Shell 编程从入门到精通》

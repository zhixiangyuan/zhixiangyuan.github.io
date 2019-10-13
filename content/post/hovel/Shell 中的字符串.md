---
title: "Shell 中的字符串处理"
date: 2019-10-13T19:27:09+08:00
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

# 1 字符串截取

使用 %、\# 实现字符串截取

# 1.1 %

`${variable%pattern}`，这种模式时，shell 在 variable 中查找，看它是否一给的模式 pattern 结尾，如果是，就从命令行把 variable 中的内容去掉右边最短的匹配模式

# 1.2 %%

`${variable%%pattern}`，这种模式时，shell 在 variable 中查找，看它是否一给的模式 pattern 结尾，如果是，就从命令行把 variable 中的内容去掉右边最长的匹配模式

# 1.3 \#

`${variable#pattern}` 这种模式时，shell 在 variable 中查找，看它是否一给的模式 pattern 开始，如果是，就从命令行把 variable 中的内容去掉左边最短的匹配模式

# 1.4 \#\#

`${variable##pattern}` 这种模式时，shell 在 variable 中查找，看它是否一给的模式 pattern 开始，如果是，就从命令行把 variable 中的内容去掉左边最长的匹配模式

# 1.5 关于 pattern

pattern 可以使用 \*、?，其中 \* 表示零个或多个任意字符，? 表示仅与一个任意字符匹配

# 1.6 案例

```shell
#!/usr/bin/env bash
var="testcase"

echo "去掉右边最短的匹配模式："
echo '${var%s*e}'" = $var -> ${var%s*e}"
echo 

echo "去掉右边最长的匹配模式："
echo '${var%%s*e}'" = $var -> ${var%%s*e}"
echo

echo "去掉左边最短的匹配模式："
echo '${var#?e}'" = $var -> ${var#?e}"
echo

echo "去掉左边最长的匹配模式："
echo '${var##?e}'" = $var -> ${var##?e}"
echo

echo "去掉左边最长的匹配模式："
echo '${var##*e}'" = $var -> ${var##*e}"
echo

echo "去掉左边最长的匹配模式："
echo '${var##*s}'" = $var -> ${var##*s}"
echo

# 以下是输出结果
去掉右边最短的匹配模式：
${var%s*e} = testcase -> testca

去掉右边最长的匹配模式：
${var%%s*e} = testcase -> te

去掉左边最短的匹配模式：
${var#?e} = testcase -> stcase

去掉左边最长的匹配模式：
${var##?e} = testcase -> stcase

去掉左边最长的匹配模式：
${var##*e} = testcase ->

去掉左边最长的匹配模式：
${var##*s} = testcase -> e
```

# 2 字符串提取

## 2.1 ${var:num}

`${var:num}` 表示从第 num 个字母开始提取向后的所有字母，数组下标以 0 开始

## 2.2 ${var:num1:num2}

`${var:num1:num2}` 表示提取数组下标从 num1 开始的字母，并向后取 num2 个字母，数组下标以 0 开始

## 2.3 案例

```shell
#!/usr/bin/env bash

var=123451

echo '${var:1}'" = ${var} -> ${var:1}"

echo '${var:1:1}'" = ${var} -> ${var:1:1}"
# 以下为输出结果
${var:1} = 123451 -> 23451
${var:1:1} = 123451 -> 2
```

# 3 字符串的截取

## 3.1 ${var/pattern/pattern}

`${var/pattern/pattern}` 表示将 var 字符串的第一个匹配的 pattern 替换为另一个 pattern

## 3.2 ${var//pattern/pattern}

`${var//pattern/pattern}` 表示将 var 字符串中的所有能匹配的 pattern 替换为另一个 pattern

## 3.3 案例

```shell
#!/usr/bin/env bash

var=123451

echo '${var/1/2}'" = ${var} -> ${var/1/2}"

echo '${var//1/2}'" = ${var} -> ${var//1/2}"

# 以下是输出结果
${var/1/2} = 123451 -> 223451
${var//1/2} = 123451 -> 223452
```



# 参考资料

1. [Shell 基石：模式匹配](https://www.jianshu.com/p/776fccbef083)
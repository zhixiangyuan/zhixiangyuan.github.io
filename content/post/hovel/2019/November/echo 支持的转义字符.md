---
title: "Echo 支持的转义字符"
date: 2019-11-06T09:51:39+08:00
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

echo 需要加上 -e 才能够处理转义字符

| 转义字符 | 含义                                                 |
| -------- | ---------------------------------------------------- |
| \a       | 报警符，计算机会发出一声报警音                       |
| \b       | 退格符（Backspace）                                  |
| \c       | 禁止尾随，这个字符后面的所有字符都会被忽略掉，不打印 |
| \f       | 换页符（清除屏幕）                                   |
| \n       | 换行符（Newline）                                    |
| \r       | 回车符（Carriage return）                            |
| \t       | 水平制表符（Horizontal tab）                         |
| \v       | 垂直制表符（Vertical tab）                           |
| `\\`     | 反斜线                                               |

以下是例子：

```shell
# 演示 \a
echo -e "\a"
# 计算机发出一声报警音

# 演示 \b
echo -e "aaa\bbb"
# 输出：aabb

# 演示：\c
echo -e "This is a test.\cThis content can't output."
# 输出：This is a test.

# 演示：\f
# 这个 \f 没什么卵用，并不能清除屏幕
echo -e "before"
echo -e "before-\fafter"
echo -e "after"
# 输出：
# before
# before-
#        after
# after

# 演示：\n
echo -e "before-\nafter"
# 输出：
# before-
# after

# 演示：\r
# 这个 \r 很奇怪，仅仅帮我换到了行首，
# 然后输出的内容覆盖了一部分 before，
# mac 系统里面的 \r 应该是换到下一行
# 的行首才对吧
echo -e "before-\rafter"
# 输出：aftere-

# 演示：\t
echo -e "before-\tafter"
# 输出：before-	after

# 演示：\v
echo -e "before-\vafter"
# 输出：
# before-
#        after

# 演示：\\
# 很奇怪，\\ 解析的有问题
echo -e "before-\\after"
# 输出：before-fter
# 还输出了一声报警音
```

# 参考资料

1. 《Linux Shell 编程从入门到精通》
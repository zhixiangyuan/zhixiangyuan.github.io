---
title: "Redis 客户端通信协议"
date: 2019-10-11T13:30:14+08:00
keywords: []
description: ""
tags: [

]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

redis 的协议客户端通信协议称为 RESP（Redis Serialization Protocol），这种协议简单高效即能被机器解析，又能被人类识别。

# 1 发送命令的格式

CRLF 指的是 \r\n

```redis
*<参数数量> CRLF
$<参数 1 的字节数量> CRLF
<参数1> CRLF
...
$<参数 N 的字节数量> CRLF
<参数N> CRLF

举例：
比如说 set hello wrold 命令会构造成

*3
$3
SET
$5
hello
$5
world

实际格式化的传输结果为下面这样，看起来有点累
*3\r\n$3\r\nSET\r\n$5\r\nhello\r\n$5\r\nworld\r\n
```

# 2 返回结果格式

redis 的返回结果类型分为以下五种

- 状态回复：在 RESP 中第一个字节为 "+"
- 错误回复：在 RESP 中第一个字节为 "-"
- 整数回复：在 RESP 中第一个字节为 ":"
- 字符串回复：在 RESP 中第一个字节为 "$"
- 多条字符串回复：在 RESP 中第一个字节为 "*"

# 参考资料

1.[Redis 开发与运维](https://gitee.com/zhixiangyuan/bookStorage/raw/master/%E7%BC%96%E7%A8%8B/Redis%20%E5%BC%80%E5%8F%91%E4%B8%8E%E8%BF%90%E7%BB%B4.pdf)
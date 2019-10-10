---
title: "Redis 中的发布订阅模式"
date: 2019-10-10T15:04:46+08:00
keywords: []
description: ""
tags: [
    "redis"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

redis 提供发布消息、订阅消息、取消订阅、按照模式订阅和取消订阅和查阅订阅的功能

# 1 命令

## 1.1 发布消息

`publish <channel> <message>`

```shell
# 下面的命令向 channel:sports 频道发布一条消息 "Tim won the championship"
# 返回的结果为订阅者的个数
$redis-cli> publish channel:sports "Tim won the championship"
```

## 1.2 订阅消息

`subscribe <channel> [channel...]`

订阅者可以订阅一个或多个频道

```shell
$redis-cli> subscribe channel:sports
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channel:sports"
3) (integer) 1

# 如果此时别的客户端发送一条消息
$redis-cli-other> publish channel:sports hello

# 那么现在订阅的客户端就会收到以下消息
...
1) "message"
2) "channel:sports"
3) "hello"
```

需要注意的是，redis 和专业的消息队列不同，它不会对消息进行缓存，所以如果在消息发送之后订阅的客户端是没法收到之前的消息的。

## 1.3 取消订阅

`unsubscribe <channel> [channel...]`

客户端可以通过 unsubscribe 命令取消对指定频道的订阅

## 1.4 按照模式订阅和取消订阅

`psubscribe <pattern> [pattern...]`

`punsubscribe <pattern> [pattern...]`

```shell
# 订阅以 it 开头的所有频道
$redis-cli> psubscribe it*
```

## 1.5 查阅订阅

### 1.5.1 查看活跃的频道

`pubsub channels <pattern>`

所谓活跃的频道是指当前频道至少有一个订阅者，其中 [pattern] 可以是指定的具体的模式

### 1.5.2 查看频道订阅数

`pubsub numsub <channel> [channel...]`

### 1.5.3 查看模式订阅数

`pubsub numpat`

```shell
$redis-cli> pubsub numpat
(integer) 1
# 当前只有一个客户端通过模式来订阅
```

# 参考资料

1.[Redis 开发与运维](https://gitee.com/zhixiangyuan/bookStorage/raw/master/%E7%BC%96%E7%A8%8B/Redis%20%E5%BC%80%E5%8F%91%E4%B8%8E%E8%BF%90%E7%BB%B4.pdf)
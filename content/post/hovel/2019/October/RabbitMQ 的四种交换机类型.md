---
title: "RabbitMQ 的四种交换机类型"
date: 2019-10-26T11:09:38+08:00
keywords: []
description: ""
tags: [
    "RabbitMQ"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

消息在 RabbitMQ 中的运动流程是先从生产者推送消息到交换机，交换机再将消息路由到队列，最后消费者消费队列中的消息。

RabbitMQ 的交换机类型分为四种：

1. 扇形交换机：fanout
2. 直连交换机：direct
3. 主题交换机：topic
4. 首部交换机：headers

# 1 扇形交换机

扇形交换机会将收到的消息广播到绑定在自身的队列上，不做任何处理，所以扇形交换机的处理速度很快

![扇形交换机](/media/hovel/59.png)

# 2 直连交换机

直连交换机与扇形交换机不同，扇形交换机接收到消息就直接转发到与自身绑定的队列，直接交换机接收到之后还需要判断一下。当队列与直连交换机绑定的时候还会绑定一个 routing_key，在消息发送过来的时候会带一个 binding_key，直接交换机收到消息后会将消息发送到 binding_key 与 rooting_key 完全相同的队列。

适用场景：有优先级的任务，根据任务的优先级把消息发送到对应的队列，这样可以指派更多的资源去处理高优先级的队列。

![直连交换机](/media/hovel/61.png)

# 3 主题交换机

主题交换机比直连交换机更加强大，如果我们需要多种形式的 routing_key，比如说 `test.key.dog`、`test.key.cat` 等，那么我们需要写很多的 routing_key 才行。主题交换机提供了一个功能，通过类似于正则表达式的方式来实现 routing_key 与 binding_key 的匹配，比如说 `test.key.#`，那么  `test.key.dog` 和 `test.key.cat` 发送过来都会被识别成 `test.key.#`，然后发送到 相应的队列

- `*` 表示一个单词
- `#` 表示任意数量（零个或多个）单词

![主题交换机](/media/hovel/62.png)

# 4 首部交换机

首部交换机是忽略 routing_key 的一种路由方式，路由规则是通过 Headers 信息来交换的，这个有点像 HTTP 的 Headers。将一个交换机声明成首部交换机，绑定一个队列的时候，定义一个 Hash 的数据结构，消息发送的时候，会携带一组 hash 数据结构的信息，当 Hash 的内容匹配上的时候，消息就会被写入队列。

绑定交换机和队列的时候，Hash 结构中要求携带一个键 `x-match`，这个键的 value 可以是 any 或者 all，这代表消息携带的 Hash 是需要全部匹配（all），还是仅匹配一个键（any）就可以。相比于直连交换机，首部交换机的优势是匹配的规则不被限定为字符串（String）。

# 参考资料

1. [RabbitMQ 的四种交换机](https://www.jianshu.com/p/469f4608ce5d)
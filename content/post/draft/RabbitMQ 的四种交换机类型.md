---
title: "RabbitMQ 的四种交换机类型"
date: 2019-10-23T14:09:38+08:00
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
draft: true
---

消息在 RabbitMQ 中的运动流程是先从生产者推送消息到交换机，交换机再将消息路由到队列，最后消费者消费队列中的消息。

RabbitMQ 的交换机类型分为四种：

1. 直连交换机：direct
2. 扇形交换机：fanout
3. 主题交换机：topic
4. 首部交换机：headers

# 1 直连交换机

# 2 扇形交换机

扇形交换机会将收到的消息广播到绑定在自身的队列上，不做任何处理，所以扇形交换机的处理速度很快

![扇形交换机](/media/hovel/59.png)

# 3 主题交换机

# 4 首部交换机

# 参考资料

1. [RabbitMQ 的四种交换机](https://www.jianshu.com/p/469f4608ce5d)
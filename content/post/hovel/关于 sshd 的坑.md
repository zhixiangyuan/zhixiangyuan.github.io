---
title: "关于 sshd 的坑"
date: 2019-01-16T20:30:59+08:00
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

# 1 sshd 死都打不开

试试先用 /usr/sbin/sshd stop 关掉再 start 看看行不行

莫名其妙，直接运行 /usr/sbin/sshd 就好了

# 1.1 开启 sshd 

/usr/sbin/sshd start

# 1.2 sshd 的配置文件权限必须在 root 下为 600 才行

配置文件在 /etc/ssh 目录下，目录下的文件权限必须在 root 下为 600 才行





---
title: "Centos 配置无密登录"
date: 2019-01-17T20:34:37+08:00
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

# 生成密钥方法

1. ssh-keygen -t rsa // 一路回车
2. cat id_rsa.pub >> authorized_keys

# 配置权限

- chmod 700 /home/hadoop
- chmod 700 -R ~/.ssh 
- chmod 600 authorized_keys
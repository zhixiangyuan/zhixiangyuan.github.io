---
title: "Google 云配置 SSH 为公网可登陆"
date: 2020-02-14T21:43:55+08:00
keywords: []
description: ""
tags: [
    "M78 星云"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 使用浏览器 SSH 登陆服务器

1. 使用 `sudo -i` 切换到 root 用户
2. `vi /etc/ssh/sshd_config` 修改配置文件
   - 修改以下两项配置为 yes
   - `PermitRootLogin yes // 默认为 no，需要开启 root 用户访问改为 yes`
   - `PasswordAuthentication yes // 默认为 no，改为 yes 开启密码登陆`
3. 给 root 用户设置密码 `passwd root`
4. 重启 SSH 服务使修改生效 `/etc/init.d/ssh restart`
---
title: "Linux 下误删 Secure 文件，系统不记录日志问题"
date: 2019-01-13T20:16:21+08:00
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

第一步创建文件

`touch /var/log/secure`

第二步修改权限

`chmod 600 /var/log/secure`

第三步重启服务

1. `service sshd restart`
2. `service syslog(rsyslog) restart`

注意：如果不重启服务的话，原来的程序依然向被删除的文件中写入数据，那么新建的文件就没卵用
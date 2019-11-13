---
title: "Sftp 命令小记"
date: 2019-01-14T20:26:53+08:00
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

sftp 命令会先连接到服务器，然后再下载文件

# 登陆命令

`sftp <username>@<ip>`

# 下载命令

递归下载 

`get -r <$文件> <$本地路径>
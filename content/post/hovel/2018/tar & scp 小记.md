---
title: "tar & scp 小记"
date: 2018-09-24T17:35:38+08:00
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

# 1 scp 命令

- 上传：`scp [文件路径] [用户名]@[IP 地址]:[服务器路径]`
- 下载：`scp [用户名]@[IP 地址]:[文件路径] [本地路径]`

# 2 tar 命令

- 打包命令：`tar cvf [file name].tar [dirname]`
- 解包命令：`tar xvf [tar name]`
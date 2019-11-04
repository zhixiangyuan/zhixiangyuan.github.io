---
title: "InnoDB 数据页结构"
date: 2019-11-04T22:08:30+08:00
keywords: []
description: ""
tags: [
    "MySQL"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
draft: true
---

# 1 

由于将内存中的数据写入到磁盘上会很慢，所以 InnoDB 采取的方式是将数据划分为若干个页，以页作为磁盘和内存之间交互的基本单位，InnoDB 中页的大小一般为 16 KB。
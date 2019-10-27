---
title: "B+树和 B 树的区别"
date: 2019-10-27T11:26:24+08:00
keywords: []
description: ""
tags: [
    "数据结构与算法"
]
categories: [

]
autoCollapseToc: false
author: "yuanzx"
markup: mmark
mathjax: false
---

B 树的非叶子结点存储实际记录的指针

B+ 树的非叶子节点不存储实际记录的指针，所有的指针均存储在叶子结点，并且叶子结点使用指针串了起来，形成了链表的结构，适合扫描区间和顺序查找

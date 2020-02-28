---
title: "Git 提交如何关联到 Issue"
date: 2020-02-28T10:31:53+08:00
keywords: []
description: ""
tags: [
    "github"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

github 会做自动关联，需要在 commit 中提及 issue，建议使用下面的语法

```
$> git commit -m "Fix: #<$issue_id> <$commemt_content>"
```

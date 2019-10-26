---
title: "一条好用的 Git 别名命令"
date: 2019-10-25T13:32:41+08:00
keywords: []
description: ""
tags: [
    "Git"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

```shell
$> git config --global alias.loginfo "log --pretty=format:'%C(auto) %h | %ai | %Cred  %an %Cgreen %s %C(auto) %d' --date=local"
# 使用时输入 git loginfo
```

该命令是对于 `git log` 界面的格式化，格式化之后显示效果看起来更舒服，下图是命令执行过后看起来的效果。

![git loginfo](/media/hovel/60.png)
---
title: "解决升级系统后 IntelliJ IDEA 报 Can't use Subversion command line client: svn 的问题"
date: 2019-11-01T10:00:53+08:00
keywords: []
description: ""
tags: [
    "IntelliJ IDEA"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

系统版本：macOS Catalina Version 10.15.1

在升级完系统后出现了下图中的错误

![idea 的报错](/hub/2019/November/1.png)

这个问题其实是由于命令行中 svn 命令失效导致的，在命令行中输入 svn 爆出以下错误

```shell
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
```

简单来说就是系统装完了，但是库少东西了，所以执行以下命令重装一下库就可以了

```shell
$> xcode-select --install
```
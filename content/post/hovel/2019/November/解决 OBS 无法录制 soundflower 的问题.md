---
title: "解决 OBS 无法录制 Soundflower 的问题"
date: 2019-11-16T09:45:12+08:00
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

本文中使用的 OBS 为 24.0.2 版本，这个版本在最新的 catalina 有 bug，它不会自动申请麦克风权限，我又没办法把权限给他，就造成了没法录声音的问题，解决办法是通过命令行运行，`open /Applications/OBS.app/Contents/MacOS/OBS --args -picture`，通过这个命令可以让 OBS 来申请权限，我们在打开的时候点允许就解决问题了。
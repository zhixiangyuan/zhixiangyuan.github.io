---
title: "制作 MacOS 安装盘"
date: 2019-10-14T15:19:45+08:00
keywords: []
description: ""
tags: [
    "macos"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

1. 第一步，在网上或者 app store 中找到 macos 镜像，然后下载
2. 第二步，通过命令进行制作安装盘，命令如下

```shell
$> <安装包位置>/Contents/Resources/createinstallmedia --volume /Volumes/<U 盘名称> --applicationpath <安装包位置> --nointeraction
# 比如：
$> sudo /Users/zhixiangyuan/Downloads/Install\ macOS\ Mojave.app/Contents/Resources/createinstallmedia --volume /Volumes/FlashDisk --applicationpath /Users/zhixiangyuan/Downloads/Install\ macOS\ Mojave.app --nointeraction
```
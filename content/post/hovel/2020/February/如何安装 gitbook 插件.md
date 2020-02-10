---
title: "如何安装 gitbook 插件"
date: 2020-02-10T16:14:26+08:00
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

安装 Gitbook 插件，本来是一件很简单的事情，在 `book.json` 文件加入插件相关的信息，比如下面的 `mathjax` 插件，然后运行 `gitbook install` 即可完成安装。

```json
{
    "plugins": ["mathjax"]
}
```

不过由于众所周知的原因，导致网络怎么都访问不上，所以国内的正确安装方法应该如下操作。

gitbook 的插件是用 npm 安装的，并且插件的包名为 `gitbook-plugin-<$plugin_name>`，所以如果需要安装 gitbook 插件，那么先用 `cnpm install gitbook-plugin-<$plugin_name>`，然后再运行 `gitbook install` 即可。
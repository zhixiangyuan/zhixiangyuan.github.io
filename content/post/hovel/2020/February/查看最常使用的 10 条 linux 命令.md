---
title: "查看最常使用的 10 条 linux 命令"
date: 2020-02-27T14:39:57+08:00
keywords: []
description: ""
tags: [
    "linux"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

在终端中输入如下命令即可看到您最常使用的 10 条命令

```shell
$> history | awk '{print $2}' | sort | uniq -c | sort -rn | head -10

# 下面是我的输出结果
1432 cd
1384 git
1250 ll
 683 code
 538 hugo
 280 ./toGit
 272 bs
 250 togit
 194 vim
 107 docker
```

# 参考文章

1. [Linux 初学者 (学习资料)](https://zhuanlan.zhihu.com/p/21723250)
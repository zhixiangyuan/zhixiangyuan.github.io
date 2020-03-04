---
title: "Env 命令"
date: 2019-11-06T11:44:09+08:00
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

```shell
# 输出所有的环境变量
$> env 

# 删除某个环境变量
# 运行某个命令在指定的环境变量下
$> env -u <$环境变量> [运行的命令]

# 删除所有的环境变量
# [$环境变量=$环境变量的值] 为设置的新的环境变量的值
$> env -i [$环境变量=$环境变量的值] [运行的命令]
```

以下为 -u 使用案例：

```shell
# 先打印看一下环境变量
$> export -p | grep USER
# 以下为输出
export USER=zhixiangyuan
export __CF_USER_TEXT_ENCODING=0x0:0:0

# 看一下脚本
$> cat echo.sh
#!/bin/bash
export -p | grep USER

# 删除 USER 环境变量
$> env -u USER ./echo.sh
declare -x __CF_USER_TEXT_ENCODING="0x0:0:0"
```

以下为 -i 使用案例：

```shell
# 查看一下环境变量
$> env
# 输出一堆环境变量
...

# 看一下脚本
$> cat echo.sh 
#!/bin/bash
env

# 清除所有环境变量
$> env -i ./echo.sh
PWD=/Users/zhixiangyuan/Downloads/tmp
SHLVL=1
_=/usr/bin/env

# 设置一个 PATH 变量
$> env -i PATH=./:$PATH ./echo.sh
PATH=./:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Users/zhixiangyuan/workspace/script:/Users/zhixiangyuan/workspace/company_project/smyl/arc/arcanist/bin
PWD=/Users/zhixiangyuan/Downloads/tmp
SHLVL=1
_=/usr/bin/env
```

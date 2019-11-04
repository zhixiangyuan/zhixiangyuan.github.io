---
title: "启动和关闭 MySQL"
date: 2019-11-04T07:39:09+08:00
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

# 1 mysqld

mysqld 这个可执行文件就代表着 MySQL 服务器程序，运行这个可执行文件就可以直接启动一个服务器进程。

# 2 mysqld_safe

mysqld_safe 是一个启动脚本，它会间接的调用 mysqld，而且还顺便启动了另外一个监控进程，这个监控进程在服务器进程挂了的时候，可以帮助重启它。另外，使用 mysqld_safe 启动服务器程序时，它会将服务器程序的出错信息和其他诊断信息重定向到某个文件中，产生出错日志，这样可以方便我们找出发生错误的原因。

# 3 mysql.server

mysql.server 也是一个启动脚本，它会间接的调用 mysqld_safe，在调用 mysql.server 时在后边指定 start 参数就可以启动服务器程序了，就像这样：`mysql.server start`

另外，我们还可以使用 mysql.server 命令来关闭正在运行的服务器程序，只要把 start 参数换成 stop 就好了：`mysql.server stop`

# 4 mysqld_multi

其实我们一台计算机上也可以运行多个服务器实例，也就是运行多个 MySQL 服务器进程。mysql_multi 可执行文件可以对每一个服务器进程的启动或停止进行监控。


---
title: "Linux 基础"
date: 2019-10-12T13:04:55+08:00
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

# useradd

```shell
# 新建用户
$> useradd <usrename>
# 新建用户，并将用户加入指定的组
# 组名必须存在，否则会提示不存在
$> useradd <username> -g <groupname>
```

# passwd

```shell
# 更改当前用户的密码
$> passwd 
# 更改指定用户的密码
$> passwd <username>
```

# usermod

```shell
# 将用户 user_name 加入到 group_name 组
$> usermod -g <$group_name> <$user_name>
# 比如说将 peter 加入 root 组
$> usermod -g root peter
```

# 用户信息存放的位置及存储的内容

```shell
# 这里存放的是用户的信息
$> cat /etc/passwd
# <用户名>:<密码（这里存放的不是真正的密码）>:<用户 ID>:
# <组 ID>:<用户主目录，用户登录后默认进入该目录>:<用户登陆后默认使用的交互命令行>
root: x :0:0:root:/root:/bin/bash
......
newUser: x :1000:1000::/home/cliu8:/bin/bash
```

# 文件信息

```shell
# d 表示文件夹，- 表示文件
# 接下来的 rwx 分为三组，分别表示用户权限，用户所在组权限和其他用户权限
# r 是 read，w 是 write，x 是执行
# 第二个字段表示硬连接树木
# 第三个字段表示所属用户
# 第四个字段表示所属组
# 第五个字段表示文件大小
# 第六个字段是文件被修改的日期
# 最后一个是文件名
drwxr-xr-x 3 root root   4096 Jun  7 21:26 jdk
-rwxr-xr-x 2 root root  20480 Oct 12 13:08 logs
```

# 管理软件

Linux 现在常用的有两大体系，分别是 CentOS 和 Ubuntu 体系，对应的安装包分为 .rpm 和 .deb 结尾。

```shell
# 安装命令，-i 表示 install
$> rpm -i <software>.rpm
$> dpkg -i <software>.rpm

# 查看已安装的软件列表
# -q 表示 query，-a 表示 all
$> rpm -qa
# -l 表示 list
$> dpkg -l

# 可以使用 more 或 less 来分页展示
$> rpm -qa | more
$> rpm -qa | less

# 使用软件管家帮忙安装
$> yum install <software>
$> apt-get install <software>

# 卸载软件
$> yum erase <software>
$> apt-get purge <software>

# 修改下载源
# CentOS 在这个位置
$> vim /etc/yum.repos.d/CentOS-Base.repo
# Ubuntu 在这个位置
$> vim /etc/apt/sources.list
```

# 后台运行程序

```shell
# nohup 命令 no hang up 不挂起，表示命令行退出的时候程序还在
# & 后台运行
# 1”表示文件描述符 1，表示标准输出，“2”表示文件描述符 2，意思
# 是标准错误输出，“2>&1”表示标准输出和 错误输出合并了。合并到
# 哪里去呢? 到 out.file 里。
$> nohup <command> >outfile 2>&1 &
```

# 通过命令关闭程序

```shell
# awk '{print $2}'是指第二列的内容，是运行的程序 ID
# xargs 将传输传递给 kill -9
$> ps -ef |grep 关键字 |awk '{print $2}'|xargs kill -9
```

# 程序跟随系统启动

```shell
# 启动 mysql
$> systemctl start mysql
# 开启启动
# 程序会在 /lib/systemd/system 目录下创建一个 <程序名>.service 的配置文件
# 里面定义了如何启动，如何关闭
$> systemctl enable mysql
```

# 关机与重启

```shell
# 现在关机
$> shutdown -h now 
# 重启
$> reboot
```
---
title: "通过 ssh 配置公网主机端口到内网主机端口的映射"
date: 2019-11-13T14:33:39+08:00
keywords: []
description: ""
tags: [
    "Linux"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 公网主机的配置修改

1. ssh 连接到公网主机，修改 sshd 的配置文件 /etc/ssh/sshd_config，在配置文件中找到 `#GatewayPorts no` 将其改为 `GatewayPorts yes`，如果没有这个配置项，则加上它
2. 重启 sshd，`/bin/systemctl restart sshd.service`
3. 配置内网主机的 ssh 无密登录

# 2 内网主机的配置修改

1. 配置内网端口到公网端口的映射，可以是本机，也可以是内网中别的主机，`ssh -NfR <$公网端口>:<$内网IP>:<$内网端口> root@<$公网IP>`
2. 修改心跳包
   
```shell
# 在内网主机的 ~/.ssh/config 文件中，增加如下两条语句
ServerAliveInterval 60
ServerAliveCountMax 9999999999
```

下面是一个例子

1. 配置本机的 80 端口映射到公网的 8080 端口，`ssh -NfR 8080:localhost:80 root@公网主机ip`
2. 配置内网中某台服务器的 80 端口映射到公网的 8080 端口，`ssh -NfR 8080:192.168.10.22:80 root@公网主机ip`

# 参考资料

1. [外网访问内网方案](https://www.jianshu.com/p/87d3f2ce9640)
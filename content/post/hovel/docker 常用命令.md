---
title: "Docker 常用命令"
date: 2019-01-12T19:56:52+08:00
keywords: []
description: ""
tags: [
    "docker"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 保存镜像

`docker commit <$container_id> <$image_name>`

# 使用镜像新建容器

```shell
docker run
# 以下是一些可选项
# 这是两个参数，一个是 -i：交互式操作，一个是 -t 终端
# 使用这个选项则需要在容器名最后加上 /bin/bash
[-it]
# 对这两个路径做绑定，起到挂载的效果
[-v {$容器中的路径}:{$宿主机中的路径}]
[-h {$hostname}] 
[--name {$机器名称}] 
<$容器名称>
```

# 进入容器有两个命令

进入容器，使用 exit 退出时，容器不会终止（推荐）

`docker exec -it <$container_id> bash`

使用下面的命令进入容器后，使用 exit 退出容器，容器会停止运行

`docker attach <$container_id>`

# 容器管理命令

`docker container start <$container_id>`：启动已经终止的容器，但不会进入容器，容器在后台执行

`docker container stop <$container_id>`：终止容器

`docker container rm <$container_id>`：删除终止状态的容器

# 镜像管理命令

`docker image rm <$image_id>`：删除镜像


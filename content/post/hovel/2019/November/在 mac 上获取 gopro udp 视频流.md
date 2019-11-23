---
title: "在 Mac 上获取 Gopro Udp 视频流"
date: 2019-11-16T09:34:37+08:00
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

gopro 可以直接通过 wifi 传送 udp 视频流，本文讲解如何从其中获取 udp 视频流并在 mac 上播放。

本文使用 gopro 4 black 进行测试，如果使用其他型号的 gopro，触发 url 参考 https://github.com/KonradIT/goprowifihack 这个网址

1. 首先用手机 app 连接 gopro，设置好 wifi，然后使用 mac 连接 gopro 的 wifi
2. 当 mac 连接 gopro wifi 之后，向 http://10.5.5.9/gp/gpControl/execute?p1=gpStream&a1=proto_v2&c1=restart 这个地址发送 get 请求，当 gopro 收到这个 get 请求之后，便开始向我们的电脑发送 udp 数据包

这时候发送数据包一段时间之后会停止，我们需要一直让它活着，直接在命令行运行以下脚本即可

```python
# 文件名可设为 hero4-udp-keep-alive-send.py
import socket
import sys
from time import sleep

def get_command_msg(id):
	return "_GPHD_:%u:%u:%d:%1lf\n" % (0, 0, 2, 0)

UDP_IP = "10.5.5.9"
UDP_PORT = 8554
KEEP_ALIVE_PERIOD = 2500
KEEP_ALIVE_CMD = 2
MESSAGE = get_command_msg(KEEP_ALIVE_CMD)

print("UDP target IP:", UDP_IP)
print("UDP target port:", UDP_PORT)
print("message:", MESSAGE)

if sys.version_info.major >= 3:
  MESSAGE = bytes(MESSAGE, "utf-8")

while True:
	sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
	sock.sendto(MESSAGE, (UDP_IP, UDP_PORT))
	sleep(KEEP_ALIVE_PERIOD/1000)
```

此时虽然 mac 在源源不断的收到 udp 视频流，但是却无法播放，我们使用 `brew install ffmpeg` 安装 ffmpeg，在安装完成之后运行 `ffplay -fflags nobuffer -f:v mpegts -probesize 8192 udp://:8554` 即可看到画面。
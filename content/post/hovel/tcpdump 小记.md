---
title: "Tcpdump 小记"
date: 2019-11-14T09:39:39+08:00
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

![](/hub/2019/November/42.png)

```shell
# -i 表示指定哪一个网卡，any 表示任意。有哪些网卡可以用 ifconfig 来查看
$> tcpdump -i any

# 查看 eth0 网卡经过的数据
$> tcpdump -i eth0

# 查看 host 为 <$ip> 的数据包
$> tcpdump -i any host <$ip>

# 过滤源地址
$> tcpdump -i any src <$ip>

# 过滤目标地址
$> tcpdump -i any dst <$ip>

# 查看某端口通信的 tcp 数据包
$> tcpdump -i any port <$port>

# dst port 是目标端口，查找目标端口为 <$port> 的数据包
$> tcpdump -i any dst port <$port>

# src port 是源端口，查找源端口为 <$port> 的数据包
$> tcpdump -i any src port <$port>

# 过滤指定端口范围内的流量，[21, 23] 端口
$> tcpdump portrange 21-23

# 不将 ip 翻译成主机名 -n
$> tcpdump -i any -n

# 不将 ip 翻译成主机名同时不将 port 翻译成协议名
# 如将 22 端口翻译成 ssh
$> tcpdump -i any -nn

# 过滤协议
$> tcpdump -i any -nn udp

# 用 ASCII 格式查看包体内容 -A
$> tcpdump -i any -nn port 80 -A
11:04:25.793298 IP 183.57.82.231.80 > 10.211.55.10.40842: Flags [P.], seq 1:1461, ack 151, win 16384, length 1460
HTTP/1.1 200 OK
Server: Tengine
Content-Type: application/javascript
Content-Length: 63522
Connection: keep-alive
Vary: Accept-Encoding
Date: Wed, 13 Mar 2019 11:49:35 GMT
Expires: Mon, 02 Mar 2020 11:49:35 GMT
Last-Modified: Tue, 05 Mar 2019 23:30:55 GMT
ETag: W/"5c7f06af-f822"
Cache-Control: public, max-age=30672000
Access-Control-Allow-Origin: *
Served-In-Seconds: 0.002

# 同时显示 HEX 和 ASCII 格式的内容 -X
$> tcpdump -i any -nn port 80 -X
10:11:32.715043 IP 172.17.42.178.59678 > 100.100.109.104.80: Flags [P.], seq 13237:13255, ack 1, win 229, length 18: HTTP
	0x0000:  4500 003a 7fef 4000 4006 123f ac11 2ab2  E..:..@.@..?..*.
	0x0010:  6464 6d68 e91e 0050 bd9b 6de1 7eb4 0a5b  ddmh...P..m.~..[
	0x0020:  5018 00e5 a8bc 0000 3132 3632 3331 2070  P.......126231.p
	0x0030:  6167 655f 696e 3d30 200a 0307 0000 0000  age_in=0........
	0x0040:  0000 0000 0000 0000 0000

# 限制查看包体的内容大小
# 只查看包体前 500 字节的内容
$> tcpdump -i any -nn port 80 -s 500 -X
10:14:14.298880 IP 100.100.30.26.80 > 172.17.42.178.36266: Flags [.], ack 8332, win 24570, length 0
	0x0000:  4500 0028 a9b7 4000 3506 42d7 6464 1e1a  E..(..@.5.B.dd..
	0x0010:  ac11 2ab2 0050 8daa 3669 bcd0 bc1f 0a05  ..*..P..6i......
	0x0020:  5010 5ffa af3f 0000 0000 0000 0000 0000  P._..?..........
	0x0030:  0000 0000 0000 0000 0000 0000 0000       ..............

# -s 0 表示显示所有内容
$> tcpdump -i any -nn port 80 -s 0 -X

# -c 100 表示只抓去 100 个报文然后退出
$> tcpdump -i any -nn port 80 -X -c 100

# 数据报文输出到文件 -w 选项
# 生成的 test.pcap 可以使用 wireshark 打开进行更详细的分析
# 也可以加上 -U 强制立即写到本地磁盘，性能稍差
$> tcpdump -i any port 80 -w test.pcap -c 5

# 默认 tcpdump 显示的 seq 是从 0 开始的相对序号，如果想查看
# 真正的绝对序号，加上 -S 参数即可
$> tcpdump -i any -S

# 支持的布尔运算
# and 也可写成 &&
# or 也可写成 ||
# not 也可写成 !
$> tcpdump -i any -nn dst port 80 && host 100.100.109.104

# 如何使用 () 来对条件进行分组
# 加上 '' 号即可
$> tcpdump -i any 'src 10.211.55.10 and (dst port 3306 or 6379)'

# 如何过滤出 RST 不为 0 的数据包
# tcp 头中第十三个字节的第三位为 RST 
# 4 写成二进制就是 0b00000100
$> tcpdump 'tcp[13] & 4 != 0'

# 18 写成二进制就是 00010010
# 表示过滤出 SYN 和 ACK 不为 0 的数据包
$> tcpdump 'tcp[13] & 18 != 0'
```

# tcpdump 输出解读

先用 `nc -l 9090` 开启一个 tcp 的监听，然后新建一个终端用 `tcpdump -i any port 9090 -nn -A` 监听这个端口，然后再新建一个终端用 `nc 127.0.0.1 9090` 连接服务器便可以看到下面三个包

```shell
# 以下为三次握手的过程
# 14:21:30.393783 是这个包的时间，精确到微妙
# 127.0.0.1.34448 > 127.0.0.1.9090 表示 TCP 四元组，源地址、源端口、目标地址、目标端口，中间的大于号表示包的流向
# Flags [S] 表示 TCP 首部的 flags 字段，S 表示这是一个 SYN 的包
# seq 3273551255 表示的 SYN 包的序列号，需要注意的是默认在 SYN 包里面显示的是真正的序列号，在随后的包中，tcpdump 为了阅读方便将显示相对序列号
# win 43690 表示自己声明的接受窗口的大小
# `[]` 包起来的是 tcp 的选项值，选项值中的 wscale 表示的是窗口缩放的大小，后面的窗口大小都要乘上 2^7
# length 参数表示当前包的长度
14:21:30.393783 IP 127.0.0.1.34448 > 127.0.0.1.9090: Flags [S], seq 3273551255, win 43690, options [mss 65495,sackOK,TS val 2942574212 ecr 0,nop,wscale 7], length 0
E..<s.@.@.............#...m..........0.........
.d..........................
# 第二个数据包便是 SYN+ACK 的数据包，窗口大小为 43690，wscale 为 7
14:21:30.393800 IP 127.0.0.1.9090 > 127.0.0.1.34448: Flags [S.], seq 1196008388, ack 3273551256, win 43690, options [mss 65495,sackOK,TS val 2942574212 ecr 2942574212,nop,wscale 7], length 0
E..<..@.@.<.........#...GI....m......0.........
.d...d......................
# 从第三个包开始，tcpdump 显得序列号是相对序列号了，这个包的窗口大小为 342 * 2 ^ 7
14:21:30.393811 IP 127.0.0.1.34448 > 127.0.0.1.9090: Flags [.], ack 1, win 342, options [nop,nop,TS val 2942574212 ecr 2942574212], length 0
E..4s.@.@.............#...m.GI.....V.(.....
.d...d..................
```

Flags 的其他标志：

- F: FIN
- R: RST
- P: PSH
- U: URG
- .: 没有标志，ACK 情况下使用（意思应该就是这是一个 ACK 的包）

然后我们在客户端输入 hello world，看到抓到如下的数据包

```shell
# 客户端（127.0.0.1.9090）收到 helloworld 后 Flags [P.] 立刻上交给应用层，seq 为 [1,13)，length 为 12
14:37:48.120714 IP 127.0.0.1.34448 > 127.0.0.1.9090: Flags [P.], seq 1:13, ack 1, win 342, options [nop,nop,TS val 2943551939 ecr 2943543795], length 12
E..@..@.@.D!......... #.Ka.b.......V.4.....
.s	..r..hello world
................
# 客户端回复服务端（127.0.0.1.34448），ack 13，表示 13 之前的数据包都接收到了，下次从 13 开始发
14:37:48.120724 IP 127.0.0.1.9090 > 127.0.0.1.34448: Flags [.], ack 13, win 342, options [nop,nop,TS val 2943551939 ecr 2943551939], length 0
E..4uY@.@..h........#.. ....Ka.n...V.(.....
.s	..s	.................
```

然后再客户端按下 Ctrl+C 退出 nc 客户端，

```shell
# 客户端发给服务端的挥手 FIN 包
14:44:18.691143 IP 127.0.0.1.34592 > 127.0.0.1.9090: Flags [F.], seq 13, ack 1, win 342, options [nop,nop,TS val 2943942509 ecr 2943551939], length 0
E..4..@.@.D,......... #.Ka.n.......V.(.....
.x.m.s	.................
# 服务端给客户端回复的 FIN+ACK 包
14:44:18.691290 IP 127.0.0.1.9090 > 127.0.0.1.34592: Flags [F.], seq 1, ack 14, win 342, options [nop,nop,TS val 2943942509 ecr 2943942509], length 0
E..4uZ@.@..g........#.. ....Ka.o...V.(.....
.x.m.x.m................
# 客户端给服务端发送的 FIN 包的 ACK
14:44:18.691298 IP 127.0.0.1.34592 > 127.0.0.1.9090: Flags [.], ack 2, win 342, options [nop,nop,TS val 2943942509 ecr 2943942509], length 0
E..4..@.@.D+......... #.Ka.o.......V.(.....
.x.m.x.m................
```

# 参考资料

1. [深入理解 TCP 协议：从原理到实战](https://juejin.im/book/5c70dbbe51882562046911bc?referrer=5aa21ad15188255585072268)
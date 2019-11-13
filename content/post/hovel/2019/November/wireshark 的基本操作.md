---
title: "wireshark 的基本操作"
date: 2019-11-13T10:50:12+08:00
keywords: []
description: ""
tags: [
    "网络协议"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 抓包过滤

抓包的过程很耗 CPU 和内存资源而且大部分情况下我们不是对所有的包都感兴趣，因此可以只抓取满足特定条件的包，丢弃不感兴趣的包，比如只想抓取 ip 为 172.18.80.49 端口号为 3306 的包，可以输入 host 172.18.80.49 and port 3306

![](/hub/2019/November/32.png)

这里关于抓包过滤具体的语法规则暂时不太清楚，后面再补充。

# 2 显示过滤（Display filter）

显示过滤可以算是 wireshark 最常用的功能了，与抓包过滤不一样的是，显示过滤不会丢弃包的内容，不符合过滤条件的包被隐藏起来，方便我们阅读。

![](/hub/2019/November/33.png)

字段过滤器可以更加精确的过滤出想要的包，比如我们只想看锤科网站 t.tt 域名的 dns 解析，可以输入 dns.qry.name == t.tt

![](/hub/2019/November/34.png)

要想记住这些很难，有一个小技巧，比如怎么知道 域名为 t.tt 的 dns 查询要用 dns.qry.name 呢？

可以随便找一个 dns 的查询，找到查询报文，展开详情里面的内容，然后鼠标选中想过滤的字段，最下面的状态码就会出现当前 wireshark 对应的查看条件，比如下图中的 dns.qry.name

![](/hub/2019/November/35.png)

常用的查询条件有：

tcp 相关过滤器：

1. tcp.flags.syn==1：过滤 SYN 包
2. tcp.flags.reset==1：过滤 RST 包
3. tcp.analysis.retransmission：过滤重传包
4. tcp.analysis.zero_window：零窗口

http 相关过滤器：

1. http.host==t.tt：过滤指定域名的 http 包
2. http.response.code==302：过滤 http 响应状态码为 302 的数据包
3. http.request.method==POST：过滤所有请求方式为 POST 的 http 请求包
4. http.transfer_encoding == "chunked" 根据 transfer_encoding 过滤
5. http.request.uri contains "/appstock/app/minute/query"：过滤 http 请求 url 中包含指定路径的请求

通信延迟常用的过滤器：

1. http.time>0.5：请求发出到收到第一个响应包的时间间隔，可以用这个条件来过滤 http 的时延
2. tcp.time_delta>0.3：tcp 某连接中两次包的数据间隔，可以用这个来分析 TCP 的时延
3. dns.time>0.5：dns 的查询耗时

wireshakr 所有的查询条件在这里可以查到：https://www.wireshark.org/docs/dfref/

# 3 比较运算符

eshark 支持比较运算符和逻辑运算符。这些运算符可以灵活的组合出强大的过滤表达式。

1. 等于：== 或者 eq
2. 不等于：!= 或者 ne
3. 大于：> 或者 gt
4. 小于：< 或者 lt
5. 包含 contains
6. 匹配 matches
7. 与操作：AND 或者 &&
8. 或操作：OR 或者 ||
9. 取反：NOT 或者！

# 4 从 wireshark 看协议分层

下图是抓取的一次 http 请求的包 curl http://www.baidu.com：

![](/hub/2019/November/36.png)

可以看到协议的分层，从上往下依次是

1. Frame：物理层的数据帧
2. Ethernet II：数据链路层以太网帧头部信息
3. Internet Protocol Version 4：互联网层 IP 包头部信息
4. Transmission Control Protocol：传输层的数据段头部信息，此处是 TCP 协议
5. Hypertext Transfer Protocol：应用层 HTTP 的信息

# 5 解密 HTTPS 包

随着 https 和 http2.0 的流行，https 正全面取代 http，这给我们抓包带来了一点点小困难。Wireshark 的抓包原理是直接读取并分析网卡数据。 下图是访问 www.baidu.com 的部分包截图，传输包的内容被加密了。

![](/hub/2019/November/37.png)

要想让它解密 HTTPS 流量，要么拥有 HTTPS 网站的加密私钥，可以用来解密这个网站的加密流量，但这种一般没有可能拿到。要么某些浏览器支持将 TLS 会话中使用的对称加密密钥保存在外部文件中，可供 Wireshark 解密流量。 在启动 Chrome 时加上环境变量 SSLKEYLOGFILE 时，chrome 会把会话密钥输出到文件。

`SSLKEYLOGFILE=/tmp/SSLKEYLOGFILE.log /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome `

wireshark 可以在 `Wireshark -> Preferences... -> Protocols -> SSL` 打开 Wireshark 的 SSL 配置面板，在 (Pre)-Master-Secret log filename 选项中输入 SSLKEYLOGFILE 文件路径。

![](/hub/2019/November/38.png)

这样就可以查看加密前的 https 流量了

# 参考资料

1. [深入理解 TCP 协议：从原理到实战](https://juejin.im/book/5c70dbbe51882562046911bc?referrer=5aa21ad15188255585072268)
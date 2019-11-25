---
title: "TCP 中的 SO_REUSEADDR 选项"
date: 2019-11-19T19:57:55+08:00
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

# 1 地址重用

SO_REUSEADDR 选项的作用就像它的名字一样，它是用来使得地址能够重用的。比如说我的 ip 是 `192.168.3.9`，那么如果没有设置这个选项，那么当我开启两个 socket 去绑定 `192.168.3.9` 和 `0.0.0.0` 时候会触发 `java.net.BindException: Address already in use (Bind failed)` 异常，但是如果开启了这个选项，那么便能够成功绑定。

下面是一个例子

```java
    public static void openSocket(String host) throws IOException {
        ServerSocket serverSocket = new ServerSocket();
        // setReuseAddress 必须在 bind 函数调用之前执行
        serverSocket.setReuseAddress(true);
        serverSocket.bind(new InetSocketAddress(host,8080));
        System.out.printf("reuse address: [%s]%n", serverSocket.getReuseAddress());
        while (true) {
            Socket socket = serverSocket.accept();
            System.out.println("incoming socket..");
            OutputStream out = socket.getOutputStream();
            out.write("Hello\n".getBytes());
            out.close();
        }
    }

    public static void main(String[] args) throws IOException {
        // 这个是我的本地地址，你可以换成你的
        openSocket("192.168.3.9");
    }
    public static void main(String[] args) throws IOException {
        openSocket("0.0.0.0");
    }
```

# 2 修改系统对 TIME-WAIT 和 FIN_WAIT_2 的处理方式

如果用户开启了一个 TCP 监听，然后有某一个客户端连接上来了，而此时服务器出现问题主动断开了连接，那么会进行四次挥手，最终处于 TIME-WAIT 状态，等待 2MSL 时间然后退出。而如果此时服务器重启，而此时端口处于 2MSL 状态，那么服务器就会爆 `java.net.BindException: Address already in use (Bind failed)` 的异常，如果设置了 SO_REUSEADDR 为 true，那么服务器会改变对 2MSL 的处理，立刻打开。同理，对于 FIN_WAIT_2 状态也会立刻打开。

```java
    public static void openSocket(String host) throws IOException {
        ServerSocket serverSocket = new ServerSocket();
        // setReuseAddress 必须在 bind 函数调用之前执行
        serverSocket.setReuseAddress(true);
        serverSocket.bind(new InetSocketAddress(host,8080));
        System.out.printf("reuse address: [%s]%n", serverSocket.getReuseAddress());
        while (true) {
            Socket socket = serverSocket.accept();
            System.out.println("incoming socket..");
            OutputStream out = socket.getOutputStream();
            out.write("Hello\n".getBytes());
            out.close();
        }
    }

    public static void main(String[] args) throws IOException {
        // 这个是我的本地地址，你可以换成你的
        openSocket("192.168.3.9");
    }
```

# 参考文章

1. [深入理解 TCP 协议：从原理到实战](https://juejin.im/book/5c70dbbe51882562046911bc?referrer=5aa21ad15188255585072268)
---
title: "Java 阻塞 IO 的使用样例"
date: 2020-01-21T19:16:57+08:00
keywords: []
description: ""
tags: [
    "java"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

样例代码分为两个部分，一个是 server，另一个是 client

这种模式的 IO 编程模型在客户端较少的情况下运行良好，但是对于客户端比较多的业务来说，单机服务端可能需要支撑成千上万的连接，而每一个连接又需要开一个线程去维护该连接，那么这种编程模型下造成的线程频繁切换严重影响性能。

# 1 Server

```java
import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class IOServer {
    public static void main(String[] args) throws Exception {
        ServerSocket serverSocket = new ServerSocket(9999);
        // 接收新连接线程
        while (true) {
            try {
                // 阻塞方法获取新的连接
                Socket socket = serverSocket.accept();
                System.out.println("Have a connection.");
                // 每一个新的连接都创建一个线程，负责读取数据
                new Thread(() -> {
                    try {
                        int len;
                        byte[] data = new byte[1024];
                        InputStream inputStream = socket.getInputStream();
                        // 按字节流方式读取数据
                        while ((len = inputStream.read(data)) != -1) {
                            System.out.println(new String(data, 0, len));
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }).start();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

# 2 IOClient

```java
import java.io.IOException;
import java.net.Socket;
import java.time.LocalDateTime;

public class IOClient {
    public static void main(String[] args) {
        try {
            // 创建 socket
            Socket socket = new Socket("127.0.0.1", 9999);
            while (true) {
                try {
                    // 向 socket 中写入数据
                    socket.getOutputStream()
                            .write((LocalDateTime.now().toString()).getBytes());
                    Thread.sleep(2000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
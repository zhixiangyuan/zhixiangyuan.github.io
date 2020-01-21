---
title: "Java NIO 的使用样例"
date: 2020-01-21T19:28:09+08:00
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

总共两个部分，一个是 NIOServer，另一个是 NIOClient。Java 的 NIO 使用起来很复杂，推荐通过 Netty 框架来使用 Java 的 NIO。

# 1 NIOServer

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.nio.charset.Charset;
import java.util.Iterator;
import java.util.Set;

public class NIOServer {
    public static void main(String[] args) throws IOException {
        // NIO 模型中通常会有两个线程，每个线程绑定一个轮询器 selector
        // serverSelector 负责轮询是否有新的连接
        Selector serverSelector = Selector.open();
        // clientSelector 负责轮询连接是否有数据可读
        // 服务器监测到一个新连接后，不再创建一个新的线程，而是直接将新连接绑定到 clientSelector 上
        Selector clientSelector = Selector.open();

        new Thread(() -> {
            try {
                // 启动服务端
                ServerSocketChannel listenerChannel = ServerSocketChannel.open();
                // 绑定端口
                listenerChannel.socket().bind(new InetSocketAddress(9999));
                // 配置非阻塞
                listenerChannel.configureBlocking(false);
                // 注册 selector 以及其感兴趣的事件
                listenerChannel.register(serverSelector, SelectionKey.OP_ACCEPT);

                while (true) {
                    // 监测是否有新的连接，这里的 1 指的是阻塞的时间为 1ms
                    // 如果 serverSelector.select(1) 大于 0 则说明有 SelectionKey.OP_ACCEPT 的事件
                    if (serverSelector.select(1) > 0) {
                        // 连接放在 SelectionKey 中
                        Set<SelectionKey> set = serverSelector.selectedKeys();
                        Iterator<SelectionKey> keyIterator = set.iterator();

                        while (keyIterator.hasNext()) {
                            SelectionKey key = keyIterator.next();

                            if (key.isAcceptable()) {
                                try {
                                    // 进来的连接将其注册到 clientSelector
                                    SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept();
                                    clientChannel.configureBlocking(false);
                                    clientChannel.register(clientSelector, SelectionKey.OP_READ);
                                } finally {
                                    keyIterator.remove();
                                }
                            }

                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

        }).start();


        new Thread(() -> {
            try {
                while (true) {
                    // 监测是否有需要读取数据的连接，这里的 1 指的是阻塞的时间为 1ms
                    // 如果 clientSelector.select(1) 大于 0 则说明有 SelectionKey.OP_READ 的事件
                    if (clientSelector.select(1) > 0) {
                        // 连接放在 SelectionKey 中
                        Set<SelectionKey> set = clientSelector.selectedKeys();
                        Iterator<SelectionKey> keyIterator = set.iterator();

                        while (keyIterator.hasNext()) {
                            SelectionKey key = keyIterator.next();

                            if (key.isReadable()) {
                                try {
                                    SocketChannel clientChannel = (SocketChannel) key.channel();
                                    // 获取 ByteBuffer
                                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                                    // 通过 byteBuffer 来读取数据
                                    clientChannel.read(byteBuffer);
                                    byteBuffer.flip();
                                    System.out.println(
                                            Charset.defaultCharset()
                                                    .newDecoder()
                                                    .decode(byteBuffer)
                                                    .toString()
                                    );
                                } finally {
                                    keyIterator.remove();
                                    key.interestOps(SelectionKey.OP_READ);
                                }
                            }

                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

# 2 NIOClient

```java
import java.io.IOException;
import java.net.Socket;
import java.time.LocalDateTime;

public class NIOClient {
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


# 参考文章

1. [Netty 入门与实战：仿写微信 IM 即时通讯系统](https://juejin.im/book/5b4bc28bf265da0f60130116)
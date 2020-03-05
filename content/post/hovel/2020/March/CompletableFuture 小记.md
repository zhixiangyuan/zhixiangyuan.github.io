---
title: "CompletableFuture 小记"
date: 2020-03-05T15:54:03+08:00
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

# 前言

本文主要讲解使用，同时使用了 junit 框架


```xml
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
<!--            <scope>test</scope>-->
        </dependency>
    </dependencies>
```

# 1 completedFuture

```java
    static void completedFutureExample() {
        // 生成一个返回结果为 message 的 CompletableFuture
        CompletableFuture<String> cf = CompletableFuture.completedFuture("message");
        // 方法已经执行完了
        assertTrue(cf.isDone());
        // 返回值为 message
        assertEquals("message", cf.getNow(null));
    }
```

# 2 runAsync

这里面需要注意，以 `Async` 结尾的方法在没有指定 `executor` 的情况下，会使用 ForkJoinPool来执行任务，使用的是守护线程。

```java
    static void runAsyncExample() throws ExecutionException, InterruptedException {
        // 异步执行一个任务
        CompletableFuture<Void> cf = CompletableFuture.runAsync(() -> {
            assertTrue(Thread.currentThread().isDaemon());
            System.out.println("Hello world");
        });
        // 阻塞等待结果完成
        cf.get();
        // 判断是否真的已经执行完成
        assertTrue(cf.isDone());
    }
```

# 3 thenApply

这个方法需要注意的是由于它没有加 `Async` 作为结尾，所以其实它是一个同步的方法。

```java
    static void thenApplyExample() {
        CompletableFuture<String> cf = CompletableFuture
                .completedFuture("message")
                // 在上一个任务执行完之后将结果作为 s 交给下一个任务去执行
                .thenApply(s -> {
                    assertFalse(Thread.currentThread().isDaemon());
                    return s.toUpperCase();
                });
        assertEquals("MESSAGE", cf.getNow(null));
    }
```

# 4 thenApplyAsync

```java
    static void thenApplyAsyncExample() {
        CompletableFuture<String> cf = CompletableFuture
                .completedFuture("message")
                // 方法上标注了 Async，所以是异步方法
                .thenApplyAsync(s -> {
                    assertTrue(Thread.currentThread().isDaemon());
                    return s.toUpperCase();
                });
        assertNull(cf.getNow(null));
        assertEquals("MESSAGE", cf.join());
    }
```

下面是以自定义的线程池来执行任务

```java
    static ExecutorService executor = Executors.newFixedThreadPool(3, new ThreadFactory() {
        int count = 1;

        @Override
        public Thread newThread(Runnable runnable) {
            return new Thread(runnable, "custom-executor-" + count++);
        }
    });

    static void thenApplyAsyncWithExecutorExample() {
        CompletableFuture<String> cf = CompletableFuture
                .completedFuture("message")
                .thenApplyAsync(s -> {
            assertTrue(Thread.currentThread().getName().startsWith("custom-executor-"));
            assertFalse(Thread.currentThread().isDaemon());
            return s.toUpperCase();
        }, executor);

        assertNull(cf.getNow(null));
        assertEquals("MESSAGE", cf.join());
    }
```

# 5 thenAccept

`thenAccept` 与 `thenApply` 的区别在于 `thenAccept` 没有返回值

```java
    static void thenAcceptExample() {
        StringBuilder result = new StringBuilder();
        CompletableFuture
                .completedFuture("thenAccept message")
                .thenAccept(s -> {
                    result
                            .append("result: ")
                            .append(s);
                });
        assertEquals("result: thenAccept message", result.toString());
    }
```

# 5 thenAcceptAsync

```java
    static void thenAcceptExample() throws ExecutionException, InterruptedException {
        StringBuilder result = new StringBuilder();
        CompletableFuture<Void> future = CompletableFuture
                .completedFuture("thenAccept message")
                // 异步执行
                .thenAcceptAsync(s -> {
                    result
                            .append("result: ")
                            .append(s);
                });
        // 等待运行结束
        future.get();
        assertEquals("result: thenAccept message", result.toString());
    }
```

# 6 thenCombine

```java
    static void thenCombineAsyncExample() {
        CompletableFuture<String> cf = CompletableFuture.completedFuture("Result1")
                .thenCombine(
                        CompletableFuture.completedFuture("Result2"),
                        (s1, s2) -> s1 + " : " + s2
                );
        // 可以看到 thenCombine 便是将两次的 future 的返回值作为 s1 和 s2 这样的入参
        // 然后再执行后续的处理
        assertEquals("Result1 : Result2", cf.join());
    }
```

# 参考文章

1. [20 Examples of Using Java’s CompletableFuture](https://mahmoudanouti.wordpress.com/2018/01/26/20-examples-of-using-javas-completablefuture/)
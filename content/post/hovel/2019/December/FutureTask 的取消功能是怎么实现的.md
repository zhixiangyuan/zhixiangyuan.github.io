---
title: "FutureTask 的取消功能是怎么实现的"
date: 2019-12-03T20:19:16+08:00
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

>本文基于 jdk 1.8

直接看取消的函数实现

```java
public class FutureTask<V> implements RunnableFuture<V> {
    ...
    public boolean cancel(boolean mayInterruptIfRunning) {
        if (!(state == NEW &&
              UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
                  mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
            return false;
        try {    // in case call to interrupt throws exception
            if (mayInterruptIfRunning) {
                try {
                    Thread t = runner;
                    if (t != null)
                        // 关键在这里，当经过层层判断之后，如果正在运行
                        // 那么就直接执行该线程的 interrupt() 即可
                        t.interrupt();
                } finally { // final state
                    UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
                }
            }
        } finally {
            finishCompletion();
        }
        return true;
    }
}
```


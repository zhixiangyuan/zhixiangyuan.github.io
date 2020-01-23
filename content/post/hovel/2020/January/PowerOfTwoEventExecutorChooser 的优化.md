---
title: "PowerOfTwoEventExecutorChooser 的优化"
date: 2020-01-23T20:01:38+08:00
keywords: []
description: ""
tags: [
    "netty"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

Netty 内部在选择使用什么 EventLoop 的时候，会使用 Chooser 来进行判断，其实就是直接调用 Chooser 中的 next 方法，对于这个地方的实现有两种，一种是 GenericEventExecutorChooser 的实现方式，也是常用的想法，见下面的代码。

```java
    private static final class GenericEventExecutorChooser implements EventExecutorChooser {
        private final AtomicInteger idx = new AtomicInteger();
        private final EventExecutor[] executors;

        GenericEventExecutorChooser(EventExecutor[] executors) {
            this.executors = executors;
        }

        @Override
        public EventExecutor next() {
            // Math.abs(idx.getAndIncrement() % executors.length) 的作用便
            // 是选择从 0 到最后一个，然后再从 0 开始
            return executors[Math.abs(idx.getAndIncrement() % executors.length)];
        }
    }
```

可以看到这种方式很好理解，但是 Netty 在这里给出了另一种方式作为优化

```java
    private static final class PowerOfTwoEventExecutorChooser implements EventExecutorChooser {
        /** 自增序列 */
        private final AtomicInteger idx = new AtomicInteger();
        /** EventExecutor 数组 */
        private final EventExecutor[] executors;

        PowerOfTwoEventExecutorChooser(EventExecutor[] executors) {
            this.executors = executors;
        }

        @Override
        public EventExecutor next() {
            // 注意这里是先 -1 再求 &
            // 这里 idx.getAndIncrement() & (executors.length - 1) 起到的效果和上面
            // Math.abs(idx.getAndIncrement() % executors.length) 起的效果是相同的
            return executors[idx.getAndIncrement() & (executors.length - 1)];
        }
    }
```

这里 `idx.getAndIncrement() & (executors.length - 1)` 的 executors.length 始终是一个 2 的幂次的数，这是之前就经过判断的。当 executors.length 是 2 的幂次的数的时候，它起到的效果便是不断的循环，从 0 到结尾再到 0。下面来分析为什么会这样。

```
首先，executors.length 是 2 的幂次的数，当这个数 -1 后得到的二进制形式便只能是 
0b1111、0b11111111 这样的形式，只会有 1，不会有 0。假设这里的 length
为 4，那么 4 -1 为 3，写成二进制就是 0b11。

当 idx 为 0 时，二进制为 0b00
计算过程：
   idx                           0b00     
   executors.length - 1          0b11     &
   result                        0b00

可以看到等于是过滤后两位，所以从 0b00 ~ 0b11 得到的结果都会是它本身
也就是从 0 到 3 的得到的结果都是本身，下面来看 idx 为 4 的计算

当 idx 为 4 时，二进制为 0b100
计算过程：
   idx                          0b100     
   executors.length - 1         0b011     &
   result                       0b000

可以看到从 4 开始便又变成 0 了，这便是整个计算过程
```

看完上面的分析就懂了为什么能做到的相同的效果，但是还是有个问题，为什么要这样做，其实这是由于计算机底层原因，计算机底层可以直接做 `&` 这样的运算，但是无法做 `%` 这样的运算，`%` 其实是由相应库实现的，效率低于 `&`，但是具体低多少也不是很清楚。


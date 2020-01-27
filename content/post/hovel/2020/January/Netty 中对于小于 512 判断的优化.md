---
title: "Netty 中对于小于 512 判断的优化"
date: 2020-01-27T16:53:46+08:00
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

在日常编程中，对于 `num < 512` 是一个很正常的写法，但是 `<` 终究没有位运算速度快，所以 Netty 对于这个运算也作出了优化实现，下面看该优化实现

```java
    // num < 512
    public static boolean lessThan512(int num) {
        return (num & 0xFFFFFE00) == 0;
    }

    // 下面是一个测试函数
    public static void main(String[] args) {
        System.out.println(lessThan512(511)); // true
        System.out.println(lessThan512(512)); // false
        System.out.println(lessThan512(513)); // false
    }

    // 下面来分析整个计算过程
    // 0xFFFF_FE00 (8 * 4 = 32) 是一个 32 位的数字
    // 也就是说 0xFFFF_FE00 是一个 int 数字，该数字
    // 写成二进制是 0x11111111_11111111_11111110_00000000
    // 做与运算时 0 & 0 = 0, 0 & 1 = 0, 1 & 0 = 0, 1 & 1 = 1
    // 那么任意一个数和该数做与运算，等于将高位为 1 的部分保留下来
    // 而 0x10_00000000 等于 512，那么只要保留下来的高位
    // 有数字，那么就说明是一个大于 512 的数字
    // 由此，我们可以类似的对小于 2 的幂次的数做优化


    // num < 2048
    public static boolean lessThan2048(int num) {
        return (num & 0xFFFFF800) == 0;
    }

    // num < 1024
    public static boolean lessThan1024(int num) {
        return (num & 0xFFFFFC00) == 0;
    }

    // num < 256
    public static boolean lessThan256(int num) {
        return (num & 0xFFFFFF00) == 0;
    }

    // num < 128
    public static boolean lessThan128(int num) {
        return (num & 0xFFFFFF80) == 0;
    }

    // num < 64
    public static boolean lessThan64(int num) {
        return (num & 0xFFFFFFC0) == 0;
    }

    // num < 32
    public static boolean lessThan32(int num) {
        return (num & 0xFFFFFFE0) == 0;
    }

    // num < 16
    public static boolean lessThan16(int num) {
        return (num & 0xFFFFFFF0) == 0;
    }

    // num < 8
    public static boolean lessThan8(int num) {
        return (num & 0xFFFFFFF8) == 0;
    }

    // num < 4
    public static boolean lessThan4(int num) {
        return (num & 0xFFFFFFFC) == 0;
    }

    // num < 2
    public static boolean lessThan2(int num) {
        return (num & 0xFFFFFFFE) == 0;
    }

    // num < 1
    public static boolean lessThan1(int num) {
        return (num & 0xFFFFFFFF) == 0;
    }
```
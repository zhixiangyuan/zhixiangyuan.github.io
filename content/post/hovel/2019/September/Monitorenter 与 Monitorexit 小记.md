---
title: "Monitorenter 与 Monitorexit 小记"
date: 2019-09-23T10:07:48+08:00
keywords: []
description: ""
tags: [
    "JVM"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

首先对如下代码进行编译 

```java
    public void Test() {
        synchronized(this) {
            System.out.println();
        }
    }
```

将得到的字节码通过 `javap` 反编译

```java
  public void Test();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         // 从 local variable table 加载 this 到栈
         0: aload_0
         // 复制一个 this
         1: dup
         // 栈顶 this 存到 local variable table 下标 1 的位置
         2: astore_1
         // 获取 this 的锁
         3: monitorenter
         // 调用方法
         4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         7: invokevirtual #3                  // Method java/io/PrintStream.println:()V
         // 从 local variable table 下标 1 的位置加载 this
        10: aload_1
         // 退出 this 的锁
        11: monitorexit
         // 跳到 20 行的位置，也就是 return
        12: goto          20
         // 这下边应该是为了防止出现异常
         // 如果出现异常则调用下面这段字节码退出锁
        15: astore_2
        16: aload_1
        17: monitorexit
        18: aload_2
        19: athrow
        20: return
```
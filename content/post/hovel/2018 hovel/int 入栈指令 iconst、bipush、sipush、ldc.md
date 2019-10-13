---
title: "int 入栈指令 iconst、bipush、sipush、ldc"
date: 2018-08-11T18:47:43+08:00
keywords: []
description: ""
tags: [
    "Java"
]
categories: [
    "杂货铺"
]
author: "yuanzx"
---

# 1 前言

本文介绍 int 数值入栈指令 iconst、bipush、sipubh、Idc。

根据 int 取值不同分为以下几种可能性：

1. iconst: `[-1, 5]`
2. bipush: `[-128, -2]∪[6, 127]`
3. sipush: `[-32768, -129]∪[128, 32767]`
4. ldc: `[-2147483648, -32769]∪[32768, 2147483647]`

# 2 iconst

当 int 取值 `[-1, 5]` 时，JVM 采用 iconst 指令将常量压入栈中。

定义 Test.java 文件

```java
    public static void main(String[] args) {
        int i = -1;
        i = 0;
        i = 1;
        i = 2;
        i = 3;
        i = 4;
        i = 5;
    }
```

查看 class 文件

```java
public static void main(java.lang.String[]);
    Code:
       0: iconst_m1
       1: istore_1
       2: iconst_0
       3: istore_1
       4: iconst_1
       5: istore_1
       6: iconst_2
       7: istore_1
       8: iconst_3
       9: istore_1
      10: iconst_4
      11: istore_1
      12: iconst_5
      13: istore_1
      14: return
```

分析 class 文件，int 取值 `[-1, 5]` 时，分别采用以下指令将值压入栈中。

1. iconst_m1: -1
2. iconst_0: 0
3. iconst_1: 1
4. iconst_2: 2
5. iconst_3: 3
6. iconst_4: 4
7. iconst_5: 5

# 3 bipush

当 int 取值 `[-128, -2]∪[[6, 127]` 时，JVM 采用 bipush 指令将常量压入栈中。

定义 Test.java 文件

```java
    public static void main(String[] args) {
        int i = 127;
        int x = 6;
        int y = -2;
        int j = -128;
    }
```

查看 class 文件


```java
  public static void main(java.lang.String[]);
    Code:
       0: bipush        127
       2: istore_1
       3: bipush        6
       5: istore_2
       6: bipush        -2
       8: istore_3
       9: bipush        -128
      11: istore        4
      13: return
```

# 4 sipush

当 int 取值 `[-32768, -129]∪[128, 32767]` 时，JVM 采用 sipush 指令将常量压入栈中。

定义 Test.java 文件

```java
    public static void main(String[] args) {
        int i = 32767;
        int x = 128;
        int y = -129;
        int j = -32768;
    }
```

查看 class 文件

```java
  public static void main(java.lang.String[]);
    Code:
       0: sipush        32767
       3: istore_1
       4: sipush        128
       7: istore_2
       8: sipush        -129
      11: istore_3
      12: sipush        -32768
      15: istore        4
      17: return
```

# 5 ldc

当 int 取值 `[-2147483648, -32769]∪[32768, 2147483647]` 时，JVM 采用 ldc 指令将常量压入栈中。

定义 Test.java 文件

```java
    public static void main(String[] args) {
        int i = 2147483647;
        int x = 32768;
        int y = -32769;
        int j = -2147483648;
    }
```

查看 class 文件

```java
  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // int 2147483647
       2: istore_1
       3: ldc           #3                  // int 32768
       5: istore_2
       6: ldc           #4                  // int -32769
       8: istore_3
       9: ldc           #5                  // int -2147483648
      11: istore        4
      13: return
```

# 参考资料

1. [JVM字节码之整型入栈指令(iconst、bipush、sipush、ldc)](http://www.linmuxi.com/2016/02/25/jvm-int-pushstack-01/)
2. [Java bytecode instruction listings](https://en.wikipedia.org/wiki/Java_bytecode_instruction_listings)
---
title: "Java 性能优化"
date: 2019-07-02T16:22:25+08:00
keywords: [
    "Java 性能优化"
]
description: ""
tags: [
    "Java"
]
categories: [
    "杂货铺"
]
author: "yuanzx"
---

# 1 明确是否需要优化以及要优化到什么程度

首先需要在实践中检测代码是否需要优化，如果不需要优化，那么优化可能占用大量时间，而且优化完了也不会有任何体现。

其次，如果需要优化，也需要明确代码需要优化到什么程度，需要同一时间承受多少的并发量。最后才是在程序中找出需要优化的部分，并将其进行优化。

# 2 如何找出代码中的性能瓶颈

通过分析器分析代码中的每一部分的性能的信息，分析那些你所关心的信息。

# 3 拼接字符串时使用 StringBuilder

如果是拼接静态的字符串，那么直接使用 +，因为编译器会对其做优化，例子如下：

```java
String str = "This" + " is a demo."
// 编译器会优化为
String str = "This is a demo."
```

如果**在循环时**拼接动态的字符串，那么使用 StringBuilder 来代替 + 实现字符串的拼接，如果碰到多线程的时候可能需要考虑使用 StringBuffer。

# 4 尽可能使用基本类型

基本类型被存储在栈中，能够起到节省内存的作用，同时需要注意减少拆箱装箱的情况。

装箱拆箱指的是类似于下面的代码

```java
Integer num = 10;
Integer num = Integer.valueOf(10);
```

但是需要注意的是 Java 5 中引入了缓存机制，对于某一些范围的数是有缓存的，下面列出了各种包装类型的缓存范围：

1. Boolean: true, false
2. Short: `[-128,127]`
3. Integer: `[-128,127 ]`
4. Long: `[-128,127]`
5. Byte: `[-128,127]` (Byte 缓存了所有的可能值)
6. Character: `[0,127]`（0 到 127 数字所表示的字符都缓存了）
7. Double 和 Float 没有缓存

# 5 使用 Apache Commons StringUtils.Replace 代替 String.replace

如果使用的 Java 版本低于 Java 9，那么最好使用 Apache Commons StringUtils.Replace 代替 String.replace，因为它的性能比 String.replace 好很多。

# 6 对于频繁使用的东西可以添加缓存

例如 Java 包装类型中的方式，对于常用的一些数据，添加缓存来提高性能。

# 7 创建 String 的方式

```java
// 尽量使用这种方式创建字符串
// 这种方式创建字符串时 JVM 会查看内部的缓存池是否已有相同的字符串存在，
// 如果有，则不再使用构造函数构造一个新的字符串，会直接返回已有的字符串实例；
// 如果没有，则分配新的内存空间创建新的字符串。
String str = "demo";
// 下面这种方式创建字符串由于使用了 new 关键字，所以
// 如果所创建的字符串在字符串缓存池中不存在，则直接调用构造函数创建全新的字符串；
// 如果所创建的字符串在字符串缓存池中已有，则会拷贝一份到 Java 堆中。
String str = new String("demo");
```

# 参考资料

1. [9 个可以快速掌握的 Java 性能调优技巧，必须掌握！](https://mp.weixin.qq.com/s/K8LfGz3u89dqFOuvwyKG0Q)
2. [Java 性能优化之 String 篇](https://www.ibm.com/developerworks/cn/java/j-lo-optmizestring/index.html)
   - 文中提到的 String(int offset, int count, char value[]) 保存上下文的问题在 Java 8 里面是没有的，Java 8 里面直接进行了数组拷贝
---
title: "Java 程序员修炼之道"
date: 2019-07-01T20:04:41+08:00
keywords: []
description: ""
tags: [
    "Java"
]
categories: [
    "杂货铺"
]
autoCollapseToc: true
author: "yuanzx"
---

# 引言

本书给我的感觉就是什么都提一点，但是每一点都讲的不够细致，如果想要更深的挖掘，还要寻找相关领域的资料，深入学习。

# 第一部分 用 Java 7 做开发

## 第 1 章 初试 Java 7

### 1.1 语言与平台

### 1.2 Coin 项目：浓缩的都是精华

### 1.3 Coin 项目中的修改

#### 1.3.1 switch 语句中的 String

#### 1.3.2 更强的数值文本表示法

#### 1.3.3 改善后的异常处理

#### 1.3.4 Try-with-resource

因为没人能在手动关闭资源时做到 100% 正确，所以 Java 7 借助了编译器来实现这项改进。

对于实现了 AutoCloseable 的接口，就可以放到 try 的括号中，编译器会在使用完资源后自动调用 close 方法关闭资源。由于 Closeable 接口继承了 AutoCloseable 接口，所以实现了 Closeable 就可以放到 try 的括号中，实现自动关闭资源了。

![Closeable 与 AutoCloseable 的继承关系图](/media/book_abstract/2.png)

示例代码如下：

```java
    public static byte[] compress(byte[] bytes) {
        if (bytes == null || bytes.length == 0) {
            return bytes;
        }
        try (
                ByteArrayOutputStream out = new ByteArrayOutputStream();
                GZIPOutputStream gzip = new GZIPOutputStream(out);
        ) {
            gzip.write(bytes);
            gzip.finish();
            return out.toByteArray();
        } catch (IOException e) {
            log.error(e.getMessage(), e);
        }
        return new byte[]{};
    }
```

但是这里有一点需要注意，代码块中的代码不能写成 `GZIPOutputStream gzip = new GZIPOutputStream(new ByteArrayOutputStream());` 这样写的话，如果 `new GZIPOutputStream()` 的时候出现异常，那么 `new ByteArrayOutputStream()` 无法被正常关闭，所以正确的做法是为每个资源声明独立的变量。

# 第二部分

## 第 5 章 类文件与字节码

1. 书中部分位置所写的堆栈在我的理解就是栈，故将该部分摘录时直接将堆栈改写为栈。
2. 书中的操作码表中的参数和栈布局的对应关系我也不是很理解。

### 5.4 字节码

1. 通过 javac 编译源文件产生的。
2. 编译的过程中去掉了高级语言特性，比如循环结构（for、while 等）在字节码中转换成了分支指令。
3. 每个操作码都由一个字节表示（因此被叫做字节码）。
4. 字节码可以被进一步编译成机器码，通常是“即时编译”。

#### 5.4.1 如何反编译 .class 文件的字节码

通过 javap 命令可以反编译 .class 文件，实际使用的时候 javap 后面的类名加与不加 .class 都可以。

![javap 命令截图](/media/book_abstract/1.png)

```
-help --help -?     打印使用信息（也就是打印图中所示内容）
-version            打印版本信息（实际打印出来的和 java 的版本是一致的）
-v -verbose         打印详细信息（这是一个 javap 反编译文件能够打印出来的更详细的信息）
-l                  打印行号和本地变量表（这里的行号不明白有什么用）
-public             仅仅打印 public 修饰的类、成员字段和方法（这里如果 class 没有被 public 修饰，同样能打印出来，只是构造函数没有打印，编译过后的构造函数和类一样没有被 public 修饰）
-protected          仅仅打印 public/protected 修饰的类、成员字段和方法
-package            仅仅打印 public/protected/package 修饰的类、成员字段和方法
-p -private         打印所有的类、成员字段和方法
-c                  打印具体的代码执行方式
-s                  打印内部类型方法签名（这个方法签名和我们常写表示法不一样，但实际的含义是一样的）
-sysinfo            打印所在路径（随着文件移动位置发生变化）、文件大小、文件最后修改日期（精确到日）、MD5 值
-constants          打印常量的实际数值（如果是 private 修饰的还要加 -p 才行，不加这个时候，打印 final 变量只打印到变量名，后面就是分号了，加了这个，直接把常量的值也打印出来）
-classpath <path>   （不知道干什么用的）
-cp <path>          （不知道干什么用的）
-bootclasspath <path>   （不知道干什么用的）
```

javac 命令产生的字节码没有经过任何优化，优化工作是由 JIT 编译器来完成的。

#### 5.4.2 运行时环境

1. Java 在运行时使用堆栈结构，堆作为存储，栈中完成所有的计算和操作。
2. 线程在运行时又一个调用栈，栈中存放了所有的方法调用，先调用的方法在栈的下面，后调用的方法在栈的上面。
3. 方法在运行时又又一个计算栈，这个栈用来计算数值。
4. 当 A 方法调用 B 方法时，A 的计算栈会暂停工作，新开辟一个栈给 B 使用，B 计算完，如果有结果，那么结果将传到 A 的栈中参加剩下的计算。

#### 5.4.3 操作码介绍

1. 操作码由一个单字节表示，所以最多只能有 255 个操作码（很奇怪，这里是 2 的八次方，那应该有 256 个操作码才对），当前仅用了 200 个左右。
2. 操作码大多数可以归为几大族系。
3. 操作码表有四列，分别为：名称、参数、栈布局、描述。
    - 名称：操作码类型的通用名称。一般会有几个稍有变形的名称在干活，操作不同类型的数据。
    - 参数：操作码的参数。以 i 打头的参数是用来作为常量池或局部变量表中的查询索引。如果有大于一个此类参数（比如 i1，i2），那么则将他们合并（i1 和 i2 合并成为一个 16 位的索引）。如果参数出现在括号里面，说明他们是可选参数。
    - 栈布局：它展示了栈在操作码执行前后的状态。括号中的元素表明不是所有形式的操作码都使用他们，或者这些元素是可选的。
    - 描述：说明操作码的作用。

举个例子：

| 名称     | 参数   | 栈布局          | 描述                                   |
| -------- | ------ | --------------- | -------------------------------------- |
| getfield | i1, i2 | `[obj] → [val]` | 从栈顶端对象的常量池中去除指定位置的值 |

第一栏是名称，第二栏 i1，i2 两个字节合在一起是一个 16 位的数，可以用来在常量池中找到想要找的值（常量池的索引总是 16 位的）。第三栏堆栈布局中的含义是通过 i1，i2 这个 `[obj]` 找到值 `[val]` 之后，用 `[val]` 替换 `[obj]` 的位置，最后一栏为描述。

#### 5.4.4 加载和存储操作码

下面的操作码负责将值加载到栈中以及将值从栈中移走。

| 名称     | 参数   | 栈布局               | 描述                                                                                       |
| -------- | ------ | -------------------- | ------------------------------------------------------------------------------------------ |
| load     | (i1)   | `[] → [val]`         | 从局部变量加载值（原始型或引用型）到栈上                                                   |
| ldc      | i1     | `[] → [val]`         | 从常量池中加载常量到栈上，针对不同类型有不同的变体                                         |
| store    | (i1)   | `[val] → []`         | 把值（原始型或引用型）从进程的栈中移走，存到局部变量表中。有快捷形式，有针对不同类型的变体 |
| dup      |        | `[val] → [val, val]` | 复制栈顶部的值，有不同形式的变体                                                           |
| getfield | i1, i2 | `[obj] → [val]`      | 从栈顶部对象的常量池中得到指定位置的值                                                     |
| putfield | i1, i2 | `[obj, val] → []`    | 把值放入对象在常量池中指定的位置上                                                         |

#### 5.4.5 数学运算操作码

| 名称   | 参数 | 栈布局                 | 描述                                                                                             |
| ------ | ---- | ---------------------- | ------------------------------------------------------------------------------------------------ |
| add    |      | `[val1, val2] → [res]` | 把栈顶端的两个值相加（必须是相同的原始类型），并把结果存在栈中。有快捷形式，有针对不同类型的变体 |
| sub    |      | `[val1, val2] → [res]` | 把栈顶端的两个值相减（必须是相同的原始类型），并把结果存在栈中。有快捷形式，有针对不同类型的变体 |
| div    |      | `[val1, val2] → [res]` | 把栈顶端的两个值相除（必须是相同的原始类型），并把结果存在栈中。有快捷形式，有针对不同类型的变体 |
| mul    |      | `[val1, val2] → [res]` | 把栈顶端的两个值相乘（必须是相同的原始类型），并把结果存在栈中。有快捷形式，有针对不同类型的变体 |
| (cast) |      | `[value] → [res]`      | 把值从一种原始类型转换为另外一种。每一种可能的类型转换都有对应的形式                             |

- add: addition n. 加法
- sub: subtraction n. 减法
- div: division n. 除法
- mul: multiplication n. 乘法
- cast: 这是一种不存在的类型，所以用括号扩起来了。在实际强制转换时有专门的指令，比如 int 转 double 的指令是 i2d

#### 5.4.6 执行控制操作码

宽大形式表示参数为 4 个字节

| 名称         | 参数         | 栈布局                             | 描述                                                                         |
| ------------ | ------------ | ---------------------------------- | ---------------------------------------------------------------------------- |
| if           | b1, b2       | `[val1, val2] → [] 或 [val1] → []` | 如果符合特定条件，则跳转到特定分支的偏移处                                   |
| goto         | b1, b2       | `[] → []`                          | 无条件地跳转到分支偏移处。有宽大形式                                         |
| jsr          | b1, b2       | `[] → [ret]`                       | 跳到本地子流程中，并把返回地址（下一个操作码的偏移地址）放到栈中。有宽大形式 |
| ret          | 索引         | `[] → []`                          | 返回到索引的局部变量所指向的偏移地址                                         |
| tableswitch  | {依情况而定} | `[index] → []`                     | 用于实现 switch                                                              |
| lookupswitch | {依情况而定} | `[key] → []`                       | 用于实现 switch                                                              |

#### 5.4.7 调用操作码

| 名称            | 参数             | 堆栈布局                  | 描述                       |
| --------------- | ---------------- | ------------------------- | -------------------------- |
| invokestatic    | i1, i2           | `[(val1, ...)] → []`      | 调用一个静态方法           |
| invokevirtual   | i1, i2           | `[obj, (val1, ...)] → []` | 调用一个 “常规” 的实例方法 |
| invokeinterface | i1, i2, count, 0 | `[obj, (val1, ...)] → []` | 调用一个接口方法           |
| invokespecial   | i1, i2           | `[obj, (val1, ...)] → []` | 调用一个 “特殊” 的实例方法 |
| invokedynamic   | i1, i2, 0, 0     | `[val1, ...] → []`        | 动态调用，见5.5节          |

- invokeinterface: 这个命令中多出来的参数是由于历史原因向后兼容产生的，现在已经用不到了（这里面多出来的参数难道是指 count 和 0 吗？）
- invokedynamic: 这个命令中多出来的两个 0，是为了向前兼容产生的

#### 5.4.8 平台操作操作码

| 名称         | 参数   | 堆栈布局     | 描述                                       |
| ------------ | ------ | ------------ | ------------------------------------------ |
| new          | i1, i2 | `[] → [obj]` | 为新对象分配内存，类型由指定位置的常量确定 |
| monitorenter |        | `[obj] → []` | 锁住对象                                   |
| monitorexit  |        | `[obj] → []` | 解锁对象                                   |

- new: 这个操作码只分配存储空间，然后在栈顶创建一个到堆的引用，对象构建的高层概念还包括运行构造方法内的代码。
  - 通常构造对象为 new 分配空间，dup 复制栈顶的引用，invokespecial 调用构造方法。

#### 5.4.9 操作码的快捷形式

#### 5.4.10 示例：字符串拼接

先看一段代码

```java
public class Demo {
    private void run(String[] args) {
        String str = "foo";
        if (args.length > 0) str = args[0];
        System.out.println("this is my string: " + str);
    }
}
```

使用 `javap -c -p Demo` 反编译后得到的结果，`# + 数字` 表示表示寻找常量池中对应的常量。

```java
Warning: Binary file Demo contains me.yuanzx.function.Demo
Compiled from "Demo.java"
public class me.yuanzx.function.Demo {
  public me.yuanzx.function.Demo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  private void run(java.lang.String[]);
    Code:
       0: ldc           #2                  // String foo
       2: astore_2
       3: aload_1
       4: arraylength
       5: ifle          12
       8: aload_1
       9: iconst_0
      10: aaload
      11: astore_2
      12: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
      15: new           #4                  // class java/lang/StringBuilder
      18: dup
      19: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
      22: ldc           #6                  // String this is my string:
      24: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      27: aload_2
      28: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      31: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      34: invokevirtual #9                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      37: return
}
```

后面都是对于指令的讲解，由于前面的内容对于这一块儿也只是范范的讲解，很多细节都有缺失，所以我觉得在这里理解这个例子不是一个好主意，这应该在详细学习指令之后再看，后面的内容都跳过去。

### 5.5 Invokedynamic

1. 新增 Invokedynamic 指令
2. 常量池中新增类型
   - CONSTANT_InvokeDynamic：这个常量是两个 16 位的索引（也就是 4 个字节）
     - 第一个索引指向方法表（用来确定要调用什么），它们被称为引导方法（有时简写为BSM），并且必须是静态的，还要有确定的参数签名。
     - 第二个索引指向 CONSTANT_NameAndType。

// 基本没怎么看懂

## 第 6 章 理解性能调优

如果不对系统进行性能评测，就不知道系统哪里需要进行优化，也就没有合适的优化一说。如果经过测试，可能发现真正的瓶颈并非是 Java 部分，问题可能出在数据库、文件系统、网络上，那微调代码毫无意义。

所以，性能调优是研究性能评测所产生的各种指标，算得上是统计学的一个分支。

JVM 中和性能相关的两个重要子系统是：

1. 垃圾收集子系统
2. JIT 编译器

### 6.1 性能术语

- 等待时间：Latency
- 吞吐量：Throughput
- 利用率：Utilization
- 效率：Efficiency
- 容量：Capacity
- 扩展性：Scalability
- 退化：degradation

这些术语会被用在多线程代码以及集群服务器平台上

#### 6.1.1 等待时间

定义：等待时间是在负载一定的情况下处理一个任务的响应时间。

有价值的性能评测一般都是用一张二维坐标图显示在负载不断增加的情况下响应时间随之改变的函数关系。

下图展示了在负载增加的情况下，响应时间出现了一个突发的非线性退化，这通常称为**性能肘**。

![性能负载图](/media/book_abstract/3.png)

#### 6.1.2 吞吐量

定义：吞吐量是系统子在限定资源、限定时长内能完成的单位工作量。

限定资源指的是例如指明了硬件配置、操作系统的服务器。

#### 6.1.3 利用率

定义：利用率表示可用资源中用来处理工作单元的资源百分比。

人们通常会说服务器的利用率是 10%，这其实是说在正常处理时间内处理工作单元的 CPU 百分比。注意，不同资源的利用率水平可能有非常大的差异，比如 CPU 和内存之间。

#### 6.1.4 效率

定义：系统的效率等于吞吐量除以所用的资源。

比如两个集群方案，为了达到相同的吞吐量，方案 A 需要的服务器数量是方案 B 的两倍，则方案 A 的效率是 B 的一半。

#### 6.1.5 容量

定义：容量是任一时刻能够通过系统的工作单元的数量。

#### 6.1.6 扩展性

定义：当系统得到更多资源时，它的吞吐量或等待时间会发生变化。这种发生在吞吐量或者等待时间上的变化就是系统的扩展性。

例如，服务器数量翻倍，它的吞吐量也能翻倍，那我们就说它实现了完美的线性扩展。在大多数情况下，完美的线性扩展很难达到。

#### 6.1.7 退化

定义：在不增加资源的情况下增加工作单元或者网络系统的客户端，一般等待时间会上升，这便是系统出现了退化。

# 第三部分

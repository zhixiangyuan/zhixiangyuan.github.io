---
title: "Javap 命令的使用"
date: 2018-11-05T10:43:36+08:00
keywords: []
description: ""
tags: [

]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

通过 javap 命令可以反编译 .class 文件，实际使用的时候 javap 后面的类名加与不加 .class 都可以。

![javap 命令截图](/media/book_abstract/1.png)

```shell
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
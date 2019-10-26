---
title: "Java 中的类加载器"
date: 2019-10-26T21:55:24+08:00
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

Java 中的类加载器分为三种：

1. 启动类加载器
2. 扩展类加载器
3. 应用程序类加载器

![类加载器](/hub/2019/october/9.png)

# 1 启动类加载器

Bootstrap ClassLoader，C 语言实现，用于加载 JDK\jre\lib\rt.jar 

# 2 扩展类加载器

Extension ClassLoader，Java 实现，负责加载 JDK\jre\lib\ext 目录中的或者由 java.ext.dirs 系统变量指定的路径中的所有类库。

# 3 应用程序类加载器

Application ClassLoader，它负责加载用户类路径（ClassPath）所指定的类，开发者可以直接食用该类加载器


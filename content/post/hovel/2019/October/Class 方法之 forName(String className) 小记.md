---
title: "Class 方法之 forName(String className) 小记"
date: 2019-10-04T20:29:08+08:00
keywords: []
description: ""
tags: [
    "Java"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

`Class.forName(String className)` 这个方法的作用就是触发类的初始化动作

```java
    public static Class<?> forName(String className)
                throws ClassNotFoundException
```

这个方法目前就仅在 JDBC 的初始化的时候见过，他的目的就是触发 JDBC 驱动注册到代码中，代码如下。

```java
    Class.forName("com.mysql.jdbc.Driver");
```

当触发驱动类初始化时会执行其中的静态代码块，代码如下

```java
    static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
```

通过将 `Driver` 注册到 `DriverManager` 中来完成注册，这里的 JDBC 的代码采用的是桥接模式进行设计的。
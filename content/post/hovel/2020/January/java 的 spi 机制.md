---
title: "java 的 spi 机制"
date: 2020-01-03T11:12:16+08:00
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

java 的 spi 机制简单来说就是通过 `com.sun.tools.javac.util.ServiceLoader` 加载整个 classpath 下的 META-INF/services 目录下的文件以实现加载指定类的功能，这样实现的好处是可以做到动态插拔依赖实现自动配置的效果。

下面通过一个例子看一下效果

![](/hub/2020/January/1.png)

这里总共有四个项目，*spi_standard* 负责提供标准接口 *spi_invoke* 中的 Main 方法用于测试调用效果，另外两个项目中实现的是 `me.yuanzx.spi.Printer` 接口的实现类，下面是四个项目的详细代码

```java
// project: spi_invoke
|- spi_invoke/src/main/java/
    |- me.yuanzx.spi
        |- Main.java

// file: src/main/java/me/yuanzx/spi/Main.java
public class Main {
    public static void main(String[] args) {
        ServiceLoader<Printer> serviceLoader = ServiceLoader.load(Printer.class);
        for (Printer print : serviceLoader) {
            print.print();
        }
    }
}

// project: spi_standard
|- spi_standard/src/main/java/
    |- me.yuanzx.spi
        |- Printer.java

// file: src/main/java/me/yuanzx/spi/Printer.java
public interface Printer {
    /** 打印信息 */
    void print();
}

// project: spi_print_chinese_name
|- spi_print_chinese_name/
    |- src/main/
        |- java/
            |- me.yuanzx.spi.impl.
                |- PrintChineseName.java
        |- resources/META-INF/services/
            |- me.yuanzx.spi.Printer
// file: src/main/java/me/yuanzx/spi/impl/PrintChineseName.java
public class PrintChineseName implements Printer {
    public void print() {
        System.out.println("我是小白");
    }
}

// file: src/main/resources/META-INF/services/me.yuanzx.spi.Printer
me.yuanzx.spi.impl.PrintChineseName

// project: spi_print_english_name
|- spi_print_english_name/
    |- src/main/
        |- java/
            |- me.yuanzx.spi.impl.
                |- PrintEnglishName.java
        |- resources/META-INF/services/
            |- me.yuanzx.spi.Printer

// file: src/main/java/me/yuanzx/spi/impl/PrintEnglishName.java
public class PrintEnglishName implements Printer {
    public void print() {
        System.out.println("I am little white");
    }
}

// file: src/main/resources/META-INF/services/me.yuanzx.spi.Printer
me.yuanzx.spi.impl.PrintEnglishName
```

此时如果我们在 spi_invoke 项目中导入 spi_print_chinese_name 作为依赖，那么便会打印 "我是小白"，如果我们倒入 spi_print_english_name 作为依赖，那么便会打印 I am little white，如果我们同时倒入两个依赖，那么两个都会打印。
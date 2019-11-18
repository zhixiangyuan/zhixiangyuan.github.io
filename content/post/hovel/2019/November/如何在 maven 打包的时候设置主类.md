---
title: "如何在 Maven 打包的时候设置主类"
date: 2019-11-16T22:02:48+08:00
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

# 1 普通项目

在 maven 中配置以下插件即可

```xml
    <build>
        <plugins>
            ...
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>2.4</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <classpathPrefix>lib/</classpathPrefix>
                            <mainClass><$mainClass></mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

原理便是在打包的时候，向 MANIFEST.MF 文件中写入 `Main-Class: <$mainClass>`，这样java 命令在运行 jar 的时候便会读取到主类并执行。

# 2 SpringBoot 项目

SpringBoot 项目打包需要用 springboot 自己的插件，否则会出现不打包依赖的情况

```xml
    <build>
        <plugins>
            ...
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass><$mainClass></mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
---
title: "Maven 中的各种 Scope 的含义"
date: 2020-01-12T16:09:56+08:00
keywords: []
description: ""
tags: [
    "maven"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

Scope 总共有六种分别是 compile、provided、runtime、test、system、import，下面便分别来阐述每种 Scope 的作用。

# 1 compile

>This is the default scope, used if none is specified. Compile dependencies are available in all classpaths of a project. Furthermore, those dependencies are propagated to dependent projects.

compile 是默认范围，如果不设置，那么默认就是这个范围。该范围参与项目的编译、测试、打包、运行全流程。该范围同时会将依赖像子项目传递。

# 2 provided

>This is much like compile, but indicates you expect the JDK or a container to provide the dependency at runtime. For example, when building a web application for the Java Enterprise Edition, you would set the dependency on the Servlet API and related Java EE APIs to scope provided because the web container provides those classes. This scope is only available on the compilation and test classpath, and is not transitive.

该范围参与项目编译、测试，但是不参与项目的打包，这意味着打出来的包是不包含该依赖的，相当于在打包的时候做了 exclude 动作。但是如果项目没有该依赖怎么运行呢，其实该依赖应该由别的项目来提供，比如在 Web 项目里面，便是 Web 容器来提供相关的依赖，或者是希望 JDK 来提供依赖。这个范围不会将依赖传递个给子项目。

# 3 runtime

>This scope indicates that the dependency is not required for compilation, but is for execution. It is in the runtime and test classpaths, but not the compile classpath.

这个范围不参与编译，但在运行时和测试时生效。

# 4 test

>This scope indicates that the dependency is not required for normal use of the application, and is only available for the test compilation and execution phases. This scope is not transitive.

该范围只在测试时有效，并且不向子项目传递

# 5 system

>This scope is similar to provided except that you have to provide the JAR which contains it explicitly. The artifact is always available and is not looked up in a repository.

该范围的含义基本与 provided 相同，只是查找依赖的位置时不直接从中央仓库查找，而是通过指定 systemPath 的位置去查找依赖

# 6 import

>This scope is only supported on a dependency of type pom in the \<dependencyManagement\> section. It indicates the dependency to be replaced with the effective list of dependencies in the specified POM's \<dependencyManagement\> section. Since they are replaced, dependencies with a scope of import do not actually participate in limiting the transitivity of a dependency.

import 只用在 \<dependencyManagement\> 当中，并且只对 pom 类型的 maven 项目起作用，作用便是导入该项目中的 \<dependencyManagement\> 列表。

```xml
   <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            ...
        </dependencies>
    </dependencyManagement>
```

# 7 参考文章

1. [官方文档](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
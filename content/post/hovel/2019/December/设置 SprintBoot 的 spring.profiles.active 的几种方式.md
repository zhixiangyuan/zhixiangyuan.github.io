---
title: "设置 SprintBoot 的 spring.profiles.active 的几种方式"
date: 2019-12-30T08:01:36+08:00
keywords: []
description: ""
tags: [
    "spring boot"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 在配置文件中指定

在 application.xml 中指定

```yml
spring:
  profiles:
    active: dev
```

# 2 在 maven 中指定

在配置文件中使用占位符指定

```yml
spring:
  profiles:
    active: @package.target@
```

在 pom.xml 中以 profile 的形式指定

```xml
   <profiles>
     <profile>
       <id>dev</id><!-- 开发环境 -->
       <properties>
         <package.target>dev</package.target>
       </properties>
       <activation>
         <activeByDefault>true</activeByDefault>
       </activation>
       <dependencies>
         <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-devtools</artifactId>
           <optional>true</optional>
         </dependency>
         <dependency>
           <groupId>io.springfox</groupId>
           <artifactId>springfox-swagger-ui</artifactId>
           <version>2.6.1</version>
         </dependency>
       </dependencies>
     </profile>
     <profile>
       <id>prod</id><!-- 生产环境 -->
       <properties>
         <package.target>prod</package.target>
       </properties>
     </profile>
     <profile>
       <id>test</id><!-- 测试环境 -->
       <properties>
         <package.target>test</package.target>
       </properties>
       <dependencies>
         <dependency>
           <groupId>io.springfox</groupId>
           <artifactId>springfox-swagger-ui</artifactId>
           <version>2.6.1</version>
         </dependency>
       </dependencies>
     </profile>
   </profiles>
   <build>
     <plugins>
       <plugin>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-maven-plugin</artifactId>
       </plugin>
     </plugins>
     <resources>
       <resource>
         <directory>src/main/resources</directory>
         <filtering>true</filtering><!-- 使用package.target值，替换配置文件中 @package.target@ -->
       </resource>
     </resources>
   </build>
```

在打包时进行替换 `mvn package -Ptest`

# 3 JVM 参数方式指定

`java -jar app.jar --spring.profiles.active=test`

# 4 web.xml 中指定

```xml
<init-param>
   <param-name>spring.profiles.active</param-name>
   <param-value>production</param-value>
 </init-param>
```

# 5 注解方式（Junit 单元测试时使用）

` @ActiveProfiles({"unittest","productprofile"})`

# 6 系统环境变量方式

指定系统环境变量 `SPRING_PROFILES_ACTIVE` 即可

# 参考文章

1. [Spring boot 激活 profile 的几种方式](https://my.oschina.net/u/1469495/blog/1522784)
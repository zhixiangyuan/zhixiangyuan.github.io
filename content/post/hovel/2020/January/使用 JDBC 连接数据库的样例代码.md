---
title: "使用 JDBC 连接数据库的样例代码"
date: 2020-01-15T20:00:34+08:00
keywords: []
description: ""
tags: [
    "MySQL"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 pom 中引入依赖

```xml
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
```

# 2 样例代码

```java
public class Main {

    static final String DB_URL = "jdbc:mysql://192.168.3.19:3306/test";

    static final String USER = "root";
    static final String PASS = "123456";

    public static void main(String[] args) {
        try (
                Connection conn = DriverManager.getConnection(DB_URL, USER, PASS);
                Statement stmt = conn.createStatement();
        ) {
            String sql = "INSERT INTO film(name) VALUES('" + UUID.randomUUID().toString() + "');";
            stmt.execute(sql);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

# 3 如何通过 SPI 机制引入驱动

需要注意的是，很多老的教程依然会写 `Class.forName(com.mysql.jdbc.Driver)`，但是由于 Java 中 spi 机制，当引入 jar 包后便会自动注册，不再需要手动调用去注册，所以这句代码可以省去。源码片段见下面部分

```java
// java.sql.DriverManager

public class DriverManager {
    ... 
    private static void loadInitialDrivers() {
        ...
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                // 看这里，这里便是通过 ServiceLoader 加载驱动的过程
                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();
                try{
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {
                // Do nothing
                }
                return null;
            }
        });
        ...
    }
}
```
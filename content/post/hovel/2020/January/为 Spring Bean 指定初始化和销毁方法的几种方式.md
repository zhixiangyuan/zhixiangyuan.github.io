---
title: "为 Spring Bean 指定初始化和销毁方法的几种方式"
date: 2020-01-13T08:25:29+08:00
keywords: []
description: ""
tags: [
    "spring framework"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 Xml 方式指定

假设我们先创建下面这个 Bean

```java
public class CoffeeBean {
    public CoffeeBean() {
        System.out.println("Construct bean.");
    }

    public void init() {
        System.out.println("Bean init.");
    }

    public void destroy() {
        System.out.println("Bean destroy.");
    }
}
```

```xml
<bean id="coffee" class="me.yuanzx.test.CoffeeBean" init-method="init" destroy-method="destory">
</bean>
```

# 2 在 @Bean 的时候指定

这种方式和 xml 是一样的，只不过将其放到了注解上

```java
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public CoffeeBean coffeeBean() {
        return new CoffeeBean();
    }
```

# 3 实现 InitializingBean 和 DisposableBean 接口

InitializingBean 接口的回掉时机是在属性全部注入完之后，所以这个接口的实现方法可以校验各种属性值。DisposableBean 是容器销毁时的回掉方法。

```java
public class CoffeeBean implements InitializingBean, DisposableBean {
    public CoffeeBean() {
        System.out.println("Construct bean.");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Bean init.");
    }

    @Override
    public void destroy() {
        System.out.println("Bean destroy.");
    }
}
```

# 4 @PostConstruct 与 @PreDestroy

通过指定 @PostConstruct 来完成初始化，指定 @PreDestroy 来完成销毁。

```java
public class CoffeeBean {

    public CoffeeBean() {
        System.out.println("Construct bean.");
    }

    @PostConstruct
    public void init() {
        System.out.println("Bean init.");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("Bean destroy.");
    }
}
```
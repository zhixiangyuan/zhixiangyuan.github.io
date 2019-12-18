---
title: "XML 的命名空间"
date: 2019-12-18T16:02:00+08:00
keywords: []
description: ""
tags: [
    "XML"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

下面的例子是一个 XML 的命名空间的例子

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
		xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context 
        https://www.springframework.org/schema/context/spring-context.xsd"
>
	<context:annotation-config/>
</beans>
```

分析写在下面，下面为了方便用 # 作为注释符号

```
<?xml version="1.0" encoding="UTF-8"?>
<beans
          # xmlns 是 xml namespace 的缩写，表示默认的命名空间
          # beans 标签就属于这个命名空间
        xmlns="http://www.springframework.org/schema/beans"
          # 这是标准的定义命名空间的方式 xmlns:namespace-prefix="namespaceURI"
          # 后面的 namespaceURI 以我的理解来说只是一个与 namespace-prefix 一样的标识
          # 但是实际确实不能改的，比如说 context 对应的就是 http://www.springframework.org/schema/context
          # 这是由于在 Spring 的源码里面把这个写死了
        xmlns:context="http://www.springframework.org/schema/context"
          # xmlns:xsi 也是一个特殊的命名空间，它用于下面的 xsi:schemaLocation 
          # 来表示 xsd 约束文件的位置
          # 语法是 namespaceURI 和一个 xsd url 为一对，中间是空格或者换行都可以
          # namespaceURI 就是上面定义的，xsd 就是命名空间的约束文件
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context 
        https://www.springframework.org/schema/context/spring-context.xsd"
		
>
	<context:annotation-config/>
</beans>
```
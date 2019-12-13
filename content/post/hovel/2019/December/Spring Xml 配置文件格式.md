---
title: "Spring Xml 配置文件格式"
date: 2019-12-13T13:23:09+08:00
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

本文基于 Spring 源码，选项均从源码中提取出来而非 Spring 文档、别人的博客、DTD 或是 XSD，博主没在源码中找到版本号的位置，所以不清楚具体的版本号，不过是写本篇博客时的最新 Spring 源码。

```
<!-- 下面的 "" 均等价于 default -->
<!-- default-lazy-init：是否懒加载，default 则使用 parentDefaults 如果 parentDefaults 为 null，则默认为 false -->
<!-- default-merge：default 则使用 parentDefaults 如果 parentDefaults 为 null，则默认为 false -->
<beans 
    [default-lazy-init={""|"default"|"true"|"false"}]
    [default-merge={""|"default"|"true"|"false"}]
    [default-autowire={""|"default"|"no"|"byName"|"byType"|"constructor"|"autodetect"}]
    [default-autowire-candidates={}]
    [default-init-method={$自定义的 bean 初始化方法}]
    [default-destroy-method={$自定义的 bean 销毁方法}]
    [profile={$可以有多个，使用 ',' 与 ';' 作为分隔符}]
>
    <bean
        [id={$将会作为 beanName}]
        [name={$可以有多个，使用 ',' 与 ';' 作为分隔符}]
        [class={$加载的 class 的类型}]
        [parent={$这个应该是父类的 beanName}]
        [scope={"singleton"}]
        [abstract="true"|"false"]
        [lazy-init={""|"default"|"true"|"false"}]
        [autowire={""|"default"|"no"|"byName"|"byType"|"constructor"|"autodetect"}]
        [depends-on={$可以有多个，使用 ',' 与 ';' 作为分隔符}]
        [autowire-candidate={""|"default"|"true"|"false"}]
        [primary={"true"|"false"}]
        [init-method={"true"|"false"}]
        [destroy-method={}]
        [factory-method={}]
        [factory-bean={}]
    > 
        <description>[$描述性的内容]</description>
        <meta
            [key={}]
            [value={}]
        ></meta>
        <lookup-method
            [name={$methodName}]
            [bean={$beanRef}]
        ></lookup-method>
        <replaced-method
            [name={}]
            [replacer={}]
        >
            <!-- 这里需要注意的是 match 和 content 只会有一个生效，match 的优先级高于 content -->
            <arg-type [match={}]>[$content]</arg-type>
        </replaced-method>
        <!-- constructor-arg 的 attribute 中只能有 ref 或者 value 或者子标签，存在多个会报错 -->
        <constructor-arg
            [index={$这里面可以写数字，表示构造方法的第几个参数，从 0 开始}]
            [type={}]
            [name={}]
            [ref={$ref 中的内容不能为空，否则报错}]
            [value={}]
        >
            <!-- 需要注意的是这里面的子标签只能有一个，如果有多个会报错 -->
            <ref 
                [bean={$refName}]
                [parent={$如果 bean 为 null，则这个值赋值给 refName}]
            ></ref>
            <bean></bean>
            <idref></idref>
            <value [type={}]>[$value]</value>
            <null></null>
            <array></array>
            <list></list>
            <set></set>
            <map></map>
            <props></props>
        </constructor-arg>
        <property
            [name={}]
        >
            <meta
                [key={}]
                [value={}]
            ></meta>
            <!-- 需要注意的是这里面的子标签只能有一个，如果有多个会报错 -->
            <ref 
                [bean={$refName}]
                [parent={$如果 bean 为 null，则这个值赋值给 refName}]
            ></ref>
            <bean></bean>
            <idref></idref>
            <value [type={}]>[$value]</value>
            <null></null>
            <array></array>
            <list></list>
            <set></set>
            <map></map>
            <props></props>
        </property>
        <qualifier 
            [type={}]
            [value={}]
        >
            <attribute
                [key={}]
                [value={}]
            ></attribute>
        </qualifier>
    </bean>

</beans>
```


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

```
<!-- 下面的 "" 均等价于 default -->
<!-- default-lazy-init：标志是否对所有的 bean 执行懒加载，default 则使用 parentDefaults 如果 parentDefaults 为 ""，则默认为 false -->
<!-- default-merge：default 则使用 parentDefaults 如果 parentDefaults 为 null，则默认为 false -->
<!-- default-autowire: 如果使用自动绑定的话，用来标识全体 bean 使用哪一种默认绑定方式 -->
<!-- default-init-method: 如果所管辖的 <bean> 按照某种规则，都有同样名称的初始化方法的话，可以在这里统一指定这个初始化方法名，而不用再每一个 <bean> 上都重复单独指定  -->
<!-- default-destroy-method: 如果所管辖的 <bean> 有按照某种规则使用了相同名称的对象销毁方法，可以通过这个属性统一指定 -->
<beans 
    [default-lazy-init={""|"default"|"true"|"false"}]
    [default-merge={""|"default"|"true"|"false"}]
    [default-autowire={""|"default"|"no"|"byName"|"byType"|"constructor"|"autodetect"}]
    [default-autowire-candidates={}]
    [default-init-method={$自定义的 bean 初始化方法}]
    [default-destroy-method={$自定义的 bean 销毁方法}]
    [profile={$可以有多个，使用 ',' 与 ';' 作为分隔符}]
>
    
    <import [resource={$别的 xml 的位置，其中可以填写 "${user.dir}" 这样的变量}]></import>
    <alias name={$bean 的名字} alias={$别名}></alias>
    <!-- abstract: 标志 bean 是否需要实例化 -->
    <!-- parent: 指定父类之后会继承父类的默认值 -->
    <!-- scope: request 会为每个 HTTP 请求创建一个全新的对象供当前请求使用，当请求结束后，该对象实例的生命周期就结束 -->
    <!-- scope: session 为每一个 session 创建一个全新的对象 -->
    <!-- scope: global session 为一同一个端口的 session 创建一个全新的对象 -->
    <bean
        [id={$将会作为 beanName}]
        [name={$可以有多个，使用 ',' 与 ';' 作为分隔符}]
        [class={$加载的 class 的类型}]
        [parent={$这个应该是父类的 beanName}]
        [scope={"singleton"|"prototype"|"request"|"session"|"global session"}]
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
        [depends-on={}]
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
            [type={$传入值的类型，比如填入 int}]
            [name={}]
            [ref={$ref 中的内容不能为空，否则报错}]
            [value={$设置传进去的值}]
        >
            <!-- 需要注意的是这里面的子标签只能有一个，如果有多个会报错 -->
            <ref 
                [bean={$refName}]
                [parent={$如果 bean 为 null，则这个值赋值给 refName}]
            ></ref>
            <idref bean={}/>
            <bean></bean>
            <idref></idref>
            <value [type={}]>[$value]</value>
            <null></null>
            <!-- ...表示可以放 <ref>、<idref>、<bean> 等的子标签 -->
            <array [value-type={}] [merge={true|false}]>...</array>
            <!-- list 对应注入对类型为 java.util.List 以及其子类或者数组类型的依赖对象，通过 <list> 可以有序的为当前对象注入以 collection 形式声明的依赖 -->
            <list [value-type={}] [merge={true|false}]>...</list>
            <set [value-type={}] [merge={true|false}]>...</set>
            <map [key-type={}] [value-type={}]>
                <entry [key={}] [key-ref={}] [value={}] [value-ref={}] [value-type={}]>
                    <description></description>
                    <key></key>
                    ...
                </entry>
            </map>
            <props [merge={true|false}]>
                <prop [key={}]>[$value]</prop>
            </props>
        </constructor-arg>
        <property
            [name={}]
        >...</property>
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

# 参考文章

1. 《Spring 揭秘》
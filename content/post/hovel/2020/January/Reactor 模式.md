---
title: "Reactor 模式"
date: 2020-01-21T21:37:10+08:00
keywords: []
description: ""
tags: [
    "设计模式"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

网上关于 Reactor 的文章很多，不过全是和 NIO 直接相关，本文只阐述 Reactor 设计的思想，同时给出范例代码实现。

Reactor 的设计思想是通过 Reactor 去感知到不同的事件，然后根据不同的事件去派发不同的处理器处理事件，下面的代码思路是通过 Reactor 对 Integer 和 String 这两种类型的资源做不同的处理，下面看代码。

```java
// 先看一下最终效果，Reactor 对不同的事件做不同的处理
public class Main {
    public static void main(String[] args) {
        Reactor.dispatch(
            Event.IntegerResource, DefaultResource.createResource(1)
        ); // Integer resource: [1]
        Reactor.dispatch(
            Event.StringResource, DefaultResource.createResource("String")
        ); // String resource: [String]
    }
}

// 定义事件类型
public enum Event {
    /** Integer 类型的资源事件 */
    IntegerResource,
    /** String 类型的资源事件 */
    StringResource
}

// 处理器接口
public interface Handler {
    /**
     * 处理资源
     * @param resource 需要被处理的资源
     */
    void handle(Resource resource);
}

// Integer 处理器
public class IntegerHandler implements Handler {

    private static final IntegerHandler self = new IntegerHandler();

    public static IntegerHandler getInstance() {
        return self;
    }

    @Override
    public void handle(Resource resource) {
        System.out.println(String.format("Integer resource: [%s]", resource.getResource()));
    }
}

// String 处理器
public class StringHandler implements Handler {

    private static final StringHandler self = new StringHandler();

    public static StringHandler getInstance() {
        return self;
    }

    @Override
    public void handle(Resource resource) {
        System.out.println(String.format("String resource: [%s]", resource.getResource()));
    }
}

// 资源接口
public interface Resource<T> {

    /** 获取资源 */
    T getResource();
}

// 默认资源实现类
public class DefaultResource<T> implements Resource<T> {

    private T resource;

    public DefaultResource(T resource) {
        this.resource = resource;
    }

    @Override
    public T getResource() {
        return resource;
    }

    public static <T> DefaultResource createResource(T resource) {
        return new DefaultResource(resource);
    }
}

// Reactor 类
public class Reactor {
    /** 存放事件和处理器的映射 */
    private static final Map<Event, Handler> eventToHandlerMap = new ConcurrentHashMap<>();

    static {
        // 注册事件处理器
        eventToHandlerMap.put(Event.IntegerResource, IntegerHandler.getInstance());
        eventToHandlerMap.put(Event.StringResource, StringHandler.getInstance());
    }

    /**
     * 根据不同的事件将资源分发给不同的处理器处理
     *
     * @param event    事件
     * @param resource 资源
     */
    public static <T> void dispatch(Event event, Resource<T> resource) {
        Handler handler = eventToHandlerMap.get(event);
        handler.handle(resource);
    }

}
```
---
title: "JDK 中推荐的事件模型"
date: 2019-12-16T16:55:27+08:00
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

# 1 事件模型

JDK 中提供了一个接口和一个类给用户实现自己的事件机制，一个是 EventObject 事件，另一个是 EventListener 事件监听器。

一般使用是都是在事件监听器中实现相应的方法，然后将事件传进去做相应的处理。下面展示一个来自 《Spring 揭秘》的例子。

```java
/** 先自定义一个事件 */
public class MethodExecutionEvent extends EventObject {

    private String methodName;

    public MethodExecutionEvent(Object source) {
        super(source);
    }

    public MethodExecutionEvent(Object source, String methodName) {
        super(source);
        this.methodName = methodName;
    }

    public String getMethodName() {
        return methodName;
    }

    public MethodExecutionEvent setMethodName(String methodName) {
        this.methodName = methodName;
        return this;
    }
}
/** 定义定时化的监听器方法 */
public interface MethodExecutionEventListener extends EventListener {

    /** 处理方法开始执行的时候发布事件 */
    void onMethodBegin(MethodExecutionEvent evt);
    /** 处理方法执行结束的时候发布事件 */
    void onMethodEnd(MethodExecutionEvent evt);
}
/** 一个简单的事件监听器的实现 */
public class SimpleMethodExecutionEventListener implements MethodExecutionEventListener {
    @Override
    public void onMethodBegin(MethodExecutionEvent evt) {
        String methodName = evt.getMethodName();
        System.out.println("Start to execute the method [" + methodName + "].");
    }

    @Override
    public void onMethodEnd(MethodExecutionEvent evt) {
        String methodName = evt.getMethodName();
        System.out.println("Finished to execute the method [" + methodName + "].");
    }
}
/** 枚举表示事件的状态 */
public enum MethodExecutionStatus {
    /** 事件开始 */
    BEGIN, 
    /** 事件结束 */
    END
}
/** 事件的发布者 */
public class MethodExecutionEventPublisher {

    private List<MethodExecutionEventListener> listenerList = new ArrayList<>();

    public void methodToMonitor() {
        MethodExecutionEvent event2Publish = new MethodExecutionEvent(this, "methodToMonitor");
        // 任务开始的时候调一次监听器
        publishEvent(MethodExecutionStatus.BEGIN, event2Publish);
        // 执行了一堆任务...
        // 任务结束的时候调一次监听器
        publishEvent(MethodExecutionStatus.END, event2Publish);
    }

    /** 发布事件 */
    protected void publishEvent(MethodExecutionStatus status, MethodExecutionEvent methodExecutionEvent) {
        List<MethodExecutionEventListener> copyListeners = new ArrayList<>(listenerList);
        for (MethodExecutionEventListener copyListener : copyListeners) {
            if (MethodExecutionStatus.BEGIN.equals(status)) {
                copyListener.onMethodBegin(methodExecutionEvent);
            } else {
                copyListener.onMethodEnd(methodExecutionEvent);
            }
        }
    }

    public void addMethodExecutionEventListener(MethodExecutionEventListener listener) {
        this.listenerList.add(listener);
    }

    /** 移除某个 listener */
    public void removeListener(MethodExecutionEventListener listener) {
        if (this.listenerList.contains(listener)) {
            this.listenerList.remove(listener);
        }
    }

    /** 清除所有的监听器 */
    public void removeAllListeners() {
        this.listenerList.clear();
    }

    public static void main(String[] args) {
        // 创建事件的发布者
        MethodExecutionEventPublisher eventPublisher = new MethodExecutionEventPublisher();
        // 添加相应的事件监听器
        eventPublisher.addMethodExecutionEventListener(new SimpleMethodExecutionEventListener());
        // 发布事件
        eventPublisher.methodToMonitor();
    }
}
```

上述代码的核心逻辑就是先实现一个监听器，在任务开始的时候调一次监听器，在任务结束的时候再调一次监听器。

# 2 应用场景

比如说分布式中实现链路监控就可以在进入方法的时候调用一次，方法执行完毕调用一次，而这种通用逻辑可以直接使用 Spring 的 AOP 来实现，这样即能起到和业务代码解耦又能实现链路监控的效果。

# 参考文章

1. 《Spring 揭秘》
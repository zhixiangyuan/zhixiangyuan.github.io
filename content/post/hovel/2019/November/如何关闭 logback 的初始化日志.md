---
title: "如何关闭 Logback 的初始化日志"
date: 2019-11-22T10:22:25+08:00
keywords: []
description: ""
tags: [
    "Logback"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

> logback 版本 1.1.1 

先看解决方案

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    ...
    <!-- 关闭 logback 自身的初始化日志 -->
    <!-- 使用 NopStatusListener 或 StatusListenerAsList 均可以达到效果-->
<!--    <statusListener class="ch.qos.logback.core.status.NopStatusListener"/>-->
    <statusListener class="ch.qos.logback.core.status.StatusListenerAsList"/>
    ...
</configuration>
```

# 1 为什么加上一个 statusListener 就可以去掉初始化日志

通过跟踪代码，发现初始化日志的打印位置

```java
package org.slf4j.impl;
public class StaticLoggerBinder implements LoggerFactoryBinder {
    ...
    void init() {
        try {
            try {
                new ContextInitializer(defaultLoggerContext).autoConfig();
            } catch (JoranException je) {
                Util.report("Failed to auto configure default logger context", je);
            }
            // logback-292
            // 只要这个地方 return true 就可以跳过打印日志，然后我们看看里面
            if (!StatusUtil.contextHasStatusListener(defaultLoggerContext)) {
                // 初始化日志是在这一步打印的，所以只要跳过这一步就可以了
                StatusPrinter.printInCaseOfErrorsOrWarnings(defaultLoggerContext);
            }
            contextSelectorBinder.init(defaultLoggerContext, KEY);
            initialized = true;
        } catch (Exception t) { // see LOGBACK-1159
            Util.report("Failed to instantiate [" + LoggerContext.class.getName() + "]", t);
        }
    }
}

public class StatusUtil {
    ...
    static public boolean contextHasStatusListener(Context context) {
        StatusManager sm = context.getStatusManager();
        if (sm == null)
            return false;
        List<StatusListener> listeners = sm.getCopyOfStatusListenerList();
        // 只要这里有 listeners 就会返回 true
        if (listeners == null || listeners.size() == 0)
            return false;
        else
            return true;
    }
}
```

通过查看代码，能够发现只要 listeners 不为 null 且大于 0 则会不打印初始化日志，然后我们看一下 StatusListener 有哪些实现。

```java
// StatusListener 是一个接口，总共有三个实现
public interface StatusListener {
    void addStatusEvent(Status status);
}
// 下面是三个实现
public class NopStatusListener implements StatusListener {

    public void addStatusEvent(Status status) {
        // nothing to do
    }
}

public class StatusListenerAsList implements StatusListener {

    List<Status> statusList = new ArrayList<Status>();

    public void addStatusEvent(Status status) {
        statusList.add(status);
    }
}

abstract public class OnPrintStreamStatusListenerBase extends ContextAwareBase implements StatusListener, LifeCycle {
    ...
    public void addStatusEvent(Status status) {
        if (!isStarted)
            return;
        // 可以发现这种实现也打印了日志
        print(status);
    }
}
```

可以发现上面三种实现，只有 NopStatusListener 和 StatusListenerAsList 没有打印日志，所以我们加入这两种 StatusListener 的其中一个就可以让启动日志消失。
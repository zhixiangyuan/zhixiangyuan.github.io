---
title: "聊一聊 SpringApplicationRunListener 调用的时机"
date: 2019-12-30T11:09:37+08:00
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

SpringApplicationRunListener 是 SpringBoot 中的一个接口，这个接口中的方法会在 SpringBoot 应用的生命周期中不同时间进行调用。下面看一看这个接口中的相关方法

```java
/** 这其中的方法的执行顺序是从上往下执行 */
public interface SpringApplicationRunListener {

	/** 在 SpringBoot 还没有开始初始化的时候调用，调用时间是所有方法中最早的 */
	default void starting() {
	}

	/** 当 SpringBoot 应用环境变量全部初始化好的时候调用该方法 */
	default void environmentPrepared(ConfigurableEnvironment environment) {
	}

	/** 当 spring context 初始化好的时候调用该方法，但是此时还没有加载资源 */
	default void contextPrepared(ConfigurableApplicationContext context) {
	}

	/** 当 spring context 在 spring 加载好资源后调用此方法，但此时容器还没有刷新 */
	default void contextLoaded(ConfigurableApplicationContext context) {
	}

	/** 在 spring 容器刷新之后立刻调用这个方法 */
	default void started(ConfigurableApplicationContext context) {
	}

	/** 当 SpringBoot 应用已经处于运行时调用此方法 */
	default void running(ConfigurableApplicationContext context) {
	}

	/** 当初始化出现异常时调用此方法 */
	default void failed(ConfigurableApplicationContext context, Throwable exception) {
	}
}
```

下面我们来看一下源码中调用这些钩子的地方

```java
/** listeners 是监听器数组，已经在前面完成了初始化，这里直接调用即可 */
public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
        // 这里便是调用 starting() 方法的地方
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            // prepareEnvironment 方法的内部在环境创建好之后便调用了 environmentPrepared() 方法
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[]{ConfigurableApplicationContext.class}, context);
            // prepareContext 方法的内部依次调用了 contextPrepared 和 contextLoaded
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
            // 这里便是调用 started() 方法的地方
			listeners.started(context);
			callRunners(context, applicationArguments);
		} catch (Throwable ex) {
            // 在处理异常的内部便调用了 failed() 方法
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
            // 这里便是调用 running() 方法的地方
			listeners.running(context);
		} catch (Throwable ex) {
            // 在处理异常的内部便调用了 failed() 方法
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```

关于调用时机的简单描述到这里便结束了
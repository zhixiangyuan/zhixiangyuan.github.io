---
title: "Spring Aop 中 AspectJProxyFactory 的使用方式"
date: 2019-12-23T16:57:14+08:00
keywords: []
description: ""
tags: [
    "spring"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

```java
/** 定义目标类 */
public class AspectTarget {

    public void method1() {
        System.out.println("method1 execution");
    }

    public void method2() {
        System.out.println("method2 execution");
    }

}
/** 定义切面 */
@Aspect
public class PerformanceTraceAspect {

    @Pointcut("execution(public void *.method1()) || execution(public void *.method2())")
    public void pointcut() {}

    @Around("pointcut()")
    public Object performanceTrace(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        try {
            stopWatch.start();
            return joinPoint.proceed();
        } finally {
            stopWatch.stop();
            System.out.println("monitor: " + 
                    // 获取调用的方法名
                    joinPoint.getSignature().getName());
        }
    }
}

/** 测试 */
public class Main {
    public static void main(String[] args) {
        AspectJProxyFactory aspectJProxyFactory = new AspectJProxyFactory();
        // 设置需要代理的目标类
        aspectJProxyFactory.setTarget(new AspectTarget());
        // 设置切面
        aspectJProxyFactory.addAspect(PerformanceTraceAspect.class);
        AspectTarget proxy = aspectJProxyFactory.getProxy();
        proxy.method1();
        proxy.method2();
    }
}

输出：
method1 execution
monitor: method1
method2 execution
monitor: method2
```
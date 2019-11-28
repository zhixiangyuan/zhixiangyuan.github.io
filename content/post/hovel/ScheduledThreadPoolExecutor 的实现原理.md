---
title: "ScheduledThreadPoolExecutor 的实现原理"
date: 2019-11-28T14:54:19+08:00
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

# 1 如何实现定时的效果

![](/hub/2019/November/64.png)

定时效果实现的关键在 ScheduledFutureTask 的 `public void run()` 方法，下面是该方法的实现

```java
        public void run() {
            // 判断是否是周期运行的任务
            boolean periodic = isPeriodic();
            // 判断当前线程池是否可以执行任务
            if (!canRunInCurrentRunState(periodic))
                // 不能执行则取消掉任务
                cancel(false);
            else if (!periodic)
                // 不是周期性任务则调用 FutureTask 的 run 方法
                ScheduledFutureTask.super.run();
            // 如果是周期性的任务则执行任务，成功后返回 true
            else if (ScheduledFutureTask.super.runAndReset()) {
                // 设置下次的执行时间
                setNextRunTime();
                // 再次将任务添加到队列
                reExecutePeriodic(outerTask);
            }
        }

        // 判断是否是需要周期运行的方法，其实如果周期不为 0，那么就是需要周期运行的方法
        public boolean isPeriodic() {
            return period != 0;
        }
```

# 2 ScheduledThreadPoolExecutor 中的特色方法

```java

    /**
     * @param initialDelay 第一次执行任务经过指定时间再开始执行
     * @param period       从上一个任务的开始执行时间开始计算的时间周期
     */
    public ScheduledFuture<?> scheduleAtFixedRate(
            Runnable command,
            long initialDelay,
            long period,
            TimeUnit unit
    );

    /**
     * @param initialDelay 第一次执行任务经过指定时间再开始执行
     * @param delay        从上一个任务的结束时间开始计算的时间周期
     */
    public ScheduledFuture<?> scheduleWithFixedDelay(
            Runnable command,
            long initialDelay,
            long delay,
            TimeUnit unit
    );
```

以下是测试用例和测试的输出结果

```java
    // 按照设定应该出现每过三秒（延迟两秒，睡一秒）打印一次时间
    public static void main(String[] args) {
        ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor();
        scheduledExecutorService.scheduleWithFixedDelay(() -> {
            System.out.println(LocalDateTime.now());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, 0, 2, TimeUnit.SECONDS);
    }

OUTPUT：
2019-11-28T15:46:19.961
2019-11-28T15:46:22.971
2019-11-28T15:46:25.979

    // 按照设定应该出现每过两秒（延迟两秒）打印一次时间
    public static void main(String[] args) {
        ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor();
        scheduledExecutorService.scheduleAtFixedRate(() -> {
            System.out.println(LocalDateTime.now());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, 0, 2, TimeUnit.SECONDS);
    }

OUTPUT：
2019-11-28T15:48:09.179
2019-11-28T15:48:11.178
2019-11-28T15:48:13.178

    // 如果延迟两秒睡三秒会怎么样
    public static void main(String[] args) {
        ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor();
        scheduledExecutorService.scheduleAtFixedRate(() -> {
            System.out.println(sum.getAndIncrement() + ". " + LocalDateTime.now());
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, 0, 2, TimeUnit.SECONDS);
    }

// 结果是刚放回队列，又要立刻执行
OUTPUT：
1. 2019-11-28T15:49:32.482
2. 2019-11-28T15:49:35.487
3. 2019-11-28T15:49:38.492
4. 2019-11-28T15:49:41.496
```



# 参考文章

1. [并发编程 —— ScheduledThreadPoolExecutor](https://juejin.im/post/5ae75604f265da0ba56753cd)
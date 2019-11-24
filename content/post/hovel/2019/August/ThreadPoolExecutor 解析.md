---
title: "ThreadPoolExecutor 解析"
date: 2019-08-06T17:26:17+08:00
keywords: []
description: ""
tags: [
    "Java"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 ThreadPoolExecutor 的构造函数

```java
    public ThreadPoolExecutor(
            // 核心线程数，默认不会被回收掉，但是如果设置了 allowCoreThreadTimeOut 
            // 为 ture，那么当核心线程闲置超时，也会被回收
            int corePoolSize,
            // 最大线程数，线程池能容纳的最大容量
            // 上限被 CAPACITY 限制 (2^29 - 1)
            int maximumPoolSize,
            // 闲置线程的存活时间
            long keepAliveTime,
            // keepAliveTime 的单位
            TimeUnit unit,
            // 用于存放任务的队列
            BlockingQueue<Runnable> workQueue,
            // 创建线程的工厂类
            ThreadFactory threadFactory,
            // 任务失败时的执行策略，Java 自带了四个策略
            // 如果需要自定义也可以自己实现
            RejectedExecutionHandler handler
    ) {
        if (corePoolSize < 0 ||
                maximumPoolSize <= 0 ||
                maximumPoolSize < corePoolSize ||
                keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

# 2 线程池的状态

看下述源码前补充两个知识点

1. 1 与任意数进行 & 运算还是原数
2. 0 与任意数进行 | 运算还是原数

```java
    // ctl 中存储了两个参数，后 29 位当前存活的线程数，高三位表示
    // 线程池当前的状态，总共有五种状态。
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
    // Integer.SIZE 表示 int 占多少位，Integer.SIZE - 3 表示的是去
    // 除 int 高三位剩下的所有位，在下面初始化线程池状态的时候会用上
    private static final int COUNT_BITS = Integer.SIZE - 3;
    // CAPACITY 的二进制形式: 0b00011111_11111111_11111111_11111111
    // CAPACITY 就像是一个筛子，和别的值做 & 运算时，会将别的值的
    // 低 29 位全部筛出来
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

    // 以下的五个值表示线程池的状态，ctl 中的高三位存储的就是这些值
    // RUNNING 状态可以接受新任务并处理
    // RUNNING 的二进制形式 0b11100000_00000000_00000000_00000000
    private static final int RUNNING    = -1 << COUNT_BITS;
    // SHUTDOWN 状态不会接受新的任务，但是会处理队列中还存在的任务
    // SHUTDOWN 的二进制形式 0b00000000_00000000_00000000_00000000
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    // STOP 状态不会接受新的任务，也不会处理队列中的任务，直接中断
    // STOP 的二进制形式 0b00100000_00000000_00000000_00000000
    private static final int STOP       =  1 << COUNT_BITS;
    // TIDYING 状态表示所有任务都已经终止
    // TIDYING 的二进制形式 0b01000000_00000000_00000000_00000000
    private static final int TIDYING    =  2 << COUNT_BITS;
    // TERMINATED 状态表示 terminated() 方法已经执行完成
    // TERMINATED 的二进制形式 0b01100000_00000000_00000000_00000000
    private static final int TERMINATED =  3 << COUNT_BITS;

    // 将 ctl 的高三位状态筛出来
    private static int runStateOf(int c)     {
        // 从 c 中取出高三位的数字
        // c          0bxxxxxxxx_xxxxxxxx_xxxxxxxx_xxxxxxxx
        // ~CAPACITY  0b11100000_00000000_00000000_00000000
        // ~CAPACITY 就像筛子一样，将 c 的高三位筛出来
        return c & ~CAPACITY; 
    }
    // 将 ctl 除了高三位后面的数字筛出来
    // 后面筛出来的数字表示当前线程数
    private static int workerCountOf(int c)  {
        // 从 c 中取出后 29 位的数字
        // c         0bxxxxxxxx_xxxxxxxx_xxxxxxxx_xxxxxxxx
        // CAPACITY  0b00011111_11111111_11111111_11111111
        // CAPACITY 就像筛子一样，将 c 的后 29 位全部筛出来了
        return c & CAPACITY;
    }
    // rs 表示 runState 线程池的运行状态
    // wc 表示 workerCount 工作线程数
    private static int ctlOf(int rs, int wc) {
        // rs 的后 29 位必然全为 0
        // wc 的高三位必然全为 0
        // 计算过程如下所示
        // rs 0bxxx00000_00000000_00000000_00000000
        // wc 0b000xxxxx_xxxxxxxx_xxxxxxxx_xxxxxxxx
        // rs | wc 采用或运算，0 与任意数或还是等于该数
        // 所以这里就将 rs 和 wc 的高三位和低 29 位全部
        // 整合到一个数中，也就是 ctl
        return rs | wc; 
    }
    // 判断当前线程池是否处于 RUNNING 状态
    private static boolean isRunning(int c) {
        // SHUTDOWN 的值是 0，c < SHUTDOWN 也就是判断
        // c 是否小于 0，在线程池状态中只有 RUNNING 状态
        // 小于 0，非常巧妙的判断出是否处于 RUNNING 状态
        return c < SHUTDOWN;
    }
```

## 2.1 线程池的状态转换图

![线程池的状态转换图](/media/hovel/36.png)


# 3 核心方法详解

## 3.1 public void execute(Runnable command)

execute 方法便是将需要执行的任务加入到线程池当中，但是这个方法无法获取线程执行的结果。

```java
    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
         // 取出线程池中的 ctl
        int c = ctl.get();
        // 取出 ctl 中的工作线程数，并且和 corePoolSize 做比较
        if (workerCountOf(c) < corePoolSize) {
            // 如果工作线程数小于 corePoolSize 则执行 addWorker 方法
            // 创建新的线程执行任务
            if (addWorker(command, true))
                // 添加任务成功则直接返回
                return;
            // 添加失败则再次取出 ctl
            c = ctl.get();
        }
        // 判断线程池是否处于 RUNNING 状态，如果处于 RUNNING 状态
        // 则将 command 加入到工作队列当中
        if (isRunning(c) && workQueue.offer(command)) {
            // 这里之所以需要再次检查，是因为前面从 ctl.get() 中获取到 ctl
            // 的值之后，判断的都是 c，而在此期间 ctl 的状态可能会发生改变，
            // 所以这里需要再次检查，如果检查出来线程池不是 RUNNING 状态，则
            // 将添加的任务移除，实现了一个手动乐观锁的效果
            int recheck = ctl.get();
            // 添加成功需要再次检查线程池状态，如果此时线程池脱离了
            // RUNNING 状态，则将刚刚添加的 command 移除并且执行 reject(command);
            if (! isRunning(recheck) && remove(command))
                reject(command);
            // 如果线程池处于 RUNNING 状态，则再次检查工作线程数是否为 0
            // 如果工作线程数为 0，则创建线程
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        
        else if (!addWorker(command, false))
            // 如果添加任务失败，通过预先设定的拒绝策略拒绝该任务
            reject(command);
    }
```

看完上述代码，便能大概总结线程池的工作原理了，首先一个任务进入线程池，线程池会先判断是否当前工作的线程数小于核心线程数，如果小于的话那么就开启新的线程去执行任务，如果检查出来工作线程数大于等于核心线程数，那么就将任务放到任务队列里面，如果当放入任务队列失败时，那么便会去开启新的工作线程执行任务，如果开启新的工作线程失败，那么则使用指定的拒绝策略拒绝任务。

## 3.2 final void reject(Runnable command)

```java
    final void reject(Runnable command) {
        // 调用之前设定的 handler 执行该任务
        handler.rejectedExecution(command, this);
    }
```

## 3.3 private boolean addWorker(Runnable firstTask, boolean core)

```java
    /**
     * 创建一个新的线程来执行给定的任务
     * @param firstTask 是需要执行的任务
     * @param core true 使用核心线程 / false 不
     */
    private boolean addWorker(Runnable firstTask, boolean core) {
        retry:
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // 这里判断分为五种情况
            // 1. rs 为 RUNNING 通过校验
            // 2. rs 为 STOP、TIDYING、TERMINATED 则 return false
            // STOP、TIDYING、TERMINATED 状态不接受也不处理队列任务
            // 3. rs 为 SHUTDOWN，提交的任务不为空，则 return false
            // 4. rs 为 SHUTDOWN，提交的任务为空，工作队列也为空，则 reutrn false
            // 状态为 SHUTDOWN，提交的任务为空、工作队列为空，则线程池有资格关闭，
            // 则直接 return false
            // 5. rs 为 SHUTDOWN，提交的任务为空，并且工作队列不为空，则通过校验
            // 因为 SHUTDOWN 状态下刚好可以处理队列任务
            if (rs >= SHUTDOWN &&
                !(rs == SHUTDOWN && firstTask == null && !workQueue.isEmpty()))
                return false;

            // 开启循环去创建线程
            for (;;) {
                int wc = workerCountOf(c);
                // 如果工作线程数大于可记录的最大线程数，或者
                // 工作线程数超过了指定的核心线程或者最大线程数
                // 则 return false
                if (wc >= CAPACITY ||
                    wc >= (core ? corePoolSize : maximumPoolSize))
                    return false;
                // 接下来将工作线程数加一
                if (compareAndIncrementWorkerCount(c))
                    // 工作线程数增加成功则结束这个双重 for 循环
                    // 继续向下执行
                    break retry;
                c = ctl.get();
                // 如果工作线程数加一失败了，那么先判断一下线程池
                // 是否处于 RUNNING 状态，如果不处于 RUNNING 则跳到
                // retry 标签处重新执行双重 for 循环；如果处于 RUNNING
                // 则只执行这个内部 for 循环即可
                if (runStateOf(c) != rs)
                    continue retry;
            }
        }

        // 执行到这里的时候工作线程数已经加一了，接下来就创建线程干活
        // workerStarted 表示 worker 的线程是否启动
        boolean workerStarted = false;
        // workerAdded 表示 worker 是否增加成功
        boolean workerAdded = false;
        Worker w = null;
        try {
            // 创建 Worker 对象
            w = new Worker(firstTask);
            final Thread t = w.thread;
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock();
                try {
                    int rs = runStateOf(ctl.get());
                    // 重新检查线程池状态，或者是判断当前是 SHUTDOWN 状态
                    if (rs < SHUTDOWN ||
                        (rs == SHUTDOWN && firstTask == null)) {
                        if (t.isAlive()) 
                            // 如果线程已经 start 了则抛出异常
                            throw new IllegalThreadStateException();
                        // 将创建的工作者加入到工作者集合中
                        workers.add(w);
                        int s = workers.size();
                        if (s > largestPoolSize)
                            // 更新最大线程数
                            largestPoolSize = s;
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                }
                if (workerAdded) {
                    // worker 添加成功，启动 worker 线程
                    // 这里便是 worker 线程启动的地方
                    t.start();
                    workerStarted = true;
                }
            }
        } finally {
            if (! workerStarted)
                // 如果 worker 的线程没有启动成功
                // 则进行回滚，移除之前添加的 worker
                addWorkerFailed(w);
        }
        return workerStarted;
    }
```

## 3.4 private void addWorkerFailed(Worker w)

```java
    // 回滚 worker 的添加，将 worker 从 workers 中移除
    private void addWorkerFailed(Worker w) {
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            if (w != null)
                // 移除 worker
                workers.remove(w);
            // 有效线程数 -1
            decrementWorkerCount();
            // 有 worker 线程移除，可能是最后一个线程退出，
            // 需要尝试终止线程池
            tryTerminate();
        } finally {
            mainLock.unlock();
        }
    }
```

## 3.5 final void tryTerminate()

```java
    /**
     * 该方法用来尝试终止线程池，主要在移除 Worker 后调用此方法。首先进行一些状态的
     * 校验，如果通过校验，则在加锁的条件下，使用 CAS 将运行状态设为 TERMINATED，有
     * 效线程数设为 0
     */
    final void tryTerminate() {
        for (;;) {
            int c = ctl.get();
            // 只有当前状态为 STOP 或者 SHUTDOWN 并且队列为空，才会尝试整理并终止
            // 1. 当前状态为 RUNNING，则不尝试终止，直接返回
            // 2. 当前状态为 TIDYING 或 TERMINATED，代表有其他线程正在执行终止，直接返回
            // 3. 当前状态为 SHUTDOWN 并且 workQueue 不为空，则不尝试终止，直接返回
            if (isRunning(c) ||
                runStateAtLeast(c, TIDYING) ||
                (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
                return;
            // 走到这代表线程池可以终止（通过上面的校验）
            // 如果此时有效线程数不为 0，则将中断一个空闲的 worker，以确保关闭信号传播
            if (workerCountOf(c) != 0) {
                interruptIdleWorkers(ONLY_ONE);
                return;
            }

            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // 使用 CAS 将 ctl 的运行状态设置为 TIDYING，有效线程数设置为 0
                if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                    try {
                        // 供用户重写的 terminated 方法，默认为空
                        terminated();
                    } finally {
                        // 将 ctl 的运行状态设置为 TERMINATED，有效线程数设置为 0
                        ctl.set(ctlOf(TERMINATED, 0));
                        termination.signalAll();
                    }
                    return;
                }
            } finally {
                mainLock.unlock();
            }
        }
    }
```

# 4 Worker 类详解

# 5 零碎方法详解

## 5.1 private boolean compareAndIncrementWorkerCount(int expect)

```java
    private boolean compareAndIncrementWorkerCount(int expect) {
        // CAS 方法，成功则工作线程数 +1
        return ctl.compareAndSet(expect, expect + 1);
    }
```

## 5.2 private void decrementWorkerCount()

```java
    private void decrementWorkerCount() {
        // 调用 CAS 方法使工作线程数 -1
        // 如果失败则一直重试，此处为自旋锁
        do {} while (! compareAndDecrementWorkerCount(ctl.get()));
    }
```

## 5.3 private boolean compareAndDecrementWorkerCount(int expect)

```java
    private boolean compareAndDecrementWorkerCount(int expect) {
        // CAS 方法，成功则工作线程数 -1
        return ctl.compareAndSet(expect, expect - 1);
    }
```

# 参考资料

1. [重走 JAVA 之路（五）：面试又被问线程池原理？教你如何反击](https://juejin.im/post/5c99c29ee51d4559bb5c6541)
2. [线程池详解（ThreadPoolExecutor）](https://zhuanlan.zhihu.com/p/34405230)
3. 非常推荐：[深度解读 java 线程池设计思想及源码实现](https://javadoop.com/2017/09/05/java-thread-pool/)
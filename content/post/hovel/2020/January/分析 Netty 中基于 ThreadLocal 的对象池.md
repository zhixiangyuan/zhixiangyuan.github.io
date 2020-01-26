---
title: "分析 Netty 中基于 ThreadLocal 的对象池"
date: 2020-01-26T19:39:28+08:00
keywords: []
description: ""
tags: [
    "netty"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 对象池原理简述

Netty 中提供的对象池使用起来非常简单，在使用的时候只需要关注两个点，一是需要实现相应的 newObject 对象的方法，毕竟对象池里面放置什么对象是由使用者来决定的，二是回收方法，在实现 newObject 方法的时候，该方法有一个入参是 Handle，这个参数的唯一作用便是实现回收的效果。在实现完后，我们便只需要通过该对象池的 get 方法便可以获取对象，获取的时候如果对象池里面有对象便直接获取对象池中的对象，如果没有对象则生成一个对象。

下面我们先看来一个使用对象池的例子。

```java
/** 需要被对象池管理的对象 */
public class OneObject {
    private final Recycler.Handle<OneObject> handle;

    public OneObject(Recycler.Handle<OneObject> handle) {
        // 保存对象池的 handle，该对象需要在回收的时候使用
        this.handle = handle;
    }

    /** 实现回收方法，只需要调用 handle 的 recycle 方法即可 */
    public void recycle() {
        handle.recycle(this);
    }
}

/** 继承 Netty 的 {@link Recycler} 实现自定义的对象池 */
public class OneObjectPool extends Recycler<OneObject> {
    @Override
    protected OneObject newObject(Handle<OneObject> handle) {
        // new 一个 OneObject 对象
        return new OneObject(handle);
    }
}

/** 测试效果 */
public class Main {

    public static final OneObjectPool ONE_OBJECT_POOL = new OneObjectPool();

    public static void main(String[] args) {
        System.out.println(String.format("OneObject hash: %s", ONE_OBJECT_POOL.get().hashCode()));
        // OneObject hash: 1401420256
        System.out.println(String.format("OneObject hash: %s", ONE_OBJECT_POOL.get().hashCode()));
        // OneObject hash: 1530388690

        // 可以看到上面两个结果，我们直接通过 ONE_OBJECT_POOL.get() 来获取两个对象的 hashcode
        // 是不同的，下面我们试试获取一个对象，然后将该对象还回去，然后再获取的效果
    }
}

/** 测试效果 */
public class Main {

    public static final OneObjectPool ONE_OBJECT_POOL = new OneObjectPool();

    public static void main(String[] args) {
        // 获取对象
        OneObject oneObject = ONE_OBJECT_POOL.get();
        System.out.println(String.format("OneObject hash: %s", oneObject.hashCode()));
        // OneObject hash: 1401420256
        // 将对象还回对象池
        oneObject.recycle();
        // 再次从对象池中获取对象
        oneObject = ONE_OBJECT_POOL.get();
        System.out.println(String.format("OneObject hash: %s", oneObject.hashCode()));
        // OneObject hash: 1401420256

        // 可以看到两次获取的对象是 hashcode 是相同的
    }
}
```

# 2 对象池源码分析

通过上面部分的代码可以看到，最核心的方法便是获取对象的 get 方法和回收对象的 recycle 方法。我们先来看看 Recycler 的 get 方法。

```java
public abstract class Recycler<T> {
    ...
    public final T get() {
        if (maxCapacityPerThread == 0) {
            return newObject((Handle<T>) NOOP_HANDLE);
        }
        // 每一个线程都一个自己的 stack
        Stack<T> stack = threadLocal.get();
        // 从栈中获取 handle
        DefaultHandle<T> handle = stack.pop();
        // 如果 handle 为 null 则去创建一个
        if (handle == null) {
            handle = stack.newHandle();
            handle.value = newObject(handle);
        }
        // 返回 handle 中的值
        return (T) handle.value;
    }
}
```

看了上面可能会有点疑惑 handle 是什么，下面我们看看 handle 的源码

```java
    /** Handle 本身只提供一个方法，便是 recycle，通过该方法回收对象 */
    public interface Handle<T> {
        /** 回收对象 */
        void recycle(T object);
    }

    /** 该类是 Handle 的默认实现 */
    static final class DefaultHandle<T> implements Handle<T> {
        private int lastRecycledId;
        private int recycleId;

        boolean hasBeenRecycled;

        private Stack<?> stack;
        private Object value;

        DefaultHandle(Stack<?> stack) {
            this.stack = stack;
        }

        @Override
        public void recycle(Object object) {
            if (object != value) {
                throw new IllegalArgumentException("object does not belong to handle");
            }

            Stack<?> stack = this.stack;
            if (lastRecycledId != recycleId || stack == null) {
                throw new IllegalStateException("recycled already");
            }
            // 可以看到该实现的回收方法最核心的便是将该 handle 再次推入到 stack 中
            // handle 中持有的 value 需要被缓存的对象，这样下次从对象池中获取出 handle
            // 后，便可以直接从 handle 中获取出 value 对象。
            stack.push(this);
        }
    }
```

从该源码中，便能够看出这个对象池其实是线程独有的，每个线程独享自己的对象池。这样的设计有一个好处，那便是不会出现多线程竞争的问题，在多线程的环境中是天然线程安全的，不需要在其中增加任何锁。
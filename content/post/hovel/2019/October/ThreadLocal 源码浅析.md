---
title: "ThreadLocal 源码浅析与其使用场景"
date: 2019-10-18T16:36:38+08:00
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

# 1 源码浅析

ThreadLocal 用于不同线程存储不同的线程全局变量，在该线程下用 ThreadLocal 取出的值是一致的。这里先简述一下原理，然后再看代码。ThreadLocal 其实是一个 Key，在 Thread 当中有一个 threadLocals 变量，存储着 ThreadLocalMap，ThreadLocal 便是 ThreadLocalMap 当中的键，value 便是我们存储的值，由于每一个 Thread 都有自己的 threadLocals，所以我们在不同的线程中就可以用相同的 ThreadLocal 取出不同的值。

既然是使用 Map 做的存储，Map 中使用的数据结构是一个 Entry[] 数组，它在确定 ThreadLocal 存储在哪个位置的时候是使用 HashCode 来计算的，那么如果出现冲突怎么办，它通过判断数组下一个位置是否为空，如果为空则将 ThreadLocal 放入其中，否则继续找下一个数组位置，判断是否为空。

```java
    // 从下面这段代码开始
    private static ThreadLocal threadLocal = new ThreadLocal();
    public static void main(String[] args) {
        threadLocal.set(1);
    }

    // 下面是 set() 的源码
    public void set(T value) {
        // 取出当前线程
        Thread t = Thread.currentThread();
        // 取出线程中 ThreadLocalMap
        ThreadLocalMap map = getMap(t);
        if (map != null)
            // Map 存在则直接将值放进去
            map.set(this, value);
        else
            // Map 不存在测新创建 Map 并将值放进去
            createMap(t, value);
    }

    // 下面是 createMap(t, value); 的源码
    void createMap(Thread t, T firstValue) {
        // 给线程的 threadLocals 赋值新的 ThreadLocalMap
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

    static class ThreadLocalMap {
        private Entry[] table;
        // 下面是 map.set(this, value); 的源码
        private void set(ThreadLocal<?> key, Object value) {
            // table 是 ThreadLocalMap 类中存储的数组
            Entry[] tab = table;
            // 获取数组长度
            int len = tab.length;
            // 使用传入的 ThreadLocal 计算数组下标
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                e != null;
                // 寻找数组下一个位置
                // 这里面的函数如果找到了数组尾部会返回数组头部
                e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                // 如果这里 k != key，说明传入的 thradLocal 和
                // 数组中存储的 threadLocal 不同，那么可能就是
                // 出现了冲突，也可能是 null
                if (k == key) {
                    e.value = value;
                    return;
                }

                // 如果这里判断存储的 k == null 说明没存东西
                // 则直接将新的 threadLocal 和其 value 存进去
                // 如果 k != null 则说明出现了 hash 冲突
                // 则寻找数组下一个位置
                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
    }
```

上面分析了 set 值的过程，下面我们来看看 get 值的过程，这个过程里面有一个点是需要注意的，在出现 hash 冲突的时候，根据索引找到的位置已经不是那个 ThreadLocal 了，那么它是怎么处理的呢，我们将在下面的代码中看到具体的处理方式。

```java
public class ThreadLocal<T> {
    /** get 部分的源码 */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            // 关键在这里，getEntry 的源码在下面
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        // 如果没有值的话，则在其中设置初始化值
        return setInitialValue();
    }

    static class ThreadLocalMap {
        private Entry getEntry(ThreadLocal<?> key) {
            // 计算索引
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                // 可以看到当 e 为 null 或者出现冲突的时候会走下面的函数
                return getEntryAfterMiss(key, i, e);
        }
        
        private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;

            // 关键便是这个 while 循环的条件
            // 当 e != null 的时候会进入循环
            // 判断是否是我们需要的那个 ThreadLocal
            // 它会一直寻找到 e == null 才会停止循环
            // 所以整个逻辑就很清晰了，他的查找逻辑就是
            // 从冲突的点开始向后遍历，一直找到一个 null
            // 或者是找到那个 ThreadLocal
            while (e != null) {
                ThreadLocal<?> k = e.get();
                if (k == key)
                    return e;
                if (k == null)
                    expungeStaleEntry(i);
                else
                    // 如果出现冲突，那么就找下一个索引
                    i = nextIndex(i, len);
                e = tab[i];
            }
            return null;
        }
    }
}
```

# 2 使用场景

明白了其内部的原理，那么便能够理解 ThreadLocal 其实是一个线程级别的全局变量，那么如果当我们需要一个线程级别的全局变量的时候便可以使用该类。换句话说便是线程需要有自己单独的实例，该实例需要在多个方法中共享，但是不希望被多个线程共享。

# 3 内存泄漏

ThreadLocalMap 中的 key 为 ThreadLocal 的弱引用，弱引用的特点是，如果这个对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉。

所以如果 ThreadLocal 没有被外部墙引用的情况下，在垃圾回收的时候会被清理掉的，这样一来 ThreadLocalMap 中使用这个 ThreadLocal 的 key 也会被清理掉。但是 value 是墙引用，不会被清理，这样一来就会出现 key 为 null 的 value。

ThreadLocalMap 的实现中已经考虑了这种情况，在调用 set()、get()、remove() 方法的时候，会清理掉 key 为 null 的记录。如果说会出现内存泄漏，那只有在出现了 key 为 null 的记录后，没有手动调用 remove() 方法，并且之后也不再调用 get()、set()、remove() 方法的情况下会出现内存泄漏。

get() 与 set() 方法均是内部对于 key 为 null 的情况做了特殊的处理，而 remove() 方法则需要将 ThreadLocal 传递进去做移除。
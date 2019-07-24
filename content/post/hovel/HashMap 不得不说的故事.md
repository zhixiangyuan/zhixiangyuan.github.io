---
title: "HashMap 不得不说的故事"
date: 2018-09-03T10:13:43+08:00
keywords: []
description: ""
tags: [
    "不得不说的故事"
]
categories: [
    "杂货铺"
]
autoCollapseToc: true
author: "yuanzx"
---

>本文基于 Jdk 1.8

# 1 什么是 HashMap

要理解这个问题，首先需要理解什么是 Hash 以及什么是 Map

## 1.1 什么是 Hash

Hash 是一种散列算法，用于确定关键字到指定位置的对应关系

## 1.2 什么是 Map

Map 是 Java 里面的一个接口，接口规定以键值对的形式存储数据，其中键不能重复。HashMap 实现了 Map 接口。

# 2 为什么要使用 HashMap

原因就是速度快，HashMap 在没有出现 Hash 冲突时，查找的时间复杂度为 O(1)，如果出现了 Hash 冲突，时间复杂度下降为 O(n)。

# 3 HashMap 的实现细节

## 3.1 首先看一下 HashMap 的继承关系

![HashMap 的继承与实现结构](/media/hovel/4.png)

## 3.2 HashMap 使用的数据结构

HashMap 中使用数组的形式存储数据，使用链表的形式解决 Hash 冲突，以下为源码

```java
    // 数据存储在 Node 中，Node 使用数组存储
    transient Node<K,V>[] table;

    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        // 出现 Hash 冲突时使用 next
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

# 4 零零碎碎的知识点

## 4.1 Key 和 Value 是否可以为 null

Key 和 Value 均可以为 null，其中如果 Key 为 null，那么这一组键值对会被存储在数组下标为 0 的位置，以下是生成 hash 的代码。

```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

## 4.2 HashMap 中影响性能的两个参数

### 4.2.1 初始容量

1. 如果未指定初始容量，那么 HashMap 同样会使用 16 作为最小初始容量。
2. 如果指定容量大于 2^30，那么 HashMap 会使用 2^30 次方作为初始容量。
3. 如果指定容量，那么初始化的容量为大于等于输入参数且最近的 2 的整数次幂的数。
   - 比如输入 10，初始化容量为 16。输入 3 初始化容量为 4。

以下为源码：

```java
    static final int MAXIMUM_CAPACITY = 1 << 30;

    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * 返回的 n 为大于等于 cap 且最近的 2 的整数次幂的数
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

#### 4.2.1.1 关于 tableSizeFor 方法的执行过程

简单讲解以下 >>> 与 |，>>> 为无符号右移，| 为或，有一取一，即 0 | 1 = 1， 1 | 1 = 1， 0 | 0 = 0。 

先举个例子：

```
假如输入数字 10，那么它的二进制为 1010

int n = cap - 1;    1001
n |= n >>> 1;       1101
n |= n >>> 2;       1111
n |= n >>> 4;       1111
n |= n >>> 8;       1111
n |= n >>> 16;      1111    
return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;    n = 1111 + 1 = 0b10_000 

最终就取到了 16，这里总共进行了 5 次无符号右移，可以将最高位为 1 剩下 31 位的位置全部置为 1，
而 int 型总共 32 位，所以包含了所有的可能性。
```

### 4.2.2 负载因子

负载因子默认值为 0.75，如果 HashMap 中的存储的数达到容量 * 0.75 的话，那么 HashMap 会进行自动扩容，容量会扩大两倍，并且重新进行 rehash。

## 4.3 关于 table 为什么要设置为 transient

1. transient 表明该数据不参与序列化，因为 HashMap 中的数组可能还有很多空间没有被使用，那么这些空间没有被虚拟化的必要，所以 HashMap 重写了 `private void writeObject(java.io.ObjectOutputStream s)` 方法，只虚拟化需要存储的数据。
2. 由于不同虚拟机对于相同 Object 产生的 HashCode 是不一样的（这是由于 Object 对象的 hash 方法是 native，不同虚拟机的实现可能会不同），那么在使用默认反序列化之后，元素位置会保持与之前相同，这是不合理的，所以 HashMap 中重写了 `private void readObject(java.io.ObjectInputStream s)` 方法，使用自定义话的反序列化。

## 4.4 Java 8 对于链表的优化

当链表长度超过 8 时，将链表转化为红黑树存储，链表的时间复杂度为 O(n)，红黑树的时间复杂度为 O(lgn)。

## 4.5 HashMap 是在什么时候初始化数组的

在第一次 put 操作的时候

## 4.6 HashMap 的初始化长度为什么为 16

这是为了降低 Hash 冲突，只要是 2 的幂次都能降低 Hash 冲突。

## 4.7 高并发场景下 HashMap 可能会出现什么问题

在高并发的场景下 HashMap 可能会出现链表环，如果 get() 操作查找到这个链表上面，则会出现死循环。

## 4.8 一般使用什么类型的元素作为 HashMap Key？

一般使用 String、Integer 这样的不可变类作为 key，因为这些类已经很规范的重写了 hashCode() 和 equals() 方法。

### 4.8.1 为什么要使用不可变类作为 Key？

如果不使用不可变类，那么在数值存储进 HashMap 之后，里面的数值内容被修改了，那么相同的 Key 就无法再命中这个目标，也就失去了 Key 的意义。

### 4.8.2 为什么要重写 hashCode() 方法和 equals() 方法

// 待完成

# 参考资料

1. [漫画：什么是 HashMap？](https://zhuanlan.zhihu.com/p/31610616)
2. [漫画：高并发下的 HashMap](https://zhuanlan.zhihu.com/p/31614195)
3. [Java8 HashMap 之 tableSizeFor](https://www.cnblogs.com/loading4/p/6239441.html)


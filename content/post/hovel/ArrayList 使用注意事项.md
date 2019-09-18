---
title: "ArrayList 使用注意事项"
date: 2019-09-17T18:24:57+08:00
keywords: []
description: ""
tags: [

]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 ArrayList 使用注意事项

如果一边对 ArrayList 中的元素做修改，一边使用 Iterator 对 ArrayList 进行遍历，则会抛出 ConcurrentModificationException 异常。

# 2 为什么会出现 ConcurrentModificationException 这个异常

通过翻看源码，能够找到下面这段代码，可以发现，在通过 Iterator 对 ArrayList 进行遍历之前，每一次我们调用 next() 方法，next() 方法都会调用 checkForComodification() 方法，而在 checkForComodification() 方法中会检查 expectedModCount 与 modCount 是否相等，如果不相等则抛出异常，expectedModCount 这个字段的初始化是在创建 Iterator 的时候创建的，相当于当时 modCount 的一个快照。

```java
    private class Itr implements Iterator<E> {
        ...
        int expectedModCount = modCount;
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

如果我们在遍历的同时调用 ArrayList 的 add、remove 之类的方法，那么方法内部会对 modCount 进行 +1 操作，然后就出现了 modCount != expectedModCount，随后便抛出 ConcurrentModificationException 异常。

# 3 如何解决这个问题

Java 中已经提供了现成的工具类 CopyOnWriteList，当我们如果需要在并发时使用 List，可以使用这个类，这个类中每次在修改时会进行加锁，并且在修改时将数组中的类进行内存拷贝，变成一个新的数组，而如果有正在迭代的代码迭代的还是之前的那个数组，通过这个方式解决掉了这个问题。

# 参考资料

1. [Java 集合系列 04 之 fail-fast 总结 (通过 ArrayList 来说明 fail-fast 的原理、解决办法)](https://www.cnblogs.com/skywang12345/p/3308762.html)
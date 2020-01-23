---
title: "Netty 对于 Selector 的 KeySet 的优化"
date: 2020-01-23T21:56:44+08:00
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

在使用 Jdk Nio 的时候，一般我们可能就直接使用 `SelectorProvider.provider().openSelector()` 来获取一个 Selector 便直接开始使用了，但是，netty 对于 Selector 竟然也有优化操作。

Selector 默认的 KeySet 的实现方式是 HashSet，HashSet 由于底层使用的是 HashMap，而 HashMap 的底层是数组 + 链表，当出现冲突的时候可能会上升到 O(n) 的复杂度，也就是全部冲突到一条链表上面去了。Netty 这里便对此问题做了优化，通过将 HashSet 替换成自己实现的 Set 来解决这个问题，Netty 为这个问题实现的 Set 是一个基于数组的 Set，其实就是在数组外面套了层壳改名叫 Set 而已，所以它的性能可以做到 O(1)，下面来看看具体代码。

注意：Selector 中的 SelectionKey 在 select 出来之后便是放到该 KeySet 中

```java
// 其中的 AccessController.doPrivileged 大家不熟悉没有关系，这里涉及到 Java 中的安全机制
// 只需要当代码会同步执行到 run 方法内就好了
public final class NioEventLoop extends SingleThreadEventLoop {
    ...
    private SelectorTuple openSelector() {
        final Selector unwrappedSelector;
        try {
            // 通过 JDK 的 provider 去获取一个 selector
            unwrappedSelector = provider.openSelector();
        } catch (IOException e) {
            throw new ChannelException("failed to open a new selector", e);
        }

        // 这里 netty 对 selector 有优化默认 DISABLE_KEY_SET_OPTIMIZATION 为 false
        // 即开启优化，那么便不会进入 if
        if (DISABLE_KEY_SET_OPTIMIZATION) {
            return new SelectorTuple(unwrappedSelector);
        }

        Object maybeSelectorImplClass = AccessController.doPrivileged(new PrivilegedAction<Object>() {
            @Override
            public Object run() {
                try {
                    return Class.forName(
                            "sun.nio.ch.SelectorImpl",
                            false,
                            PlatformDependent.getSystemClassLoader());
                } catch (Throwable cause) {
                    return cause;
                }
            }
        });

        if (!(maybeSelectorImplClass instanceof Class) ||
                // ensure the current selector implementation is what we can instrument.
                !((Class<?>) maybeSelectorImplClass).isAssignableFrom(unwrappedSelector.getClass())) {
            if (maybeSelectorImplClass instanceof Throwable) {
                Throwable t = (Throwable) maybeSelectorImplClass;
                logger.trace("failed to instrument a special java.util.Set into: {}", unwrappedSelector, t);
            }
            return new SelectorTuple(unwrappedSelector);
        }

        final Class<?> selectorImplClass = (Class<?>) maybeSelectorImplClass;
        // 具体的优化方式在这里，通过自定义的 SelectionKeySet 来替换默认的 SelectionKeySet
        // 自定义的实现是使用数组实现的，add 方法的复杂度为 O(1)，而默认的使用 HashSet，add
        // 方法的复杂度最高可能为 O(n)
        final SelectedSelectionKeySet selectedKeySet = new SelectedSelectionKeySet();

        Object maybeException = AccessController.doPrivileged(new PrivilegedAction<Object>() {
            @Override
            public Object run() {
                try {
                    Field selectedKeysField = selectorImplClass.getDeclaredField("selectedKeys");
                    Field publicSelectedKeysField = selectorImplClass.getDeclaredField("publicSelectedKeys");

                    if (PlatformDependent.javaVersion() >= 9 && PlatformDependent.hasUnsafe()) {
                        // Let us try to use sun.misc.Unsafe to replace the SelectionKeySet.
                        // This allows us to also do this in Java9+ without any extra flags.
                        long selectedKeysFieldOffset = PlatformDependent.objectFieldOffset(selectedKeysField);
                        long publicSelectedKeysFieldOffset =
                                PlatformDependent.objectFieldOffset(publicSelectedKeysField);

                        if (selectedKeysFieldOffset != -1 && publicSelectedKeysFieldOffset != -1) {
                            PlatformDependent.putObject(
                                    unwrappedSelector, selectedKeysFieldOffset, selectedKeySet);
                            PlatformDependent.putObject(
                                    unwrappedSelector, publicSelectedKeysFieldOffset, selectedKeySet);
                            return null;
                        }
                        // We could not retrieve the offset, lets try reflection as last-resort.
                    }

                    Throwable cause = ReflectionUtil.trySetAccessible(selectedKeysField, true);
                    if (cause != null) {
                        return cause;
                    }
                    cause = ReflectionUtil.trySetAccessible(publicSelectedKeysField, true);
                    if (cause != null) {
                        return cause;
                    }
                    // 在这里使用自定义的 selectedKeySet 来替换 unwrappedSelector 中
                    // 默认的 selectedKeysField 和 publicSelectedKeysField 实现
                    selectedKeysField.set(unwrappedSelector, selectedKeySet);
                    publicSelectedKeysField.set(unwrappedSelector, selectedKeySet);
                    return null;
                } catch (NoSuchFieldException e) {
                    return e;
                } catch (IllegalAccessException e) {
                    return e;
                }
            }
        });

        if (maybeException instanceof Exception) {
            selectedKeys = null;
            Exception e = (Exception) maybeException;
            logger.trace("failed to instrument a special java.util.Set into: {}", unwrappedSelector, e);
            return new SelectorTuple(unwrappedSelector);
        }
        selectedKeys = selectedKeySet;
        logger.trace("instrumented a special java.util.Set into: {}", unwrappedSelector);
        return new SelectorTuple(unwrappedSelector,
                new SelectedSelectionKeySetSelector(unwrappedSelector, selectedKeySet));
    }
}
```

下面是优化的 Set 的实现

```java
final class SelectedSelectionKeySet extends AbstractSet<SelectionKey> {
    /** 可以看到其实现为数组 */
    SelectionKey[] keys;
    int size;

    SelectedSelectionKeySet() {
        keys = new SelectionKey[1024];
    }

    @Override
    public boolean add(SelectionKey o) {
        if (o == null) {
            return false;
        }
        // add 时按顺序添加即可
        keys[size++] = o;
        if (size == keys.length) {
            // 扩容
            increaseCapacity();
        }

        return true;
    }

    @Override
    public boolean remove(Object o) {
        // 用不上这个方法所以直接返回 false
        return false;
    }

    @Override
    public boolean contains(Object o) {
        // 用不上这个方法所以直接返回 false
        return false;
    }

    @Override
    public int size() {
        return size;
    }

    @Override
    public Iterator<SelectionKey> iterator() {
        return new Iterator<SelectionKey>() {
            private int idx;

            @Override
            public boolean hasNext() {
                return idx < size;
            }

            @Override
            public SelectionKey next() {
                if (!hasNext()) {
                    throw new NoSuchElementException();
                }
                return keys[idx++];
            }

            @Override
            public void remove() {
                // 用不上这个方法所以直接返回 false
                throw new UnsupportedOperationException();
            }
        };
    }

    void reset() {
        reset(0);
    }

    void reset(int start) {
        Arrays.fill(keys, start, size, null);
        size = 0;
    }

    private void increaseCapacity() {
        SelectionKey[] newKeys = new SelectionKey[keys.length << 1];
        System.arraycopy(keys, 0, newKeys, 0, size);
        keys = newKeys;
    }
}
```

下面是 Selector 中的默认实现

```java
public abstract class SelectorImpl extends AbstractSelector {
    // The set of keys with data ready for an operation
    protected Set<SelectionKey> selectedKeys;

    // Removal allowed, but not addition
    private Set<SelectionKey> publicSelectedKeys;     

    protected SelectorImpl(SelectorProvider sp) {
        super(sp);
        keys = new HashSet<SelectionKey>();
        // 可以看到这里 selectedKeys 会被赋值为 HashSet
        selectedKeys = new HashSet<SelectionKey>();
        if (Util.atBugLevel("1.4")) {
            publicKeys = keys;
            // 可以看到这里 publicSelectedKeys 会被赋值为 selectedKeys
            // selectedKeys 便是 HashSet
            publicSelectedKeys = selectedKeys;
        } else {
            publicKeys = Collections.unmodifiableSet(keys);
            publicSelectedKeys = Util.ungrowableSet(selectedKeys);
        }
    }
}
```
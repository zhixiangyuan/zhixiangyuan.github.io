---
title: "Netty 如何解决 JDK 空轮训的 BUG"
date: 2020-01-23T21:21:31+08:00
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

Netty 解决空轮训的 bug 的方式很简单，通过执行阻塞 select 的方式阻塞一段时间，如果在执行完毕之后阻塞的时间小于设定的阻塞的时间，那么 selectCnt 便 +1，当达到一定次数的时候 netty 便认为出现了空轮训的 bug，然后触发重建 selector 的操作，将老得的 SelectionKey 上面的 channel 注册到新的 Selector 上面，这便是 netty 的解决方法，下面看一下具体的源码。

```java
// 以下源码省略不重要的部分
public final class NioEventLoop extends SingleThreadEventLoop {
    ...
    private void select(boolean oldWakenUp) throws IOException {
        Selector selector = this.selector;
        try {
            // 初始化 selectCnt
            int selectCnt = 0;
            // 初始化当前时间
            long currentTimeNanos = System.nanoTime();
            // 当前 select 的操作的截止时间不能超过 selectDeadLineNanos 的时间
            long selectDeadLineNanos = currentTimeNanos + delayNanos(currentTimeNanos);

            for (; ; ) {
                long timeoutMillis = (selectDeadLineNanos - currentTimeNanos + 500000L) / 1000000L;
                ...
                // 阻塞式的 select 操作
                int selectedKeys = selector.select(timeoutMillis);
                // 每次 select 之后 selectCnt 便会 +1
                selectCnt++;
                ... 
                long time = System.nanoTime();
                // time 是当前时间，currentTimeNanos 是 select 之前的时间
                // 此处 time - currentTimeNanos 表示此次执行的 select 的时间
                if (time - currentTimeNanos >= TimeUnit.MILLISECONDS.toNanos(timeoutMillis)) {
                    // 如果此次 select 的时间超过了阻塞时间，那么则初始化 selectCnt 为 1
                    selectCnt = 1;
                } else if (SELECTOR_AUTO_REBUILD_THRESHOLD > 0 &&
                        // 这里可以看到 selectCnt 的次数如果超过 SELECTOR_AUTO_REBUILD_THRESHOLD 
                        // 则说明触发了空轮训的 bug
                        // 这里 SELECTOR_AUTO_REBUILD_THRESHOLD 的值默认为 512
                        selectCnt >= SELECTOR_AUTO_REBUILD_THRESHOLD) {
                    // 如果触发空轮训的 bug，那么则调用 selectRebuildSelector 重建 selector
                    selector = selectRebuildSelector(selectCnt);
                    selectCnt = 1;
                    break;
                }
                // 如果 select 的时间小于超时时间则不重置 selectCnt，继续向下开始下一次循环
                // 那么 selectCnt 随着多次 +1，那么便会最终触发空轮训 bug

                currentTimeNanos = time;
            }

            if (selectCnt > MIN_PREMATURE_SELECTOR_RETURNS) {
                if (logger.isDebugEnabled()) {
                    logger.debug("Selector.select() returned prematurely {} times in a row for Selector {}.",
                            selectCnt - 1, selector);
                }
            }
        } catch (CancelledKeyException e) {
            if (logger.isDebugEnabled()) {
                logger.debug(CancelledKeyException.class.getSimpleName() + " raised by a Selector {} - JDK bug?",
                        selector, e);
            }
            // Harmless exception - log anyway
        }
    }
}
```

下面再看一下重建 selector 的代码

```java
public final class NioEventLoop extends SingleThreadEventLoop {
    ...
    private void rebuildSelector0() {
        final Selector oldSelector = selector;
        final SelectorTuple newSelectorTuple;

        if (oldSelector == null) {
            return;
        }

        try {
            // 获取新的 selector
            newSelectorTuple = openSelector();
        } catch (Exception e) {
            logger.warn("Failed to create a new Selector.", e);
            return;
        }

        // Register all channels to the new Selector.
        int nChannels = 0;
        // 对老的 oldSelector 的 keys 进行遍历
        for (SelectionKey key : oldSelector.keys()) {
            // 获取其上的 attachment 
            Object a = key.attachment();
            try {
                if (!key.isValid() || key.channel().keyFor(newSelectorTuple.unwrappedSelector) != null) {
                    continue;
                }
                // 获取其感兴趣的事件
                int interestOps = key.interestOps();
                key.cancel();
                // 将老的 key 上面的 channel 注册到新的 selector 上面，附上其感兴趣的事件 和 attachment
                SelectionKey newKey = key.channel().register(newSelectorTuple.unwrappedSelector, interestOps, a);
                if (a instanceof AbstractNioChannel) {
                    // Update SelectionKey
                    ((AbstractNioChannel) a).selectionKey = newKey;
                }
                nChannels++;
            } catch (Exception e) {
                logger.warn("Failed to re-register a Channel to the new Selector.", e);
                if (a instanceof AbstractNioChannel) {
                    AbstractNioChannel ch = (AbstractNioChannel) a;
                    ch.unsafe().close(ch.unsafe().voidPromise());
                } else {
                    @SuppressWarnings("unchecked")
                    NioTask<SelectableChannel> task = (NioTask<SelectableChannel>) a;
                    invokeChannelUnregistered(task, key, e);
                }
            }
        }
        
        // 将新的 selector 赋值给老的 selector
        selector = newSelectorTuple.selector;
        // 将未包装的新的 selector 赋值给老的未包装的 selector
        unwrappedSelector = newSelectorTuple.unwrappedSelector;

        try {
            // time to close the old selector as everything else is registered to the new one
            oldSelector.close();
        } catch (Throwable t) {
            if (logger.isWarnEnabled()) {
                logger.warn("Failed to close the old Selector.", t);
            }
        }

        if (logger.isInfoEnabled()) {
            logger.info("Migrated " + nChannels + " channel(s) to the new Selector.");
        }
    }
}
```
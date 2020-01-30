---
title: "Netty 中从 PooledByteBufAllocator 分配直接内存的逻辑"
date: 2020-01-30T15:38:21+08:00
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

从 `PooledByteBufAllocator` 中分配直接内存会调用 `newDirectBuffer` 方法，该方法会先从 `ThreadLocal` 中取出 `PoolThreadCache`，如果取得时候没有则创建一个新的 `PoolThreadCache`，创建的时候会在其中设置各种参数，其中有两个很重要的参数，分别是 `heapArena` 和 `directArena`，这两个参数是从 `PooledByteBufAllocator` 中维护的 `heapArenas` 和 `directArenas` 中取出的，取出的是其中被线程最少使用的 `arena`，这里判断被线程最少使用是通过 `arena` 中的 `numThreadCaches` 字段判断的，该字段在每次被一个新的线程使用的时候会加一，被释放时减一。

取出线程自己的 `PoolThreadCache` 后然后从 `PoolThreadCache` 取出 `directArena`，接着通过 `directArena` 开始分配内存，分配内存的过程首先从对象池中取出一个 `PooledUnsafeDirectByteBuf`，然后去申请一块内存，接着将该内存对应的 `chunk`、偏移量、申请容量、最大容量等参数放到 `PooledUnsafeDirectByteBuf` 中，接下来 `PooledUnsafeDirectByteBuf` 便可以使用了。

接下来来聊申请内存的过程，申请内存时需要判断内存大小是 `tiny`、`small`、`小于 chunkSize` 和 `大于 chunkSize`，这里先聊内存是 `小于 chunkSize` 的部分。对于该部分的处理，进入申请内存的过程首先其内部有几个 `PoolChunkList`，这个 `PoolChunkList` 有两个作用一个是内部以链表的形式保存了存放在里面的 `chunk`，第二个是存放了当前 `chunk` 的使用百分比。这里的的队列分别是 `qInit`、`q000`、`q025`、`q050`、`q075`、`q100`，队列的使用率如下图所示。

![](/hub/2020/January/50.png)

首先在分配内存的时候会从 `PoolThreadCache` 中缓存的 `MemoryRegionCache` 数组去寻找是否有被缓存的可以分配的内存块，如果有则直接分配，如果没有则去上述的队列中找是否有可以使用的 `chunk`，如果有 `chunk` 则直接分配内存，如果没有则申请一个 `chunk` 然后进行分配，最后加入到队列中。这里面 `chunk` 管理自己的内存是使用完全二叉树来管理，不过完全二叉树只管理大于 `8K` 大小的内存，它的 `page` 最大是 `8K`，再小便要由 `subpage` 来管理，每一个 `page` 对应着一个 `subpage`，`subpage` 内部使用 `long[]` 来管理自己的内存。

对于 `tiny`、`small` 类型的内存块其实也是一样的，只不过稍有不同的是它内部管理内存的类是 `subpage` 而不是 `page`。

如果申请的是 `大于 chunkSize` 的内存，那么就直接没有上面那些麻烦的逻辑了，他就是直接申请一块足够大的 `chunk` 然后初始化到 `ByteBuf` 中返回给使用者使用。
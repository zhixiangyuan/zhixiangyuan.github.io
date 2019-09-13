---
title: "Leetcode: 146 LRU 缓存机制"
date: 2019-09-13T09:20:52+08:00
keywords: []
description: ""
tags: [
    "leetcode"
]
categories: [
    "数据结构与算法"
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 题目

[LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache/submissions/)

# 2 解

## 2.1 我的解

没解出来

## 2.2 官方解法

厉害了，对 LinkedhashMap 稍加改造就完成了这个效果，强

LRU = Last recent used，最近被使用的缓存不要被覆盖掉。

```java
class LRUCache extends LinkedHashMap<Integer, Integer> {

    private int capacity;
    
    public LRUCache(int capacity) {
        // 这里的第三个参数 true 非常关键
        // 这个 true 打开了 LinkedHashMap 的
        // 隐藏功能，每次 get 一个节点的数
        // 的时候，将这个节点移动到链表的
        // 最后一位
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }
    
    public int get(int key) {
        return super.getOrDefault(key, -1);
    }
    
    public void put(int key, int value) {
        super.put(key, value);
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        // 这里控制着什么时候删除链表头节点
        // 因为只要 get 过的节点都在后边
        return size() > capacity;
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```
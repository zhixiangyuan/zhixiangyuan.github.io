---
title: "Leetcode: 151 最小栈"
date: 2019-09-22T07:43:01+08:00
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

[最小栈](https://leetcode-cn.com/problems/min-stack/)

# 2 解

## 2.1 我的解

题意要求的常数时间 getMin，只需要在程序中添加一个最小值成员变量，在每次 push 时判断是不是更小，若更小则将最小值更新，同时在 pop 时需要判断 pop 的值是否与最小值相等，如果相等，则在 pop 时需要遍历链表找出第二最小值，然后更新最小值，这样 getMin 时直接返回最小值即可。

```java
class MinStack {
    
    private LinkedList<Integer> stack;

    /** initialize your data structure here. */
    public MinStack() {
        stack = new LinkedList();
    }
    
    public void push(int x) {
        stack.push(x);
    }
    
    public void pop() {
        if (stack.size() == 0) return;
        stack.pop();
    }
    
    public int top() {
        if (stack.size() == 0) return 0;
        return stack.element();
    }
    
    public int getMin() {
        if (stack.size() == 0){
            return 0;
        }
        int min = stack.get(0);
        for (Integer num : stack) {
            min = Math.min(min, num);
        }
        return min;
    }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(x);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.getMin();
 */
```
---
title: "Leetcode: 141 环形链表"
date: 2019-09-21T09:53:26+08:00
keywords: []
description: ""
tags: [
    "leetcode"
]
categories: [
]
autoCollapseToc: false
author: "yuanzx"
markup: mmark
mathjax: true  
---

# 1 题目

[环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

# 2 解

## 2.1 我的解

- 时间复杂度: O(n)
- 空间负载的: O(n)

```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public boolean hasCycle(ListNode head) {
        HashSet hashSet = new HashSet();
        ListNode index = head;
        while(index != null) {
            if (hashSet.contains(index)) {
                return true;
            } else {
                hashSet.add(index);
                index = index.next;
            }
        }
        return false;
    }
}
```

## 2.2 官方的解

- 时间复杂度: O(n)
- 空间复杂度: O(1)

运用快慢指针，如果是没有环，快指针必然出现 fast.next == null，如果有环，就能跑出 slow == fast。

```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) {
            return false;
        }
        ListNode slow = head;
        ListNode fast = head.next;
        while (slow != fast) {
            if (fast.next == null || fast.next.next == null) {
                return false;
            }
            slow = slow.next;
            fast = fast.next.next;
        }
        return true;
    }
}
```
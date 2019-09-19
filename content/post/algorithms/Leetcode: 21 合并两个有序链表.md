---
title: "Leetcode: 21 合并两个有序链表"
date: 2019-09-19T07:54:04+08:00
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
markup: mmark
mathjax: true 
---

# 1 题目

[合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

# 2 解

## 2.1 我的解

最近什么情况，为什么写出来的解又臭又长，可能是我又想写递归，又写不出简洁的递归的原因吧。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null && l2 == null) {
            return null;
        } else if (l1 == null) {
            return l2;
        } else if (l2 == null) {
            return l1;
        } else {
            ListNode node = null;
            if (l1.val > l2.val) {
                node = l2;
                ListNode nextL2 = l2.next;
                node.next = null;
                mergeTwoLists(l1, nextL2, node);
            } else if (l1.val < l2.val) {
                node = l1;
                ListNode nextL1 = l1.next;
                node.next = null;
                mergeTwoLists(nextL1, l2, node);
            } else if (l1.val == l2.val) {
                node = l2;
                ListNode nextL2 = l2.next;
                node.next = l1;
                ListNode nextL1 = l1.next;
                node.next.next = null;
                mergeTwoLists(nextL1, nextL2, node.next);
            }
            return node;
        }
        
    }
    
    private void mergeTwoLists(ListNode l1, ListNode l2, ListNode node) {
        if (l1 == null && l2 == null) {
            return;
        } else if (l1 == null) {
            node.next = l2;
            ListNode nextL2 = l2.next;
            l2.next = null;
            mergeTwoLists(l1, nextL2, node.next);
        } else if (l2 == null) {
            node.next = l1;
            ListNode nextL1 = l1.next;
            l1.next = null;
            mergeTwoLists(nextL1, l2, node.next);
        } else {
            if (l1.val > l2.val) {
                node.next = l2;
                ListNode nextL2 = l2.next;
                l2.next = null;
                mergeTwoLists(l1, nextL2, node.next);
            } else if (l1.val < l2.val) {
                node.next = l1;
                ListNode nextL1 = l1.next;
                l1.next = null;
                mergeTwoLists(nextL1, l2, node.next);      
            } else {
                node.next = l1;
                ListNode nextL1 = l1.next;
                node.next.next = l2;
                ListNode nextL2 = l2.next;
                node.next.next.next = null;
                mergeTwoLists(nextL1, nextL2, node.next.next);
            }
        }
    }
}
```

## 2.2 官方题解

### 2.2.1 递归

- 时间复杂度: O(n + m)，遍历所有节点
- 空间复杂度: O(n + m)，每一个节点占一个栈贞

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        }
        else if (l2 == null) {
            return l1;
        }
        else if (l1.val < l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        }
        else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }
    }
}
```

### 2.2.2 迭代

- 时间复杂度: O(n + m)，遍历所有节点
- 空间复杂度: O(1)

这类题目用迭代写比用递归写好，毕竟空间复杂度为 O(1)，如果用递归那么在链表很长的情况下搞不好会出问题。

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        // maintain an unchanging reference to node ahead of the return node.
        ListNode prehead = new ListNode(-1);

        ListNode prev = prehead;
        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) {
                prev.next = l1;
                l1 = l1.next;
            } else {
                prev.next = l2;
                l2 = l2.next;
            }
            prev = prev.next;
        }

        // exactly one of l1 and l2 can be non-null at this point, so connect
        // the non-null list to the end of the merged list.
        prev.next = l1 == null ? l2 : l1;

        return prehead.next;
    }
}
```
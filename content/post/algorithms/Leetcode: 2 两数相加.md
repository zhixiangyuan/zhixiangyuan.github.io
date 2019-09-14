---
title: "Leetcode: 2 两数相加"
date: 2019-09-14T09:14:06+08:00
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

[两数相加](https://leetcode-cn.com/problems/add-two-numbers/submissions/)

# 2 解

## 2.1 我的解

- 时间复杂度: O(n)
- 空间复杂度: O(n)

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 * int val;
 * ListNode next;
 * ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {

        boolean addOne = false;
        ListNode head = null;
        ListNode index = null;

        while (l1 != null || l2 != null) {
            int num;
            if (l1 == null) {
                num = l2.val;
                l2 = l2.next;
            } else if (l2 == null) {
                num = l1.val;
                l1 = l1.next;
            } else {
                num = l1.val + l2.val;
                l1 = l1.next;
                l2 = l2.next;
            }
            if (addOne) {
                num++;
                addOne = false;
            }
            if (num >= 10) {
                addOne = true;
                num = num - 10;
            }
            ListNode node = new ListNode(num);
            if (index == null) {
                index = node;
                head = node;
            } else {
                index.next = node;
                index = node;
            }
        }
        if (addOne) {
            index.next = new ListNode(1);
        }
        return head;
    }
}
```

## 2.2 官方解法

官方解法的思路和我的写法是相同的，不过官方写法运用三目表达式使得计算过程变得简洁，而我的解法代码冗长，看起来很啰嗦。

- 时间复杂度: O(max(m,n))
- 空间复杂度: O(max(m,n))

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummyHead = new ListNode(0);
    ListNode p = l1, q = l2, curr = dummyHead;
    int carry = 0;
    while (p != null || q != null) {
        int x = (p != null) ? p.val : 0;
        int y = (q != null) ? q.val : 0;
        int sum = carry + x + y;
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
        if (p != null) p = p.next;
        if (q != null) q = q.next;
    }
    if (carry > 0) {
        curr.next = new ListNode(carry);
    }
    return dummyHead.next;
}
```
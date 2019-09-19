---
title: "如何在 Leetcode 上调试代码"
date: 2019-09-19T09:39:08+08:00
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

最近经常在 leetcode 上面刷题，不过发现在上面写的代码如果遇到一些 bug 只能肉眼 debug，如果 不能肉眼 debug 出来就比较难受了，还要打开 ide 真实模拟一下。不过今天发现一个小技巧，虽然 leetcode 上面不能够实现命令行打印内容，但是可以打印错误的异常日志，所以我们可以利用一下这个异常日志输出我们想看到的内容，比如说对于 [合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/) 这道题，如果我们想输出点东西可以想像下面这段代码一样。

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        print("test");
        if (l1 == null) {
            return l2;
        } else if (l2 == null) {
            return l1;
        } else if (l1.val < l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        } else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        } 
    }
    
    public void print(String str) {
        throw new CustomException(str);
    }
    
    class CustomException extends RuntimeException {
        private String message;
        public CustomException(String message) {
            this.message = message;
        }
        @Override
        public String toString() {
            return message;
        }
    }
}
```

关键代码如下

```java
    public void print(String str) {
        throw new CustomException(str);
    }
    
    class CustomException extends RuntimeException {
        private String message;
        public CustomException(String message) {
            this.message = message;
        }
        @Override
        public String toString() {
            return message;
        }
    }
```

所以以后如果想打印点东西，把这段代码 copy 过去就可以实现打印了。
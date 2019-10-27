---
title: "Leetcode: 27 移除元素"
date: 2019-09-28T07:57:58+08:00
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

[移除元素](https://leetcode-cn.com/problems/remove-element/)

# 2 解

## 2.1 我的解

- 时间复杂度: O(n)
- 空间复杂度: O(1)

```java
class Solution {
    public int removeElement(int[] nums, int val) {
        int newIndex = 0;
        int oldIndex = 0;
        for (; oldIndex < nums.length; oldIndex++) {
            if (val != nums[oldIndex])  {
                nums[newIndex] = nums[oldIndex];
                newIndex++;
            }
        }
        return newIndex;
    }
}
```
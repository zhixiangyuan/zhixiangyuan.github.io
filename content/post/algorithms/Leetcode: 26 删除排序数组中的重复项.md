---
title: "Leetcode: 26 删除排序数组中的重复项"
date: 2019-09-27T08:12:28+08:00
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

[删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

# 2 解

## 2.1 我的解

- 时间复杂度: O(n)
- 空间复杂度: O(1)

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int oldIndex = 1;
        int newIndex = 1;
        
        for (; oldIndex < nums.length; oldIndex++) {
            if (nums[oldIndex - 1] != nums[oldIndex]) {
                nums[newIndex] = nums[oldIndex];
                newIndex++;
            }
        }
        return newIndex;
    }
}
```
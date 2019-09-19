---
title: "Leetcode: 136 只出现一次的数字"
date: 2019-09-20T06:45:42+08:00
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

[只出现一次的数字](https://leetcode-cn.com/problems/single-number/)

# 2 解

## 2.1 我的解

### 2.1.1 HashSet 解

- 时间复杂度: O(n)

```java
class Solution {
    public int singleNumber(int[] nums) {
        HashSet hashset = new HashSet();
        for (int i = 0; i < nums.length; i++) {
            if (hashset.contains(nums[i])) {
                hashset.remove(nums[i]);
            } else {
                hashset.add(nums[i]);
            }
        }
        return (int) hashset.toArray()[0];
    }
}
```

### 2.2 官方的解

### 2.2.1 异或解

简直骚的不行

如果我们对 0 和二进制位做 XOR 运算，得到的仍然是这个二进制位

$$a \oplus 0 = a$$

如果我们对相同的二进制位做 XOR 运算，返回的结果是 0

$$a \oplus a = 0$$

XOR 满足交换律和结合律

$$a \oplus b \oplus a = (a \oplus a) \oplus b = 0 \oplus b = b$$

所以我们只需要将所有的数进行 XOR 操作，得到那个唯一的数字

- 时间复杂度: O(n)
- 空间复杂度: O(1)

```java
class Solution {
    public int singleNumber(int[] nums) {
        int result = 0;
        for (int i = 0; i < nums.length; i++) {
            result = result ^ nums[i];
        }
        return result;
    }
}
```
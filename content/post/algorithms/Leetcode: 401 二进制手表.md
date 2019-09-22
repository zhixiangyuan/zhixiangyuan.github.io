---
title: "Leetcode: 401 二进制手表"
date: 2019-09-23T07:20:33+08:00
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

[二进制手表](https://leetcode-cn.com/problems/binary-watch/)

# 2 解

## 2.1 别人的解

很巧妙的解法

```java
class Solution {
    public List<String> readBinaryWatch(int num) {
       List<String> result = new ArrayList<>();
        // 直接遍历  0:00 -> 12:00   每个时间有多少 1
        for (int i = 0; i < 12; i++) {
            for (int j = 0; j < 60; j++) {
                if (count1(i) + count1(j) == num) {
                    result.add(i + ":" + (j < 10 ? "0" + j : j));
                }
            }
        }
        return result;
    }
    // 计算二进制中 1 的个数
    // 这种写法是固定写法，可以统计出
    // int 数中二进制形式 1 的个数
    private int count1(int n) {
        int result = 0;
        while(n != 0){
            n &= n - 1;
            result++;
        }
        return result;
    }
}
```
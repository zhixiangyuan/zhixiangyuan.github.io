---
title: "Leetcode: 64 最小路径和"
date: 2020-02-08T14:13:57+08:00
keywords: []
description: ""
tags: [
    "leetcode"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
markup: mmark
mathjax: false
draft: true
---

# 1 题目

[最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)

# 2 解

## 2.1 动态规划

遍历一遍数组就出答案

从最后一个格子开始遍历，一直向前加数值，直到加到 `[0,0]` 的位置，然后取出 `[0,0]` 内部存储的数据。

```java
class Solution {
    public int minPathSum(int[][] grid) {
        for (int i = grid.length - 1; i >= 0; i--) {
            for (int j = grid[0].length - 1; j >= 0; j--) {
                if (i == grid.length - 1 && j != grid[0].length - 1) {
                    grid[i][j] += grid[i][j + 1];
                } else if (i != grid.length - 1 && j == grid[0].length - 1) {
                    grid[i][j] += grid[i + 1][j];
                } else if (i != grid.length - 1 && j != grid[0].length - 1) {
                    grid[i][j] += Math.min(grid[i + 1][j], grid[i][j + 1]);
                }
            }
        }
        return grid[0][0];
    }
}
```
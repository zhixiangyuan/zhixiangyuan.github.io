---
title: "Leetcode: 6 不同路径"
date: 2020-02-07T13:30:49+08:00
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
draft: true
---

# 1 题目

[不同路径](https://leetcode-cn.com/problems/unique-paths/)

# 2 解

## 2.1 使用排列组合的解法

本题其实是一个排列组合问题，由于只能向下和向右移动，所以最终会移动 (n - 1) + (m - 1) 步，假设 n - 1 是向下移动，m - 1 是向右移动，那么计算过程就是总共移动 (n - 1) + (m - 1)，从其中挑出 n - 1 也就是向下移动的步骤的位置，然后将 m - 1 次向右移动按照顺序放置进去，计算过程就是 `C(n + m - 2, n - 1)` 或者 `C(n + m - 2, m - 1)`

下面是实现的代码：

```java
class Solution {
    public int uniquePaths(int m, int n) {
        return C(n + m - 2, n - 1);
    }

    private static int C(int n, int k) {
        return factorial(n) / (factorial(n - k) * factorial(k));
    }

    private static int factorial(int num) {
        if (num <= 1) {
            return 1;
        } else {
            return num * factorial(num - 1);
        }
    }
}
```

这个代码有一个缺陷，就是阶乘运算很容易超出 int 上限，改成 long 也很容易超，所以 leetcode 上面的示例没法完全跑过去。这里我所想的是，对其计算过程进行改造，将乘法改造成会约分的乘法，这样可能就可以跑过去了。

## 2.2 使用递归模拟走路径

这方法跑结果集会超时

```java
class Solution {
    public int uniquePaths(int m, int n) {
        return move(m, n, 0, 0);
    }

    private int move(int rightLimit, int downLimit, int curRightIndex, int curDownIndex) {
        if (curRightIndex == rightLimit - 1 && curDownIndex == downLimit - 1) {
            // 抵达终点
            return 1;
        } else if (curRightIndex >= rightLimit || curDownIndex >= downLimit) {
            // 越界
            return 0;
        } else {
            // 收集达到终点的次数
            int sum = move(rightLimit, downLimit, curRightIndex + 1, curDownIndex);
            sum += move(rightLimit, downLimit, curRightIndex, curDownIndex + 1);
            return sum;
        }
    }
}
```

## 2.3 动态规划解法

先初始化一个下面这样的表格，然后用左边的值加上上边的值就可以了

| 1   | 1   | 1   | 1   | 1   | 1   | 1   |
| --- | --- | --- | --- | --- | --- | --- |
| 1   |     |     |     |     |     |     |
| 1   |     |     |     |     |     |     |

右下角的值就是答案

| 1   | 1   | 1   | 1   | 1   | 1   | 1   |
| --- | --- | --- | --- | --- | --- | --- |
| 1   | 2   | 3   | 4   | 5   | 6   | 7   |
| 1   | 3   | 6   | 10  | 15  | 21  | 28  |

下面是计算过程

![](/hub/2020/February/1.png)

代码使用一维数组来实现

```java
class Solution {
    public int uniquePaths(int m, int n) {
        // 初始化一维数组
        int[] memo = new int[n];
        // 一维数组全部填入 1
        Arrays.fill(memo, 1);

        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                memo[j] += memo[j - 1];
            }
        }

        return memo[n - 1];
    }
}
```

# 参考文章

1. [第三种解法来源于此](https://leetcode-cn.com/problems/unique-paths/solution/xiao-xue-ti-java-by-biyu_leetcode/)
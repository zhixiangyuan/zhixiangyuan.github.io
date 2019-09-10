---
title: "Leetcode: 70 爬楼梯"
date: 2019-09-10T09:43:30+08:00
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

[爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/submissions/)

# 2 解

## 2.1 我的解

没解出来

## 2.2 官方题解

### 2.2.1 暴力法

通过递归的方式模拟爬楼梯，不过这个方法在 leetcode 上面跑会超时

- 时间复杂度: O($$2^n$$)
- 空间复杂度: O(1)

```java
class Solution {
    public int climbStairs(int n) {
        return climbStairs(0, n);
    }
    
    public int climbStairs(int current, int n) {
        if (current > n) {
            return 0;
        }
        if (current == n) {
            return 1;
        }
        return climbStairs(current + 1, n) + climbStairs(current + 2, n);
    }
}
```

这种解法关于时间复杂度的运算可以这样来算，算法在运行的时候的执行路径如下

![算法奔跑路径](/media/algorithms/9.jpeg)

如上如所知，运行路径是一个二叉树的效果，树深为 n，所以时间复杂度为 $$2^n$$，如果为三叉树，那么时间复杂度为 $$3^n$$，不过 $$2^n$$ 和 $$3^n$$ 在 n 为无穷大的时候是同一个级别的无穷大，所以如果是三叉树，时间复杂度还是为 $$2^n$$。

### 2.2.2 动态规划法

第 i 阶可以由一下两种方法得到：

1. 在第（i - 1）阶后向上爬一阶
2. 在第（i - 2）阶后向上爬二阶

所以达到第 i 阶的方法总是就是第（i - 1）阶和第（i - 2）阶方法数的和。

令 dp[i] 表示能到达第 i 阶的方法总数：

$$
dp[i] = dp[i - 1] + dp[i - 2]
$$

写成代码就是：

- 时间复杂度 O(n)
- 空间复杂度 O(n)

```java
public class Solution {
    public int climbStairs(int n) {
        if (n == 1) {
            return 1;
        }
        int[] dp = new int[n + 1];
        // 这里为了帮助理解从数组的 1 位置开始，放弃使用 0 位置
        // 不过这里的数组可以优化掉，这样空间复杂度就变成了 1
        dp[1] = 1;
        dp[2] = 2;
        for (int i = 3; i <= n; i ++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        return dp[n];
    }
}
```

不过这种思路真的是挺厉害的，要是想不到到达 第 i 阶 的所有方法 = 第 i - 1 阶的所有方法 + 第 i - 2 阶的所有方法，然后递推下来，真的是想不到这个解。

### 2.2.3 斐波那契数

在上述方法中，我们使用了 dp 数组，其中 $$dp[i] = dp[i - 1] + dp[i - 2]$$ 可以很容易通过分析得出 dp[i] 其实就是第 i 个斐波那契数

$$
Fib(n) = Fib(n - 1) + Fib(n - 2)
$$

现在我们必须找出以 1 和 2 作为第一项和第二项的斐波那契数列中的第 n 个数，也就是说 $$Fib(1) = 1$$ 且 $$Fib(2) = 2$$。

写成代码：

- 时间复杂度: O(n)
- 空间复杂度: O(1)

```java
public class Solution {
    public int climbStairs(int n) {
        if (n == 1) {
            return 1;
        }
        int first = 1;
        int second = 2;
        for (int i = 3; i <= n; i ++) {
            int third = first + second;
            first = second;
            second = third;
        }
        return second;
    }
}
```

这样子写空间复杂度就变成 O(1) 了
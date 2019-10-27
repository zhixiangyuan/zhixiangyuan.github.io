---
title: "Leetcode: 53 最大子序和"
date: 2019-09-09T08:17:52+08:00
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

[最大子序和](https://leetcode-cn.com/problems/maximum-subarray/submissions/)

# 2 解

## 2.1 我的解

双重循环遍历，把所有可能性都算一遍

- 时间复杂度: O($$n^2$$)
- 空间复杂度: O(1)

执行时间: 95ms

```java
class Solution {
    public int maxSubArray(int[] nums) {
        if (nums.length == 0) return 0;
        else if (nums.length == 1) return nums[0];

        int maxSum = nums[0];
        for (int i = 0; i < nums.length; i++) {
            int tempSum = nums[i];
            if (tempSum > maxSum) {
                maxSum = tempSum;
            }
            for (int j = i + 1; j < nums.length; j++) {
                tempSum += nums[j];
                if (tempSum > maxSum) {
                    maxSum = tempSum;
                }
            }
        }
        return maxSum;
    }
}
```

## 2.2 动态规划解法

1. 如果 `sum > 0`，则说明 sum 对结果有增益效果，则 sum 保留并加上当前遍历数字
2. 如果 `sum <= 0`，则说明 sum 对结果无增益效果，需要舍弃，则 sum 直接更新为当前遍历数字
3. 每次比较 sum 和 ans 的大小，将最大值置为 ans，遍历结束返回结果

- 时间复杂度: O(n)
- 空间复杂度: O(1)

执行时间: 2ms

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int ans = nums[0];
        int sum = 0;
        for(int num: nums) {
            if(sum > 0) {
                sum += num;
            } else {
                sum = num;
            }
            ans = Math.max(ans, sum);
        }
        return ans;
    }
}
```


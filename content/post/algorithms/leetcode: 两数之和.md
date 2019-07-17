---
title: "leetcode: 两数之和"
date: 2019-07-02T20:23:42+08:00
keywords: []
description: ""
tags: [
    "leetcode"
]
categories: [
    "数据结构与算法"
]
author: "yuanzx"
---

# 一、题目

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那两个整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

# 二、示例

给定 nums = {2, 7, 11, 15}, target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9，所以返回 {0, 1}

# 三、解

## 我的解

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length - 1; i++) {
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[i] + nums[j] == target) {
                    return new int[]{i, j};
                }
            }
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}
```

执行时间： 51ms

- 时间复杂度：O(n^2)
- 空间复杂度：O(1)

## 最优解

### 一遍哈希表

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (map.containsKey(complement)) {
                return new int[] { map.get(complement), i };
            }
            map.put(nums[i], i);
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}
```

执行时间： 7ms

- 时间复杂度：O(n)
- 空间复杂度：O(n)

在迭代的同时将数值插入到 HashMap 中，使得查找的时间复杂度为 O(1) 来降低时间复杂度

### 提交里最快的解法: 手动构造哈希表

```java
class Solution {
    public int[] twoSum(int[] nums, int target) { 
        int indexArrayMax = 4095;
        int[] indexArrays = new int[indexArrayMax + 1];
        for (int i = 0; i < nums.length; i++) {
            int diff = target - nums[i];
            int index = diff & indexArrayMax;
            if (indexArrays[index] != 0) {
                return new int[] { indexArrays[index] - 1, i };
            }
            indexArrays[nums[i] & indexArrayMax] = i + 1;
        }
        throw new IllegalArgumentException("No two sum solution");
    }
}
```

执行时间： 2ms

- 时间复杂度：O(n)
- 空间复杂度：O(n)

这种解法通过手动构造的哈希表来降低冲突的可能性，但是这种解法有个问题，如果出现哈希冲突，那么这种解法是直接覆盖的，那答案不就错了么。
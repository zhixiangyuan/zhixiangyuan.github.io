---
title: "Leetcode: 912 排序数组"
date: 2019-10-18T08:35:40+08:00
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

[排序数组](https://leetcode-cn.com/problems/sort-an-array/)

# 2 解

## 2.1 我的解

### 2.1.1 归并排序

- 时间复杂度: O(nlogn)

```java
class Solution {
    public int[] sortArray(int[] nums) {
        mergerSort(nums, 0, nums.length - 1);
        return nums;
    }
    
    public void mergerSort(int[] nums, int low, int high) {
        if (low == high) return;
        int mid = (low + high) / 2;
        mergerSort(nums, low, mid);
        mergerSort(nums, mid + 1, high);
        merge(nums, low, mid, high);
    }
    
    // 从小到大排序
    private void merge(int[] nums, int low, int mid, int high) {
        int[] tmpArray = new int[high - low + 1];
        int tmpArrayIndex = 0;
        int lowEndIndex = mid + 1;
        int highEndIndex = high + 1;
        int leftIndex = low;
        int rightIndex = mid + 1;
        while (leftIndex != lowEndIndex || rightIndex != highEndIndex) {
            if (leftIndex <= mid && rightIndex <= high) {
                if (nums[leftIndex] < nums[rightIndex]) {
                    tmpArray[tmpArrayIndex++] = nums[leftIndex++];
                } else {
                    tmpArray[tmpArrayIndex++] = nums[rightIndex++];
                }
            } else if (leftIndex != lowEndIndex) {
                tmpArray[tmpArrayIndex++] = nums[leftIndex++];
            } else if (rightIndex != highEndIndex){
                tmpArray[tmpArrayIndex++] = nums[rightIndex++];
            }
        }
        
        for (tmpArrayIndex = 0; tmpArrayIndex < tmpArray.length; tmpArrayIndex++) {
            nums[low++] = tmpArray[tmpArrayIndex];
        }
    }
}
```
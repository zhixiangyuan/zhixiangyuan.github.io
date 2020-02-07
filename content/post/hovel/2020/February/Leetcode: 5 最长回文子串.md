---
title: "Leetcode: 5 最长回文子串"
date: 2020-02-06T15:37:18+08:00
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

[最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

# 2 解

```java
/**
 * 核心思想便是从某个字母中间向两边找，如果找到的字母是相等的，说明是回文
 * 那么就接着找，一直到不想等为止
 */
class Solution {
    public String longestPalindrome(String s) {
        if (s == null) return "";
        if (s.length() == 0 || s.length() == 1) {
            return s;
        }
        int start = 0;
        int end = 0;
        for (int i = 0; i < s.length(); i++){
            // 寻找中心点周围没有相等字母的情况
            int len1 = expandAroundCenter(s, i, i);
            // 寻找中心点的前一个字母和中心点相等的情况
            // 这种情况上面的找法判断不出来
            int len2 = expandAroundCenter(s, i, i + 1);
            int len = Math.max(len1, len2);
            // 此处判断 len > end - start 和 len > end - start + 1 均可
            // len > end - start 说明相等也会重新计算起始点和结束点
            // len > end - start + 1 则如果 len 相等则不会重新计算起始点和结束点
            if (len > end - start) {
                start = i - (len - 1)/2;
                end = i + len/2;
            }
        }
        return s.substring(start, end + 1);
    }

    private int expandAroundCenter(String str, int left, int right){
        while (left > -1 && right < str.length() && str.charAt(left) == str.charAt(right)) {
            left--;
            right++;
        }
        return right - left - 1;
    }
}
```
---
title: "Leetcode: 3 无重复字符的最长子串"
date: 2019-09-26T07:56:34+08:00
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

[无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

# 2 解

## 2.1 我的解

- 时间复杂度: O(n)

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        char[] result = s.toCharArray();
        HashMap<Object, Integer> map = new HashMap();
        int maxLength = 0;
        int offset = 0;
        for (int i=0; i < result.length; i++) {
            if (!map.containsKey(result[i])) {
                map.put(result[i], i);
                offset++;
            } else {
                Integer index = map.get(result[i]);
                i = index + 1;
                map.clear();
                map.put(result[i], i);
                offset = 1;
            }
            maxLength = Math.max(maxLength, offset);
        }
        return maxLength;
    }
}
```

## 2.2 官方的解

有点厉害，直接用数组解决了这个问题，随机访问的效率远远比别的高

```java
public class Solution {

    public static int lengthOfLongestSubstring(String s) {
        int n = s.length(), ans = 0;
        int[] index = new int[128];
        // try to extend the range [i, j]
        for (int j = 0, i = 0; j < n; j++) {
            // 默认 index 全是 0，如果 index[s.charAt(j)] 的值为 0
            // 那么 Math.max(index[s.charAt(j)], i) 必然结果为 i 的值
            // 如果 index[s.charAt(j)] 不为 0，则说明之前碰到过这个字符
            // 则将这个字符的后一位的位置赋值给 i，从该位置开始遍历.
            i = Math.max(index[s.charAt(j)], i);
            ans = Math.max(ans, j - i + 1);
            // 存储当前字符的后一位在 String 中的下标
            index[s.charAt(j)] = j + 1;
        }
        return ans;
    }
}
```
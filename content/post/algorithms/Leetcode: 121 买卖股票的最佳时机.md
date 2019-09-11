---
title: "Leetcode: 121 买卖股票的最佳时机"
date: 2019-09-11T07:58:02+08:00
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

[买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

# 2 解

## 2.1 一遍遍历

- 时间复杂度: O(n)
- 空间复杂度: O(1)

碰到最低价格就将价格保存下来，然后将最低价格和后面的最高价格做比较，保存利润，这样始终就算出了最高利润，一遍遍历就能算出来

```java
class Solution {
    public int maxProfit(int prices[]) {
        int minprice = Integer.MAX_VALUE;
        int maxprofit = 0;
        for (int i = 0; i < prices.length; i++) {
            if (prices[i] < minprice)
                minprice = prices[i];
            else if (prices[i] - minprice > maxprofit)
                maxprofit = prices[i] - minprice;
        }
        return maxprofit;
    }
}
```


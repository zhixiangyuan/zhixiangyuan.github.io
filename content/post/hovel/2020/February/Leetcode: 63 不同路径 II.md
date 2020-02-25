---
title: "Leetcode: 63 不同路径 II"
date: 2020-02-07T21:00:28+08:00
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
---

# 1 题目

[不同路径 II](https://leetcode-cn.com/problems/unique-paths-ii/)

# 2 解

## 2.1 模拟移动

这个方法跑数据集会超时

```java
class Solution {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        // 从 [0, 0] 节点开始走
        return move(obstacleGrid, 0, 0);
    }

    private int move(int[][] obstacleGrid, int rowIndex, int columnIndex) {
        if (
            // 行越界
            rowIndex >= obstacleGrid.length 
            // 列越界
            || columnIndex >= obstacleGrid[0].length
            // 碰到障碍物
            || obstacleGrid[rowIndex][columnIndex] == 1
        ) {
            return 0;
        } else if (
            // 碰到终点
            rowIndex == obstacleGrid.length - 1
            && columnIndex == obstacleGrid[0].length - 1
        ) {
            return 1;
        } else {
            // 计算总和
            int sum = move(obstacleGrid, rowIndex + 1, columnIndex);
            sum += move(obstacleGrid, rowIndex, columnIndex + 1);
            return sum;
        }
    }
}
```

## 2.2 动态规划

```java
class Solution {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        // 处理数据集的特殊情况
        if (obstacleGrid[0][0] == 1) {
            return 0;
        }
        if (obstacleGrid[obstacleGrid.length - 1][obstacleGrid[0].length - 1] == 1) {
            return 0;
        }
        if (obstacleGrid.length == 1) {
            return 1;
        }
        // 初始化第一个位置为 1
        obstacleGrid[0][0] = 1;
        // 初始化第一列数值为 0 的数据变成 1，数值为 1 的墙以及其后的数据为 0
        for (int rowIndex = 1; rowIndex < obstacleGrid.length; rowIndex++) {
            obstacleGrid[rowIndex][0] = (obstacleGrid[rowIndex][0] == 0 && obstacleGrid[rowIndex - 1][0] == 1)? 1:0;
        }
        // 初始化第一行数值为 0 的数据变成 1，数值为 1 的墙以及其后的数据为 0
        for (int columnIndex = 1; columnIndex < obstacleGrid[0].length; columnIndex++) {
            obstacleGrid[0][columnIndex] = (obstacleGrid[0][columnIndex] == 0 && obstacleGrid[0][columnIndex - 1] == 1)? 1:0;
        }
        // 做相加操作，如果碰到墙直接置为 0
        for (int rowIndex = 1; rowIndex < obstacleGrid.length; rowIndex++) {
            for (int columnIndex = 1; columnIndex < obstacleGrid[0].length; columnIndex++) {
                if (obstacleGrid[rowIndex][columnIndex] == 1) {
                    obstacleGrid[rowIndex][columnIndex] = 0;
                } else {
                    obstacleGrid[rowIndex][columnIndex] = obstacleGrid[rowIndex - 1][columnIndex] + obstacleGrid[rowIndex][columnIndex - 1];
                }
            }
        }
        // 取出终点的数据
        return obstacleGrid[obstacleGrid.length - 1][obstacleGrid[0].length - 1];
    }
}
```
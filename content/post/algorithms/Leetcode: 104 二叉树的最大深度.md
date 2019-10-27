---
title: "Leetcode: 104 二叉树的最大深度"
date: 2019-09-16T07:08:02+08:00
keywords: []
description: ""
tags: [
    "leetcode"
]
categories: [
]
autoCollapseToc: false
author: "yuanzx"
---

# 1 题目

[二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

# 2 解

## 2.1 我的解

- 时间复杂度: O(n)
- 空间复杂度: O(1)

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    
    private int depth = 0;
    public int maxDepth(TreeNode root) {
        maxDepth(root, 1);
        return depth;
    }
    
    public void maxDepth(TreeNode root, int depth) {
        if (root == null) {
            return;
        }
        if (root.left == null && root.right == null) {
            this.depth = Math.max(depth, this.depth);
        }
        maxDepth(root.left, depth + 1);
        maxDepth(root.right, depth + 1);
    }
}
```

## 2.2 另一种解法

- 时间复杂度: O(n)
- 空间复杂度: O(1)

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int leftDepth = maxDepth(root.left);
        int rightDepth = maxDepth(root.right);
        return Math.max(leftDepth, rightDepth) + 1;
    }
}
```
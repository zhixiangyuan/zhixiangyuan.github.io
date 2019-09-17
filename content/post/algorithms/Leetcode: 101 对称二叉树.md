---
title: "Leetcode: 101 对称二叉树"
date: 2019-09-17T07:47:27+08:00
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

[对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

# 2 解

## 2.1 我的解

思路拿取对称的节点进行比较，相同返回 true，不相同返回 false

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
    public boolean isSymmetric(TreeNode root) {
        if (root == null) {
            return true;
        }
        if (root.left != null && root.right != null) {
            return isSymmetric(root.left, root.right);
        } else if (root.left != null && root.right == null) {
            return false;
        } else if (root.left == null && root.right != null) {
            return false;
        } 
        return true;
    }
    
    private boolean isSymmetric(TreeNode left, TreeNode right) {
        if (left.val != right.val) {
            return false;
        }
        
        if (left.left != null && right.right != null) {
            if (!isSymmetric(left.left, right.right)) {
                return false;
            }
        } else if (left.left != null && right.right == null) {
            return false;
        } else if (left.left == null && right.right != null) {
            return false;
        }
        if (left.right != null && right.left != null) {
             if (!isSymmetric(left.right, right.left)) {
                 return false;
             }
        } else if (left.right != null && right.left == null) {
            return false;
        } else if (left.right == null && right.left != null) {
            return false;
        }
        return true;
    }
}
```

## 2.2 官方题解

### 2.2.1 递归（DFS）

- 时间复杂度: O(n)
- 空间复杂度: O(n)

对于复杂度的 O(n) 官方是这样解释的，如果树是线性的，那么在栈上的递归调用造成的空间复杂度为 O(n)。但是这里如果树是线性的，那么 `if (t1 == null || t2 == null) return false;` 就会直接返回 false。所以这里的线性不能直接理解为一条线，而是两条线，左边的二叉树线和右边的二叉树线，并且是镜像的，这样才会出现递归调用到 O(n) 的情况。

```java
public boolean isSymmetric(TreeNode root) {
    return isMirror(root, root);
}

public boolean isMirror(TreeNode t1, TreeNode t2) {
    if (t1 == null && t2 == null) return true;
    if (t1 == null || t2 == null) return false;
    return (t1.val == t2.val)
        && isMirror(t1.right, t2.left)
        && isMirror(t1.left, t2.right);
}
```

### 2.2.2 迭代（BFS）

- 时间复杂度: O(n)
- 空间复杂度: O(n)

```java
public boolean isSymmetric(TreeNode root) {
    Queue<TreeNode> q = new LinkedList<>();
    q.add(root);
    q.add(root);
    while (!q.isEmpty()) {
        TreeNode t1 = q.poll();
        TreeNode t2 = q.poll();
        if (t1 == null && t2 == null) continue;
        if (t1 == null || t2 == null) return false;
        if (t1.val != t2.val) return false;
        q.add(t1.left);
        q.add(t2.right);
        q.add(t1.right);
        q.add(t2.left);
    }
    return true;
}
```
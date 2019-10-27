---
title: "Leetcode: 450 删除二叉搜索树中的节点"
date: 2019-10-21T10:13:37+08:00
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

[二叉搜索树中的搜索](https://leetcode-cn.com/problems/binary-watch/)

# 2 解

## 2.1 我的解

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
    private static final String LEFT = "left";
    private static final String RIGHT = "right";
    public TreeNode deleteNode(TreeNode root, int key) {
        if (root == null) return null;
        if (root.val == key) {
            if (root.left == null && root.right == null) {
                return null;
            } else if (root.left != null && root.right == null) {
                TreeNode left = root.left;
                root.left = null;
                return left;
            } else if (root.right != null && root. left == null) {
                TreeNode right = root.right;
                root.right = null;
                return right;
            } else {
                root.val = deleteRightChildTreeMinimum(root, root.right, RIGHT);
                return root;
            }
        }
        if (root.val > key) internalDeleteNode(root, root.left, LEFT, key);
        if (root.val < key) internalDeleteNode(root, root.right, RIGHT, key);
        return root;
    }
    
    private void internalDeleteNode(TreeNode root, TreeNode child, String direction, int key){
        if (child == null) return;
        if (child.val == key) {
            // 这里分为三种情况处理
            if (child.left == null && child.right == null) {
                // 1. 父节点无左右节点，直接将祖父节点置为 null 即可
                switch (direction) {
                    case LEFT: {
                        root.left = null;
                        break;
                    }
                    case RIGHT: {
                        root.right = null;
                        break;
                    }
                }
            } else if (child.left != null && child.right == null) {
                switch (direction) {
                    case LEFT: {
                        root.left = child.left;
                        break;
                    }
                    case RIGHT: {
                        root.right = child.left;
                        break;
                    }
                }
            } else if (child.left == null && child.right != null) {
                switch (direction) {
                    case LEFT: {
                        root.left = child.right;
                        break;
                    }
                    case RIGHT: {
                        root.right = child.right;
                        break;
                    }
                }
            } else {
                // 走到这里必然满足 child.left != null && child.right != null
                // 那么就要在右子数中找到最小节点然后替换需要被删除的节点
                child.val = deleteRightChildTreeMinimum(child, child.right, RIGHT);
            }
            return;
        }
        if (child.val > key) internalDeleteNode(child, child.left, LEFT, key);
        if (child.val < key) internalDeleteNode(child, child.right, RIGHT, key);
    }
    
    private int deleteRightChildTreeMinimum(TreeNode root, TreeNode child, String direction) {
        if (child.left == null && child.right == null) {
            // 说明这个 child 节点就是最小的节点
            switch (direction) {
                case LEFT: {
                    root.left = null;
                    break;
                }
                case RIGHT: {
                    root.right = null;
                    break;
                }
            }
            return child.val;
        } else if (child.left != null) {
            // 如果左子树不为 null，则继续从左子树中寻找最小节点
            return deleteRightChildTreeMinimum(child, child.left, LEFT);
        } else {
            // 左子树为 null，那么最小的数就是当前的 child 了
            int min = child.val;
            switch (direction) {
                case LEFT: {
                    root.left = child.right;
                    break;
                }
                case RIGHT: {
                    root.right = child.right;
                    break;
                }
            }
            return min;
        }
    }
}
```
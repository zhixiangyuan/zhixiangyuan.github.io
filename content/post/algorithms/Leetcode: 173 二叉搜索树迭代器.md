---
title: "Leetcode: 173 二叉搜索树迭代器"
date: 2019-09-25T07:46:04+08:00
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

[二叉搜索树迭代器](https://leetcode-cn.com/problems/binary-search-tree-iterator/)

# 2 解

## 2.1 我的解

先保存树左边的节点进队列，当队列空了再让后边的节点进队列，这样似乎还是没有满足使用 O(h) 的内存空间 h 是树的高度。


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
class BSTIterator {
    
    Queue<Integer> queue = new LinkedList<>();
    boolean recursionRight = false;
    TreeNode root;

    public BSTIterator(TreeNode root) {
        if (root == null) return;
        this.root = root;
        recursion(root.left);
        queue.add(root.val);
    }
    
    /** @return the next smallest number */
    public int next() {
        return queue.poll();
    }
    
    /** @return whether we have a next smallest number */
    public boolean hasNext() {
        if (!recursionRight && queue.peek() == null && root != null) {
            recursion(root.right);
            recursionRight = true;
        }
        return queue.peek() != null ? true : false;
    }
    
    private void recursion(TreeNode root) {
        if (root == null) return;
        recursion(root.left);
        queue.add(root.val);
        recursion(root.right);
    }
}

/**
 * Your BSTIterator object will be instantiated and called as such:
 * BSTIterator obj = new BSTIterator(root);
 * int param_1 = obj.next();
 * boolean param_2 = obj.hasNext();
 */
```
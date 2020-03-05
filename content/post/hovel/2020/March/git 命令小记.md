---
title: "git 命令小记"
date: 2020-03-04T14:56:38+08:00
keywords: []
description: ""
tags: [
    "Git"
]
categories: [
    "杂货铺"
]
autoCollapseToc: false
author: "yuanzx"
---

下面的记录中 `branch` 表示分支，`branch: <$分支名>`，`branch: <$分支名>*` 其中的 `*` 表示所处的分支。

```shell

# 运行前 git 分支情况
*
|\
| * branch: bugFix*
* | branch: master

$> git rebase master
# 运行后 git 分支情况
*
|\
| * branch: bugFix
* | branch: master
|/
*   branch: unknow*

----------------------------------

# 查看 HEAD 当前指向的记录
$> cat .git/HEAD
9e68ee1ed185d541c42ec6fee6792ce11ad11e32

# 上文 `branch: <$分支名>*` 中的 * 表示的就是 HEAD 
# 比如说当指向 master 的时候 执行 `cat .git/HEAD` 看看输出
$> cat .git/HEAD
ref: refs/heads/master

# 此时也可以通过下面的命令查看 HEAD 的指向
$> git symbolic-ref HEAD
refs/heads/master

----------------------------------

# 相对引用
# ^ 表示上一个记录，^^ 表示上上个
# ~<$num> 表示上 num 个记录
# 比如下面这个例子便是去 master 的上个记录
$> git checkout master^
$> git checkout master~1

----------------------------------

# 将分支指向某个提交
$> git branch -f <$branch_name> <{$hash_record|$branch_name}>

# 下面是一个例子
# 移动之前的分支情况
*
|\
| * branch: bugFix*
* | branch: master

$> git branch -f master c1_hash_record
# 运行后 git 分支情况
*   c1_hash_record branch: master
|\
| * c2_hash_record branch: bugFix
* | c3_hash_record 

----------------------------------
# 撤销到上一次的记录，对远程仓库不适用
# 执行前
*
* branch: master*

# 执行过后，前一次的提交的内容会回到未追踪的状态
# 通过 git status 便可以看到改变的内容
# 这种撤销的方式无法改变远程仓库的内容
$> git reset HEAD^
# 执行后
* branch: master*
*

----------------------------------
# 撤销上一次的提交，对远程仓库适用，
# 原理是创建提交，然后用相反的提交抵消之前的提交
# 执行前
*
* branch: master*

# 反向抵消之前提交的内容在未追踪的状态
# 需要 git add 和 commit
$> git revert master
# 执行后
* 
*
* branch: master*

----------------------------------
# 将指定的提交复制到当前的提交下面
# 执行前
* C1
* C2
* C3 branch: master*

# 将 C1 C2 这两次提交再次复制到当前的提交下面
$> git cherry-pick C1 C2
# 执行后
* C1
* C2
* C3 
* C1'
* C2' branch: master*

----------------------------------


----------------------------------
----------------------------------
```

# 参考文章

1. [很棒的 Git 可视化教学](https://learngitbranching.js.org/)
---
title: Git 实战
comments: true
mathjax: false
date: 2021-05-25 21:47:00
tags:
    - Git
categories:
    - [Learning, Computer, Git]
---

实用 Git 操作技巧。

<!-- more -->

# 从远程仓库获取最新代码合并到本地分支

原文地址：https://blog.csdn.net/hanchao5272/article/details/79162130

查询当前远程的版本：

```shell
git remote -v
```

获取最新代码到本地（本地当前分支为 \<branch\>，获取的远端的分支为 \<remote\>/\<branch\>）：

```shell
# 获取远端 origin 的 master 分支，保存为本地 origin/master 分支
git fetch origin master
```

查看版本差异：

```shell
# 查看 master 与 origin/master 的版本差异
git log -p master..origin/master
```

合并最新代码到本地分支：

```shell
git merge origin/master
```


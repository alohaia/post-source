---
title: Vue 学习记录
comments: true
mathjax: false
date: 2021-04-28 22:58:06
tags:
    - Vue
categories:
    - [Learning, Computer, Front end]
---

@[toc]

<!-- more -->

# 从空目录开始搭建 Electron 项目

## 准备工具的安装

安装 npm：
```bash
sudo pacman -Sy npm
```

安装 cnpm（可选）：
```bash
sudo npm install cnpm -g --registry=https://registry.nlark.com
```

安装 yarn（可选）：
```bash
sudo pacman -Sy yarn
```

## 搭建 Electron 环境

### 全局安装

使用 npm（可用 cnpm 或者 yarn 代替）安装：

```bash
# 稳定版
npm i -D electron@latest

# 测试版
npm i -D electron@beta

# github 上的最新版
npm i -D electron-nightly
```

使用 Linux 软件包管理器安装：

```bash
paru -Sy electron
```

### 为指定项目安装

> 非必需，但是可以使 LSP server 能提供相应的语法补全。

```bash
npm install electron --save-dev
```

## 开始项目

准备两个基本文件：

```bash
touch main.js index.html
```

初始化 `packages.json`：

```bash
cnpm init --yes
```

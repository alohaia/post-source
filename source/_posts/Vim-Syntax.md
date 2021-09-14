---
title: Vim Syntax
comments: true
mathjax: false
date: 2021-03-16 16:56:12
tags:
    - Vim/NeoVim
categories:
    - [Learning, Computer, Vim/NeoVim]
---

> 写 [NeoVim 插件](https://github.com/alohaia/vim-hexowiki)过程中，对 Vim 语法高亮的一些总结。

<!-- more -->

# 优先级

1. 多个 Match 或 Region 的匹配项在相同位置起始时，最后定义的优先。
2. Keyword 优先级高于 Match 和 Region。
3. 在更靠前的位置起始的匹配项优先。

# 其他

- 大小写：`:h syn-case`
- 折叠深度：`:h syn-foldlevel`
- 语法检查：`:h syn-spell`

```vimscript
syn keyword vimCommand tag
syn keyword vimSetting contained tag
```


---
title: Lua 和 C
comments: true
mathjax: false
date: 2021-04-15 10:49:03
tags:
    - Lua
    - C/C++
categories:
    - [Learning, Computer, Lua]
    - [Learning, Computer, C/C++]
---

Lua 是一种
- **嵌入式语言**——C 语言拥有控制权，Lua 被当作库，C 代码称为**应用代码**
- **可扩展的语言**——Lua 拥有控制权，C 语言被用作库，C 代码称为**库代码**

应用代码和库代码都使用相同的 API 和 Lua 通信，这些 API 被称为 ==C API==。

<!-- more -->

## 头文件

- `lua.h`：Lua 提供的基础函数，如创建新 Lua 环境、调用 Lua 函数、读写环境中全局变量以及注册供 Lua 调用的新函数的函数。这些函数都以 `lua_` 开头。
- `lauxlib.h`：**辅助库**（auxlibary library, auxlib）所提供的函数，使用 `lua.h` 中的基础 API 实现更高层次的抽象。这些函数以 `luaL_` 开头。
- `lualib.h`：用于操作 Lua 标准库。其中的函数也以 `luaL_` 开头。


## 常用函数

{% note info%}
完整函数列表见：
- [Lua 参考手册（Lua5.3 - 菜鸟教程）](https://www.runoob.com/manual/lua53doc/contents.html#index)
- [Lua 参考手册（Lua5.4 - 官网](http://www.lua.org/manual/5.4/#index)
{% endnote %}


`lua_State *luaL_newstate (void);`
: 创建一个新的 **Lua 状态机**。新环境中没有任何预定义的函数。

`void luaL_openlibs (lua_State *L);`
: 加载所有 Lua 标准库。

`int luaL_loadstring (lua_State *L, const char *s);`
: 将一个字符串加载为 Lua 代码块。 这个函数使用 `lua_load` 加载一个零结尾的字符串 `s`。




---
title: Lua 迭代器和泛型 for
comments: true
mathjax: false
date: 2021-04-08 10:57:34
tags:
    - Lua
categories:
    - [Learning, Computer, Lua]
---

> 《Lua 程序设计》的“迭代器”和“工厂”（返回迭代器及其他值的函数）经常混用。
> 本文也未刻意区分。

<!-- more -->

# 迭代器

迭代器（iterator）
: 一种可以用来遍历一个集合中所有元素的代码结构（函数）。

闭包（closure）
: 一个可以访问其自身环境中一个或多个局部变量的函数。
: 通常这一概念还包含其可访问的外部局部变量。但也可将其简单理解为有状态（保存在外部局部变量中）的迭代器，与下文中的“无状态迭代器”相对。

工厂（factory）
: 用于创建闭包及其封装变量的函数。

这里所说的闭包内可以访问局部变量称为”状态“。除了局部变量，宿主语言（如 C）中实现的迭代器还可以将状态保存在其自身的变量中（如 `io.read()` 将状态保存在 C 结构体中）。

# 泛型 for

泛型 for 的语法：

```lua
for <var-list> in <exp-list> do
    <body>
end
```

`for` 做的第一件事就是对 `<exp-list>` 求值。

表达式列表 `<exp-list>` 应返回三个值**供 `for` 保存**：
1. 迭代函数
2. 不可变状态（invariant state）（`ipairs` 和 `pairs` 简单地将其参数作为不可变状态返回）
3. 控制变量的初始值

控制变量（control variable）
: 其值为 `nil` 时循环结束。

表达式列表的结果最多有三个，多余的会被舍弃；而不足的则以 `nil` 补齐。
其中只有最后一项能产生多个值。

`for` 做的第二件事是以不可变状态和控制变量为参数来调用选代函数，并将返回值赋给 `<var-list>`，如果第一个返回值为 `nil` 则结束循环，否则执行循环体并再次调用迭代函数，并不断重复此过程。

确切地说，形如

```lua
for var_1, ..., var_n in exp-list do
    block
end
```

与以下代码等价：

```lua
do
    local _f, _s, _var = exp-list
    while true do
        local var_1, ..., var_n = _f(_s, _var)
        _var = var_1
        if var == nil then
            break
        end
        block
    end
end
```

因此，假设迭代函数为 f，不可变状态为 s，控制变量初始值为 a~0~，那么在循环中控制变量的值依次为 a~1~=f(s,a~0~), a~2~=f(s,a~1~), ...，直到 a~i~ 为 `nil`。如果 `<var-list>` 还有其他变量（var_1、var_2……），那么这些变量只是简单地在每次调用 f 后得到额外的返回值（可以在 `block` 中使用这些变量）。

# 无状态迭代器

无状态迭代器（stateless iterator）
: 一种自身不保存任何状态（即闭包所访问的“局部变量”）的选代器。

无状态迭代器只根据不可变状态和控制变量（即其参数）的值来生成下一个元素。
生成无状态迭代器的工厂函数不用为其提供局部变量，一般只需简单地返回迭代器和不可变状态即可。所以可以将无状态迭代器放在工厂外而不是工厂函数的 `return` 语句中。

`ipairs` 是一个典型的无状态迭代器，可以在 Lua 中这样实现：

```lua
local function iter(t, i)
    i = i + 1
    local v = t[i]
    if v then
        return i, v
    end
end

function ipairs(t)
    return iter, t, 0
end

-- 以下两个等效

-- 1
for i,v in ipairs({1,2,3,4}) do
    print(i,v)
end

-- 2
for i,v in iter,{1,2,3,4},0 do
    print(i,v)
end
```

`pairs` 和 `ipairs` 类似，不过 `pairs` 返回的迭代函数是 Lua 中的一个基本函数 `next`：

```lua
function pairs(t)
    return next, t, nil
end
```

```c
int main() {
    printf();
    return 0;
}
```




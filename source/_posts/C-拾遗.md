---
title: C++ 拾遗
comments: true
mathjax: false
date: 2021-07-24 11:55:27
tags:
    - C/C++
categories:
    - [Learning, Computer, C/C++]
---

一些之前没注意或忘掉的 C\+\+ 的语法或用法细节，C\+\+ 的基本语法则不在此记录。

<!-- more -->

# 异常

C\+\+ 的**异常**（exception）机制为处理函数非正常运行提供了另一种途径。在 C 中，函数多通过返回值来告知调用者函数发生了非预期的情况。然而若是返回值有其他用途，那么如何将正常返回值和用于提示的返回值相区分可能会是个问题：

```c
double devide(double lhs, double rhs) {
    if(rhs == 0) {
        // 应该返回什么值？
    }
    return lhs/rhs;
}
```

我们想要通过一个特殊的值来提示调用者发生了“devide-by-zero”的情况，但是所有实数（当然是在 `double` 的范围内）都可以作为正常的返回值。

我们可以用 C\+\+ 的异常来解决这一问题：

```cpp
#include <iostream>
#include <cstdio>
#include <stdexcept>

double devide(double lhs, double rhs) {
    if(rhs == 0) {
        throw std::domain_error("Devide by zero.\n");
    }
    return lhs/rhs;
}

int main() {
    try {
        std::cout << devide(10, 0);
    } catch (std::exception &e) {
        std::cout << e.what();
    } catch (...) {
        std::cout << "undefined exception.\n";
    }
}
```

> C\+\+ 中除以 0 不会引发异常或错误，而是会得到 [infinity](https://zh.cppreference.com/w/cpp/types/numeric_limits/infinity)。

## 抛出异常

`throw` 有两种语法[^throw]：
1. `throw <expression>`：复制表达式结果（使用复制构造函数）然后抛出。
2. `throw`：重抛当前异常，中止当前 `catch` 块的执行并移动到下一个匹配的 `catch` 块。

[^throw]: https://zh.cppreference.com/w/cpp/language/throw

除了内置基本类型、用户自定义类型，C++ STL 还提供了 [std::exception](https://zh.cppreference.com/w/cpp/error/exception) 及其派生类用于异常抛出。

## 捕捉和处理异常

捕捉异常的原则与传递函数参数的原则类似：{% hzl %}对于基本类型，按值传递比较有效率；|而对类则应该按引用传递以保证效率。{% endhzl %}


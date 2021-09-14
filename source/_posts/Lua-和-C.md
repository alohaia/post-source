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

{% note info%}
完整函数列表见：
- [Lua 参考手册（Lua5.3 - 菜鸟教程）](https://www.runoob.com/manual/lua53doc/contents.html#index)
- [Lua 参考手册（Lua5.4 - 官网）](http://www.lua.org/manual/5.4/#index)
{% endnote %}

<!-- more -->

# 头文件

- `lua.h`：Lua 提供的基础函数，如创建新 Lua 环境、调用 Lua 函数、读写环境中全局变量以及注册供 Lua 调用的新函数的函数。这些函数都以 `lua_` 开头。
- `lauxlib.h`：**辅助库**（auxlibary library, auxlib）所提供的函数，使用 `lua.h` 中的基础 API 实现更高层次的抽象。这些函数以 `luaL_` 开头。
- `lualib.h`：用于操作 Lua 标准库。其中的函数也以 `luaL_` 开头。

# 栈

Lua 通过维护一个虚拟栈来完成 Lua 代码和 C 代码之间的交流。C API 中的函数若需要传入栈索引，这个索引必须是**有效索引**或**可接受索引**。

有效索引（valid indices）
: 对应一个可以修改的 Lua 值所在位置的索引，包括 1 到栈顶（1 ≤ abs(index) ≤ `lua_gettop(L)`）之间的栈索引和**伪索引**。

可接受索引（acceptable indices）
: 栈顶之后的索引（abs(index) > `lua_gettop(L)`）以及大于**上值**数目的伪索引。前者可以以只读方式访问内存。
: 除非特别说明，API 中的函数能正确处理可接受索引。

伪索引（pseudo-indices）
: 可以用来访问**注册表**（registry）（使用 `LUAREGISTRYINDEX` 访问）和 C 函数的**上值**（upvalue）。
: 注册表和上值都是不在栈中但是可以被 C 代码访问的位置。

Lua 代码严格按照后进先出的规则操纵栈，而 C 代码则可以对栈进行随机访问，甚至可以在任何位置插入和删除元素。

C 代码通过 `lua_push*` 函数将元素压入栈，通过 `lua_to*` 函数取得指定位置的元素值。

对于数字（Number 和 Integer），返回值 0 有意义，不能视为错误。
Lua5.2 中引入了新函数进行处理：

```c
lua_Number lua_tonumberx (lua_State *L, int index, int *isnum);
lua_Integer lua_tointegerx (lua_State *L, int index, int *isnum);
```

当获得的值为 0 或指定的值类型不对时，两个函数都会返回 0。
`isnum` 返回一个布尔值表示指定的值的类型是否为所需类型，以区分上述两种情况。

```c
lua_pushstring(L, "hello");

int msg;
if(!lua_tonumberx(L, -1, &msg)) {
    printf("Not a number, %d\n", msg);
    // -> Not a number, 0
}
```

```c
lua_pushinteger(L, 0)

int msg;
if(!lua_tonumberx(L, -1, &msg)) {
    printf("Not a number, %d\n", msg);
    // -> Not a number, 1
    // "Not a number" 判断错误，msg 表明指定位置的值的类型为 number
}
```

{% note warning 确保栈中有足够的空间 %}
Lua 保证在使用栈时栈中至少有 20 个空闲的位置（slot）（`lua.h` 中定义的单位大小为 `LUA_MINSTACK`），但有些时候会需要更多的空间，这时便需要 `lua_checkstack` 函数：

```c
int lua_checkstack(lua_State *L, int sz);
```

`lua_checkstack` 确保堆栈上至少有 `n` 个额外空位。
如果不能把堆栈扩展到相应的尺寸，函数返回假（0）。

辅助库 `lauxlib.h` 中定义了一个高层函数：

```c
void luaL_checkstack(lua_Stack *L, int sz, const char *msg);
```

该函数类似于 `lua_ckeckstack`，但是当无法扩展空间时，会抛出一个错误。
`msg` 是描述错误的额外文本信息，`NULL` 表示不需要额外文本。
{% endnote %}

其他栈操作（详细说明见[手册](#)）：

```c
int  lua_gettop    (lua_State *L);
void lua_settop    (lua_State *L, int index);
void lua_pushvalue (lua_State *L, int index);
void lua_rotate    (lua_State *L, int index);
void lua_remove    (lua_State *L, int index);
void lua_insert    (lua_State *L, int index);
void lua_replace   (lua_State *L, int index);
void lua_copy      (lua_State *L, int fromidx, int toidx);
```

# C API 中的错误处理

> 用异常来提示错误，而没有在 API 的每个操作中使用错误码。与 C++ 或 Java 不同，C 语言没有提供异常处理机制。为了解决这个问题，Lua 使用了 C 语言中的 `setjmp` 机制，`setjmp` 营造了一个*类似异常处理的机制*。
> 因此，大多数 API 都可以抛出异常（即调用 `longjmp`）而不是直接返回。

## 处理应用代码中的错误

在应用代码中，如果没有为 `longjmp` 准备对应的 `setjmp`，API 中的任何错误都会导致 Lua 调用紧急函数（panic function），这个函数返回时，应用就会退出。
我们可以通过 `lua_atpanic` 设置自己的紧急函数，但是作用不大。

要正确处理应用代码中的错误，必须通过 Lua 提供的 API 调用我们自己的代码，即在 `setjmp` 的上下文中运行代码。
我们可以将 C 代码封装到一个 C 函数中，再通过 `lua_pcall` 调用，这样我们的 C 代码会在**保护模式**（见 [`lua_pcall`](https://www.runoob.com/manual/lua53doc/manual.html#lua_pcall)）下运行。

```c
static int foo(lua_State *L) {
    // code to run in protected mode
    return 0;
}

int secure_foo(lua_State *L) {
    lua_pushfunction(L, foo);
    return(lua_pcall(L, 0, 0, 0) == 0);
}
```

## 处理库代码中的错误

在编写**库代码**时，通常无需进行额外的操作。库函数抛出的错误要么被 Lua 中的 `pcall` 捕获，要么被应用代码中的 `lua_pcall` 捕获（这就要求应用代码需要按照上文中的方式编写）。
因此，当库代码中的函数检测到错误时，只需简单地调用 `lua_error`（或 `luaL_error`）即可。`lua_error` 会以栈顶的值作为**错误对象**(error object)（“a single value”，`lua_pcall` 会在检测到错误时会将其置于栈顶），抛出一个 Lua 错误。

# 内存分配

Lua 的 C API 并不一定要通过调用 `malloc` 等函数来分配内存，而是可以通过用户自己实现的一个**分配函数**（allocation function）来分配和释放内存。

`luaL_newstate` 通过默认分配函数来创建 Lua 状态机，该分配函数使用了 C 的 `malloc`、`realloc`、`free` 函数。
`lua_newstate` 则将其参数作为分配函数创建 Lua 状态机。

```c
lua_State* lua_newstate(lua_Alloc f, void* ud);
```

- `lua_Alloc f`：分配函数
- `void* ud`：用户数据

分配函数需满足以下格式：

```c
typedef void* (*lua_Alloc)(void* ud,
                           void* ptr,
                           size_t osize,
                           size_t nsize);
```

- `void* ud`：始终为 `lua_newstate` 提供的用户数据
- `void* ptr`：正要被分配、重分配、释放的块的地址
- `size_t osize`：原始块的大小
- `size_t nsize`：请求的块的大小

分配函数的行为像
- `malloc`：`ptr` 为 `NULL`，`nsize` 为 0，返回新分配的块的地址
- `realloc`：`ptr` 不为 `NULL` 且 `nsize` 不为 0，释放 `ptr` 指向的块并返回新地址
- `free`：`ptr` 不为 `NULL`，`nsize` 为 0，分配函数必须释放 `ptr` 指向的块并返回 `NULL`

注意：
- 如果 `ptr` 是 `NULL`，那么 `osize` 则用来存放其他信息。若同时 `nsize` 为 0，则分配函数什么也不做，并返回 `NULL`
- 如果无法分配指定的块，则必须返回 `NULL`
- Lua 假定 `nsize` 等于 `osize` 时不会失败

`luaL_newstate` 使用的默认分配函数如下：

```c
void* l_alloc(void* ud, void* ptr, size_t osize, size_t nsize) {
    (void) ud; (void)osize; /* 未使用 */
    if(nsize == 0) {
        free(ptr);
        return NULL;
    }
    else
        return realloc(ptr, nsize);
}
```

# 在 Lua 中调用 C 语言

Lua 调用 C 函数时，每个 C 函数痘有其独立的**局部栈**，其第一个参数总是位于局部栈中索引为 1 的位置。

创建一个 C 库一般包含三步：
1. 创建 C 函数
2. 填充函数注册表：定义 C 函数到 Lua 函数的映射
3. 定义入口函数：将 C 函数注册为 Lua 函数

首先，库函数都必须使用同一个原型：

```c
typedef int (*lua_CFunction)(lua_State *L);
```

C 函数通过返回值（`return`）决定要返回（通过虚拟栈）给 Lua 的参数个数。

第二步中，函数注册表为一个 `luaL_Reg` 数组，如：

```c
static const struct luaL_Reg _lib_reg[] = {
    {"dir", l_dir},
    {NULL, NULL} // 哨兵
};
```

其中，`l_dir` 函数被注册为 Lua 函数 `dir`。

最后就是使用第二步中的函数注册表将 C 函数注册为 Lua 函数。这一步需要我们定义一个**入口函数**。
Lua 会根据库文件名在库文件中寻找入口函数，一般为 `luaopen_<file_name_without_extention_name>`，如：

```c
int luaopen_libdir(lua_State *L) {
    luaL_newlib(L, _lib_reg);
    return 1;
}
```

入口函数通常直接使用 `luaL_newlib` 方法创建新的 Lua 表，其中包含注册好的 Lua 函数。

```c
void luaL_newlib(lua_State *L, const luaL_Reg l[]);
```

`luaL_newlib` 其实是一个宏，详见[共享上值](#共享上值)。

完整示例见[实现 Lua 的 dir 函数](#实现-lua-的-dir-函数)。

{% note info "require 函数参数中的连字符" %}
Lua 的 `require` 函数在寻找入口函数时，入口函数名为库文件第一个连字符（-）之前的部分或文件名去除末尾的 `.so`（Windows 为 `.dll`）。

另外，`require` 函数中的句点 `.` 表示**子模块**，如：`require'libdir-v0.1'` 中的 `libdir-v0.1` 表示 `libdir-v0/1`：

```xxx
> dir = require'libdir-v0.1'
stdin:1: module 'libdir-v0.1' not found:
        no field package.preload['libdir-v0.1']
        no file '/usr/share/lua/5.4/libdir-v0/1.lua'
        no file '/usr/share/lua/5.4/libdir-v0/1/init.lua'
        no file '/usr/lib/lua/5.4/libdir-v0/1.lua'
        no file '/usr/lib/lua/5.4/libdir-v0/1/init.lua'
        no file './libdir-v0/1.lua'
        no file './libdir-v0/1/init.lua'
        no file '/usr/lib/lua/5.4/libdir-v0/1.so'
        no file '/usr/lib/lua/5.4/loadall.so'
        no file './libdir-v0/1.so'
        no file '/usr/lib/lua/5.4/libdir-v0.so'
        no file '/usr/lib/lua/5.4/loadall.so'
        no file './libdir-v0.so'
stack traceback:
        [C]: in function 'require'
        stdin:1: in main chunk
        [C]: in ?
```
{% endnote %}

## 注册表

Lua API 中通过使用注册表来实现“全局变量”。使用这一功能需要一对函数：

```c
int luaL_ref (lua_State *L, int t);
void luaL_unref (lua_State *L, int t, int ref);
```

注册表总是位于**伪索引** `LUA_REGISTRYINDEX` 中，可以像使用一个普通的索引一样使用它。
需要注意的是，所有在 Lua 中加载的库代码都共用一个注册表，在给表的键取名时要注意命名冲突。

{% note warning %}
在注册表中不能使用数值类型的键，因为 Lua 将其作为**引用系统**的保留字。
{% endnote %}


{% note info 轻量用户数据 %}
Lua 中提供轻量用户数据（light userdata）来获取唯一值。

```c
// 具有唯一地址的变量
static char Key = 'k';

// 保存字符串
lua_pushlightuserdata(L, (void*)&Key);
lua_pushstring(L, myStr);
lua_settable(L, LUA_REGISTRYINDEX);

// 获取字符串
lua_pushlightuserdata(L, (void*)&Key);
lua_gettable(L, LUA_REGISTRYINDEX);
*myStr = lua_tostring(L, -1);
```

为了简化将变量地址作为唯一值的用法，Lua5.2 引入了两个函数：`lua_rawgetp` 和 `lua_rawsetp`。

```c
// 具有唯一地址的变量
static char Key = 'k';

// 保存字符串
lua_pushstring(L, myStr);
lua_rawsetp(L, LUA_REGISTRYINDEX, (void*)&Key);

// 获取字符串
lua_rawgetp(L, LUA_REGISTRYINDEX, (void*)&Key);
myStr = lua_tostring(L, -1);
```
{% endnote %}

## 上值和共享上值

Lua 中可以为一个函数绑定若干个**上值**，这些上值仅在绑定的函数中可见。
一个函数和其上值的关联称为**闭包**（closure）。

```c
// 创建并返回闭包
void lua_pushcclosure(lua_State *L, lua_CFunction fn, int n);
// 在闭包函数中获取上值的索引
int lua_upvalueindex(int i);
```

上值的索引可以被当作普通索引使用，Lua 会处理好一切。

使用 `lua_pushcclosure` 在工厂函数中返回一个闭包是上值的一种使用方法，实例见：[上值和工厂函数](#上值和工厂函数)。

如果要在多个函数中共用上值（共享上值），可以使用以下代码替代 `luaL_newlib`。

```c
// 创建库函数表
luaL_newlibtable(L, _lib_reg);

// 准备要共享的值（这里是一个表）
lua_newtable(L);
/* ...（这里假设这些代码保证上面的表最终在栈顶） */

// 将 _lib_reg 中的函数添加到新库中，指定栈顶一定数量（这里是 1）的共享上值
// lua_setfuncs 会将栈中的值删除
lua_setfuncs(L, lib, 1);
```

# 实例

## 输出整个栈中的内容

```c
static void stackDump (lua_State* L) {
    int i;
    int top = lua_gettop(L);
    for (i = 0; i <= top; i++) {
        int t = lua_type(L, i);
        switch (t) {
            case LUA_TSTRING: {
                printf("'%s'", lua_tostring(L, i));
                break;
            }
            case LUA_TBOOLEAN: {
                printf(lua_toboolean(L, i) ? "true" : "false");
                break;
            }
            case LUA_TNUMBER: {
                printf("%g", lua_tonumber(L, i));
                break;
            }
            default: {
                printf("%s", lua_typename(L, t));
                break;
            }
        }
        printf("\t");
    }
    printf("\n");
}
```

## 实现 Lua 的 dir 函数

Lua 标准库中没有 `dir` 函数（读取目录下的所有文件），但是我们可以自己实现：

```c libdir-v0-1.c
#include <dirent.h>
#include <errno.h>
#include <string.h>

#include <lua.h>
#include <lauxlib.h>

static int l_dir(lua_State *L) {
    DIR *dir;
    struct dirent *entry;
    int i;
    const char *path = luaL_checkstring(L, 1);

    // 打开目录
    dir = opendir(path);
    if (dir == NULL) {
        lua_pushnil(L);
        lua_pushstring(L, strerror(errno));
        return 2;
    }

    lua_newtable(L);
    i = 1;
    while ((entry = readdir(dir)) != NULL) {
        lua_pushinteger(L, i++);
        lua_pushstring(L, entry->d_name);
        lua_settable(L, -3);
    }

    closedir(dir);
    return 1;
}

static const struct luaL_Reg _lib_reg[] = {
    {"dir", l_dir},
    {NULL, NULL} // 哨兵
};

// 入口函数，不加 static，lua_open<libname>
int luaopen_libdir(lua_State *L) {
    luaL_newlib(L, _lib_reg);
    return 1;
}
```

通过 `gcc -fPIC -shared -o libdir-v0-1.so libdir-v0-1.c` 编译得到动态库。
再通过以下 Lua 脚本测试：

```lua
dir = require"libdir"   --> 
dir.dir('.')  --> table: 0x5580923c1af0
```
## 上值和工厂函数

```c
// 前置声明
static int counter(lua_State* L);

// 工厂函数，返回（通过虚拟栈）一个 closure
int newCounter(lua_State* L) {
    lua_pushinteger(L, 10);
    lua_pushcclosure(L, &counter, 1);   // 状态机，指定闭包中的函数，要绑定的上值数量
    return 1;
}

// counter 的定义
static int counter(lua_State* L) {
    int val = lua_tointeger(L, lua_upvalueindex(1));
    lua_pushinteger(L, ++val);
    lua_copy(L, -1, lua_upvalueindex(1));
    return 1;
}
```

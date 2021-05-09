---
title: CMake 学习记录
comments: true
mathjax: false
date: 2021-05-03 20:35:04
tags:
    - CMake
    - C/C++
categories:
    - [Learning, Computer, C/C++]
---

GitHub repository: https://github.com/ttroy50/cmake-examples

CMake 是个一个开源的跨平台自动化建构系统，用来管理软件建置的程序，并不依赖于某特定编译器，并可支持多层目录、多个应用程序与多个库。<br/>
它用配置文件控制建构过程（build process）的方式和 Unix 的 make 相似，只是 CMake
的配置文件取名为 CMakeLists.txt。<br/>
CMake 并不直接建构出最终的软件，而是产生标准的建构档（如 Unix 的 Makefile 或
Windows Visual C++ 的 projects/workspaces），然后再依一般的建构方式使用。

<!-- more -->

> 原文中将 CMake 命令称为“函数”。本文沿用了这种称法。

## 基础

最基本的 Hello World CMakeLists.txt 文件：

```cmake CMakeLists.txt
# （可选）指定 所支持的 CMake 的最低版本
# 可以通过以下命令查看 CMake 版本
# $ cmake --version
cmake_minimum_required(VERSION 3.5)

# 设置项目名，这样做会设置一些变量
project(hello_cmake)

# 添加一个要生成的可执行文件
add_executable(hello_cmake main.cpp)
```

{% note info Note %}
有些时候会使用一种简便方法——将项目名和可执行文件名设成同一个名字。
然后就可以像下面这样写：

```cmake
cmake_minimum_required(VERSION 2.6)
project(hello_cmake)
add_executable(${PROJECT_NAME} main.cpp)
```

在这个例子中，[`project()`](https://cmake.org/cmake/help/latest/command/project.html) 函数将变量 `PROJECT_NAME` 设置为 `hello_cmake`，在 [`add_executable()`](https://cmake.org/cmake/help/latest/command/add_executable.html) 函数中则使用了这个变量指定可执行文件名。
{% endnote %}

### 二进制目录

我们运行 `cmake`（可带参数）命令时所在的目录（即当前工作目录、`pwd` 显示的目录）被称为
`CMAKE_BINARY_DIR`。这是你的生成的二进制文件所在的根目录（通常还会有一些其他文件）。

CMake 支持两种生成二进制文件的方式——就地构建（in-place
build）和源外构建（out-of-source build）。

#### 就地构建

要以这种方式构建，只需在项目根目录（其实是 CMakeLists.txt 所在的目录）执行
`cmake`（不指定项目根目录，见下）命令。这样做会使 CMake
生成的临时文件和二进制文件和项目文件混在一起。

#### 源外构建

要采用源外构建，在任一非项目根目录的目录中执行 `cmake <project_root>` 命令，其中
`<project_root>` 为项目根目录。这种方式生成的临时文件和二进制文件等会被放在执行
`cmake` 命令时所在的目录，而不会污染源目录（项目根目录），一般采用这种方式。

习惯上这样做（假设当前目录已经为源目录）：

```shell
mkdir build && cd build && cmake ..
make
```

### 其他目录路径变量

完整变量列表见：https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html

一些术语：
- **源（码）树（source tree）**和**构建树（build tree）**：分别指项目 *源代码根目录*
    和 *指定的生成二进制文件（等文件）的根目录* 和各自的子目录形成的树状目录结构。
- **源（码）目录（source directory）** 和 **构建目录（build directory）**：*源码树* 和
    *构建树* 中的位置，一般指顶层位置，但是可以移动，表示当前位置的变量一般含有 `CURRENT`。
- CMake 目录种类 TODO
    - 顶层/根
    - 当前所处
    - 正在处理

| 变量名                                                                                                  | 解释                             |
|---------------------------------------------------------------------------------------------------------|----------------------------------|
| [`CMAKE_SOURCE_DIR`](https://cmake.org/cmake/help/latest/variable/CMAKE_SOURCE_DIR.html)                 | 源码树（source tree）顶层的路径  |
| [`CMAKE_CURRENT_SOURCE_DIR`](https://cmake.org/cmake/help/latest/variable/CMAKE_CURRENT_SOURCE_DIR.html) | 正在处理的源目录                 |
| [`PROJECT_SOURCE_DIR`](https://cmake.org/cmake/help/latest/variable/PROJECT_SOURCE_DIR.html)             | 当前项目的源目录                 |
| [`CMAKE_BINARY_DIR`](https://cmake.org/cmake/help/latest/variable/CMAKE_BINARY_DIR.html)                 | 构建树（build tree）顶层的路径   |
| [`CMAKE_CURRENT_BINARY_DIR`](https://cmake.org/cmake/help/latest/variable/CMAKE_CURRENT_BINARY_DIR.html) | 当前所在的构建目录（build tree） | 
| [`PROJECT_BINARY_DIR`](https://cmake.org/cmake/help/latest/variable/PROJECT_BINARY_DIR.html)             | 项目的构建目录                   |

### 源文件变量

可以声明一个变量保存要使用的源文件名，以便之后使用：

```cmake
set(SOURCES
    src/Hello.cpp
    src/main.cpp
)

add_executable(${PROJECT_NAME} ${SOURCES})
```

也可以使用 `GOLB` 命令以用通配符寻找相符的文件：

```cmake
file(GLOB SOURCES "src/*.cpp")
```

{% note warning %}
这种方式已经不推荐，取而代之的是直接在 `add_xxx()` 函数中写明源文件。
{% endnote %}

### 包含目录

使用 [`target_include_directories()`](https://cmake.org/cmake/help/latest/command/target_include_directories.html) 函数为目标（target）添加包含目录（include path），这样添加的目录会在生成目标时以 `-I` 选项传递给编译器。

```cmake
target_include_directories(target
    PRIVATE
        ${PROJECT_SOURCE_DIR}/include
)
```

`INTERFACE`、`PUBLIC`、`PRIVATE` 关键字用来指定其后的参数的作用域（scope，也就是这三个关键字）
- `PUBLIC`、`PRIVATE`：参数会被用来填充目标的 [`INCLUDE_DIRECTORIES`](https://cmake.org/cmake/help/latest/prop_tgt/INCLUDE_DIRECTORIES.html#prop_tgt:INCLUDE_DIRECTORIES) 属性（property）
- `INTERFACE`、`PUBLIC`：参数会被用来填充目标的 [`INTERFACE_INCLUDE_DIRECTORIES`](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_INCLUDE_DIRECTORIES.html#prop_tgt:INTERFACE_INCLUDE_DIRECTORIES) 属性

详细解释见下面的[库](#库)。

### 库

#### 创建库

使用 [`add_library()`](https://cmake.org/cmake/help/latest/command/add_library.html) 函数以用指定源文件创建一个库文件。

如要创建静态库，可以像这样写：

```cmake
add_library(hello_library STATIC
    src/Hello.cpp
)
```

指定库的类型：
- `STATIC`：静态库，编译时链接
- `SHARED`：动态库，运行时链接
- `MODULE`：不会链接到其他目标，但是可以在运行时通过 *dlopen-like* 功能
  （见 [man 3 dlopen](https://man7.org/linux/man-pages/man3/dlopen.3.html)）手动加载

> 像之前推荐的那样，这里直接在 `add_xxx()` 函数中写明了要使用的源文件。

[上一节](#包含目录)中作用域的含义：
- `PRIVATE`：参数会被添加为**目标**的包含目录之一
- `INTERFACE`：参数会被添加为**与该库链接的所有的目标**的包含目录之一
- `PUBLIC`：相当于前两个作用域相加的效果

#### 链接库和目标

要将指定目标与库链接，使用 `target_link_libraries()` 函数：

```cmake
add_executable(hello_binary
    src/main.cpp
)

target_link_libraries(hello_binary
    PRIVATE
        hello_library
)
```

{% note info 目标别名 %}
可以在 `add_xxx()` 函数中使用 `ALIAS` 为目标创建一个别名，也可以理解为创建了一个“伪目标”。

如为库创建别名：

```cmake
add_library(hello::library ALIAS hello_library)

add_executable(hello_binary
    src/main.cpp
)

target_link_libraries(hello_binary
    PRIVATE
        hello::library
)
```

当然，也可以为可执行文件创建别名：

```cmake
add_executable(main ALIAS hello_binary)
```
{% endnote %}

### 安装

CMake 提供了生成 `make install` 目标（是 Makefile 中的 target，而不是之前提到的
CMake 中的 target）的功能。<br/>
安装目录可以通过 `CMAKE_INSTALL_PREFIX` 指定，详见 [指定安装目录](#指定安装目录)。

#### install 函数

安装由 [`install()`](https://cmake.org/cmake/help/latest/command/install.html) 函数控制。

```cmake
install (TARGETS cmake_examples_inst_bin
    DESTINATION bin)

install (TARGETS cmake_examples_inst
    LIBRARY DESTINATION lib)
```

上面的代码会将 `cmake_examples_inst_bin` 生成的二进制文件安装到 `${CMAKE_INSTALL_PREFIX}/bin` 目录下，将 `cmake_examples_inst` 生成的二进制文件安装到 `${CMAKE_INSTALL_PREFIX}/bin` 目录下。

{% note warning %}
在有 DLL 目标的平台（如 Windows）需要添加以下内容：

```cmake
install (TARGETS cmake_examples_inst
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin)
```
{% endnote %}

使用 `DIRECTORY` 以安装目录：

```cmake
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
    DESTINATION include)
```

使用 `FILES` 直接安装文件：

```cmake
install (FILES cmake-examples.conf
    DESTINATION etc)
```

#### install_manifest.txt

运行完 `make install` 后，CMake 会生成一个名为 install_manifest.txt
的文件，其中包含了所有安装文件的详细信息。

如果以 root 运行 `make install`，install_manifest.txt 会为 root 所有。

#### 指定安装目录

`CMAKE_INSTALL_PREFIX` 的值默认为 `/usr/local/`，要改变其值，可以在添加
*可执行文件和库文件* 前在顶层（top level）CMakeLists.txt 中添加如下内容：

```cmake
if( CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT )
    message(STATUS "Setting default CMAKE_INSTALL_PREFIX path to ${CMAKE_BINARY_DIR}/install")
    set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install" CACHE STRING "The path to use for make install" FORCE)
endif()
```

也可以通过终端命令的参数（`-DCMAKE_INSTALL_PREFIX=/install/location`）设置该变量。

----------------------------------------------------------------

还可以通过 make 的 `DESTDIR` 改变安装目录：

```shell
make install DESTDIR=/tmp/stage
```

这会创建 `${DESTDIR}/${CMAKE_INSTALL_PREFIX}` 作为安装目录。<br/>
在这个例子中，安装目录为 `/tmp/stage/usr/local`。

#### 卸载

CMake 默认不提供 `make uninstall` 对象。<br/>
要添加 `make uninstall` 对象，可以参考 [Can I do "make uninstall" with CMake?](https://gitlab.kitware.com/cmake/community/-/wikis/FAQ)。

一个简单的做法是利用 install_manifest.txt 文件：

```cmake
sudo xargs rm < install_manifest.txt
```
### 构建类型

CMake 内置了几种构建类型，其原理就是向编译器传递一些标志（flags）：

| Levels         | Compiler Flags    |
|----------------|-------------------|
| Release        | `-O3 -DNDEBUG`    |
| Debug          | `-g`              |
| MinSizeRel     | `-Os -DNDEBUG`    |
| RelWithDebInfo | `-O2 -g -DNDEBUG` |

- 可以通过图形界面设置构建类型：

{% asset_img "Configuring-Using-Cmake-GUI.png" "Configuring Using Cmake GUI'Configuring Using Cmake GUI'" %}

- 也可以在终端命令中定义变量：
    ```shell
    cmake .. -DCMAKE_BUILD_TYPE=Release
    ```

CMake 提供的默认的构建类型不包含任何相关的编译器标志，以优化程序。<br/>
但是有时可能会需要设置默认构建类型，可以在顶层 CMakeList.txt 中添加如下内容：

```cmake
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message("Setting build type to 'RelWithDebInfo' as none was specified.")
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
endif()
```

### 编译器标志

CMake 支持用若干种方式设置编译器标志：
- 使用函数 [`target_compile_definitions()`](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html) 和 [`target_compile_options()`](https://cmake.org/cmake/help/latest/command/target_compile_options.html)
- 使用变量 `CMAKE_C_FLAGS` 和 `CMAKE_CXX_FLAGS`，见 [`CMAKE_<LANG>_FLAGS`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS.html)

#### 为指定目标设置

在现代 CMake 中设置 C++ 标志的推荐方法是使用按对象标志（per-target flags），可以通过 `target_compile_definitions()` 函数将其填充到其他对象。

```cmake
target_compile_definitions(cmake_examples_compile_flags
     PRIVATE EX3
)
```

这会使编译器在变异目标（`cmake_examples_compile_flags`）时添加 `-DEX3` 标志。

使用该函数会填充 **库** 的 [`INTERFACE_COMPILE_DEFINITIONS`](https://cmake.org/cmake/help/v3.0/prop_tgt/INTERFACE_COMPILE_DEFINITIONS.html#prop_tgt:INTERFACE_COMPILE_DEFINITIONS)
变量，并会根据作用域（scope）将定义传递给链接的对象。<br/>
如果目标是一个库并且选择的作用域是 `PUBLIC` 或 `INTERFACE`，那么定义还会被包含在链接到该库的所有对象中。

#### 设置编译器标志

`CMAKE_CXX_FLAGS` 要么为空，要么包含所选构建类型的对应标志。<br/>
要设置 **额外** 的编译标志，可以在顶层 CMakeLists.txt 中添加如下内容：

```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DEX2" CACHE STRING "Set C++ Compiler Flags" FORCE)
```

{% note info %}
`CACHE STRING "Set C++ Compiler Flags" FORCE` 用来强制变量在 CMakeLists.txt 中被设置，详见 [`set`#设置缓存项](https://cmake.org/cmake/help/latest/command/set.html#set-cache-entry)。
{% endnote %}


类似地，`CMAKE_C_FLAGS` 等变量也可以以同样的方式设置。其他变量见 [`CMAKE_<LANG>_FLAGS`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS.html)。<br/>
一旦设置了这些变量，CMake 会为此目录及其子目录下的所有目标设置编译器标志，因此更推荐使用 `target_compile_definitions` 函数。

#### 设置链接器标志

参考：[How do I add a linker or compile flag in a CMake file?](https://stackoverflow.com/questions/11783932/how-do-i-add-a-linker-or-compile-flag-in-a-cmake-file)

先将标志储存为常量（现在不推荐这样做）：

```cmake
SET(GCC_COVERAGE_COMPILE_FLAGS "-fprofile-arcs -ftest-coverage")
SET(GCC_COVERAGE_LINK_FLAGS    "-lgcov")
```

然后可以通过几种方式添加它们：
1. 最简单的方式（not clean, but easy and convenient, and works only for compile flags, C & C++ at once）：
    ```cmake
    add_definitions(${GCC_COVERAGE_COMPILE_FLAGS})
    ```
2. 追加到现有的 CMake 变量：
    ```cmake
    SET(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} ${GCC_COVERAGE_COMPILE_FLAGS}")
    SET(CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} ${GCC_COVERAGE_LINK_FLAGS}") 
    ```
3. 使用目标属性（target properties）（需要指定目标名）[COMPILE_FLAGS](https://cmake.org/cmake/help/latest/prop_tgt/COMPILE_FLAGS.html)
    ```cmake
    get_target_property(TEMP ${THE_TARGET} COMPILE_FLAGS)
    if(TEMP STREQUAL "TEMP-NOTFOUND")
      SET(TEMP "") # Set to empty string
    else()
      SET(TEMP "${TEMP} ") # A space to cleanly separate from existing content
    endif()
    # Append our values
    SET(TEMP "${TEMP}${GCC_COVERAGE_COMPILE_FLAGS}" )
    set_target_properties(${THE_TARGET} PROPERTIES COMPILE_FLAGS ${TEMP} )
    ```

### 第三方库

CMake 提供了 [`find_package()`](https://cmake.org/cmake/help/latest/command/find_package.html) 函数来寻找第三方软件包文件所在路径。<br/>
该函数会在 [`CMAKE_MODULE_PATH`](https://cmake.org/cmake/help/latest/variable/CMAKE_MODULE_PATH.html) 中的目录列表中的目录中寻找形似 "FindXXX.cmake" 的文件。Linux 中的默认搜索路径为 `/usr/share/cmake/Modules`（或者 `/usr/share/cmake-3.20/Modules`）。

如要寻找 Boost：

```cmake
find_package(Boost 1.46.1 REQUIRED COMPONENTS filesystem system)
```

- `Boost`：包名，这会使 `find_package()` 函数寻找 FindBoost.cmake
- `1.46.1`：Boost 的最小版本
- `REQUIRED`：指定为必须，如果没找到符合要求的包则构建失败
- `COMPONENTS <...>`：库的组件列表

#### 检查是否找到软件包

大多数软件包（的 FindXXX.cmake 文件）会设置 `XXX_FOUND` 变量，可以通过检查该变量判断软件包是否被找到。

```cmake
if(Boost_FOUND)
    message ("boost found")
    include_directories(${Boost_INCLUDE_DIRS})
else()
    message (FATAL_ERROR "Cannot find Boost")
endif()
```

#### 暴露变量

找到指定的包（其实是加载了指定 FindXXX.cmake 文件）后，我们可以获得一些变量来帮助我们确定所需文件位置，如 `Boost_INCLUDE_DIRS` 就指明了 Boost 的包含文件所在目录。

有时可以通过 ccmake 或 cmake-gui 查看环境变量以确定所需变量。

#### 别名和导入目标

大多数 CMake 库，在其模块文件中导出别名目标。

Boost 的所有导出目标以 `Boost::<subsystem>` 命名，如：
- `Boost::boost`：仅头文件（header-only）的库
- `Boost::system`：Boost system library
- `Boost::filesystem`：文件系统相关的库

```cmake
target_link_libraries(third_party_include
    PRIVATE
        Boost::filesystem
)
```


这些目标会包含了它们的依赖，链接 `Boost::filesystem` 会自动添加 `Boost::boost` 和 `Boost::system` 依赖。

#### 非别名目标

有些第三方库没有使用可导入的目标，这时就必须使用以下两个变量：
- `xxx_INCLUDE_DIRS`：include directory
- `xxx_LIBRARY`：library path

```cmake
# Include the boost headers
target_include_directories( third_party_include
    PRIVATE ${Boost_INCLUDE_DIRS}
)

# link against the boost libraries
target_link_libraries( third_party_include
    PRIVATE
    ${Boost_SYSTEM_LIBRARY}
    ${Boost_FILESYSTEM_LIBRARY}
)
```


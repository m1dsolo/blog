---
title: '在 C++ 中使用 CMake 构建专属于自己的轮子库'
date: '2025-02-21'
tags: ['C++', 'CMake']
draft: false
keywords: [
    'C++',
    'CMake',
    'GoogleTest',
]
---

## 1. 前言

掌握一门编程语言的关键之一就是多动手写代码。
当我不知道该写什么代码时，通常会选择造轮子。
造轮子的过程中不仅能让我更加熟悉语言的特性，还能让我深入思考这些轮子的底层实现原理。
同时，造好的轮子也可以用于以后的项目中，从而提高开发效率。

C++ 是一门非常适合造轮子的语言，搭配上 CMake 这个构建工具，可以轻松构建出一个专属于自己的轮子库。
这样在以后的项目中，只需要引入这个库，就可以使用自己造的轮子了。

例如，当我曾经实现了一个线程池，未来如果要使用它，
只需要 `include` 我的 `wheel` 库：
```cpp
#include <wheel/thread_pool.hpp>

int main() {
    wheel::ThreadPool thread_pool(4);

    ...
}
```
本文将会用一个简单的例子来介绍如何使用 C++ 和 CMake 来构建一个专属于自己的轮子库。

## 2. 例子介绍

本文的目标是实现一个加法器，输入两个整数，加法器会输出它们的和。
这个加法器用到了`wheel`库中的`math.hpp`文件里面的`add`函数。

## 3. 构建轮子库

首先来看一下最终的 `wheel` 库目录结构：
```bash
wheel
├── CMakeLists.txt
├── include
│   └── wheel
│       └── math.hpp
└── src
    └── math.cpp
```

约定`include/wheel`内存的头文件是对外公开的接口，`src`内存源文件（也可以存私有头文件），`test`内存的测试文件。

接下来我们分别来看一下这几个文件的内容。

`include/wheel/math.hpp`定义要暴露的接口：
```cpp
#pragma once

namespace wheel {

int add(int a, int b);

}  // namespace wheel
```
{{< notice info >}}
任何 C++ 项目都要有单独的命名空间，以避免与其他库命名冲突。
{{< /notice >}}


`src/math.cpp`实现接口的功能:
```cpp
#include <wheel/math.hpp>

namespace wheel {

int add(int a, int b) {
    return a + b;
}

}  // namespace wheel
```

`CMakelists.txt`文件内容如下：
```cmake
cmake_minimum_required(VERSION 3.16)

option(BUILD_WHEEL_TEST "build wheel test" OFF)

set(TARGET wheel)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)

project(
    ${TARGET}
    VERSION 0.1.0
    DESCRIPTION "wheel"
    HOMEPAGE_URL "https://github.com/m1dsolo/wheel"
    LANGUAGES C CXX
)

file(GLOB_RECURSE SRC CONFIGURE_DEPENDS "src/*.cpp")
add_library(${TARGET} OBJECT ${SRC})
target_include_directories(${TARGET} PUBLIC include)

if (BUILD_WHEEL_TEST)
    add_subdirectory(test)
endif()
```

如果有看不懂的选项，可以让 GPT 帮你讲解～

接下来使用以下命令来进行构建和编译：
```bash
cmake -B build -G Ninja
cmake --build build -j8
```

## 4. 使用轮子库

创建一个新项目`adder`，其目录结构如下：
```bash
adder
├── CMakeLists.txt
├── src
│   └── main.cpp
└── third_party
    └── wheel
```

`src/main.cpp`内容如下：
```cpp
#include <iostream>

#include <wheel/math.hpp>

int main() {
    int res = wheel::add(1, 2);
    std::cout << res << std::endl;

    return 0;
}
```

`CMakeLists.txt`内容如下：
```cmake
cmake_minimum_required(VERSION 3.16)

set(TARGET adder)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)

project(
    ${TARGET}
    VERSION 0.1.0
    DESCRIPTION "adder"
    HOMEPAGE_URL "https://github.com/m1dsolo/adder"
    LANGUAGES C CXX
)

set(WHEEL_ROOT ${PROJECT_SOURCE_DIR}/third_party/wheel)

add_subdirectory(${WHEEL_ROOT})

file(GLOB_RECURSE SRC CONFIGURE_DEPENDS "src/*.cpp")
add_executable(${TARGET} ${SRC})
target_include_directories(${TARGET}
    PRIVATE include
    PRIVATE ${WHEEL_ROOT}/include
)
target_link_libraries(${TARGET}
    PRIVATE wheel
)
```
也就是编译时同时需要编译并链接`wheel`库。

接下来构建并编译`adder`程序：
```bash
cmake -B build -G Ninja
cmake --build build -j8
```
编译后我们就可以成功运行`adder`了：
```bash
./build/adder
```

可以看到终端上输出了我们的预期结果。

## 5. 测试轮子库（可选）

轮子库本质上就是一个 C++项目，推荐在写好轮子后，写一些测试用例来验证轮子的正确性。
这里我们选择使用 [GoogleTest](https://github.com/google/googletest) 来进行测试。

在`wheel`内添加：
```bash
wheel
├── test
│   ├── CMakeLists.txt
│   └── math.cpp
└── third_party
```

安装 GoogleTest 到`third_party/googletest`：
```bash
git submodule add --depth=1 https://github.com/google/googletest.git third_party/googletest
```

在`test/CMakeLists.txt`中添加：
```cmake
set(TARGET test)
set(GTEST_ROOT ${PROJECT_SOURCE_DIR}/third_party/googletest)

add_subdirectory(${GTEST_ROOT} ${CMAKE_BINARY_DIR}/googletest)

enable_testing()

file(GLOB_RECURSE SRC "*.cpp")
add_executable(${TARGET} ${SRC})
target_include_directories(${TARGET}
    PRIVATE ${GTEST_ROOT}/include
    PRIVATE ${PROJECT_SOURCE_DIR}/include
)
target_link_libraries(${TARGET}
    PRIVATE GTest::gtest_main
    PRIVATE wheel
)

add_test(NAME ${TARGET} COMMAND ${TARGET})
```

在`test/math.cpp`添加对 math 的测试用例：
```cpp
#include <wheel/math.hpp>

#include <gtest/gtest.h>

namespace wheel {

TEST(MathTest, AddFunction) {
    EXPECT_EQ(wheel::add(1, 1), 2);
    EXPECT_EQ(wheel::add(-1, 1), 0);
    EXPECT_EQ(wheel::add(-1, -1), -2);
    EXPECT_EQ(wheel::add(0, 0), 0);
}

}  // namespace wheel
```

由于之前我们已经配置了`BUILD_WHEEL_TEST`这个 option ，所以只需要在构建项目的时候指定这个参数：
```bash
cmake -B build -G Ninja -DBUILD_WHEEL_TEST=ON
cmake --build build -j8
```

编译成功后运行测试：
```bash
./build/test/test
```

## 6. 结语

在完成轮子库的构建后，建议撰写一些笔记或博客，记录下整个过程和思考。
这不仅有助于巩固所学知识，还能在未来需要时快速回顾和使用这些轮子库。
通过不断地实践和总结，能够更深入地理解编程语言和工具的特性，从而提升开发效率和代码质量。

你可以参考我分享的 [轮子库](https://github.com/m1dsolo/wheel) ，
代码仅供学习参考，不建议直接用于生产环境。

以后有时间，我会分享一些曾经造过的轮子的实现原理，敬请期待！

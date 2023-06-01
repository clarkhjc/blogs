---
title: cmake基本用法
date: 2023-06-01 17:33:33
tags:
- cmake
---

通过一个例子，熟悉常用的cmake命令，这个例子包括：

- 具有多个、多层的目录结构
- 编译静态库并链接

目录树如下：
![cmake-1](cmake-1.png)

helplib提供一个函数，函数返回帮助信息；printlib提供一个函数，负责打印信息到终端。

tool_help.cpp调用helplib中的函数；tool_print.cpp调用printlib中的函数。

在主目录生成CMakeLists.txt文件，内容如下：
```
# cmake版本要求
cmake_minimum_required(VERSION 2.8...3.24)

# 项目名称
project(CmakeSample)

# 版本变量
set(VERSION_MAJOR 1)
set(VERSION_MINOR 2)

# 生成配置文件
configure_file(
    "${PROJECT_SOURCE_DIR}/config.h.in"
    "${PROJECT_BINARY_DIR}/config.h"
)
include_directories("${PROJECT_BINARY_DIR}")

# 添加helplib子目录
add_subdirectory("${PROJECT_SOURCE_DIR}/helplib")
include_directories("${PROJECT_SOURCE_DIR}/helplib")

# 添加printlib子目录
add_subdirectory("${PROJECT_SOURCE_DIR}/printlib")
include_directories("${PROJECT_SOURCE_DIR}/printlib")

# 定义静态库列表变量
set(EXTRA_LIBS ${EXTRA_LIBS} helplib)
set(EXTRA_LIBS ${EXTRA_LIBS} printlib)

# 获取当前目录源文件列表
aux_source_directory(. DIR_SRCS)

# 添加目标文件
add_executable(CmakeSample ${DIR_SRCS})

# 链接静态库
target_link_libraries(CmakeSample ${EXTRA_LIBS})
```
我们将程序版本号写到配置文件中，方便修改，在主目录下编写config.h.in文件，内容如下:
```
#define PROJECT_VERSION_MAJOR @VERSION_MAJOR@
#define PROJECT_VERSION_MINOR @VERSION_MINOR@
```

helplib目录下的CMakeLists.txt文件，内容如下：
```
aux_source_directory(. DIR_SRCS)
add_library(helplib ${DIR_SRCS})
```

printlib目录下的CMakeLists.txt文件，内容如下：
```
aux_source_directory(. DIR_SRCS)
add_library(printlib ${DIR_SRCS})
```

新的目录结构如下：
![cmake-2](cmake-2.png)

我们在主目录下新建build目录，并进入build目录执行`cmake ..`
![cmake-3](cmake-3.png)

构建成功后，编译并执行程序:
![cmake-4](cmake-4.png)
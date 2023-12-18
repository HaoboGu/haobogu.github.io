---
title: "Learn Cmake"
author: "Haobo Gu"
tags: [makefile, c, cmake]
date: 2023-07-04T23:06:12+08:00
summary: Learn Cmake
draft: true
---

# Cmake

Cmake相当于是一个makefile generator，Cmake里面封装了很多常用的操作，因此写起来比makefile简单不少。话不多说，直接看如何使用。

## CMake 命令

### 基础设置
首先，CMake的核心文件是 `CmakeLists.txt`。在文件中，首先会列举一些必要的信息，如要求的最小CMake版本、工程名称等：
```cmake
cmake_minimum_required(VERSION 3.22)
project(your_project_name)

# 也可以声明语言
project("my_project" C CXX ASM)
```

使用`project()`的时候，同时会定义下面几个变量，后面可以直接使用：

- PROJECT_NAME: 就是设置的工程名
- CMAKE_PROJECT_NAME: 顶层工程的名称。如果当前的CMakeLists.txt是第一个，那么这个值会更新。否则，说明当前的CMakeLists.txt是子目录的，则不会更新这个值

在CMake中，还可以设置变量，用于后面使用：
```cmake
# 设置var变量，值为value
set(var value) 

# 使用变量，message命令是打印信息
message("变量是： ${var}")
```

### 编译目标

然后就是确定编译目标，这里使用`add_executable()`来定义编译目标为executable，即可执行文件。同理，还有`add_library()`，即编译为库。编译目标可以就设置为上面定义的${PROJECT_NAME}:

```cmake
# 设置编译目标
add_executable(${PROJECT_NAME})

# 设置编译目标并且添加源文件
add_executable(${PROJECT_NAME} main.c)
```

注意这里可以只设置编译目标，不添加对应的源文件。这是因为源文件可以在后面添加。此时，添加源文件就需要使用cmake的其他命令了。

```
add_executable(${PROJECT_NAME})
# 在声明编译目标之后，添加源文件
target_sources(runtest${PROJECT_NAME} main.c)
```

### 添加文件

然后就是添加文件了。如果提前使用`add_executable()`定义好了编译目标，那么可以使用`target_include_directories()`和`target_sources()`添加对应的头文件目录和源文件

```cmake
target_sources(${PROJECT_NAME}
    PRIVATE
        bar.c
        bar.h
)

target_include_directories(${PROJECT_NAME} 
	PRIVATE
		"/path/to/include"
)
```

这里，第一个参数是编译目标的名称，第二个参数是可见范围，如果不是要编译一个别人也能用的库，一般都是PRIVATE，然后后面就是对应的源文件列表，以及头文件的目录了

### 编译、链接选项

添加完文件之后，还需要添加编译和链接选项，还有一些定义。使用下面的两个函数添加：

```cmake
# 添加编译选项
target_compile_options(
    ${TARGET_NAME} PRIVATE
    g3
)

# 添加链接选项
target_link_options(${PROJECT_NAME} PRIVATE
    -mcpu=cortex-m7
    -mfpu=fpv5-d16
    -mfloat-abi=hard
)

# 添加定义
target_compile_definitions(${PROJECT_NAME} PRIVATE
    DEBUG # 等同与添加了-DDebug
)
```

## 进阶语法

在完成上面的配置之后，应该是已经可以完成项目的编译了。为了让整个编译流程和CMakeLists.txt文件更清晰，下面介绍一些进阶的内容。

### 生成器表达式

在上面添加定义的时候，如果我想在Debug的时候，添加-DDebug，而在生成Release的时候不加-DDebug，此时就需要生成器表达式了。生成器表达式的格式是：`$<>`。在<>里面，大写的条件是内置的，小写的内容是自定义的。可以添加如下的格式：

- 逻辑运算

    ```cmake
    # 如果value的值为空（0、空字符串、etc），那么整个表达式的值为0，否则为1
    $<BOOL:value> 
    # 和BOOL类似，如果后面的所有condition都为真，则返回1，否则0
    $<AND:condition1,condition2,...>
    ```

- 条件表达式

    ```cmake
    # 如果condition为真，则结果为后面的true_string，否则为空
    $<condition:true_string>
    # 如果condition为真，返回v1，否则v2
    $<IF:condition,v1,v2>
    ```

- （内置）变量查询

    ```cmake
    # 判断编译类型配置是否包含在cfgs列表（比如"release,debug"）中，不区分大小写
    $<CONFIG:cfgs>
    ```

使用生成器表达式，就可以把Debug和Release的配置分开了。比如我想设置优化等级，Debug的时候为-O0，Release时为-O2，可以这么写：
```cmake
add_compile_options(${PROJECT_NAME}
    PRIVATE
    "$<$<CONFIG:debug>:-O0>"
    "$<$<CONFIG:release>:-O2>"
)

# 也可以用if表达式，设置release为-O2，其他所有情况都是-O0
add_compile_options(${PROJECT_NAME}
    PRIVATE
    "$<IF:$<CONFIG:release>,-O2,-O0>"
)
```

### 配置子目录

在一些情况下，需要添加第三方的库，这些库的目录下也有一个CMakeLists.txt。在这种情况下，我们可以使用

```cmake
add_subdirectory(sub_dir)
```

来添加子目录，以及相应的CMakeLists.txt

### 一次性添加所有源文件

在使用`target_sources`的时候，我们只能一个文件一个文件地添加。CMake同样提供的一次性添加的方法，可以把一个目录下的所有文件，全都添加进来：

```cmake
# 查找目录path下的所有源文件，并将名称保存到 PATH_SRCS 变量
aux_source_directory(path PATH_SRCS)
```

需要注意的是，aux_source_directory不是递归的，如果需要递归地查找，则使用：

```cmake
# 递归地搜索 src 目录下所有的.c文件，并且添加到MY_SRC变量中
file(GLOB_RECURSE MY_SRCS src/*.c)
```

在添加完毕之后，使用
```cmake
target_sources(${PROJECT_NAME}
    PRIVATE
        ${PATH_SRCS}
        ${MY_SRCS}
)
```

就可以把这些文件都添加到源文件列表中了


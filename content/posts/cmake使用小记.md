+++
title = 'Cmake使用小记'
date = 2024-08-06T22:59:39+08:00
draft = false
description = "Cmake"
tags = [
    "CMAKE",
]

+++
# CMAKE 小记

最近参与一个小的项目，由于之前没有仓库承载代码，所以建立了新仓，从零开始搭建了构建工程，也算学习了部分cmake的知识，因此在这里记录下。本篇wiki主要记录的是一些踩坑的地方，常规用法不再这里赘述。:)

### 1. CMAKE\_\<LANG\>\_COMPILER

以c语言为例，cmake_c_compiler变量的作用是为了指明编译c语言的编译器，一般来说此变量的设置是跟随toolchain的文件或者通过-D CLI来设置；但也可以通过在CMakeLists.txt中set(CMAKE_C_COMPILER xxx)来设置。

##### NOTE: 

在CMakeLists.txt中set(CMAKE_C_COMPILER xxx)来设置编译器的话，需要在project(xxx)前面；且一旦设置，此project不可改变。

### 2. ExternalProject_Add

此项目有个特殊点在于交付件有一个动态库xxx.so，一个驱动xxx.ko。编译动态库的时候需要用clang编译器，编译驱动的时候需要gcc编译器，这就涉及了cmake_c_compiler的设置；但是由于一个project设置编译器后不能改变，我又想方便本地编译，所以就采用了ExternalProject_Add接口。

此接口的作用是可以在一个project内使用外部扩展的project，这样我就可以分别设置cmake_c_compiler，互不影响。

当然想做到同样效果还有其他方式，比如通过-D cmake_c_compiler设置，然后再通过另一选项-D xxx来控制编译驱动还是动态库。

### 3. add_library

add_library的常规用法是添加一个target为STATIC静态库或者SHARED动态库，它还有一种用法是添加Object Library。由于项目的其中一个交付件为xxx.so，那么自然的其有对外的API；在编写ST或者UT用例时，我们可以选择直接把动态库源码编进来，也可以选择链接xxx.so。

考虑第一种场景，我们就需要：

```cmake
add_executable(st st1.c xxx1.c xxx2.c ... xxxn.c)
```
如果n过于多，显得就很繁琐，这时候就可以使用:
```cmake
add_library(objlib OBJECT xxx1.c xxx2.c ... xxxn.c)
add_executable(st st1.c $<TARGET_OBJECTS:objlib>)
```

当我们需要编写很多ST用例时，就会方便简洁很多。


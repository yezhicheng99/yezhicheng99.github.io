---
title: cmake常用指令
date: 2022-05-05 18:34:20
categories:
tags:
top_img:
---
在<b>构建</b>一个C++项目工程时，通常需要使用cmake来写CMakeLists.txt文件来管理工程的编译和运行。

# cmake_minimum_required

- 设置cmake版本号：
``` cmake
cmake_minimum_required (VERSION 2.6) # 设置cmake最低版本号
cmake_minimum_required　(VERSION 3.12...3.15) # 设置允许的cmake版本号范围
```

- 一个最简单的CMakeLists.txt文件内容如下：
``` cmake
cmake_minimum_required (VERSION 2.6)
project (Tutorial)
add_executable(Tutorial tutorial.cxx)
```

# project

使用语法：project(项目名字, [VERSION 版本号] [DESCRIPTION 描述字符串] )

- 举例
``` cmake
project(myproject)
project(myproject, VERSION 1.0.0 DESCRIPTION "my first project" )
```

# set

使用语法：set(<variable> <value>）

- 添加c++ 11标准支持
``` cmake
set( CMAKE_CXX_FLAGS "-std=c++11" )
```

- 设置opencv路径
``` cmake
set(OpenCV_DIR "/usr/share/opencv")
```

# include_directories
添加头文件搜索路径

# link_directories
添加库文件搜索路径

# find_package

- 执行后会定义以下变量
<name>_FOUND
<name>_INCLUDE_DIRS or <name>_INCLUDES
<name>_LIBS or <name>_LIBRARIES

- 如果找不到对应库，添加如下命令，值为对应的.cmake库文件所在路径
``` cmake
set(OpenCV_DIR /opt/ros/kinetic/share/OpenCV-3.3.1-dev)
```

- find_package请和include_directories(${OpenCV_INCLUDE_DIRS}) target_link_libraries( test ${OpenCV_LIBS})配合食用，三步走

- 举例
``` cmake
FIND_PACKAGE(CURL)
IF(CURL_FOUND)
   INCLUDE_DIRECTORIES(${CURL_INCLUDE_DIR})
   TARGET_LINK_LIBRARIES(curltest ${CURL_LIBRARY})
ELSE(CURL_FOUND)
     MESSAGE(FATAL_ERROR ”CURL library not found”)
ENDIF(CURL_FOUND)
```
``` cmake
cmake_minimum_required( VERSION 2.8 )

project( imageBasics )

set( CMAKE_CXX_FLAGS "-std=c++11" ) # 添加c++ 11标准支持

#find_package( OpenCV 3.3 REQUIRED ) # 添加OPENCV库
find_package( OpenCV REQUIRED )

include_directories( ${OpenCV_INCLUDE_DIRS} ) # 添加OpenCV头文件
#include_directories(/usr/local/include) # 同上，找根目录
 
# 添加一个可执行程序
# 语法：add_executable( 程序名 源代码文件 ）
add_executable( main main.cpp )
 
# 将库文件链接到可执行程序上
target_link_libraries( main ${OpenCV_LIBS} )
```

# file

使用语法：file(GLOB variable 通配符表达式)

- 参数说明
GLOB: 查找所有在当前目录（CMakeLists.txt所在目录，即CMAKE_CURRENT_SOURCE_DIR变量）下所有满足匹配通配符表达式的文件
variable: 保存文件名的变量

- 举例
``` cmake
file(GLOB source_list "src/*.cpp")
```

# message
用于记录log消息

使用语法：message([<mode>] "message text" ...)

- mode:要记录的log的类型，包括
FATAL_ERROR: cmake errer, 遇到该错误，cmake 直接停止处理
WARNING: 警告信息， 但是cmake会继续执行下去的
STATUS: 用于记录一些cmake 运行过程中的有用的信息

- 举例
``` cmake
message(STATUS "hello, world.")
set(NAME xiaoming)
messaeg(STATUS "hello, ${NAME}")
```

# add_library
将所有指定的文件(target)打包成一个库

使用语法：add_library(<name> [STATIC | SHARED | MODULE] [EXCLUDE_FROM_ALL] [<source>...]) 

- 参数说明
name： 生成的目标库的名字
[STATIC | SHARED | MODULE]: STATIC 表示静态库，SHARED 表示动态库，MODULE表示模块库
EXCLUDE_FROM_ALL: 与add_executable命令中的参数相同，当指定该参数时，要构建的target 就会被排除在all target 列表之外
source: 源文件列表

- 举例
``` cmake
add_library(hello_library STATIC src/Hello.cpp)
```

# target_include_directories
当编译给定target时，该命令用于指令要包含的路径名

使用语法：target_include_directories(<target> [SYSTEM] [AFTER|BEFORE] <INTERFACE|PUBLIC|PRIVATE> [items1...] [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])

- 参数
AFTER 或 BEFORE: 控制把需要添加的路径append 还是 prepend 到已有的路径列表中
INTERFACE/PUBLIC/PRIVATE: 使用PRIVATE关键字时，添加的路径只会在编译当前target时使用到。当使用INTERFACE关键字时，添加的路径只会在编译任何链接到当前target的目标文件时使用到。使用PUBLIC关键字时，会包含以两种使用。因此，通常情况下，如果target是可执行文件，建议使用PRIVATE，如果target是动态或静态库，建议使用PUBLIC，比较方便。

- 举例
``` cmake
target_include_directories(mylib PUBLIC include/1.h PUBLIC ${PROJECT_SOURCE_DIR}/include)
target_include_directories(mytarget PRIVATE ${PROJECT_SOURCE_DIR}/include)
```

# add _executable

使用语法：add_executable(name [source1] [source2] ...)

- 举例
``` cmake
add_executable(main main.cpp utils.cpp)
```

# target_link_libraries
指定一个target的链接阶段的依赖库

使用语法：target_link_libraries(<target> <PRIVATE|PUBLIC|INTERFACE> <item>...[<PRIVATE|PUBLIC|INTERFACE> <item>...]...)

- 举例
``` cmake
target_link_libraries( hello_binary PRIVATE hello_library)
```

# add_subdirectory
增加子目录
- 举例
``` cmake
add_subdirectory(subdir1)
add_subdirectory(subdir2)
```



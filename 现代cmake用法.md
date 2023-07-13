### 为什么要学习现代CMake

传统CMake构建方式
```cpp
mkdir build         // 需要先创建 build 目录
cd build            // 切换到 build 目录
cmake ..            // 在 build 目录运行 cmake <源码目录> 生成 Makefile
make -j4            // 执行本地的构建系统 make 真正开始构建（4进程并行）
sudo make install   // 让本地的构建系统执行安装步骤
cd ..               // 回到源码目录
```

现代CMake例子
可见 CMake 项目的构建分为两步：
第一步是 **cmake -B build**，称为配置阶段（configure），这时只检测环境并生成构建规则
**会在 build 目录下生成本地构建系统能识别的项目文件（Makefile 或是 .sln）**
第二步是 **cmake --build build**，称为构建阶段（build），这时才实际调用编译器来编译代码
* -D选项
在**配置阶段**可以通过 -D 设置缓存变量。第二次配置时，之前的 -D 添加仍然会被保留。
cmake -B build -DCMAKE_INSTALL_PREFIX=/opt/openvdb-8.0
↑设置安装路径为 /opt/openvdb-8.0（会安装到 /opt/openvdb-8.0/lib/libopenvdb.so）
cmake -B build -DCMAKE_BUILD_TYPE=Release
↑设置构建模式为发布模式（开启全部优化）
cmake -B build   ←第二次配置时没有 -D 参数，但是之前的 -D 设置的变量都会被保留
（此时缓存里仍有你之前定义的 CMAKE_BUILD_TYPE 和 CMAKE_INSTALL_PREFIX）
* -G选项
指向要用的生成器
Linux 系统上的 CMake 默认用是 Unix Makefiles 生成器；Windows 系统默认是 Visual Studio 2019 生成器；MacOS 系统默认是 Xcode 生成器。

性能上：Ninja > Makefile > MSBuild
Makefile 启动时会把每个文件都检测一遍，浪费很多时间。特别是有很多文件，但是实际需要构建的只有一小部分，从而是 I/O Bound 的时候，Ninja 的速度提升就很明显。
然而某些专利公司的 CUDA toolkit 在 Windows 上只允许用 MSBuild 构建，不能用 Ninja
<img src="../pic/cmake-1.png" style="zoom:60%" align="center"/>
<img src="../pic/cmake-2.png" style="zoom:60%" align="center"/>

### 添加一个源文件
```cpp
// CMake 中添加一个可执行文件作为构建目标
add_executable(main main.cpp)

// 另一种方式：先创建目标，稍后再添加源文件
add_executable(main)
target_sources(main PUBLIC main.cpp)

// 多个源文件，逐个添加
add_executable(main)
target_sources(main PUBLIC main.cpp other.cpp)

// 多个源文件，创建变量
add_executable(main)
// set(sources main.cpp other.cpp)
// 建议把头文件也加上
set(sources main.cpp other.cpp other.h)
target_sources(main PUBLIC ${sources})

// 使用 GLOB 自动查找当前目录下指定扩展名的文件，实现批量添加源文件
add_executable(main)
file(GLOB sources *.cpp *.h)
target_sources(main PUBLIC ${sources})

// 启用 CONFIGURE_DEPENDS 选项，当添加新文件时，自动更新变量
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})

// 如果源码放在子文件夹里怎么办
// 全写出来？
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h mylib/*.cpp mylib/*.h)
target_sources(main PUBLIC ${sources})

// 可以用aux_source_directory 自动搜集需要的文件后缀名
add_executable(main)
aux_source_directory(. sources)
aux_source_directory(mylib sources)
target_sources(main PUBLIC ${sources})

// 建议放到src下
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS src/*.cpp)
target_sources(main PUBLIC ${sources})
```

### 项目的配置变量
* CMAKE_BUILD_TYPE 构建的类型，调试模式还是发布模式
  CMAKE_BUILD_TYPE 是 CMake 中一个特殊的变量，用于控制构建类型，他的值可以是：
Debug 调试模式，完全不优化，生成调试信息，方便调试程序
Release 发布模式，优化程度最高，性能最佳，但是编译比 Debug 慢
MinSizeRel 最小体积发布，生成的文件比 Release 更小，不完全优化，减少二进制体积
RelWithDebInfo 带调试信息发布，生成的文件比 Release 更大，因为带有调试的符号信息
默认情况下 CMAKE_BUILD_TYPE 为空字符串，这时相当于 Debug。

#### **Release模式**
在Release模式下，追求的是程序的最佳性能表现，在此情况下，编译器会对程序做最大的代码优化以达到最快运行速度。另一方面，由于代码优化后不与源代码一致，此模式下一般会丢失大量的调试信息。

```cpp
cmake_minimum_required(VERSION 3.15)
project(hellocmake LANGUAGES CXX)

set(CMAKE_BUILD_TYPE Release)

add_executable(main main.cpp)
```

大多数 CMakeLists.txt 的开头都会有这样三行，为的是让默认的构建类型为发布模式（高度优化）而不是默认的调试模式（不会优化）。

#### **project**
初始化项目信息，并把当前 CMakeLists.txt 所在位置作为根目录

PROJECT_SOURCE_DIR：当前项目源码路径（存放main.cpp的地方）
PROJECT_BINARY_DIR：当前项目输出路径（存放main.exe的地方）
CMAKE_SOURCE_DIR：根项目源码路径（存放main.cpp的地方）
CMAKE_BINARY_DIR：根项目输出路径（存放main.exe的地方）
PROJECT_IS_TOP_LEVEL：BOOL类型，表示当前项目是否是（最顶层的）根项目
PROJECT_NAME：当前项目名
CMAKE_PROJECT_NAME：根项目的项目名

* **Language**字段
默认
```cpp
project(hellocmake LANGUAGES C CXX)

```
* **set C++ standard**
最好是在 project 指令前设置 CMAKE_CXX_STANDARD 这一系列变量，这样 CMake 可以在 project 函数里对编译器进行一些检测，看看他能不能支持 C++17 的特性
```cpp
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON) // 可以为 ON 或 OFF，默认 OFF。发现不支持报错，更安全
set(CMAKE_CXX_EXTENSIONS ON)        // OFF 则关闭 GCC 的扩展功能，只使用标准的 C++
// 要兼容其他编译器（如 MSVC）的项目，都会设为 OFF 防止不小心用了 GCC 才有的特性。

// 手动添加 -std=c++17 会有兼容性问题
```

* **VERSION**字段
**project(项目名 VERSION x.y.z) 可以把当前项目的版本号设定为 x.y.z**
之后可以通过 PROJECT_VERSION 来获取当前项目的版本号。
PROJECT_VERSION_MAJOR 获取 x（主版本号）。
PROJECT_VERSION_MINOR 获取 y（次版本号）。
PROJECT_VERSION_PATCH 获取 z（补丁版本号）。

**项目名的另一大作用：会设置另外 <项目名>_SOURCE_DIR 等变量**
```cpp
// cmake_minimum_required 指定最低所需的 CMake 版本
cmake_minimum_required(VERSION 3.15)
project(hellocmake VERSION 0.2.3)

message("PROJECT_NAME: ${PROJECT_NAME}")
message("PROJECT_VERSION: ${PROJECT_VERSION}")
message("PROJECT_SOURCE_DIR: ${PROJECT_SOURCE_DIR}")
message("PROJECT_BINARY_DIR: ${PROJECT_BINARY_DIR}")
message("hellocmake_VERSION: ${hellocmake_VERSION}")
// message("hellocmake_SOURCE_DIR: ${hellocmake_SOURCE_DIR}")
// message("hellocmake_BINARY_DIR: ${hellocmake_BINARY_DIR}")
// CMake 的 ${} 表达式可以嵌套
message("hellocmake_SOURCE_DIR: ${${PROJECT_NAME}_SOURCE_DIR}")
message("hellocmake_BINARY_DIR: ${${PROJECT_NAME}_BINARY_DIR}")
add_executable(main main.cpp)
```

通过 CMAKE_VERSION 这个变量来获得当前 CMake 版本号
CMAKE_MINIMUM_REQUIRED_VERSION 获取 cmake_minimum_required中指定最小所需版本号
```cpp
cmake --version

message("CMAKE_VERSION: ${CMAKE_VERSION}")
message("CMAKE_MINIMUN_REQUIRED_VERSION: ${CMAKE_MINIMUN_REQUIRED_VERSION}")
```

**其他指令**
CMAKE_BUILD_TOOL: 执行构建过程的工具。该变量设置为CMake构建时输出所需的程序。对于VS 6， CMAKE_BUILD_TOOL设置为msdev， 对于Unix，它被设置为make 或 gmake。 对于 VS 7， 它被设置为devenv. 对于Nmake构建文件，它的值为nmake。
CMAKE_DL_LIBS: 包含dlopen和dlclose的库的名称。
CMAKE_COMMAND: 指向cmake可执行程序的全路径。
CMAKE_CTEST_COMMAND: 指向ctest可执行程序的全路径。
CMAKE_EDIT_COMMAND: cmake-gui或ccmake的全路径。
CMAKE_EXECUTABLE_SUFFIX: 该平台上可执行程序的后缀。
CMAKE_SIZEOF_VOID_P: void指针的大小。
CMAKE_SKIP_RPATH: 如果为真，将不添加运行时路径信息。默认情况下是如果平台支持运行时信息，将会添加运行时信息到可执行程序当中。这样从构建树中运行程序将很容易。为了在安装过程中忽略掉RPATH，使用CMAKE_SKIP_INSTALL_RPATH。
CMAKE_GENERATOR: 构建工程的产生器。它将产生构建文件 (e.g. "Unix Makefiles", "Visual Studio 2019", etc.)


**例子**
```cpp
cmake_minimum_required(VERSION 3.15)

// C++版本
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

// 项目名称 启用的语言
project(zeno LANGUAGES C CXX)

// 检查源目录和输出目录是否相同
if (PROJECT_BINARY_DIR STREQUAL PROJECT_SOURCE_DIR)
    message(WARNING "The binary directory of CMake cannot be the same as source directory!")
endif()

// 构建类型Release模式
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

// windows缺少默认定义宏
if (WIN32)
    add_definitions(-DNOMINMAX -D_USE_MATH_DEFINES)
endif()

if (NOT MSVC)
// 让编译带有缓存 更快
    find_program(CCACHE_PROGRAM ccache)
    if (CCACHE_PROGRAM)
        message(STATUS "Found CCache: ${CCACHE_PROGRAM}")
        set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ${CCACHE_PROGRAM})
        set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ${CCACHE_PROGRAM})
    endif()
endif()
```

### 链接库文件

**add_executable**
* main.cpp 调用 mylib.cpp 里的 say_hello 函数
```cpp
add_executable(main main.cpp mylib.cpp)
```

**改进：mylib 作为一个静态库**
```cpp
add_library(mylib STATIC mylib.cpp)

add_executable(main main.cpp)

target_link_libraries(main PUBLIC mylib)
```

**改进：mylib 作为一个对象库**
// 不懂
对象库类似于静态库，但不生成 .a 文件，只由 CMake 记住该库生成了哪些对象文件
对象库是 CMake 自创的，绕开了编译器和操作系统的各种繁琐规则，保证了跨平台统一性。
在自己的项目中，我推荐全部用对象库(OBJECT)替代静态库(STATIC)避免跨平台的麻烦。
对象库仅仅作为组织代码的方式，而实际生成的可执行文件只有一个，减轻了部署的困难。

对象库可以绕开编译器的不统一：保证不会自动剔除没引用到的对象文件
```cpp

```

**add_library 无参数时，是静态库还是动态库**
会根据 BUILD_SHARED_LIBS 这个变量的值决定是动态库还是静态库。
ON 则相当于 SHARED，OFF 则相当于 STATIC。
如果未指定 BUILD_SHARED_LIBS 变量，则默认为 STATIC。
因此，如果发现一个项目里的 add_library 都是无参数的，意味着你可以用：
cmake -B build -DBUILD_SHARED_LIBS:BOOL=ON
来让他全部生成为动态库。
```cpp
set(BUILD_SHARED_LIBS ON)

// defacult value
if (NOT DEFINED BUILD_SHARED_LIBS)
    set(BUILD_SHARED_LIBS ON)
endif()
```

**常见坑点：动态库无法链接静态库**
动态比静态多了PIC代码，让静态动起来，出现reallocate错误
```cpp
add_library(otherlib STATIC otherlib.cpp)
// 只针对一个库，只对他启用位置无关的代码(PIC)
set_property(TARGET otherlib PROPERTY POSITION_INDEPENDENT_CODE ON)

add_library(mylib SHARED mylib.cpp)
target_link_libraries(mylib PUBLIC otherlib)

add_executable(main main.cpp)
target_link_libraries(main PUBLIC mylib)
```

### 对象的属性

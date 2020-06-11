# 编译相关

## 安全编译选项

### 栈保护

**说明：**避免栈溢出被攻击者利用，在缓存区和控制信息之间插入一个canary word。

**作用阶段：**编译选项

**参数：**`-fstack-protector-all/-fstack-protector-strong`

### GOT表保护

**说明：**动态链接的ELF二进制程序使用称为全局偏移表（GOT）的查找表去动态解析位于共享库中的函数。攻击者通过缓冲区溢出修改GOT表项的函数地址值来达到攻击的目的。通过增加RELRO选项，可以防止GOT表被恶意重写。

**作用阶段：**链接选项

**参数：**`-Wl,-z,relro`

### 立即执行BindNow

**说明：**开启部分重定项只读保护后，再开启立即绑定可实现全部重定向只读保护。

**作用阶段：**链接选项

**参数：**`-Wl,-z,now`

### 堆栈不可执行

**说明：**缓冲区溢出成功后都是通过执行shellcode来达到攻击的目的，而shellcode一般都是在缓存区里，只要操作系统限制堆栈中的数据只可读写，不可执行，一旦堆栈中的数据被执行立即报告错误并退出，那么溢出成功后也不能执行shellcode，从而达到对应用安全防护的目的。

**作用阶段：**链接选项

**参数：**`-Wl,-z,noexecstack `

### 地址无关

**说明：**地址无关选项将发生在代码段的重定位移到数据段实现，so文件加载时代码段不会发生任何变化，做到所有进程共用一个代码段副本。

**作用阶段：**编译选项

**参数：**`-fPIC`

### 随机化

**说明：**具备PIE的可执行文件，在加载执行时可像共享库一样随机加载。有研究表明：PIE可有效降低固定地址类攻击、缓冲溢出类攻击的成功概率。

**作用阶段：**编译、链接选项

**参数：**`-fPIE -pie`

### rpath

**说明：**主要用于防护LD_LIBRARY_PATH替换同名动态库的攻击。通过加入此选项可以指定一个运行时动态库搜索的路径，该路径的搜索优先级高于LD_LIBRARY_PATH指定的路径。

**作用阶段：**链接选项

**参数：**`-Wl,--disable-new-dtags,--rpath,/libpath1:/libpath2;-Wl,--enable-new-dtags,--rpath,/libpath1:/libpath2`

### Fortify Source

**说明：**程序中使用到静态的固定大小的缓冲区，增加了该选项之后，编译器或运行时库会对相关函数的调用在编译时或运行时进行检查。

**作用阶段：**编译选项

**参数：**`-D_FORTIFY_SOURCE=2 -O2`

### Visibility

**说明：**设置默认的ELF镜像中符号的可见性为隐藏。

**作用阶段：**编译选项

**参数：**`-fvisibility=hidden`

### 整型溢出

**说明：**使用了-ftrapv选项后，执行带符号的整数间的加、减、乘运算时，不是通过CPU的指令，而是用包含在GCC附属库libgcc.c里的函数来实现。

**作用阶段：**编译选项

**参数：**`-ftrapv`

### 栈检查

**说明：**stack-check在编译时检查程序中栈空间，如果超过编译告警阀值则产生告警；然后在程序中生成额外的指令来检查运行时栈不会被溢出，stack-check选项会在每个栈空间最低底部设置一个安全的缓冲区，如果函数中申请的栈空间进入安全缓冲区，则触发一个Storage_Error异常。但它所生成的代码实际上并不处理异常，如果检测到异常则会发出一个消息，通知操作系统处理。它只保证操作系统可以检测到栈扩展。

**作用阶段：**编译选项

**参数：**`-fstack-check`

### 符号表删除

**说明：**符号在链接过程中，发挥着至关重要的作用，链接过程的本质就是把多个不同的目标文件“粘”到一起，符号可看作链接的粘合剂，整个链接过程正是基于符号才正确完成的。链接完成后，符号表对可执行文件运行已经无任何作用，反而会成为攻击者构造攻击的工具，因此删除符号表可防御黑客攻击。事实上删除符号表除防攻击外，还可对文件减肥，降低文件大小。

**作用阶段：**链接选项

**参数：**`-s`

## CMake

### 日志

日志级别如下：

- `FATAL_ERROR`：cmake出错，停止编译和生成(信息红色)
- `SEND_ERROR`：cmake出错，继续编译，但是停止生成(信息红色)
- `WARNING`：cmake警告，继续编译(信息红色)
- `AUTHOR_WARNING`：开发者警告，继续编译(信息红色)
- `DEPRECATION`：如果使用set方法设置CMAKE_ERROR_DEPRECATED为true(不区分大小写)，编译出错，否则继续编译
- (none) or `NOTICE`：不设置mode，默认是NOTICE模式，不影响编译和生成，用于打印消息(信息白色)
- `STATUS`：编译时状态信息，左边以`--`开头(信息白色)
- `DEBUG`：针对开发人员的调试信息(信息白色)
- `TRACE`：日志级别的临时信息(信息白色)

日志样例：`message(FATAL_ERROR "Insufficient gcc version")`

### 环境变量

**当前目录：**`CMAKE_CURRENT_SOURCE_DIR`

### 语法

**函数定义：**

```shell
function(SET_COMPILE_OPTIONS)
    add_definitions(-fPIC)
endfunction(SET_COMPILE_OPTIONS)
```

**设置工程：**`project(xxx)`

**编译子目录：**`add_subdirectory(xx)`

**加载其他编译文件：**`include(xxx)`

**变量定义：**`set (LOCAL_LIB /usr/local/lib)`

**添加编译选项：**`add_denifition(-Wall -Werror)`

**搜索lib库：**`find_library(VAR_NAME lib_name)`

**搜索程序：**`find_program(PRO_NAME exe_name)`

**设置链接目录：**`link_directory(${link_path})`

**设置头文件目录：**`include_directories(SYSTEM ${include_dir})`

**获取源文件：**`file(GLOB SRCS [^.~]*.[hc][px][px] [^.~]*.[hc])`

**执行其他程序：**`execute_process(COMMAND sh ${CMAKE_CURRENT_SOURCE_DIR}/build.sh)`

### 控制

使用if/elseif/else进行分支处理：

```
if (${CMAKE_BUILD_TYPE} STREQUAL "RELEASE")
elseif (${CMAKE_BUILD_TYPE} STREQUAL "DEBUG")
else (${CMAKE_BUILD_TYPE} STREQUAL "CHECK")
endif ()
```

**使用for循环语句：**

```shell
foreach(sourcefile ${source_files})
    get_filename_component(basename "${sourcefile}" NAME)
endforeach()
```

### 输出

**编译动态库：**`add_library(${PROJECT_NAME} SHARED ${SRCS})`

**编译静态库：**`add_library(${PROJECT_NAME} STATIC ${SRCS})`

**设置链接库：**`target_link_libraries(${PROJECT_NAME} ${LIBRARY_LIST})`

**编译可执行程序：**`add_executable(${PROJECT_NAME} ${SRCS})`

*默认为动态链接，如果需求静态链接则显式指定库名为libxxx.a*

#### 静态链接顺序

静态库中，包含着所有的 obj(*.o) 文件，连接器从左至右搜索，维护着一个 **undefined 列表**，一旦遇到没有定义的内容，就会将它加到列表中，如果搜索到了定义的内容，则抽取出 obj 文件，进行链接，并将 undefined 内容移出列表，而其它 obj 文件就会被丢弃（为了减少最终的体积大小），于是**一个静态库如果不能在搜索过程中被链接，它就会被丢弃**，而在后面一旦遇到依赖它的库，就会造成引用无法被链接，一直留在**undefined 列表**中，最终导致编译错误。因此，**被依赖的库应该放在依赖库列表的后面**。

### 其他

**禁用rpath：**`set (CMAKE_SKIP_RPATH TRUE)`

**执行脚本：**`execute_process(COMMAND ${command} ${param})`

## make

**输出详细打印命令：**`make VERBOSE=1`

**并发编译：**`make -j8`

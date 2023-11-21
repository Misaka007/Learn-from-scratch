# Cmake基础教程

## 指定cmake版本

```cmake
cmake_minimum_required(VERSION 3.10)
```

## 定义工程名

```cmake
project(Example)

# PROJECT 指令的语法是：
project(<PROJECT-NAME> [<language-name>...])
project(<PROJECT-NAME>
       [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
       [DESCRIPTION <project-description-string>]
       [HOMEPAGE_URL <url-string>]
       [LANGUAGES <language-name>...])
```

## 指定c++版本（标准）

```cmake
#增加-std=c++11
set(CMAKE_CXX_STANDARD 11)
#增加-std=c++14
set(CMAKE_CXX_STANDARD 14)
#增加-std=c++17
set(CMAKE_CXX_STANDARD 17)
```

## 构建可执行文件

```cmake
add_executable(Example main.cpp add.c mult.c)
```

## 指定输出的路径

在CMake中指定可执行程序输出的路径，也对应一个宏，叫做EXECUTABLE_OUTPUT_PATH，它的值还是通过set命令进行设置

```cmake
# 定义一个变量用于存储一个绝对路径
set(HOME /home/robin/Linux/Sort)
# 将拼接好的路径值设置给EXECUTABLE_OUTPUT_PATH宏
# 如果这个路径中的子目录不存在，会自动生成，无需自己手动创建
set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin)
```

## 搜索文件

### 1. aux_source_directory

aux_source_directory 命令可以查找某个路径下的所有源文件

```cmake
# dir：要搜索的目录
# variable：将从dir目录下搜索到的源文件列表存储到该变量中
aux_source_directory(< dir > < variable >)
```

例：

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
# 搜索 src 目录下的源文件
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src SRC_LIST)
add_executable(app  ${SRC_LIST})
```

### 2. file()

```cmake
# GLOB: 将指定目录下搜索到的满足条件的所有文件名生成一个列表，并将其存储到变量中。
# GLOB_RECURSE：递归搜索指定目录，将搜索到的满足条件的文件名生成一个列表，并将其存储到变量中。
file(GLOB/GLOB_RECURSE 变量名 要搜索的文件路径和文件类型)
```

例：

```cmake
file(GLOB MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB MAIN_HEAD ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)

```

## 包含头文件
在编译项目源文件的时候，很多时候都需要将源文件对应的头文件路径指定出来，这样才能保证在编译过程中编译器能够找到这些头文件，并顺利通过编译。在CMake中设置要包含的目录也很简单，通过一个命令就可以搞定了，他就是include_directories:

CMAKE
include_directories(headpath)
举例说明，有源文件若干，其目录结构如下：

```C++

$ tree
.
├── build
├── CMakeLists.txt
├── include
│   └── head.h
└── src
    ├── add.cpp
    ├── div.cpp
    ├── main.cpp
    ├── mult.cpp
    └── sub.cpp

```

CMakeLists.txt文件内容如下:

```cmake
cmake_minimum_required(VERSION 3.0)
project(CALC)
set(CMAKE_CXX_STANDARD 11)
set(HOME /home/robin/Linux/calc)
set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin/)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
add_executable(app  ${SRC_LIST})
# 其中，第六行指定就是头文件的路径，PROJECT_SOURCE_D
```

## 制作动态库或静态库
有些时候我们编写的源代码并不需要将他们编译生成可执行程序，而是生成一些静态库或动态库提供给第三方使用，下面来讲解在cmake中生成这两类库文件的方法。

### 制作静态库
在cmake中，如果要制作静态库，需要使用的命令如下：

```CMAKE
add_library(库名称 STATIC 源文件1 [源文件2] ...) 
```
在Linux中，静态库名字分为三部分：lib+库名字+.a，此处只需要指定出库的名字就可以了，另外两部分在生成该文件的时候会自动填充。

在Windows中虽然库名和Linux格式不同，但也只需指定出名字即可。

下面有一个目录，需要将src目录中的源文件编译成静态库，然后再使用：

```SHELL
.
├── build
├── CMakeLists.txt
├── include           # 头文件目录
│   └── head.h
├── main.cpp          # 用于测试的源文件
└── src               # 源文件目录
    ├── add.cpp
    ├── div.cpp
    ├── mult.cpp
    └── sub.cpp
```

根据上面的目录结构，可以这样编写CMakeLists.txt文件:

```CMAKE
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
add_library(calc STATIC ${SRC_LIST})
```

这样最终就会生成对应的静态库文件libcalc.a。

## 制作动态库

在cmake中，如果要制作动态库，需要使用的命令如下：

```cmake
add_library(库名称 SHARED 源文件1 [源文件2] ...) 
在Linux中，动态库名字分为三部分：lib+库名字+.so，此处只需要指定出库的名字就可以了，另外两部分在生成该文件的时候会自动填充。
```

在Windows中虽然库名和Linux格式不同，但也只需指定出名字即可。

根据上面的目录结构，可以这样编写CMakeLists.txt文件:

```CMAKE
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
add_library(calc SHARED ${SRC_LIST})
# 这样最终就会生成对应的动态库文件libcalc.so。
```

### 指定输出的路径
#### 方式1 - 适用于动态库
对于生成的库文件来说和可执行程序一样都可以指定输出路径。由于在Linux下生成的动态库默认是有执行权限的，所以可以按照生成可执行程序的方式去指定它生成的目录：

```CMAKE
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
# 设置动态库生成路径
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
add_library(calc SHARED ${SRC_LIST})
对于这种方式来说，其实就是通过set命令给EXECUTABLE_OUTPUT_PATH宏设置了一个路径，这个路径就是可执行文件生成的路径。
```

#### 方式2 - 都适用

由于在Linux下生成的静态库默认不具有可执行权限，所以在指定静态库生成的路径的时候就不能使用EXECUTABLE_OUTPUT_PATH宏了，而应该使用LIBRARY_OUTPUT_PATH，这个宏对应静态库文件和动态库文件都适用。

```CMAKE
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
# 设置动态库/静态库生成路径
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
# 生成动态库
#add_library(calc SHARED ${SRC_LIST})
# 生成静态库
add_library(calc STATIC ${SRC_LIST})
```

## 包含库文件
在编写程序的过程中，可能会用到一些系统提供的动态库或者自己制作出的动态库或者静态库文件，cmake中也为我们提供了相关的加载动态库的命令。

### 链接静态库

```SHELL
src
├── add.cpp
├── div.cpp
├── main.cpp
├── mult.cpp
└── sub.cpp
```

现在我们把上面src目录中的add.cpp、div.cpp、mult.cpp、sub.cpp编译成一个静态库文件libcalc.a。通过命令制作并使用静态链接库

测试目录结构如下：

```SHELL
$ tree 
.
├── build
├── CMakeLists.txt
├── include
│   └── head.h
├── lib
│   └── libcalc.a     # 制作出的静态库的名字
└── src
    └── main.cpp
```

在cmake中，链接静态库的命令如下：

```CMAKE
link_libraries(<static lib> [<static lib>...])
# 参数1：指定出要链接的静态库的名字
# 可以是全名 libxxx.a
# 也可以是掐头（lib）去尾（.a）之后的名字 xxx
# 参数2-N：要链接的其它静态库的名字
#如果该静态库不是系统提供的（自己制作或者使用第三方提供的静态库）可能出现静态库找不到的情况，此时可以将静态库的路径也指定出来：
```

```CMAKE
# 包含静态库路径
link_directories(<lib path>)
```

这样，修改之后的CMakeLists.txt文件内容如下:

```CMAKE
cmake_minimum_required(VERSION 3.0)
project(CALC)
# 搜索指定目录下源文件
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
# 包含头文件路径
include_directories(${PROJECT_SOURCE_DIR}/include)
# 包含静态库路径
link_directories(${PROJECT_SOURCE_DIR}/lib)
# 链接静态库
link_libraries(calc)
add_executable(app ${SRC_LIST})
添加了第8行的代码，就可以根据参数指定的路径找到这个静态库了。
```

### 链接动态库
在程序编写过程中，除了在项目中引入静态库，好多时候也会使用一些标准的或者第三方提供的一些动态库，关于动态库的制作、使用以及在内存中的加载方式和静态库都是不同的，在此不再过多赘述，如有疑惑请参考Linux 静态库和动态库

在cmake中链接动态库的命令如下:

```CMAKE
target_link_libraries(
    <target> 
    <PRIVATE|PUBLIC|INTERFACE> <item>... 
    [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)

#target：指定要加载动态库的文件的名字

#PRIVATE|PUBLIC|INTERFACE：动态库的访问权限，默认为PUBLIC
#如果各个动态库之间没有依赖关系，无需做任何设置，三者没有没有区别，一般无需指定，使用默认的 PUBLIC 即可。

动态库的链接具有传递性，如果动态库 A 链接了动态库B、C，动态库D链接了动态库A，此时动态库D相当于也链接了动态库B、C，并可以使用动态库B、C中定义的方法。

```CMAKE
target_link_libraries(A B C)
target_link_libraries(D A)
# PUBLIC：在public后面的库会被Link到前面的target中，并且里面的符号也会被导出，提供给第三方使用。
# PRIVATE：在private后面的库仅被link到前面的target中，并且终结掉，第三方不能感知你调了啥库
# INTERFACE：在interface后面引入的库不会被链接到前面的target中，只会导出符号。
```

#### 链接系统动态库

动态库的链接和静态库是完全不同的：

* 静态库会在生成可执行程序的链接阶段被打包到可执行程序中，所以可执行程序启动，静态库就被加载到内存中了。

* 动态库在生成可执行程序的链接阶段不会被打包到可执行程序中，当可执行程序被启动并且调用了动态库中的函数的时候，动态库才会被加载到内存

因此，在cmake中指定要链接的动态库的时候，应该将命令写到生成了可执行文件之后：

```CMAKE
cmake_minimum_required(VERSION 3.0)
project(TEST)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
# 添加并指定最终生成的可执行程序名
add_executable(app ${SRC_LIST})
# 指定可执行程序要链接的动态库名字
target_link_libraries(app pthread)
在target_link_libraries(app pthread)中：

# app: 对应的是最终生成的可执行程序的名字
# pthread：这是可执行程序要加载的动态库，这个库是系统提供的线程库，全名为libpthread.so，在指定的时候一般会掐头（lib）去尾（.so）。
```

链接第三方动态库
现在，自己生成了一个动态库，对应的目录结构如下：

```SHELL

$ tree 
.
├── build
├── CMakeLists.txt
├── include
│   └── head.h            # 动态库对应的头文件
├── lib
│   └── libcalc.so        # 自己制作的动态库文件
└── main.cpp              # 测试用的源文件

```

假设在测试文件main.cpp中既使用了自己制作的动态库libcalc.so又使用了系统提供的线程库，此时CMakeLists.txt文件可以这样写：

```CMAKE
cmake_minimum_required(VERSION 3.0)
project(TEST)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
# 指定源文件或者动态库对应的头文件路径
include_directories(${PROJECT_SOURCE_DIR}/include)
# 指定要链接的动态库的路径
link_directories(${PROJECT_SOURCE_DIR}/lib)
# 添加并生成一个可执行程序
add_executable(app ${SRC_LIST})
# 指定要链接的动态库
target_link_libraries(app pthread calc)
通过link_directories指定了动态库的路径之后，在执行生成的可执行程序的时候，就不会出现找不到动态库的问题了。
```

温馨提示：使用 target_link_libraries 命令就可以链接动态库，也可以链接静态库文件。

## 安装和打包

### 安装规则(install rules)

在CMake中，安装是指将二进制文件、库文件和头文件等安装到指定目录中。

语法：

```cmake
install(TARGETS <target>... [...])
install({FILES | PROGRAMS} <file>... [...])
install(DIRECTORY <dir>... [...])
install(SCRIPT <file> [...])
install(CODE <code> [...])
install(EXPORT <export-name> [...])
```

该命令是为项目配置安装规则。在项目中会按照install()命令调用的顺序执行按照规则。

首先解释几个选项：

* DESTINATION：指定要安装到的目录，如果是相对路径，则是相对于CMAKE_INSTALL_PREFIX变量的，如果是绝对路径，则直接使用，由于cpack的安装程序生成器不支持绝对路径，所有最好使用相对路径。
* PERMISSIONS: 指定安装文件的权限，有效权限有OWNER_READ，OWNER_WRITE，OWNER_EXECUTE，GROUP_READ，GROUP_WRITE，GROUP_EXECUTE，WORLD_READ，WORLD_WRITE，WORLD_EXECUTE，SETUID和SETGID。在某些平台上没有意义的权限将被忽略。
* CONFIGURATIONS: 指定安装规则适用的构建配置(Debug, Relase等)，如果要为Debug和Relase单独配置安装路径，如下：

```cmake
install(TARGETS target
        CONFIGURATIONS Debug
        RUNTIME DESTINATION Debug/bin)
install(TARGETS target
        CONFIGURATIONS Release
        RUNTIME DESTINATION Release/bin)
```

EXCLUDE_FROM_ALL: 指定该文件从完整安装中排除。
RENAME: 指定已安装文件的名称，该名称可能与原始文件不同。仅当命令安装了单个文件时，才允许重命名.
安装二进制文件
语法：

```cmake
install(TARGETS targets... [EXPORT <export-name>]
        [[ARCHIVE|LIBRARY|RUNTIME|OBJECTS|FRAMEWORK|BUNDLE|
          PRIVATE_HEADER|PUBLIC_HEADER|RESOURCE]
         [DESTINATION <dir>]
         [PERMISSIONS permissions...]
         [CONFIGURATIONS [Debug|Release|...]]
         [COMPONENT <component>]
         [NAMELINK_COMPONENT <component>]
         [OPTIONAL] [EXCLUDE_FROM_ALL]
         [NAMELINK_ONLY|NAMELINK_SKIP]
        ] [...]
        [INCLUDES DESTINATION [<dir> ...]]
        )
```

常用参数：

ARCHIVE: 静态库
LIBRARY: 动态库
RUNTIME: 可执行文件
如果没有指定下列目录的路径，则使用内置的默认值：

| Target	| Variable |	Built-In Default |
| --- | --- | --- | --- |
| RUNTIME	| 	${CMAKE_INSTALL_BINDIR}	       | bin |
| LIBRARY	|      ${CMAKE_INSTALL_LIBDIR}		| lib |
| ARCHIVE	|      ${CMAKE_INSTALL_LIBDIR}		| lib |
| PRIVATE_HEADER |     ${CMAKE_INSTALL_INCLUDEDIR}		| include  |
| PUBLIC_HEADER	 |    ${CMAKE_INSTALL_INCLUDEDIR}		| include  |
安装文件和目录
语法：

```cmake
install(<FILES|PROGRAMS> files...
        TYPE <type> | DESTINATION <dir>
        [PERMISSIONS permissions...]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>]
        [RENAME <name>] [OPTIONAL] [EXCLUDE_FROM_ALL])

install(DIRECTORY dirs...
        TYPE <type> | DESTINATION <dir>
        [FILE_PERMISSIONS permissions...]
        [DIRECTORY_PERMISSIONS permissions...]
        [USE_SOURCE_PERMISSIONS] [OPTIONAL] [MESSAGE_NEVER]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>] [EXCLUDE_FROM_ALL]
        [FILES_MATCHING]
        [[PATTERN <pattern> | REGEX <regex>]
         [EXCLUDE] [PERMISSIONS permissions...]] [...])

```
当通过命令行使用cmake --install .或者make install时，将会运行安装步骤。如果要指定安装的根目录，需要使用cmake --install --prefix "/home/user/installdir" .来指定。

## 日志
在CMake中可以用用户显示一条消息，该命令的名字为message：

```CMAKE
message([STATUS|WARNING|AUTHOR_WARNING|FATAL_ERROR|SEND_ERROR] "message to display" ...)
(无) ：重要消息
# STATUS ：非重要消息
# WARNING：CMake 警告, 会继续执行
# AUTHOR_WARNING：CMake 警告 (dev), 会继续执行
# SEND_ERROR：CMake 错误, 继续执行，但是会跳过生成的步骤
# FATAL_ERROR：CMake 错误, 终止所有处理过程
```

CMake的命令行工具会在stdout上显示STATUS消息，在stderr上显示其他所有消息。CMake的GUI会在它的log区域显示所有消息。

CMake警告和错误消息的文本显示使用的是一种简单的标记语言。文本没有缩进，超过长度的行会回卷，段落之间以新行做为分隔符。

```CMAKE
# 输出一般日志信息
message(STATUS "source path: ${PROJECT_SOURCE_DIR}")
# 输出警告信息
message(WARNING "source path: ${PROJECT_SOURCE_DIR}")
# 输出错误信息
message(FATAL_ERROR "source path: ${PROJECT_SOURCE_DIR}")
```

## 变量操作

#### 追加  

有时候项目中的源文件并不一定都在同一个目录中，但是这些源文件最终却需要一起进行编译来生成最终的可执行文件或者库文件。如果我们通过file命令对各个目录下的源文件进行搜索，最后还需要做一个字符串拼接的操作，关于字符串拼接可以使用set命令也可以使用list命令。

使用set拼接
如果使用set进行字符串拼接，对应的命令格式如下：

```CMAKE
set(变量名1 ${变量名1} ${变量名2} ...)
# 关于上面的命令其实就是将从第二个参数开始往后所有的字符串进行拼接，最后将结果存储到第一个参数中，如果第一个参数中原来有数据会对原数据就行覆盖。
```

```CMAKE
cmake_minimum_required(VERSION 3.0)
project(TEST)
set(TEMP "hello,world")
file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/src1/*.cpp)
file(GLOB SRC_2 ${PROJECT_SOURCE_DIR}/src2/*.cpp)
# 追加(拼接)
set(SRC_1 ${SRC_1} ${SRC_2} ${TEMP})
message(STATUS "message: ${SRC_1}")
```

#### 使用list拼接

如果使用list进行字符串拼接，对应的命令格式如下：

```CMAKE
list(APPEND <list> [<element> ...])
list命令的功能比set要强大，字符串拼接只是它的其中一个功能，所以需要在它第一个参数的位置指定出我们要做的操作，APPEND表示进行数据追加，后边的参数和set就一样了。
```

```CMAKE
cmake_minimum_required(VERSION 3.0)
project(TEST)
set(TEMP "hello,world")
file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/src1/*.cpp)
file(GLOB SRC_2 ${PROJECT_SOURCE_DIR}/src2/*.cpp)
# 追加(拼接)
list(APPEND SRC_1 ${SRC_1} ${SRC_2} ${TEMP})
message(STATUS "message: ${SRC_1}")
```

在CMake中，使用set命令可以创建一个list。一个在list内部是一个由分号;分割的一组字符串。例如，set(var a b c d e)命令将会创建一个list:a;b;c;d;e，但是最终打印变量值的时候得到的是abcde。

```CMAKE
set(tmp1 a;b;c;d;e)
set(tmp2 a b c d e)
message(${tmp1})
message(${tmp2})
```

输出的结果:

```SHELL
1
2
abcde
abcde
```

### 字符串移除

我们在通过file搜索某个目录就得到了该目录下所有的源文件，但是其中有些源文件并不是我们所需要的，比如：

```SHELL
$ tree
.
├── add.cpp
├── div.cpp
├── main.cpp
├── mult.cpp
└── sub.cpp
```

在当前这么目录有五个源文件，其中main.cpp是一个测试文件。如果我们想要把计算器相关的源文件生成一个动态库给别人使用，那么只需要add.cpp、div.cp、mult.cpp、sub.cpp这四个源文件就可以了。此时，就需要将main.cpp从搜索到的数据中剔除出去，想要实现这个功能，也可以使用list

```CMAKE
list(REMOVE_ITEM <list> <value> [<value> ...])
```

通过上面的命令原型可以看到删除和追加数据类似，只不过是第一个参数变成了REMOVE_ITEM。

```CMAKE
cmake_minimum_required(VERSION 3.0)
project(TEST)
set(TEMP "hello,world")
file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/*.cpp)
# 移除前日志
message(STATUS "message: ${SRC_1}")
# 移除 main.cpp
list(REMOVE_ITEM SRC_1 ${PROJECT_SOURCE_DIR}/main.cpp)
# 移除后日志
message(STATUS "message: ${SRC_1}")
```

可以看到，在第8行把将要移除的文件的名字指定给list就可以了。但是一定要注意通过 file 命令搜索源文件的时候得到的是文件的绝对路径（在list中每个文件对应的路径都是一个item，并且都是绝对路径），那么在移除的时候也要将该文件的绝对路径指定出来才可以，否是移除操作不会成功。

#### list命令还有其它功能

* 获取 list 的长度。

```CMAKE
list(LENGTH <list> <output variable>)
LENGTH：子命令LENGTH用于读取列表长度
<list>：当前操作的列表
<output variable>：新创建的变量，用于存储列表的长度。
```
读取列表中指定索引的的元素，可以指定多个索引

```CMAKE
list(GET <list> <element index> [<element index> ...] <output variable>)
<list>：当前操作的列表
<element index>：列表元素的索引
从0开始编号，索引0的元素为列表中的第一个元素；
索引也可以是负数，-1表示列表的最后一个元素，-2表示列表倒数第二个元素，以此类推
当索引（不管是正还是负）超过列表的长度，运行会报错
<output variable>：新创建的变量，存储指定索引元素的返回结果，也是一个列表。
将列表中的元素用连接符（字符串）连接起来组成一个字符串
```

```CMAKE
list (JOIN <list> <glue> <output variable>)
<list>：当前操作的列表
<glue>：指定的连接符（字符串）
<output variable>：新创建的变量，存储返回的字符串
查找列表是否存在指定的元素，若果未找到，返回-1
```

```CMAKE
list(FIND <list> <value> <output variable>)
<list>：当前操作的列表
<value>：需要再列表中搜索的元素
<output variable>：新创建的变量
如果列表<list>中存在<value>，那么返回<value>在列表中的索引
如果未找到则返回-1。
将元素追加到列表中
```

```CMAKE
list (APPEND <list> [<element> ...])
在list中指定的位置插入若干元素
```

```CMAKE
list(INSERT <list> <element_index> <element> [<element> ...])
将元素插入到列表的0索引位置
```

```CMAKE
list (PREPEND <list> [<element> ...])
将列表中最后元素移除
```

```CMAKE
list (POP_BACK <list> [<out-var>...])
将列表中第一个元素移除
```

```CMAKE
list (POP_FRONT <list> [<out-var>...])
将指定的元素从列表中移除
```

```CMAKE
list (REMOVE_ITEM <list> <value> [<value> ...])
将指定索引的元素从列表中移除
```

```CMAKE
list (REMOVE_AT <list> <index> [<index> ...])
移除列表中的重复元素
```

```CMAKE
list (REMOVE_DUPLICATES <list>)
列表翻转
```

```CMAKE
list(REVERSE <list>)
列表排序
```

```CMAKE

list (SORT <list> [COMPARE <compare>] [CASE <case>] [ORDER <order>])
COMPARE：指定排序方法。有如下几种值可选：
STRING:按照字母顺序进行排序，为默认的排序方法
FILE_BASENAME：如果是一系列路径名，会使用basename进行排序
NATURAL：使用自然数顺序排序
CASE：指明是否大小写敏感。有如下几种值可选：
SENSITIVE: 按照大小写敏感的方式进行排序，为默认值
INSENSITIVE：按照大小写不敏感方式进行排序
ORDER：指明排序的顺序。有如下几种值可选：
ASCENDING:按照升序排列，为默认值
DESCENDING：按照降序排列
```

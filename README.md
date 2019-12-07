# NeCMake CMake语法详解

## 一、概念
### 1.1 什么是CMake
> 在Android Studio 2.2及以上，构建原生库的默认工具是CMake.  
> CMake是一个跨平台的构建工具，可以用简单的语句来描述所有平台的安装（编译过程）。能够输出各种各样的makefile或者project文件。
CMake并不直接构建出最终的软件，而是产生其他工具的脚本（如Makefile），然后再依据这个工具的构建方式使用。  
> CMake是一个比make更高级的编译配置工具，它可以根据不同的平台、不同的编译器生成相应的Makefile或者vcproj项目，从而达到跨平台的目的。
Android Studio利用CMake生成的是ninja. ninja是一个小型的关注速度的构建系统。我们不需要关注ninja的脚本，知道怎么配置CMake就可以了。  
> CMake其实是一个跨平台的支持产出各种不同的构建脚本的一个工具。

### 1.2 CMake源文件
> CMake的源文件可以包含命令、注释、空格和换行。  
> 以CMake编写的源文件以CMakeLists.txt命名或以.cmake为扩展名。  
> 可以通过add_subdirectory()命令把子目录的CMake源文件添加进来。  
> CMake源文件中所有有效的语句都是命令，可以是内置命令或自定义的函数/宏命令。  

### 1.3 CMake注释
* 单行注释：#注释内容（注释从#开始到行尾结束）
```cmake
# 单行注释
```
* 多行注释：可以使用中括号来实现多行注释：#[[多行注释]]
```cmake
#[[多行注释
多行注释
多行注释]]
```

### 1.4 CMake变量
* CMake中所有的变量都是string类型。可以使用set()和unset()命令来声明或移除一个变量
```cmake
# 声明变量：set(变量名 变量值)
set(var 123)
```
* 变量的引用：${变量名}
```cmake
# 引用变量 message命令用来打印
# 如果想直接看到打印信息，使用 WARNING 以上的级别进行打印。
message(WARNING "var = ${var}")
```

### 1.5 CMake列表（lists）
> 列表也是字符串，可以把列表看做一个特殊的变量，这个变量有多个值。  
> 语法格式：set(列表名 值1 值2 ... 值N) 或 set(列表名 "值1;值2;...;值N")。  
> 列表的引用：${列表名}。  
```cmake 
# 声明列表 set(列表名 值1 值2 ... 值N)
# 或 set(列表名 "值1;值2;...;值N")
set(list_var 1 2 3 4 5)
# 或
set(list_var "1;2;3;4;5")

message("list_var = ${list_var}")
```

### 1.6 CMake流程控制
#### 1.6.1 操作符 
类型 | 名称
--  | ---
一元 | EXIST,COMMAND,DEFINED
二元 | EQUAL,LESS,LESS_EQUAL,GREATER,GREATER_EQUAL,STREQUAL,STRLESS,STRLESS_EQUAL,
STRGREATER,STRGREATER_EQUAL,VERSION_EQUAL_VERSION_LESSS,VERSION_LESS_EQUAL,VERSION_GREATER,
VERSION_GREATER_EQUAL,MATCHES
逻辑 | NOT,AND,OR


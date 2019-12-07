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

## 二、CMake流程控制
### 2.1 操作符 （大小写敏感）
类型 | 名称
--  | ---
一元 | EXIST,COMMAND,DEFINED
二元 | EQUAL,LESS,LESS_EQUAL,GREATER,GREATER_EQUAL,STREQUAL,STRLESS,STRLESS_EQUAL,STRGREATER,STRGREATER_EQUAL,VERSION_EQUAL_VERSION_LESSS,VERSION_LESS_EQUAL,VERSION_GREATER,VERSION_GREATER_EQUAL,MATCHES
逻辑 | NOT,AND,OR  

优先级： () > 一元 > 二元 > 逻辑  
### 2.2 布尔常量值 （大小写敏感） 
类型  | 值 
--   | -- 
true | 1,ON,YES,TRUE,Y,非0的值
false| 0,OFF,NO,FALSE,N,IGNORE,NOTFOUND,空字符串,以-NOTFOUND结尾的字符串  

### 2.3 条件命令 
> 语法格式：  
> if(表达式)  
>       COMMAND(ARGS...)  
> elseif(表达式)  
>       COMMAND(ARGS...)  
> else(表达式)  
>       COMMAND(ARGS...)  
> endif(表达式)   
>  
> elseif和else部分是可选的，也可以有多个elseif部分，缩进和空格对语句解析没有影响。  
  
```cmake
set(if_tap 0FF)
set(elseif_tap ON)

if(${if_tap})
    message(WARNING "if")
elseif(${elseif_tap})
    message(WARNING "elseif")
else(${if_tap})
    message(WARNING "else")
endif(${if_tap)
```

### 2.4 循环遍历-while
> 语法格式：  
> while(表达式)  
>       COMMAND(ARGS...)  
> endwhile(表达式)  
>  
> break()命令可以跳出整个循环，continue()可以跳出当前循环  
```cmake
set(a "")

while(NOT a STREQUAL "xxx")
    set(a "${a}x")
    message(WARNING "a = ${a}")
endwhile()
```

### 2.5 循环遍历-foreach
#### 2.5.1 foreach循环变量 + 参数1 参数2... 参数N
> 语法格式：  
> foreach(循环变量 参数1 参数2 ... 参数N)  
>       COMMAND(ARGS...)  
> endforeach(循环变量)  
>  
> 每次迭代设置循环变量为参数。  
> foreach也支持break()和continue()命令跳出循环。  
```cmake
foreach(item 1 2 3)
    message(WARNING "item = ${item}")
endforeach(item)
```
#### 2.5.2 foreach循环变量 + RANGE total
> 语法格式：  
> foreach(循环变量 RANGE total)  
>       COMMAND(ARGS...)  
> endforeach(循环变量)  
>  
> 此种方式的取值范围为[0, total]   
```cmake
foreach(item2 RANGE 3)
    message(WARNING "item2 = ${item2}")
endforeach(item2)
```
#### 2.5.3 foreach循环变量 + RANGE start stop step
> 语法格式：  
> foreach(循环变量 RANGE start stop step)  
>       COMMAND(ARGS...)  
> endforeach(循环变量)  
>  
> 循环范围为[start, stop],循环增量为step。   
```cmake
foreach(item3 RANGE 1 5 2)
    message(WARNING "item3 = ${item3}")
endforeach(item3)
```
#### 2.5.4 foreach循环变量 + IN LISTS 列表
> 语法格式：  
> foreach(循环变量 IN LISTS 列表)  
>       COMMAND(ARGS...)  
> endforeach(循环变量)    
```cmake
set(list_var 1 2 3)

foreach(item4 IN LISTS list_var)
    message(WARNING "item4 = ${item4}")
endforeach(item4)
```  

## 三、CMake函数、宏及变量作用域
### 3.1 CMake自定义函数命令
> 自定义函数命令格式：  
> function(<name>[arg1 [arg2 [arg3 ...]]])  
>       COMMAND()  
> endfunction(<name>)  

> 函数命令调用格式：  
> name(实参列表)  
```cmake
function(func x y z)
    message(WARNING "call function func")
    message(WARNING "x = ${x}")     # x = 1
    message(WARNING "y = ${y}")     # y = 2
    message(WARNING "z = ${z}")     # z = 3
    message(WARNING "ARGC = ${ARGC}")   # ARGC = 3
    message(WARNING "arg1 = ${ARGV0} arg2 = ${ARGV1} arg3 = ${ARGV2}")  # arg1 = 1 arg2 = 2 arg3 = 3
    message(WARNING "all args = ${ARGV}")   # all args = 1;2;3
    endfunction(func)
    
func(1 2 3)
```

### 3.2 CMake自定义宏命令
> 自定义宏命令格式：  
> macro(<name>[arg1 [arg2 [arg3 ...]]])  
>       COMMAND()  
> endmacro(<name>)  

> 宏命令调用格式：  
> name(实参列表)  
```cmake
macro(ma x y z)
    message(WARNING "call macro ma")
    message(WARNING "x = ${x}")     # x = 1
    message(WARNING "y = ${y}")     # y = 2
    message(WARNING "z = ${z}")     # z = 3
    message(WARNING "ARGC = ${ARGC}")   # ARGC = 3
    message(WARNING "arg1 = ${ARGV0} arg2 = ${ARGV1} arg3 = ${ARGV2}")  # arg1 = 1 arg2 = 2 arg3 = 3
    message(WARNING "all args = ${ARGV}")   # all args = 1;2;3
    endmacro(func)
    
ma(1 2 3)
```

### 3.3 CMake变量的作用域
优先级：函数层 > 目录层 > 全局层  
> 全局层：cache变量，在整个项目范围内可见，一般在set定义变量时，指定CACHE参数就能定义为cache变量。  
> 目录层：在当前目录CMakeLists.txt中定义，以及在该文件包含的其它CMake源文件中定义的变量。  
> 函数层：在命令函数中定义的变量，属于函数作用域内的变量。

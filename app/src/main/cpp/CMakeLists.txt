# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)




# message([<mode>] "message to display" ...)
# <mode>关键字，可以指定消息的类型：
# (none)         = 重要消息
# STATUS         = 附带消息
# WARNING        = CMake警告，继续处理
# AUTHOR_WARNING = CMake警告（dev），继续处理
# SEND_ERROR     = CMake错误，继续处理，但跳过生成
# FATAL_ERROR    = CMake错误，停止处理和生成
# DEPRECATION    = 如果分别启用了变量CMAKE_ERROR_DEPRECATED或CMAKE_WARN_DEPRECATED，则CMake弃用错误或警告，否则无消息

#set(var 11123)
## 如果想直接看到打印信息，使用 WARNING 以上的级别进行打印。
#message(WARNING "var = ${var}")
#
#set(list_var 1 2 3 4 5)
#message(WARNING "list_var: ${list_var}")
#
#while(NOT a STREQUAL "xxx")
#    set(a "${a}x")
#    message(WARNING "a = ${a}")
#endwhile()
#
#foreach(item 1 2 3)
#    message(WARNING "item = ${item}")
#endforeach(item)

#foreach(item2 RANGE 3)
#    message(WARNING "item2 = ${item2}")
#endforeach(item2)

#foreach(item3 RANGE 1 5 2)
#    message(WARNING "item3 = ${item3}")
#endforeach(item3)

#set(list_var 1 2 3)
#
#foreach(item4 IN LISTS list_var)
#    message(WARNING "item4 = ${item4}")
#endforeach(item4)

#function(func x y z)
#    message(WARNING "call function func")
#    message(WARNING "x = ${x}")     # x = 1
#    message(WARNING "y = ${y}")     # y = 2
#    message(WARNING "z = ${z}")     # z = 3
#    message(WARNING "ARGC = ${ARGC}")   # ARGC = 3
#    message(WARNING "arg1 = ${ARGV0} arg2 = ${ARGV1} arg3 = ${ARGV2}")  # arg1 = 1 arg2 = 2 arg3 = 3
#    message(WARNING "all args = ${ARGV}")   # all args = 1;2;3
#endfunction(func)
#
#func(1 2 3)

macro(ma x y z)
    message(WARNING "call macro ma")
    message(WARNING "x = ${x}")
    message(WARNING "y = ${y}")
    message(WARNING "z = ${z}")
    message(WARNING "ARGC = ${ARGC}")
    message(WARNING "arg1 = ${ARGV0} arg2 = ${ARGV1} arg3 = ${ARGV2}")
    message(WARNING "all args = ${ARGV}")
endmacro(func)

ma(1 2 3)

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.

add_library( # Sets the name of the library.
        native-lib

        # Sets the library as a shared library.
        SHARED

        # Provides a relative path to your source file(s).
        native-lib.cpp)

# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

find_library( # Sets the name of the path variable.
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
        native-lib

        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})
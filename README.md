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
```bash
# 单行注释
```
* 多行注释：可以使用中括号来实现多行注释：#[[多行注释]]
```bash
#[[多行注释
多行注释
多行注释]]
```

### 1.4 CMake变量
* CMake中所有的变量都是string类型。可以使用set()和unset()命令来声明或移除一个变量
```bash
# 声明变量：set(变量名 变量值)
set(var 123)
```
* 变量的引用：${变量名}
```bash
# 引用变量 message命令用来打印
message("var = ${var}")
```


#### 1.2.2 FFmpeg开发库
* Libavcodec：包含音视频编解码库  
* Libavutil：包含简化编写程序的库（随机数字产生器、数据结构、核心多媒体应用程序）   
* Libavformat：包含多媒体容器的格式（复用器、解复用器） 
* Libavdevice：包含输入、输出的设备，用于多媒体输入输出的实现  
* Libavfilter：包含多媒体过滤器的库  
* Libswscale：执行高度图像缩放和色彩空间、像素格式的转换操作  
* Libswresample：执行高度优化的音视频重采样工具，重新矩阵化和样本格式转换工作    

### 1.3 如何使用FFmpeg
> FFmpeg是由c代码编写而成，功能多，代码量大  
> 在Android平台使用需要先编译，后使用，编译可以通过MakeFile语法来进行编译  

## 二、NDK编译FFmpeg
### 2.1 准备工作
#### 2.1.1 下载NDK r17 版本
从[https://developer.android.google.cn/ndk/downloads/older_releases.html](https://developer.android.google.cn/ndk/downloads/older_releases.html)下载 r17 版本，并解压。  
```bash
unzip android-ndk-r17c-linux-x86_64.zip
```
#### 2.1.2 配置NDK环境变量
* 进入NDK解压目录，查看当前路径  
```bash
pwd		# /Users/tian/NeCloud/NDKWorkspace/linuxdir/NDK/android-ndk-r17c
```
* 修改配置文件  
```bash
vim /etc/profile    # 若没有写权限，请先添加权限
```
* 添加NDK路径到配置文件中  
```bash
NDKROOT=/Users/tian/NeCloud/NDKWorkspace/linuxdir/NDK/android-ndk-r17c
export PATH=$NDKROOT:$PATH 		# 添加到环境变量中
```
* 使配置生效  
```bash
source /etc/profile
```  
* 测试配置是否成功
```bash
ndk-build
```  
若打印以下信息则表示配置成功：  
```bash
Android NDK: Could not find application project directory !    
Android NDK: Please define the NDK_PROJECT_PATH variable to point to it.    
/Users/tian/NeCloud/NDKWorkspace/linuxdir/NDK/android-ndk-r17c/build/core/build-local.mk:151: *** Android NDK: Aborting    .  Stop.
```  

#### 2.1.3 下载FFmpeg
从[http://www.ffmpeg.org/download.html](http://www.ffmpeg.org/download.html)下载FFmpeg 4.0.5 版本，并解压。  
```bash
tar xvf ffmpeg-4.0.5.tar.gz
```

### 2.2 编译FFmpeg(Ubuntu的Linux 环境)
#### 2.2.1 修改configure文件
进入FFmpeg目录，修改configure文件：不修改的话编译出来的.so文件会有一串数字，无法使用，所以得修改命名规则，使编译出来的so后缀不带数字，可以被Android识别。
```bash
cd ffmpeg-4.0.5
vim configure
```
修改内容如下：  
```bash
#SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
#LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
#SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
#SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```
#### 2.2.2 编写编译动态库脚本
在FFmpeg目录中，编写build.sh脚本  
```bash 
vim build.sh
``` 
脚本内容如下：  
```bash
#!/bin/bash
NDK_ROOT=/home/tian/NDKWorkspace/tools/NDK/android-ndk-r17c
#TOOLCHAIN 变量指向ndk中的交叉编译gcc所在的目录
TOOLCHAIN=$NDK_ROOT/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64/
#FLAGS与INCLUDES变量 可以从AS ndk工程的.externativeBuild/cmake/debug/armeabi-v7a/build.ninja中拷贝，需要注意的是**地址**
FLAGS="-isystem $NDK_ROOT/sysroot/usr/include/arm-linux-androideabi -D__ANDROID_API__=21 -g -DANDROID -ffunction-sections -funwind-tables -fstack-protector-strong -no-canonical-prefixes -march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16 -mthumb -Wa,--noexecstack -Wformat -Werror=format-security -std=c++11  -O0 -fPIC"
INCLUDES="-isystem $NDK_ROOT/sources/cxx-stl/llvm-libc++/include -isystem $NDK_ROOT/sources/android/support/include -isystem $NDK_ROOT/sources/cxx-stl/llvm-libc++abi/include"

#执行configure脚本，用于生成makefile
#--prefix : 安装目录
#--enable-small : 优化大小
#--disable-programs : 不编译ffmpeg程序(命令行工具)，我们是需要获得静态(动态)库。
#--disable-avdevice : 关闭avdevice模块，此模块在android中无用
#--disable-encoders : 关闭所有编码器 (播放不需要编码)
#--disable-muxers :  关闭所有复用器(封装器)，不需要生成mp4这样的文件，所以关闭
#--disable-filters :关闭视频滤镜
#--enable-cross-compile : 开启交叉编译（ffmpeg比较**跨平台**,并不是所有库都有这么happy的选项 ）
#--cross-prefix: 看右边的值应该就知道是干嘛的，gcc的前缀 xxx/xxx/xxx-gcc 则给xxx/xxx/xxx-
#disable-shared enable-static 不写也可以，默认就是这样的。
#--sysroot: 
#--extra-cflags: 会传给gcc的参数
#--arch --target-os :
PREFIX=./android/armeabi-v7a2
./configure \
--prefix=$PREFIX \
--enable-small \
--disable-programs \
--disable-avdevice \
--disable-encoders \
--disable-muxers \
--disable-filters \
--enable-cross-compile \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--enable-shared \
--disable-static \
--sysroot=$NDK_ROOT/platforms/android-21/arch-arm \
--extra-cflags="$FLAGS $INCLUDES" \
--extra-cflags="-isysroot $NDK_ROOT/sysroot" \
--arch=arm \
--target-os=android 

make clean
make install
```
#### 2.2.3 执行脚本
```bash
sh build.sh
```
编译一段时间，成功后会在当前目录下生成 android/armeabi-v7a2 文件夹


### 2.3 编译FFmpeg(MAC 环境)
#### 2.3.1 编写编译脚本
进入FFmpeg目录，编写buildmac.sh脚本  
```bash
cd ffmpeg-4.0.5 
vim buildmac.sh
``` 
脚本内容如下：  
```bash
#!/bin/bash
ADDI_CFLAGS="-marm"
# 指明交叉编译时使用的是哪个版本的头文件和库文件，它是SYSROOT的一部分
PLATFORM=arm-linux-androideabi
API=27
CPU=armv7-a
# ndk环境
NDK_ROOT=/Users/tian/NeCloud/NDKWorkspace/linuxdir/NDK/android-ndk-r17c
# 指明交叉编译目标机器的头文件和库文件目录
SYSROOT=$NDK_ROOT/platforms/android-$API/arch-arm/
ISYSROOT=$NDK_ROOT/sysroot
ASM=$ISYSROOT/usr/include/$PLATFORM
# 指明交叉编译工具链的位置
TOOLCHAIN=$NDK_ROOT/toolchains/$PLATFORM-4.9/prebuilt/darwin-x86_64
# 交叉编译后的输出目录，这里保存在当前目录下的android/armeabi-v7a2
PREFIX=/Users/tian/NeCloud/NDKWorkspace/linuxdir/ffmpeg/ffmpeg-4.0.5/android/armeabi-v7a2
export TMPDIR="/Users/tian/NeCloud/NDKWorkspace/linuxdir/ffmpeg/ffmpeg-4.0.5/temp"

function build_one
{
./configure \
--prefix=$PREFIX \
--disable-shared \
--enable-static \
--disable-doc \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-avdevice \
--disable-doc \
--disable-symver \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--target-os=android \
--arch=arm \
--enable-cross-compile \
--sysroot=$SYSROOT \
--extra-cflags="-I$ASM -isysroot $ISYSROOT -0s -fpic -marm" \
--extra-ldflags="-marm" \
$ADDITIONAL_CONFIGURE_FLAG
make clean
# 不确定自己在上面的目录或者环境有没有错误时，可以先注销一下下面两个命令
# make
# make install
}
echo "开始编译ffmpeg"
build_one
echo "完成编译..."
```
#### 2.3.2 执行脚本
```bash
sh buildmac.sh
```
编译一段时间，成功后会在当前目录下生成 android/armeabi-v7a2 文件夹


## 三、采坑
### 3.1 坑一
Mac环境下执行脚本时报`nasm/yasm not found `错误 
```bash
nasm/yasm not found or too old. Use --disable-x86asm for a crippled build.
```
**解决方案**  
1. Download yasm source code from   
[http://www.tortall.net/projects/yasm/releases/yasm-1.2.0.tar.gz](http://www.tortall.net/projects/yasm/releases/yasm-1.2.0.tar.gz)  
2. 在终端执行命令
```bash
tar xvzf yasm-1.2.0.tar.gz  # unpack
cd yasm-1.2.0
./configure && make -j 4 && sudo make install    #Configure and build   -j 4 表示4个并发执行线程
```



/Users/tian/NeCloud/NDKWorkspace/linuxdir/NDK/android-ndk-r17c/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64/bin/arm-linux-androideabi-gcc is unable to create an executable file.


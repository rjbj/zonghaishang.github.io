---
title: 源码编译openjdk8
subtitle:  openjdk 的模块，部分使用 C/C++ 编写实现，部分使用 Java 实现。因此除了需要 C/C++ 相关编译工具外，还需要有一个 JDK (Bootstrap JDK)。编译 openjdk8 时可使用 jdk1.7 作为 Bootstrap JDK 。
cover: /images/jdk.jpg
author: 
  nick: 诣极
  link: https://github.com/zonghaishang
tags:
- OpenJDK
categories:
- OpenJDK
date: 2018-09-30 23:00:19
---

## macOS High Sierra 编译openjdk 8

本次编译使用的系统是 `macOS High Sierra`，版本为 `10.13.2`。使用的 jdk 是 openjdk 8 。

### 概述

openjdk 的模块，部分使用 C/C++ 编写实现，部分使用 Java 实现。因此除了需要 C/C++ 相关编译工具外，还需要有一个 JDK (Bootstrap JDK)。编译 openjdk8 时可使用 jdk1.7 作为 Bootstrap JDK 。

我当前系统已经安装了jdk1.7 ：

```
$ java -version
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
```

### 安装准备

#### 源码下载

因为代码比较大，国内采用镜像下载：

```
git clone https://gitee.com/gorden5566/jdk8u.git
cd jdk8u/
git checkout --track origin/fix
sh ./getModules.sh
```

### 安装依赖

* 安装freetype

```
brew install freetype
```
或者进入官网[XQuartx](https://www.xquartz.org/)下载dmg安装。

* 安装xcode

直接从 `App Store` 中下载安装 或命令行安装 `xcode-select --install` 

* 安装gcc编译器

不要安装编译器版本高于5的，因为默认启用c++14 导致编译中断
```
brew install gcc@4.9
```

* 链接gcc编译器(4.9版本)

```
sudo ln -s /usr/local/Cellar/gcc@4.9/4.9.4/bin/gcc-4.9 /usr/bin/gcc
sudo ln -s /usr/local/Cellar/gcc@4.9/4.9.4/bin/g++-4.9 /usr/bin/g++
```

如果安装gcc版本和我的不一样，需要自行替换。

* 添加环境变量(~/.bash_profile)

```
export LFLAGS='-Xlinker -lstdc++'
```

添加执行命令生效：

```
source ~/.bash_profile
```

* 源码修改

修改openjdk/hotspot/src/share/vm/opto/loopPredicate.cpp 第775行
```
 assert(rng->Opcode() == Op_LoadRange || _igvn.type(rng)->is_int()->_lo >= 0, "must be");
```

 在is_int()后在添加 ->_lo 。

 修改openjdk/jdk/src/macosx/native/sun/osxapp/ThreadUtilities.m 第一个函数

 ```
 static inline void attachCurrentThread(void** env);
 ```

函数名前面添加static 关键字。

## 开始编译

为了方便我直接指定我当前bootstrap jdk1.7的版本，我的~/.bash_profile :

```
export ANT_HOME=/Users/Jason/tools/apache-ant-1.10.1
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk/Contents/Home
# export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home
export CLASSPATH=.:${JAVA_HOME}/lib:${JAVA_HOME}/jre/lib:${ANT_HOME}/lib
```

### 生成配置

```
export MACOSX_DEPLOYMENT_TARGET=10.13.2

bash ./configure --with-target-bits=64 --enable-ccache --with-boot-jdk-jvmargs="-Xlint:deprecation -Xlint:unchecked"  --disable-zip-debug-info --with-freetype-include=/usr/local/Cellar/freetype/2.9/include/freetype2 --with-freetype-lib=/usr/local/Cellar/freetype/2.9/lib --with-debug-level=slowdebug 
```
 其中freetype是前面安装的路径，可以进/usr/local/Cellar目录查看自己对应版本

 执行命令后我电脑输出：

```
====================================================
A new configuration has been successfully created in
/Users/Jason/openjdk/jdk8u-default/build/macosx-x86_64-normal-server-slowdebug
using configure arguments '--with-target-bits=64 --enable-ccache --with-boot-jdk-jvmargs=-Xlint:deprecation -Xlint:unchecked --disable-zip-debug-info --with-freetype-include=/usr/local/Cellar/freetype/2.9/include/freetype2 --with-freetype-lib=/usr/local/Cellar/freetype/2.9/lib --with-debug-level=slowdebug'.

Configuration summary:
* Debug level:    slowdebug
* JDK variant:    normal
* JVM variants:   server
* OpenJDK target: OS: macosx, CPU architecture: x86, address length: 64

Tools summary:
* Boot JDK:       java version "1.7.0_79" Java(TM) SE Runtime Environment (build 1.7.0_79-b15) Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)  (at /Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk/Contents/Home)
* C Compiler:      version Configured with: --prefix=/Applications/Xcode.app/Contents/Developer/usr --with-gxx-include-dir=/usr/include/c++/4.2.1 (at /Applications/Xcode.app/Contents/Developer/usr/bin/gcc)
* C++ Compiler:    version Configured with: --prefix=/Applications/Xcode.app/Contents/Developer/usr --with-gxx-include-dir=/usr/include/c++/4.2.1 (at /Applications/Xcode.app/Contents/Developer/usr/bin/gcc)

Build performance summary:
* Cores to use:   4
* Memory limit:   16384 MB
* ccache status:  installed, but disabled (version older than 3.1.4)

WARNING: The result of this configuration has overridden an older
configuration. You *should* run 'make clean' to make sure you get a
proper build. Failure to do so might result in strange build problems.
```

### 生成jdk8

```
make all COMPILER_WARNINGS_FATAL=false
```

生成jdk8成功会输出耗时信息：

```
## Finished docs (build time 00:01:59)

----- Build times -------
Start 2018-01-25 11:08:51
End   2018-01-25 11:22:23
00:00:22 corba
00:00:28 demos
00:01:59 docs
00:05:21 hotspot
00:00:59 images
00:00:14 jaxp
00:00:23 jaxws
00:02:50 jdk
00:00:41 langtools
00:00:12 nashorn
00:13:32 TOTAL
-------------------------
Finished building OpenJDK for target 'all'
```

## 使用openjdk8

1. 生成的jdk Home 在源码目录build ：

```
build/macosx-x86_64-normal-server-slowdebug/images/j2sdk-bundle/jdk1.8.0.jdk/Contents/Home
```

直接在intellij idea 或者 eclipse 中指定上面的Home即可。

2. 验证jdk版本

```
$ cd build/macosx-x86_64-normal-server-slowdebug/images/j2sdk-bundle/jdk1.8.0.jdk/Contents/Home
$ ./bin/java -version -version
openjdk version "1.8.0-internal-debug"
OpenJDK Runtime Environment (build 1.8.0-internal-debug-jason_2018_01_25_11_07-b00)
OpenJDK 64-Bit Server VM (build 25.71-b00-debug, mixed mode)
```

### 为什么要编译JDK源码

1. 已发布jdk版本去除了调试信息和运行时信息，降低内存占用提升运行速度，但是不适合开发者调试jdk代码

2. 深入jvm细节，自己动手编译为深入学习打基础

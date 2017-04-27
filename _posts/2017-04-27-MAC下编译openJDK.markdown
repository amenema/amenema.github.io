---
layout:     post
title:      "Mac下编译openJDK"
subtitle:   " \"osx编写openJdk7填坑指南\""
date:       2017-04-27 12:00:00
author:     "Mzx"
header-img: "img/post-bg-2015.jpg"
catalog: true
keyword: mac,osx,openjdk,jdk,Sierra,编译
description: 本文描述了如何在macs上使用osx编译openjdk7
tags:
    - Java
---


# **Mac**下编译**openJDK**   
>最近阅读《深入理解**Java**虚拟机：JVM高级特性与最佳实践》时，按照书上的教程打算编译一个**JDK**。由于书中使用的版本是**openJdk7u4**年代比较久远，而我的**osx**是**Sierra**,真是各种坑。所以在此建议：如果需要编译**openJdk**那么尽量选用最新版本的**openJdk**。尽量使用**linux**进行编译，因为一是新版的**OSX**加入**Rootless**机制，二十**XCODE**等一系列基础组件(因为编译要用**c/c++**)与之前的版本环境不太一样，所以会遇到很多问题。如果确实需要使用**OSX**编译那请看如下教程。   

## 准备工作  
### 下载与安装  
* 升级**Xcode**到最新版，(目前版本8.3.2)  
* 安装**XQuartz**  
* 安装**ant**(如果是**openJdk7**对应最低版本是**ant-1.7.1** ant版本也不能太高，因为需要最新**JDK的支持**) [下载](http://archive.apache.org/dist/ant/binaries/)   
* 安装**JDK 1.7.0_u4**（jdk版本尽量低于openJDK版本，否则坑太多 [下载](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html)    
* 下载**cups**  [下载](https://www.cups.org/)    
* 下载openJDK  [下载](http://www.java.net/download/openjdk/jdk7/promoted/b147/openjdk-7-fcs-src-b147-27_jun_2011.zip)  

### 执行命令构建环境  
* 关闭**Rootless**   
	* 重启 Mac 并按住 **Command+R**，进入恢复模式
	* 打开终端 **Terminal**
	* **csrutil disable** (重新开启：**csrutil enable**)  


* 编译**cups**  

	```
		$ tar -zxvf cups-2.1.4-source.tar.gz
		$ cd cups-2.1.4-source
		$ ./configure --prefix=/usr/local/cups
	   	$ sudo make
		$ sudo make install
	```  

* 添加软链  
	
	为了防止**openJDK**找不不到库，所以需要加个软链链接到他们历史上曾经存在位置  
	
	```  
	//ant 必须在关闭**rootless**后执行，否则提示权限不够
	$ sudo ln -s ${your_ant_home}/apache-ant-1.9.7/bin/ant /usr/bin/ant
	
	//安装 Command Line Tools 
	$ xcode-select --install 
	 
	//llvm 
	$ sudo ln -s /usr/bin/llvm-gcc /Applications/Xcode.app/Contents/Developer/usr/bin/llvm-gcc
	$ sudo ln -s /usr/bin/llvm-g++ /Applications/Xcode.app/Contents/Developer/usr/bin/llvm-g++
	
	//XQuartz
	$ sudo ln -s /usr/X11/include/X11 /usr/include/X11
	$ sudo ln -s /usr/X11/include/freetype2/freetype/ /usr/X11/include/freetype
	```  

* 编写配置文件  

	```
	
	$ cd ${youre_openJDK_source_code_HOME}/openJdk/
	$ vi buildConfig
	在 buildCofig添加如下内容：
	#to build openJDK
	# 设定编译预约
	export LANG=C
	export LC_ALL=C
	export CC=clang
	export LFLAGS='-Xlinker -lstdc++'
	export USE_CLANG=true
	export LP64=1
	export ARCH_DATE_MODEL=64
	# 设置为增量编译，因为问题可能比较多，节约编译时间
	export INCREMENTAL_BUILD=true
	export COMPILER_WARNINGS_FATAL=false
	export ALLOW_DOWNLOADS=true
	export HOTSPOT_BUILD_JOBS=8
	export ALT_PARALLEL_COMPILE_JOBS=8
	export SKIP_COMPARE_IMAGES=true
	export USE_PRECOMPILED_HEADER=true
	export BUILD_LANGTOOLS=true
	export BUILD_JAXP=true
	export BUILD_JAXWS=true
	export BUILD_CORBA=true
	export BUILD_HOTSPOT=true
	export BUILD_JDK=true
	export SKIP_DEBUG_BUILD=true
	export SKIP_FASTDEBUG_BUILD=false
	export DEBUG_NAME=debug
	export SHOW_ALL_WARNINGS=false
	export BUILD_DEPLOY=false
	export BUILD_INSTALL=false
	#三方的库
	export ALT_BOOTDIR=/Library/Java/JavaVirtualMachines/1.7.0.jdk/Contents/Home
	export ALT_CUPS_HEADERS_PATH='${your_cups_home}/cups/include'
	#这里如果做了软链可以不加
	export ANT_HOME=/usr/local/ant-1.7.1
	export FREETYPE_LIB_PATH=/usr/X11/lib
	export FREETYPE_HEADERS_PATH=/usr/X11/include
	export ALT_FREETYPE_LIB_PATH=/usr/local/Cellar/freetype/2.6.3/lib
	export ALT_FREETYPE_HEADERS_PATH=/usr/local/Cellar/freetype/2.6.3/include
	export ALT_OUTPUTDIR=/Users/menzhongxin/Downloads/openjdkBuild
	export DYLD_FALLBACK_LIBRARY_PATH=/usr/local/
	export MILESTONE=internal
	export BUILD_NUMBER=b25
	
	unset JAVA_HOME
	unset CLASSPATH
	export _JAVA_OPTIONS=-Dfile.encoding=ASCII
	
	#make sanity
	make 2>&1 | tee $ALT_OUTPUTDIR/build.log

	```  
	
### 开始编译  

编译之前可以先读一下这篇文章，里边提供了很多错误的解决方法
[http://www.voidcn.com/blog/j754379117/article/p-6347581.html](http://www.voidcn.com/blog/j754379117/article/p-6347581.html)
我在编译时遇到了文中提到的问题 4，5， 7   
另外还有一个文中没有提到的问题如下：   

	```
	Undefined symbols for architecture x86_64:
	"_attachCurrentThread", referenced from:
	    +[ThreadUtilities getJNIEnv] in ThreadUtilities.o
	    +[ThreadUtilities getJNIEnvUncached] in ThreadUtilities.o
	ld: symbol(s) not found for architecture x86_64
	clang: error: linker command failed with exit code 1 (use -v to see invocation)
	```
解决方法：  

	```
	修改 **jdk/src/macosx/native/sun/osxapp/ThreadUtilities.m ** 文件的38行
	将
		inline void attachCurrentThread(void** env) {
	修改为：
		static inline void attachCurrentThread(void** env) {
	```

开始编译   
  
	``` 
	$ cd ${your_open_source_code_jdk_home}/
	$ source configBuild
		
	```  
	
编译成功后结果显示    


```  	
	>>>Finished making images @ Wed Apr 26 22:40:00 CST 2017 ...
########################################################################
##### Leaving jdk for target(s) sanity all docs images             #####
########################################################################
##### Build time 00:06:57 jdk for target(s) sanity all docs images #####
########################################################################

#-- Build times ----------
Target debug_build
Start 2017-04-26 22:32:33
End   2017-04-26 22:40:00
00:00:11 corba
00:00:09 hotspot
00:00:02 jaxp
00:00:06 jaxws
00:06:57 jdk
00:00:02 langtools
00:07:27 TOTAL
-------------------------
	
```

# 参考文章  

* [第一章 Mac os下编译openJDK 7](http://www.voidcn.com/blog/j754379117/article/p-6347581.html) 
* [OS X El Capitan에서 OpenJDK7 빌드하기](http://tiramisu79.umask.io/build-openjdk7-on-osx-el-capitan/)
		

---
title: JVM-Debug-Openjdk-On-Mac
date: 2017-12-08 12:54:59
tags: JVM
---

### hotspot官方调试工具——hsdb
http://www.bubuko.com/infodetail-1858803.html

### 大神博客
http://rednaxelafx.iteye.com/

### openjdk源码
https://github.com/unofficial-openjdk/openjdk

### 切换分支
git checkout origin/jdk/jdk10
git clean -df

### boot jdk
系统装的是jdk8
```
$ java -version
java version "1.8.0_25"
Java(TM) SE Runtime Environment (build 1.8.0_25-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.25-b02, mixed mode)
```

但是必须再装一个boot jdk完成configure

```
$ bash configure --with-debug-level=slowdebug --enable-dtrace --with-jvm-variants=server --with-target-bits=64 --enable-ccache --with-num-cores=8 --with-memory-size=8000  --disable-warnings-as-errors

Running generated-configure.sh
configure: Configuration created at Fri Sep 28 17:41:37 CST 2018.
configure: configure script generated at timestamp 1516225089.
checking for basename... /usr/bin/basename
checking for bash... /usr/local/bin/bash
checking for cat... /bin/cat
checking for chmod... /bin/chmod
checking for cmp... /usr/bin/cmp
checking for comm... /usr/bin/comm
...
====================================================
A new configuration has been successfully created in
/Users/eric/Code/middleware/openjdk/build/macosx-x86_64-normal-server-slowdebug
using configure arguments '--with-debug-level=slowdebug --enable-dtrace --with-jvm-variants=server --with-target-bits=64 --enable-ccache --with-num-cores=8 --with-memory-size=8000 --disable-warnings-as-errors'.

Configuration summary:
* Debug level:    slowdebug
* HS debug level: debug
* JDK variant:    normal
* JVM variants:   server
* OpenJDK target: OS: macosx, CPU architecture: x86, address length: 64
* Version string: 10-internal+0-adhoc.eric.openjdk (10-internal)

Tools summary:
* Boot JDK:       java version "10.0.2" 2018-07-17 Java(TM) SE Runtime Environment 18.3 (build 10.0.2+13) Java HotSpot(TM) 64-Bit Server VM 18.3 (build 10.0.2+13, mixed mode)  (at /Library/Java/JavaVirtualMachines/jdk-10.0.2.jdk/Contents/Home)
* Toolchain:      clang (clang/LLVM from Xcode 8.3.3)
* C Compiler:     Version 8.1.0 (at /usr/bin/clang)
* C++ Compiler:   Version 8.1.0 (at /usr/bin/clang++)

Build performance summary:
* Cores to use:   7
* Memory limit:   8000 MB
* ccache status:  Active (3.3.4)
```


然后make
```
$ make images
Building target 'images' in configuration 'macosx-x86_64-normal-server-slowdebug'
Compiling 8 files for BUILD_TOOLS_LANGTOOLS
Creating hotspot/variant-server/tools/adlc/adlc from 13 file(s)
Compiling 2 files for BUILD_JVMTI_TOOLS
Creating support/modules_libs/java.base/libjsig.dylib from 1 file(s)
Warning: No mercurial configuration present and no .src-rev
Parsing 1 properties into enum-like class for jdk.compiler
Compiling 13 properties into resource bundles for jdk.javadoc
Compiling 12 properties into resource bundles for jdk.jdeps
Compiling 16 properties into resource bundles for jdk.compiler
Compiling 7 properties into resource bundles for jdk.jshell
...
Creating support/demos/image/jfc/CodePointIM/CodePointIM.jar
Creating support/demos/image/applets/MoleculeViewer/MoleculeViewer.jar
Creating support/demos/image/jfc/SwingApplet/SwingApplet.jar
Creating support/demos/image/applets/WireFrame/WireFrame.jar
Creating support/demos/image/jfc/FileChooserDemo/FileChooserDemo.jar
Creating support/demos/image/jfc/Font2DTest/Font2DTest.jar
Creating support/demos/image/jfc/Metalworks/Metalworks.jar
Creating support/demos/image/jfc/Notepad/Notepad.jar
Creating support/demos/image/jfc/SampleTree/SampleTree.jar
Creating support/demos/image/jfc/TableExample/TableExample.jar
Creating support/demos/image/jfc/TransparentRuler/TransparentRuler.jar
Creating support/classlist.jar
Creating images/jmods/jdk.jlink.jmod
Creating images/jmods/java.base.jmod
Creating jre jimage
Creating jdk jimage
WARNING: Using incubator modules: jdk.incubator.httpclient
WARNING: Using incubator modules: jdk.incubator.httpclient
Stopping sjavac server
Finished building target 'images' in configuration 'macosx-x86_64-normal-server-slowdebug'
```

### 验证
```
eric at EricdeMacBook-Pro in ~/Code/middleware/openjdk/build/macosx-x86_64-normal-server-slowdebug/jdk/bin (remotes/origin/jdk/jdk10●) 
$ ./java -version
openjdk version "10-internal" 2018-03-20
OpenJDK Runtime Environment (slowdebug build 10-internal+0-adhoc.eric.openjdk)
OpenJDK 64-Bit Server VM (slowdebug build 10-internal+0-adhoc.eric.openjdk, mixed mode)
```

### 导入Clion
```
/Users/eric/Code/middleware/openjdk/src/hotspot
```

### 配置调试

### Clion导致CPU彪高
File -> Power Save Mode 可以解决问题


Refer:
https://blog.csdn.net/wd2014610/article/details/81664062
https://blog.csdn.net/wd2014610/article/details/81703203#commentBox
https://www.jianshu.com/p/ee7e9176632c
---
title: JVM-Debug-Openjdk-On-Mac
date: 2017-12-08 12:54:59
tags: JVM
---

### 用VSCode搭建OpenJDK调试环境
https://zhuanlan.zhihu.com/p/50220757

### hotspot官方调试工具——hsdb
http://www.bubuko.com/infodetail-1858803.html

### 英文，如何读OpenJDK源码
https://www.infoq.com/articles/Introduction-to-HotSpot/

### 大神博客
http://rednaxelafx.iteye.com/

### openjdk源码
https://github.com/unofficial-openjdk/openjdk

### xcode版本
Version 11.1 (11A1027)


```
➜  ~ gcc -v
Configured with: --prefix=/Applications/Xcode.app/Contents/Developer/usr --with-gxx-include-dir=/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/4.2.1
Apple clang version 11.0.0 (clang-1100.0.33.8)
Target: x86_64-apple-darwin18.7.0
Thread model: posix
InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
```

### boot jdk
系统装的是jdk 8和jdk 12(但是必须再装一个boot jdk完成configure)
```
$ java -version
java version "12.0.2" 2019-07-16
Java(TM) SE Runtime Environment (build 12.0.2+10)
Java HotSpot(TM) 64-Bit Server VM (build 12.0.2+10, mixed mode, sharing)
```
```
➜  ~ /usr/libexec/java_home -V
Matching Java Virtual Machines (2):
    12.0.2, x86_64:	"Java SE 12.0.2"	/Library/Java/JavaVirtualMachines/jdk-12.0.2.jdk/Contents/Home
    1.8.0_221, x86_64:	"Java SE 8"	/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home

/Library/Java/JavaVirtualMachines/jdk-12.0.2.jdk/Contents/Home
```


### configure build makefile 不要显示制定freetype
```

```


### make all
```

```

### 验证
```
➜  bin pwd
/Users/ericfei/Code/opensource/openjdk-jdk12u/build/macosx-x86_64-serverANDclient-slowdebug/jdk/bin

➜  bin ./java -version
openjdk version "12.0.2-internal" 2019-07-16
OpenJDK Runtime Environment (slowdebug build 12.0.2-internal+0-adhoc.ericfei.openjdk-jdk12u)
OpenJDK 64-Bit Server VM (slowdebug build 12.0.2-internal+0-adhoc.ericfei.openjdk-jdk12u, mixed mode)
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
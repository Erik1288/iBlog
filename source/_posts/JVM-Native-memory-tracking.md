---
title: JVM-Native-memory-tracker
date: 2018-09-25 20:04:32
tags:
---

Java内存之本地内存分析神器： NMT 和 pmap

https://blog.csdn.net/jicahoo/article/details/50933469

首先，你要在Java启动项中，加入启动项： -XX:NativeMemoryTracking=detail 然后，重新启动Java程序。执行如下命令： jcmd 14179 VM.native_memory detail 你会在标准输出得到类似下面的内容（本文去掉了许多与本文无关或者重复的信息）：


https://my.oschina.net/foxty/blog/1934968
这篇文章写得也非常详细
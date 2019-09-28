---
title: Linux-Memory-Alignment
date: 2019-04-01 16:44:15
tags:
---



### 为什么需要进行内存对齐
https://www.jianshu.com/p/730c23e57249

很多 CPU（如基于 Alpha，IA-64，MIPS，和 SuperH 体系的）拒绝读取未对齐数据。当一个程序要求这些 CPU 读取未对齐数据时，这时 CPU 会进入异常处理状态并且通知程序不能继续执行。举个例子，在 ARM，MIPS，和 SH 硬件平台上，当操作系统被要求存取一个未对齐数据时会默认给应用程序抛出硬件异常。所以，如果编译器不进行内存对齐，那在很多平台的上的开发将难以进行。
那么，为什么这些 CPU 会拒绝读取未对齐数据？是因为未对齐的数据，会大大降低 CPU 的性能。

作者：xuyafei86
链接：https://www.jianshu.com/p/a371e2613ec8
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。


https://www.jianshu.com/p/a371e2613ec8
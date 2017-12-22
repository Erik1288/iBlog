---
title: JVM-Debug-Openjdk-On-Mac
date: 2017-12-08 12:54:59
tags:
---

https://segmentfault.com/a/1190000005082098

http://rednaxelafx.iteye.com/
大神博客

http://www.javaranger.com/archives/1636
http://www.cnblogs.com/dennyzhangdd/p/6734638.html
http://hllvm.group.iteye.com/group/topic/35385


debug on linux
https://segmentfault.com/a/1190000008346240


### netbeans看hotspot代码

http://marcelinorc.com/2016/02/17/using-netbeans-to-hack-openjdk9-in-ubuntu/

./configure --with-target-bits=64 \
--with-freetype=/usr/local/Cellar/freetype/2.8.1 \
--enable-ccache \
--with-jvm-variants=server,client \
--with-boot-jdk-jvmargs="-Xlint:deprecation -Xlint:unchecked" \
--disable-zip-debug-info \
--disable-warnings-as-errors \
--with-debug-level=slowdebug 2>&1 | tee configure_mac_x64.log


bash configure --with-target-bits=64 --with-freetype=/usr/local/Cellar/freetype/2.8.1 --enable-ccache --with-jvm-variants=server,client --with-boot-jdk-jvmargs="-Xlint:deprecation -Xlint:unchecked" --disable-zip-debug-info --disable-warnings-as-errors --with-debug-level=slowdebug
---
title: Mac-Multi-JDK
date: 2019-10-08 14:58:44
tags:
---


```
# 设置 JDK 8
export JAVA_8_HOME=`/usr/libexec/java_home -v 1.8`
# 设置 JDK 12
export JAVA_12_HOME=`/usr/libexec/java_home -v 12`

export JAVA_HOME=$JAVA_8_HOME
#alias命令动态切换JDK版本
alias jdk8="export JAVA_HOME=$JAVA_8_HOME"
alias jdk12="export JAVA_HOME=$JAVA_12_HOME"
```
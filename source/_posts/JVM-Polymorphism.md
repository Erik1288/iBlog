---
title: JVM-Polymorphism
date: 2018-10-08 21:30:08
tags:
---

Java 的方法调用有两类，动态方法调用与静态方法调用。静态方法调用是指对于类的静态方法的调用方式，是静态绑定的；而动态方法调用需要有方法调用所作用的对象，是动态绑定的。类调用 (invokestatic) 是在编译时就已经确定好具体调用方法的情况。实例调用 (invokevirtual)则是在调用的时候才确定具体的调用方法，这就是动态绑定，也是多态要解决的核心问题。JVM 的方法调用指令有四个，分别是 invokestatic，invokespecial，invokesvirtual 和 invokeinterface。前两个是静态绑定，后两个是动态绑定的。本文也可以说是对于JVM后两种调用实现的考察。


### 这篇文章讲得很好
https://www.ibm.com/developerworks/cn/java/j-lo-polymorph/index.html
---
title: SourceCode-Map
date: 2017-11-12 21:54:26
tags: 源码分析
---

http://www.importnew.com/26049.html
ConcurrentHashMap

JDK ConcurrentHashMap如何提高并发度
concurrenthashmap在读写并发的时候，什么时候可见，什么时候不可见

http://pettyandydog.com/2016/08/28/HashMap_infinite_loop/#more
HashMap

HashMap

### HashMap resize() 扩容
https://blog.csdn.net/aichuanwendang/article/details/53317351

HashMap的工作原理是什么
hashmap在 扩容 时为什么是乘以2. newCap = oldCap << 1 ，看看是不是和旧数据在新的Map中是否要产生偏移相关，不然所有的数据都挤在数组的前段。
内部的数据结构是什么
HashMap 的 table的容量如何确定？loadFactor 是什么？ 该容量如何变化？这种变化会带来什么问题？
HashMap 实现的数据结构是什么？如何实现
HashMap 和 HashTable、ConcurrentHashMap 的区别
HashMap的遍历方式及效率
HashMap、LinkedMap、TreeMap的区别
如何决定选用HashMap还是TreeMap
如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办
HashMap 是线程安全的吗？并发下使用的 Map 是什么，它们内部原理分别是什么，比如存储方式、 hashcode、扩容、 默认容量等

JDK1.8 中 concurrentHashMap 在扩容的时候是怎么做的，如何提高效率的，什么时候是多线程执行，什么时候单线程执行，如果某个线程正在扩容一个槽里的数据，如何保证后续线程不会重复计算。面蚂蚁金服暑期实习生的时候遇到的。

分段锁
http://guochenglai.com/2016/06/04/java-concurrent4-java-subsection-decompose/



线程池源码分析： ExecutorService  定时任务怎么实现的？提交给线程池的任务实现runnable，线程执行完它就dead了，怎么做到线程复用的？
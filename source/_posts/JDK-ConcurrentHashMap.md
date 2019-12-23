---
title: JDK-ConcurrentHashMap
date: 2019-11-25 17:05:54
tags:
---


### 内部的talbe用volatile修饰，`volatile Node<K,V>[] table`，这样做是无法让多个线程在读取数组的某一个元素时有volatile的语义（内存可见性语义），那为什么还要加volatile关键字？

在ConcurrentHashMap的内部数组长度达到loadFactor阈值时（数组长度不大，但也发生treefy的时候也有可能扩容），需要进行扩容，
```
if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
    try {
        if (table == tab) {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
            table = nt;
            sc = n - (n >>> 2);
        }
    } finally {
        sizeCtl = sc;
    }
}
```

### HashMap是怎么进行扩容的？
作者：自闭玩家
链接：https://www.nowcoder.com/discuss/176593?type=2&order=0&pos=11&page=1
来源：牛客网

讲了一下hashMap, 结构和插入，然后问我插入会有resize

问我resize是直接原数组还是新数组(我说新的

然后问我插入的情况下，resize，数据量很大的话，后面讲了map的二进制优化这些，插入是不是很慢(是喽

然后问我有没有优化的方式(我说redis的map扩容优化，就是有个计数器，保证前面的已经resize了..

### ConcurrentHashMap内部是一个volatile Node[]实现的，那么多线程获取数组的引用时，确实是达到了volatile的效果，但是多线程想要去获取数组Node[]的某一个元素时，怎么能保证每个线程看到的是最新的值？
```
/*
    * Volatile access methods are used for table elements as well as
    * elements of in-progress next table while resizing.  All uses of
    * the tab arguments must be null checked by callers.  All callers
    * also paranoically precheck that tab's length is not zero (or an
    * equivalent check), thus ensuring that any index argument taking
    * the form of a hash value anded with (length - 1) is a valid
    * index.  Note that, to be correct wrt arbitrary concurrency
    * errors by users, these checks must operate on local variables,
    * which accounts for some odd-looking inline assignments below.
    * Note that calls to setTabAt always occur within locked regions,
    * and so in principle require only release ordering, not
    * full volatile semantics, but are currently coded as volatile
    * writes to be conservative.
    */

@SuppressWarnings("unchecked")
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
```

---
title: LevelDB
date: 2019-03-17 21:14:04
tags:
---



### 这篇文章很好的说明了Compaction
https://www.cnblogs.com/KevinT/p/3819134.html

### LevelDB compact 过程. 画图
https://blog.csdn.net/baijiwei/article/details/83240078

https://www.zhihu.com/question/19887265


### LevelDB某一个值的多版本是怎么做的？


### LevelDB参数调优


### LevelDB: Read the Fucking Source Code
http://www.grakra.com/2017/06/17/Leveldb-RTFSC/

### 架构性
https://zhuanlan.zhihu.com/p/54510835

### LevelDB Cache —— Table Cache and Block Cache
https://www.cnblogs.com/liuhao/archive/2012/11/29/2795455.html

### LSM优化在SSD
http://catkang.github.io/2017/04/30/lsm-upon-ssd.html

### 牛逼啊
https://www.bookstack.cn/read/system-design/cn-key-value-store.md

### LSM理解
所有的方法都可以有效的提高了读操作的性能（最少提供了O(log(n)) )，但是，却丢失了日志文件超好的写性能。上面这些方法，都强加了总体的结构信息在数据上，数据被按照特定的方式放置，所以可以很快的找到特定的数据，但是却对写操作不友善，让写操作性能下降。
他们通过顺序的在文件末尾重复写对结构来实现写操作，之前的树结构的相关部分，包括最顶层结点都会变成孤结点。尽管通过这种方法避免了本地更新，但是因为每个写操作都要重写树结构，放大了写操作，降低了写性能。

# LSM-tree以及基于其实现的存储引擎有以下特点：
## 小的随机写IO被顺序化为大的连续IO；
## 磁盘上的物理文件（sstable）被分层为多个level，且随着系统的运行，层级低的数据会向层级高的数据合并。

因为LSM-tree将大量的随机IO变成了顺序IO，因此对HDD此类存储介质有着比较明显的性能提升。但同时，此架构也会带来如下一些问题，最主要的便是IO放大以及IO放大带来的副作用。

LSM-tree结构之所以带来IO放大，是因为：
## 写入时，除了将数据顺序写入日志外，还考虑将memtable写入level 0 的sstable以及由此带来的sstable合并问题，根据论文作者的计算，在极端情况下，可能带来的放大系数为10*Level；
## 读取时，首先需要查找文件所在的sstable，由于sstable分层，因此该过程要由低向高level逐渐查找，在极端情况下，需要遍历所有sstable的元数据块，也带来了相当程度的读放大。

而这种IO放大会带来怎样的副作用呢？
## 显而易见的是IO效率的降低；
## 对于SSD存储器件，这种放大还会导致存储硬件的寿命缩减。

### 在SSD中，如何尽可能避免LSM读/写放大带来的影响？
https://zhuanlan.zhihu.com/p/30773636
```
回顾上面的问题，当LSM中数据的长度很大时，这个问题变得尤为突出，这是因为：

1. 数据长度越大，越容易触发Compaction，从而造成写放大；
2. 如果把上层文件看做下层文件的cache，大数据长度会造成这个cache能cache的数据个数变少，从而读请求更大概率的需要访问下层数据，从而造成读放大；
3. 每条数据每次Merge需要更多的写入量
```
但是，进一步分析，LSM需要的其实是key的有序，而跟value无关。所以自然而然的思路是
Key Value 分离存储


### 为什么会有归并排序
https://www.codedump.info/post/20190215-leveldb/
MergingIterator
用于合并流程的迭代器。在合并过程中，需要操作memtable、immutable memtable、level 0 sstable以及非level 0的sstable，该迭代器将这些存储数据结构体的迭代器，统一进行归并排序


EncodeVarint
为什么要把整型（int）编码成变长整型（varint）呢？是为了尽可能的节约存储空间。
varint是一种紧凑的表示数字的方法，它用一个或多个字节来表示一个数字，值越小的数字使用越少的字节数。比如int32类型的数字，一般需要4个字节。但是采用Varint，对于很小的int32类型的数字，则可以用1个字节来表示。当然凡事都有好的也有不好的一面，采用varint表示法，大的数字则可能需要5个字节来表示。从统计的角度来说，一般不会所有消息中的数字都是大数，因此大多数情况下，采用varint后，可以用更小的字节数来表示数字信息。
varint中的每个字节的最高位（bit）有特殊含义，如果该位为1，表示后续的字节也是这个数字的一部分，如果该位为0，则结束。其他的7位（bit）都表示数字。7位能表示的最大数是127，因此小于128的数字都可以用一个字节表示。大于等于128的数字，比如说300，会用两个字节在内存中表示为：
低               高
1010 1100 0000 0010
正常情况下，int需要32位，varint用一个字节的最高位作为标识位，所以，一个字节只能存储7位，如果整数特别大，可能需要5个字节才能存放（5*8-5（标志位）>32），下面if语句的第五个分支就是处理这种情况。
---------------------
作者：灿哥哥
来源：CSDN
原文：https://blog.csdn.net/caoshangpa/article/details/78815940
版权声明：本文为博主原创文章，转载请附上博文链接！
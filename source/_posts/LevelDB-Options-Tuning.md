---
title: LevelDB-Options-Tuning
date: 2019-04-11 11:00:23
tags:
---

### 常用语调优的参数
// memtable 的最大 size
size_t write_buffer_size;
// db 中打开的文件最大个数
// db 中需要打开的文件包括基本的 CURRENT/LOG/MANIFEST/LOCK, 以及打开的 sstable 文件。 // sstable 一旦打开，就会将 index 信息加入 TableCache，所以把
// (max_open_files - 10)作为 table cache 的最大数量.
int max_open_files;
// 传入的 block 数据的 cache 管理
Cache* block_cache;
// sstable 中 block 的 size
size_t block_size;
// block 中对 key 做前缀压缩的区间长度
int block_restart_interval;
// 压缩数据使用的压缩类型(默认支持 snappy，其他类型需要使用者实现)
CompressionType compression;

namespace config {
// level 的最大值
static const int kNumLevels = 7;
// level-0 中 sstable 的数量超过这个阈值，触发 compact
static const int kL0_CompactionTrigger = 4;
// level-0 中 sstable 的数量超过这个阈值, 慢处理此次写(sleep1ms) static const int kL0_SlowdownWritesTrigger = 8;
// level-0 中 sstable 的数量超过这个阈值, 阻塞至 compact memtable 完成。 static const int kL0_StopWritesTrigger = 12;
// memtable dump 成的 sstable，允许推向的最高 level
// (参见 Compact 流程的 VersionSet::PickLevelForMemTableOutput()) static const int kMaxMemCompactLevel = 2;
}


### leveldb中的写放大

一条记录写到leveldb，则会写一次到log，写一次到level 0，随着后面更多数据的写入，level n里面的记录会与level n+1的记录合并排序，按照level n+1的数据量是level n的10倍，预计每提升一级，需要11个写，因此如果是7级的话，大约为68个写，即写放大为68。leveldb存放在内存中的数据大小write_buffer_size默认为4M，level 1的大小默认为10M。因此一个1TB的数据库会有7层。通过加大内存中的数据以及level 1的大小，例如设置write_buffer_size=1G，level1大小为4G，那么总共的层级只有5级，写放大大约为46，性能可以提升大约30%


### leveldb中的读放大

前一篇提到一次记录的读取需要多次文件读取，通过bloomfilter可以讲实际读取数据降到到接近1。我们来看看bloomfilter降低读取的原理。假设filter的bits为10，一个文件有1000个key，那么filter一共有1w个bit，对每个key计算hash值，把hash值模1w后对应的bit置为1，多计算几种hash值，都设置相应的bit为1。获取一个指定的key1时，按照前面的算法计算hash值检查bit值，如果这些bit值不是全1，那么可以肯定这个key1不在文件中。这样就可以通过不读取文件就判断key1是否在文件中。当然可能会出现全1但是key1又不在文件中，这样的情况就相当于是hash冲突了。通过查阅bloomfilter的资料，可以看到hash冲突的概率的情况为：
10 bits     0.0081
15 bits     0.0007
19 bits      0.0001
假设每次读取需要查找10个文件，那么10bits的hash冲突导致的多余读取为10 * 0.0081 = 0.081大约是8%
我在实际使用tair中，当写入较多时，导致level0的数据可能达到32，这种情况下10bits的多余夺取为35 * 0.0081 = 0.2835大约是28%，此时应当增加bits，例如15bits则额外读为2.45%


这两点是是实际使用中碰到的，可以提升实际的应用性能
---------------------
作者：dongfuye
来源：CSDN
原文：https://blog.csdn.net/dongfuye/article/details/46818023
版权声明：本文为博主原创文章，转载请附上博文链接！
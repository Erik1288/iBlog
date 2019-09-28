---
title: RocksDB
date: 2019-03-20 14:39:31
tags:
---


# RocksDB对LevelDB的优化——内联跳跃表
https://zhuanlan.zhihu.com/p/29277585

# RocksDB参数调优
https://xiking.win/2018/12/05/rocksdb-tuning/
# 官网
https://github.com/facebook/rocksdb/wiki/RocksDB-Tuning-Guide

### LSM优化在SSD
http://catkang.github.io/2017/04/30/lsm-upon-ssd.html

### Features Not in LevelDB
https://github.com/facebook/rocksdb/wiki/Features-Not-in-LevelDB
Performance
Multithread compaction
Multithread memtable inserts
Reduced DB mutex holding
Optimized level-based compaction style and universal compaction style
Prefix bloom filter
Memtable bloom filter
Single bloom filter covering the whole SST file
Write lock optimization
Improved Iter::Prev() performance
Fewer comparator calls during SkipList searches
Allocate memtable memory using huge page.
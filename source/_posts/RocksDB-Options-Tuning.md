---
title: RocksDB-Options-Tuning
date: 2019-04-11 14:28:03
tags:
---

https://github.com/tikv/tikv/blob/master/docs/op-guide/rocksdb-option-config.md

公司TiKV关于RocksDB的配置:conf/tikv.toml
[rocksdb]
wal-dir = ""

[rocksdb.defaultcf]
block-cache-size = "25GB"

[rocksdb.lockcf]

[rocksdb.writecf]
block-cache-size = "15GB"

### RocksDB 的常用调优参数
https://blog.csdn.net/weixin_36145588/article/details/78541070
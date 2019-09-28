---
title: Linux-DirectIO
date: 2019-03-18 18:38:35
tags:
---


### 为什么需要DirectIO?
Why do we need it?
With buffered I/O, the data is copied twice between storage and memory because of the page cache as the proxy between the two. In most cases, the introduction of page cache could achieve better performance. But for self-caching applications such as RocksDB, the application itself should have a better knowledge of the logical semantics of the data than OS, which provides a chance that the applications could implements more efficient replacement algorithm for cache with any application-defined data block as a unit by leveraging their knowledge of data semantics. On the other hand, in some situations, we want some data to opt-out of system cache. At this time, direct I/O would be a better choice.
当我们的系统自己有设计Cache时
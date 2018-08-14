---
title: Linux-Pagecache
date: 2018-08-14 17:17:43
tags:
---


### 什么是pagecache
```
In computing, a page cache, sometimes also called disk cache,[2] is a transparent cache for the pages originating from a secondary storage device such as a hard disk drive (HDD). The operating system keeps a page cache in otherwise unused portions of the main memory (RAM), resulting in quicker access to the contents of cached pages and overall performance improvements. A page cache is implemented in kernels with the paging memory management, and is mostly transparent to applications.
```

### pagecache查看
free -h
pcstat

### pagecache清理
```
释放缓存区内存的方法
1）清理pagecache（页面缓存）
[root@backup ~]# echo 1 > /proc/sys/vm/drop_caches     或者 # sysctl -w vm.drop_caches=1
 
2）清理dentries（目录缓存）和inodes
[root@backup ~]# echo 2 > /proc/sys/vm/drop_caches     或者 # sysctl -w vm.drop_caches=2
 
3）清理pagecache、dentries和inodes
[root@backup ~]# echo 3 > /proc/sys/vm/drop_caches     或者 # sysctl -w vm.drop_caches=3
```

### pagecache应用
RocketMQ和Kafka的应用

### pagecache参考文章
https://www.cnblogs.com/kevingrace/p/5991604.html
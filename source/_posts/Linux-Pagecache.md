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

推荐一款工具pcstat(https://github.com/tobert/pcstat)，它可以直观得显示出指定文件是否被cache，甚至可以知道cache的比例是多少。

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

### page cache与脏页
https://blog.csdn.net/yunsongice/article/details/5850855

```
页面类型 

Linux 中内存页面有三种类型： 

·    Read pages，只读页（或代码页），那些通过主缺页中断从硬盘读取的页面，包括不能修改的静态文件、可执行文件、库文件等。当内核需要它们的时候把它们读到 内存中，当内存不足的时候，内核就释放它们到空闲列表，当程序再次需要它们的时候需要通过缺页中断再次读到内存。 

·    Dirty pages，脏页，指那些在内存中被修改过的数据页，比如文本文件等。这些文件由 pdflush 负责同步到硬盘，内存不足的时候由 kswapd 和 pdflush 把数据写回硬盘并释放内存。 

·    Anonymous pages，匿名页，那些属于某个进程但是又和任何文件无关联，不能被同步到硬盘上，内存不足的时候由 kswapd 负责将它们写到交换分区并释放内存。 
```

### sync
```
SYNC(1)                          User Commands                         SYNC(1)

NAME
       sync - flush file system buffers

SYNOPSIS
       sync [OPTION]

DESCRIPTION
       Force changed blocks to disk, update the super block.

       --help display this help and exit

       --version
              output version information and exit
```

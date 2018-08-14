---
title: Linux-Pagecache
date: 2018-08-14 17:17:43
tags:
---


### 什么是pagecache

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
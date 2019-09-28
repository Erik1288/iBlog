---
title: Linux-Core-dump
date: 2018-10-10 11:15:10
tags:
---

ulimint -a 用来显示当前的各种用户进程限制

```
$ ulimit -a
-t: cpu time (seconds)              unlimited
-f: file size (blocks)              unlimited
-d: data seg size (kbytes)          unlimited
-s: stack size (kbytes)             8192
-c: core file size (blocks)         0
-v: address space (kbytes)          unlimited
-l: locked-in-memory size (kbytes)  unlimited
-u: processes                       709
-n: file descriptors                256
```


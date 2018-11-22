---
title: RocketMQ-Performance-tunning
date: 2018-09-22 15:25:34
tags:
---


### 红帽linux性能调优 参考
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html-single/performance_tuning_guide/index

### 是否用ReentrantLock
```
/**
 经过多人测试，在竞争非常激烈的情况下，用ReentrantLock性能会好过SpinLock
 * introduced since 4.0.x. Determine whether to use mutex reentrantLock when putting message.<br/>
 * By default it is set to false indicating using spin lock when putting message.
 */
private boolean useReentrantLockWhenPutMessage = false;
```


### 
```
// Whether check the CRC32 of the records consumed.
// This ensures no on-the-wire or on-disk corruption to the messages occurred.
// This check adds some overhead,so it may be disabled in cases seeking extreme performance.
private boolean checkCRCOnRecover = true;
```

```
private boolean warmMapedFileEnable = false;
```


###
```
    @ImportantField
    private boolean transientStorePoolEnable = false;
```

###
```
    private boolean transferMsgByHeap = true;
```


###
```
    private boolean slaveReadEnable = false;
```

###
```
    private boolean serverPooledByteBufAllocatorEnable = true;
```

###
```
    /**
     * make make install
     *
     *
     * ../glibc-2.10.1/configure \ --prefix=/usr \ --with-headers=/usr/include \
     * --host=x86_64-linux-gnu \ --build=x86_64-pc-linux-gnu \ --without-gd
     */
    private boolean useEpollNativeSelector = false;
```
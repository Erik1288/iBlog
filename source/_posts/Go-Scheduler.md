---
title: Go-Scheduler
date: 2018-07-24 21:15:58
tags:
---

### 分享
首先，思考几个问题。

什么叫做并行，基础是什么？
在同一时刻内，多件时间一起做。
对处理器，或者单处理器多核心。
有个问题，如果系统总共有4个线程，在4核的机器上，应该不会有上下文切换吧？

什么叫做并发，基础是什么？
并发的概念没有并行严格，指在一个时段内，多件事情一起做。并发的基础是并行和多线程调度，线程是操作系统进程中能够并发执行的实体，是处理器调度和分派的基本单位。 P101:一个线程什么时候获得CPU时间，能够运行多久，是由调度器根据某种调度策略所定。叫做线程的调度，也叫线程的上下文切换

程序在处理一个问题时，是不是线程越多越好？
送分题，不是，1分拿到。从煎鸡蛋和数学题开始讨论，哪一个线程可以适当多一点，哪一个可以少一点，多到多少最好，少到多少最好？

注意到，既然我们在煎鸡蛋的时候无法自己直观得确定内核线程的数量，开小了，并发度不够，开大了，上下文切换太严重，在golang的设计哲学里，决定到底要创建多少个线程合适，这个事情就由golang来做。所以，大部分情况下，go程序运行时，无法控制线程的数量。

更加细粒度的并发执行的调度单位：协程(goroutine)。线程的调度是由操作系统来做的，协程的调度就是由go scheduler (runtime)来做的。本质上，协程如果要执行，还是需要线程的支持。
模仿上面的话，一个协程什么时候获得线程的时间，能够占用线程运行多久，就是go scheduler做的事情。也叫协程的上下文切换，是一种非常轻量级的上下文切换。

当然事情远远没有那么简单


用刚刚所说的理论，来解释下，为什么net包中的IO一定是非阻塞的。
假如，IO是阻塞的，会发生什么情况
在zeroProxy或者tidb中，proxy层来一个用户就启用一个goroutine来单独处理读和写，假如有1万个连接同时进来，所有的连接都在read上阻塞，由于是阻塞IO，那么阻塞的级别就不在goroutine上，而是在内核线程上。这时

查看下源码:
```go
func (fd *FD) Read(p []byte) (int, error) {
	if err := fd.readLock(); err != nil {
		return 0, err
	}
	defer fd.readUnlock()
	if len(p) == 0 {
		// If the caller wanted a zero byte read, return immediately
		// without trying (but after acquiring the readLock).
		// Otherwise syscall.Read returns 0, nil which looks like
		// io.EOF.
		// TODO(bradfitz): make it wait for readability? (Issue 15735)
		return 0, nil
	}
	if err := fd.pd.prepareRead(fd.isFile); err != nil {
		return 0, err
	}
	if fd.IsStream && len(p) > maxRW {
		p = p[:maxRW]
	}
	for {
		n, err := syscall.Read(fd.Sysfd, p)
		if err != nil {
			n = 0
			if err == syscall.EAGAIN && fd.pd.pollable() {
				if err = fd.pd.waitRead(fd.isFile); err == nil {
					continue
				}
			}

			// On MacOS we can see EINTR here if the user
			// pressed ^Z.  See issue #22838.
			if runtime.GOOS == "darwin" && err == syscall.EINTR {
				continue
			}
		}
		err = fd.eofError(n, err)
		return n, err
	}
}
```


http://morsmachine.dk/go-scheduler
https://www.zhihu.com/question/20862617
https://www.zhihu.com/question/22345230
#### 进击的Golang专栏——说说Golang的runtime
https://zhuanlan.zhihu.com/p/27328476

从监控Go进程中的线程开始讲起:

```
[nanxing@zerodb-proxy001 zeroproxy]$ sudo sh start_zeroproxy_public.sh 
[nanxing@zerodb-proxy001 zeroproxy]$ ps axu|grep zero
root     25086  4.6  0.2  34100 18748 pts/0    Sl   14:35   0:00 ./zeroProxy.out -config app.yaml
nanxing  25098  0.0  0.0 112704   972 pts/0    S+   14:35   0:00 grep --color=auto zero
[nanxing@zerodb-proxy001 zeroproxy]$ pstree -p 25086
zeroProxy.out(25086)─┬─{zeroProxy.out}(25087)
                     ├─{zeroProxy.out}(25088)
                     ├─{zeroProxy.out}(25089)
                     ├─{zeroProxy.out}(25090)
                     ├─{zeroProxy.out}(25091)
                     ├─{zeroProxy.out}(25092)
                     ├─{zeroProxy.out}(25093)
                     ├─{zeroProxy.out}(25094)
                     └─{zeroProxy.out}(25095)
[nanxing@zerodb-proxy001 zeroproxy]$ pstree -p 25086|wc -l
9
[nanxing@zerodb-proxy001 zeroproxy]$ pstree -p 25086
zeroProxy.out(25086)─┬─{zeroProxy.out}(25087)
                     ├─{zeroProxy.out}(25088)
                     ├─{zeroProxy.out}(25089)
                     ├─{zeroProxy.out}(25090)
                     ├─{zeroProxy.out}(25091)
                     ├─{zeroProxy.out}(25092)
                     ├─{zeroProxy.out}(25093)
                     ├─{zeroProxy.out}(25094)
                     └─{zeroProxy.out}(25095)
[nanxing@zerodb-proxy001 zeroproxy]$ pstree -p 25086
zeroProxy.out(25086)─┬─{zeroProxy.out}(25087)
                     ├─{zeroProxy.out}(25088)
                     ├─{zeroProxy.out}(25089)
                     ├─{zeroProxy.out}(25090)
                     ├─{zeroProxy.out}(25091)
                     ├─{zeroProxy.out}(25092)
                     ├─{zeroProxy.out}(25093)
                     ├─{zeroProxy.out}(25094)
                     └─{zeroProxy.out}(25095)
[nanxing@zerodb-proxy001 zeroproxy]$ pstree -p 25086
zeroProxy.out(25086)─┬─{zeroProxy.out}(25087)
                     ├─{zeroProxy.out}(25088)
                     ├─{zeroProxy.out}(25089)
                     ├─{zeroProxy.out}(25090)
                     ├─{zeroProxy.out}(25091)
                     ├─{zeroProxy.out}(25092)
                     ├─{zeroProxy.out}(25093)
                     ├─{zeroProxy.out}(25094)
                     ├─{zeroProxy.out}(25095)
                     ├─{zeroProxy.out}(25120)
                     └─{zeroProxy.out}(25121)
[nanxing@zerodb-proxy001 zeroproxy]$ pstree -p 25086
zeroProxy.out(25086)─┬─{zeroProxy.out}(25087)
                     ├─{zeroProxy.out}(25088)
                     ├─{zeroProxy.out}(25089)
                     ├─{zeroProxy.out}(25090)
                     ├─{zeroProxy.out}(25091)
                     ├─{zeroProxy.out}(25092)
                     ├─{zeroProxy.out}(25093)
                     ├─{zeroProxy.out}(25094)
                     ├─{zeroProxy.out}(25095)
                     ├─{zeroProxy.out}(25120)
                     ├─{zeroProxy.out}(25121)
                     └─{zeroProxy.out}(25123)
[nanxing@zerodb-proxy001 zeroproxy]$ pstree -p 25086
zeroProxy.out(25086)─┬─{zeroProxy.out}(25087)
                     ├─{zeroProxy.out}(25088)
                     ├─{zeroProxy.out}(25089)
                     ├─{zeroProxy.out}(25090)
                     ├─{zeroProxy.out}(25091)
                     ├─{zeroProxy.out}(25092)
                     ├─{zeroProxy.out}(25093)
                     ├─{zeroProxy.out}(25094)
                     ├─{zeroProxy.out}(25095)
                     ├─{zeroProxy.out}(25120)
                     ├─{zeroProxy.out}(25121)
                     └─{zeroProxy.out}(25123)
```
注意到，随着性能测试工具的压力逐渐增大，进程自己创建了少量线程。但是内部的协程

pprof -http "0.0.0.0:12345" 'http://10.12.0.241:50011/debug/pprof/profile'


## 这里的图片可以参考下
https://blog.csdn.net/u010824081/article/details/78186611


## 为什么 Go 不实现分代和紧凑(Compact) gc
Generational and Compact gc have already been thought best practice. But golang doesn't adopt it. Who can tell me the reason? 

官方回答：
https://lingchao.xin/post/why-golang-garbage-collector-not-implement-generational-and-compact-gc.html
https://groups.google.com/forum/#!msg/golang-nuts/KJiyv2mV2pU/wdBUH1mHCAAJ


## google's tcmalloc
https://www.jianshu.com/p/7c55fbdef679

在Netty-Jemalloc中提到了jemalloc，tcmalloc是与jemalloc齐名的内存分配算法。

## 有几个重要的问题在这里抛出
### 怎么写一个程序，这个程序会大量创建线程？？
https://stackoverflow.com/questions/28186361/why-does-it-not-create-many-threads-when-many-goroutines-are-blocked-in-writing
这里面有答案
如果能理解这个问题，就基本上可以理解golang的调度器，可以参考下面的这个链接
https://stackoverflow.com/questions/27600587/why-my-program-of-golang-create-so-many-threads


## 如果linux的cpu使用率非常低，cpu的load会不会很高？

## 调度器初始化源码分析
```
func schedinit() {
	// raceinit must be the first call to race detector.
	// In particular, it must be done before mallocinit below calls racemapshadow.
	_g_ := getg()
	if raceenabled {
		_g_.racectx, raceprocctx0 = raceinit()
	}

	sched.maxmcount = 10000

	tracebackinit()
	moduledataverify()
	stackinit()
	mallocinit()
	mcommoninit(_g_.m)
	alginit()       // maps must not be used before this call
	modulesinit()   // provides activeModules
	typelinksinit() // uses maps, activeModules
	itabsinit()     // uses activeModules

	msigsave(_g_.m)
	initSigmask = _g_.m.sigmask

	goargs()
	goenvs()
	parsedebugvars()
	gcinit()

	sched.lastpoll = uint64(nanotime())
	procs := ncpu
	if n, ok := atoi32(gogetenv("GOMAXPROCS")); ok && n > 0 {
		procs = n
	}
	if procresize(procs) != nil {
		throw("unknown runnable goroutine during bootstrap")
	}

	// For cgocheck > 1, we turn on the write barrier at all times
	// and check all pointer writes. We can't do this until after
	// procresize because the write barrier needs a P.
	if debug.cgocheck > 1 {
		writeBarrier.cgo = true
		writeBarrier.enabled = true
		for _, p := range allp {
			p.wbBuf.reset()
		}
	}

	if buildVersion == "" {
		// Condition should never trigger. This code just serves
		// to ensure runtime·buildVersion is kept in the resulting binary.
		buildVersion = "unknown"
	}
}
```
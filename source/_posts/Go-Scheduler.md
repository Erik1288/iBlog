---
title: Go-Scheduler
date: 2018-07-24 21:15:58
tags:
---

### 分享
首先，思考几个问题。

什么叫做并行，基础是什么？
在同一时刻内，多件时间一起做。
有个问题，如果系统总共有4个线程，在4核的机器上，应该不会有上下文切换吧？

什么叫做并发，基础是什么？
并发的概念没有并行严格，指在一个时段内，多件事情一起做。并发的基础是并行和多线程调度，线程是操作系统进程中能够并发执行的实体，是处理器调度和分派的基本单位。

程序在处理一个问题时，是不是线程越多越好？
送分题，不是，1分拿到。从煎鸡蛋和数学题开始讨论，哪一个线程可以适当多一点，哪一个可以少一点，多到多少最好，少到多少最好？

注意到，既然我们在煎鸡蛋的时候无法自己直观得确定内核线程的数量，开小了，并发度不够，开大了，上下文切换太严重，在golang的设计哲学里，这个事情就由golang来做。所以，大部分情况下，go编译的程序运行时，无法控制线程的数量。

更加细粒度的并发执行的调度单位：协程(goroutine)。线程的调度是由操作系统来做的，协程的调度就是由go scheduler (runtime)来做的。






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
如果能理解这个问题，就基本上可以理解golang的调度器，可以参考下面的这个链接
https://stackoverflow.com/questions/27600587/why-my-program-of-golang-create-so-many-threads


## 如果linux的cpu使用率非常低，cpu的load会不会很高？
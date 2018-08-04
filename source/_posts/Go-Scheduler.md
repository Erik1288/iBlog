---
title: Go-Scheduler
date: 2018-07-24 21:15:58
tags:
---


http://morsmachine.dk/go-scheduler
https://www.zhihu.com/question/20862617
https://www.zhihu.com/question/22345230


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


## google's tcmalloc
https://www.jianshu.com/p/7c55fbdef679

在Netty-Jemalloc中提到了jemalloc，tcmalloc是与jemalloc齐名的内存分配算法。

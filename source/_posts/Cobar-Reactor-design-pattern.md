---
title: Cobar——Reactor设计模式
date: 2017-07-24 18:00:58
tags: Cobar
---

### I/O多路复用（multiplexing）和 行为分用（demultiplexing）

### Redis为什么是单线程的？
http://www.dengshenyu.com/%E5%90%8E%E7%AB%AF%E6%8A%80%E6%9C%AF/2016/01/09/redis-reactor-pattern.html

### C10K问题
https://segmentfault.com/a/1190000007240744

### 为什么我们要用NIO而不是BIO？
IO复用模型，Linux下的select、poll和epoll就是干这个的，这三个东西本质上都上做同一种事情。将用户socket对应的fd注册进epoll，然后epoll帮你监听哪些socket上有消息到达，这样就避免了大量的无用操作。此时的socket应该采用非阻塞模式。这样，整个过程只在调用select、poll、epoll这些调用的时候才会阻塞，这个阻塞而且是可以设置超时时间，超时时间过后，程序还可以处理除了epoll之外的逻辑，收发客户消息是不会阻塞的，整个进程或者线程就被充分利用起来，这就是事件驱动，所谓的reactor模式。
但是，和熟悉Linux的同事交流后，发现阻塞IO其实也是能复用线程的，也就是说，用几个线程也是可以处理成千上万的连接的。那怎么做呢？关键在于，阻塞IO的read和write可以向OS提供一种超时机制，如果read或者write没有数据，是会阻塞，但是时间非常短，这依赖于程序向OS发送的超时信号。那既然可以，为什么我们一定是用非阻塞的IO去处理C10K问题，而不是用阻塞IO呢？注意到，如果用阻塞IO，CPU在使用上一直是去探测Socket上究竟有没有数据，而没有正儿八经用在数据的处理上；非阻塞IO的好处在于，这个探测的过程是OS用epoll这种事件机制帮你做的，不仅在探测数据时间的效率上比轮询高，并且CPU还没有浪费在探测上。

### Cobar Reactor

![](Cobar-Reactor-design-pattern/CobarReactorSign.gif)

#### NIO原生API

![](Cobar-Reactor-design-pattern/NioRegister.gif)

#### Cobar NIO Server线程模型

![](Cobar-Reactor-design-pattern/CobarReactor.gif)

#### Cobar的整个模型非常像 Fork/Join
Forks and Joins: When a job arrives at a fork point, it is split into N sub-jobs which are then serviced by n tasks. After being serviced, each sub-job waits until all other sub-jobs are done processing. Then, they are joined again and leave the system. Thus, in parallel programming, we require synchronization as all the parallel processes wait for several other processes to occur.


### 如何深刻理解reactor和proactor？

reactor：能收了你跟俺说一声。
proactor: 你给我收十个字节，收好了跟俺说一声。

Refer:
http://www.dre.vanderbilt.edu/~schmidt/PDF/reactor-siemens.pdf


---
title: Netty-FAQ
date: 2017-10-16 14:25:04
tags:
---

### Netty 编解码器
ByteToMessageDecoder与LengthFieldBasedFrameDecoder的区别

### 重要概念 Future and Promise

### 如何调试时间循环线程
当我们用debug启动netty server时，我们不知道boss线程运行的代码，那怎么样才能发现boss线程当前的执行轨迹呢。如果能找到轨迹，对我们研究boss线程有非常大的帮助。

给boss时间循环线程池起个名字
``` java
 NioEventLoopGroup boss = new NioEventLoopGroup(0, new ThreadFactory() {
    @Override
    public Thread newThread(Runnable r) {
        return new Thread(r, "boss-event-loop");
    }
});
```
如果用的Intellij，就能实现这个效果，首先用debug模式启动netty server。在debug tag下，我们进入Threads，展开**Thread Group "main"**，发现**boss-event-loop**正在处于Running状态。选中**boss-event-loop**，右键点击**suspend**，之后就能看到代码停了下来，去**Frames**tab中选择某一行进行断点调试。

### 聊天程序
Web Socket技术
Long Pooling技术

### 原生NIO可能会被问到的问题

### Netty线程管理，高低水位线(watermark)控制
https://stackoverflow.com/questions/25281124/netty-4-high-and-low-write-watermarks
http://adolgarev.blogspot.ru/2013/12/pipelining-and-flow-control.html?view=flipcard

### Netty线程模型，Netty异常对Inbound(入站)和Outbound(出站) Handler的影响

### Netty内存管理，怎么防止内存过度使用

io模型，上面图里的问题，内存池怎么管理，怎么防止泄露。
mq主从切换，但是网络原因master假死， 这时候slave升级为主，怎么办？
和mysql主从切换一个道理，我不知道怎么办。或者怎么屏蔽。



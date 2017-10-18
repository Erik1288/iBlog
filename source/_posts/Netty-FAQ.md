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


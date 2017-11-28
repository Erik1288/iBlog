---
title: Netty-原理整理
date: 2017-10-16 14:25:04
tags:
---

### Netty 编解码器
ByteToMessageDecoder与LengthFieldBasedFrameDecoder的区别

### 重要概念 Future and Promise

### Netty处理的对象

bytes and messages.

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

### Netty线程管理，高低水位线(watermark)控制，高低水位指的是线程
https://stackoverflow.com/questions/25281124/netty-4-high-and-low-write-watermarks
http://adolgarev.blogspot.ru/2013/12/pipelining-and-flow-control.html?view=flipcard

看下这个文章，是不是可以用Netty的限流来完成这个事情

### Netty线程模型，Netty异常对Inbound(入站)和Outbound(出站) Handler的影响

### Netty内存管理，怎么防止内存过度使用

io模型，上面图里的问题，内存池怎么管理，怎么防止泄露。
mq主从切换，但是网络原因master假死， 这时候slave升级为主，怎么办？
和mysql主从切换一个道理，我不知道怎么办。或者怎么屏蔽。


### Netty bind()方法

``` java
@Override
protected void doRegister() throws Exception {
    boolean selected = false;
    for (;;) {
        try {
            selectionKey = javaChannel().register(eventLoop().unwrappedSelector(), 0, this);
            return;
        } catch (CancelledKeyException e) {
            if (!selected) {
                // Force the Selector to select now as the "canceled" SelectionKey may still be
                // cached and not removed because no Select.select(..) operation was called yet.
                eventLoop().selectNow();
                selected = true;
            } else {
                // We forced a select operation on the selector before but the SelectionKey is still cached
                // for whatever reason. JDK bug ?
                throw e;
            }
        }
    }
}
```
注意到，register的第二个参数为0，也就是说，selector和serverSocketChannel之间仅仅有了注册关系，但没有指定selector到底interest什么事件，那问题是，selector和serverSocketChannel之间的OP_ACCEPT是什么时候完成的？

### 用sendBuf和RecBuf做系统之间的限流，这好像是一个天然的事情

### Netty高性能开发备忘录
http://blog.csdn.net/asdfayw/article/details/71730543

### Netty中的那些坑
http://www.jianshu.com/p/890525ff73cb
http://www.jianshu.com/p/8f22675d71ac

### 用Netty开发中间件：高并发性能优化
http://blog.csdn.net/dc_726/article/details/48978891
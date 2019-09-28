---
title: JVM-NIO-Epoll-and-Kqueue
date: 2019-02-04 13:47:15
tags: JVM
---


https://www.cnblogs.com/moonz-wu/p/4740908.html

```
implRegister(SelectionKeyImpl):227, KQueueSelectorImpl (sun.nio.ch), KQueueSelectorImpl.java
register(AbstractSelectableChannel, int, Object):132, SelectorImpl (sun.nio.ch), SelectorImpl.java
register(Selector, int, Object):212, AbstractSelectableChannel (java.nio.channels.spi), AbstractSelectableChannel.java
doRegister():386, AbstractNioChannel (io.netty.channel.nio), AbstractNioChannel.java
register0(ChannelPromise):508, AbstractChannel$AbstractUnsafe (io.netty.channel), AbstractChannel.java
access$200(AbstractChannel$AbstractUnsafe, ChannelPromise):427, AbstractChannel$AbstractUnsafe (io.netty.channel), AbstractChannel.java
run():486, AbstractChannel$AbstractUnsafe$1 (io.netty.channel), AbstractChannel.java
safeExecute$$$capture(Runnable):163, AbstractEventExecutor (io.netty.util.concurrent), AbstractEventExecutor.java
safeExecute(Runnable):-1, AbstractEventExecutor (io.netty.util.concurrent), AbstractEventExecutor.java
 - Async stack trace
addTask:-1, SingleThreadEventExecutor (io.netty.util.concurrent)
execute:756, SingleThreadEventExecutor (io.netty.util.concurrent)
register:483, AbstractChannel$AbstractUnsafe (io.netty.channel)
register:80, SingleThreadEventLoop (io.netty.channel)
register:74, SingleThreadEventLoop (io.netty.channel)
register:86, MultithreadEventLoopGroup (io.netty.channel)
channelRead:255, ServerBootstrap$ServerBootstrapAcceptor (io.netty.bootstrap)
invokeChannelRead:362, AbstractChannelHandlerContext (io.netty.channel)
invokeChannelRead:348, AbstractChannelHandlerContext (io.netty.channel)
fireChannelRead:340, AbstractChannelHandlerContext (io.netty.channel)
channelRead:1408, DefaultChannelPipeline$HeadContext (io.netty.channel)
invokeChannelRead:362, AbstractChannelHandlerContext (io.netty.channel)
invokeChannelRead:348, AbstractChannelHandlerContext (io.netty.channel)
fireChannelRead:930, DefaultChannelPipeline (io.netty.channel)
read:93, AbstractNioMessageChannel$NioMessageUnsafe (io.netty.channel.nio)
processSelectedKey:677, NioEventLoop (io.netty.channel.nio)
processSelectedKeysOptimized:612, NioEventLoop (io.netty.channel.nio)
processSelectedKeys:529, NioEventLoop (io.netty.channel.nio)
run:491, NioEventLoop (io.netty.channel.nio)
run:905, SingleThreadEventExecutor$5 (io.netty.util.concurrent)
run:748, Thread (java.lang)
```

```
class KQueueSelectorImpl extends SelectorImpl {

    private HashMap<Integer, KQueueSelectorImpl.MapEntry> fdMap;

    KQueueSelectorImpl(SelectorProvider var1) {
        xxx
        this.fdMap = new HashMap();
        xxx
    }

    protected void implRegister(SelectionKeyImpl var1) {
        if (this.closed) {
            throw new ClosedSelectorException();
        } else {
            int var2 = IOUtil.fdVal(var1.channel.getFD());
            this.fdMap.put(var2, new KQueueSelectorImpl.MapEntry(var1));
            ++this.totalChannels;
            this.keys.add(var1);
        }
    }

    protected void implDereg(SelectionKeyImpl var1) throws IOException {
        int var2 = var1.channel.getFDVal();
        this.fdMap.remove(var2);
        this.kqueueWrapper.release(var1.channel);
        --this.totalChannels;
        this.keys.remove(var1);
        this.selectedKeys.remove(var1);
        this.deregister(var1);
        SelectableChannel var3 = var1.channel();
        if (!var3.isOpen() && !var3.isRegistered()) {
            ((SelChImpl)var3).kill();
        }

    }
}
```
---
title: Netty-FileRegion
date: 2019-01-28 09:56:21
tags:
---


doWrite(ChannelOutboundBuffer):226, AbstractNioByteChannel (io.netty.channel.nio), AbstractNioByteChannel.java
doWrite(ChannelOutboundBuffer):314, NioSocketChannel (io.netty.channel.socket.nio), NioSocketChannel.java
flush0():802, AbstractChannel$AbstractUnsafe (io.netty.channel), AbstractChannel.java
flush0():313, AbstractNioChannel$AbstractNioUnsafe (io.netty.channel.nio), AbstractNioChannel.java
flush():770, AbstractChannel$AbstractUnsafe (io.netty.channel), AbstractChannel.java
flush(ChannelHandlerContext):1256, DefaultChannelPipeline$HeadContext (io.netty.channel), DefaultChannelPipeline.java
invokeFlush0():781, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
invokeFlush():773, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
access$1500(AbstractChannelHandlerContext):36, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
run():761, AbstractChannelHandlerContext$16 (io.netty.channel), AbstractChannelHandlerContext.java
runAllTasks(long):408, SingleThreadEventExecutor (io.netty.util.concurrent), SingleThreadEventExecutor.java
run():455, NioEventLoop (io.netty.channel.nio), NioEventLoop.java
run():140, SingleThreadEventExecutor$2 (io.netty.util.concurrent), SingleThreadEventExecutor.java
run():748, Thread (java.lang), Thread.java
```
if (msg instanceof FileRegion) {
    FileRegion region = (FileRegion) msg;
    boolean done = region.transfered() >= region.count();

    if (!done) {
        long flushedAmount = 0;
        if (writeSpinCount == -1) {
            writeSpinCount = config().getWriteSpinCount();
        }

        for (int i = writeSpinCount - 1; i >= 0; i--) {
            long localFlushedAmount = doWriteFileRegion(region);
            if (localFlushedAmount == 0) {
                setOpWrite = true;
                break;
            }

            flushedAmount += localFlushedAmount;
            if (region.transfered() >= region.count()) {
                done = true;
                break;
            }
        }

        in.progress(flushedAmount);
    }

    if (done) {
        in.remove();
    } else {
        // Break the loop and so incompleteWrite(...) is called.
        break;
    }
}
```
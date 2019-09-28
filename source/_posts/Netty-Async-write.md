---
title: Netty-Async-write
date: 2019-01-14 22:34:47
tags:
---

Netty中调用``，其实可以分为两个部分--`write` and `flush`，仅仅调用write是不会触发nio底层channel的write操作，而是将数据先放入
`ChannelOutboundBuffer`中，调用了flush之后，才会真实调用channel的write。
关于ChannelOutboundBuffer的部分，查看
https://www.jianshu.com/p/311425d1c72f

```
ch.pipeline().addLast(handlerExecutorGroup, "in1", new ChannelInboundHandlerAdapter() {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("handler(in1) write buffer in thread:" + Thread.currentThread().getName());
        ChannelFuture channelFuture = ctx.writeAndFlush(msg);

        System.out.println(channelFuture.isDone());

        channelFuture.addListener(new GenericFutureListener<Future<? super Void>>() {
            @Override
            public void operationComplete(Future<? super Void> future) throws Exception {
                // 既然是回调，一定要明确，这个回调是哪个线程执行的。

                // 这里有两种情况会触发调用`operationComplete`

                // 1 在addListener时，会主动尝试调用notifyListeners，只要future.idDone()是true的，那执行这段代码的线程就是执行
                // `channelRead`的线程，也就是当前Pipeline节点(AbstractChannelHandlerContext)的Executor。

                // 2 在addListener时，future.idDone()返回的是false，所以不会在第一种情况下触发。当nioEventLoop底层的线程在调用了
                // flush0()之后，也会调用notifyListeners，触发`operationComplete`。调用栈如下所示。但会去判断下是不是在Promise的
                // EventExecutor执行的，如果不是，会把`notifyListenersNow`作为任务给Promise的EventExecutor执行。Promise的
                // EventExecutor其实就是当前Pipeline节点(AbstractChannelHandlerContext)的Executor。

                System.out.println("write complete in thread:" + Thread.currentThread().getName());
            }
        });
    }
});

notifyListeners():417, DefaultPromise (io.netty.util.concurrent), DefaultPromise.java
trySuccess(Object):103, DefaultPromise (io.netty.util.concurrent), DefaultPromise.java
trySuccess(Promise, Object, InternalLogger):48, PromiseNotificationUtil (io.netty.util.internal), PromiseNotificationUtil.java
safeSuccess(ChannelPromise):703, ChannelOutboundBuffer (io.netty.channel), ChannelOutboundBuffer.java
remove():258, ChannelOutboundBuffer (io.netty.channel), ChannelOutboundBuffer.java
removeBytes(long):338, ChannelOutboundBuffer (io.netty.channel), ChannelOutboundBuffer.java
doWrite(ChannelOutboundBuffer):411, NioSocketChannel (io.netty.channel.socket.nio), NioSocketChannel.java
flush0():938, AbstractChannel$AbstractUnsafe (io.netty.channel), AbstractChannel.java
flush0():360, AbstractNioChannel$AbstractNioUnsafe (io.netty.channel.nio), AbstractNioChannel.java
flush():905, AbstractChannel$AbstractUnsafe (io.netty.channel), AbstractChannel.java
flush(ChannelHandlerContext):1370, DefaultChannelPipeline$HeadContext (io.netty.channel), DefaultChannelPipeline.java
invokeFlush0():776, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
invokeFlush():768, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
access$1500(AbstractChannelHandlerContext):38, AbstractChannelHandlerContext (io.netty.channel), AbstractChannelHandlerContext.java
write(AbstractChannelHandlerContext, Object, ChannelPromise):1175, AbstractChannelHandlerContext$WriteAndFlushTask (io.netty.channel), AbstractChannelHandlerContext.java
run():1098, AbstractChannelHandlerContext$AbstractWriteTask (io.netty.channel), AbstractChannelHandlerContext.java
safeExecute$$$capture(Runnable):163, AbstractEventExecutor (io.netty.util.concurrent), AbstractEventExecutor.java
safeExecute(Runnable):-1, AbstractEventExecutor (io.netty.util.concurrent), AbstractEventExecutor.java
 - Async stack trace
addTask:-1, SingleThreadEventExecutor (io.netty.util.concurrent)
execute:756, SingleThreadEventExecutor (io.netty.util.concurrent)
safeExecute:1036, AbstractChannelHandlerContext (io.netty.channel)
write:825, AbstractChannelHandlerContext (io.netty.channel)
writeAndFlush:794, AbstractChannelHandlerContext (io.netty.channel)
writeAndFlush:837, AbstractChannelHandlerContext (io.netty.channel)
channelRead:41, HandlerExecutor$1$1 (com.eric)
invokeChannelRead:362, AbstractChannelHandlerContext (io.netty.channel)
access$600:38, AbstractChannelHandlerContext (io.netty.channel)
run:353, AbstractChannelHandlerContext$7 (io.netty.channel)
run:54, DefaultEventLoop (io.netty.channel)
run:905, SingleThreadEventExecutor$5 (io.netty.util.concurrent)
run:30, FastThreadLocalRunnable (io.netty.util.concurrent)
run:748, Thread (java.lang)
```
---
title: Netty-DirectMemory-leak
date: 2019-02-20 19:22:54
tags:
---


http://calvin1978.blogcn.com/articles/netty-leak.html
江南白衣的文章

https://www.jianshu.com/p/4e96beb37935

netty 堆外内存泄露排查盛宴


所谓内存泄漏，主要是针对池化的ByteBuf。ByteBuf对象被JVM GC掉之前，没有调用release()去把底下的DirectByteBuffer或byte[]归还到池里，会导致池越来越大。而非池化的ByteBuf，即使像DirectByteBuf那样可能会用到System.gc()，但终归会被release掉的，不会出大事。

如果ByteBuf调用了Release，是不会被GC清理掉的。


### 会自动释放ByteBuf
ByteToMessageDecoder
SimpleChannelInboundHandler
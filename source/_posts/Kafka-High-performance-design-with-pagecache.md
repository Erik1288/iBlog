---
title: Kafka-High-performance-design-with-pagecache
date: 2017-11-12 22:47:44
tags: Kafka
---


在开始所有的分析之前，有几段文章一定要先读一读

http://blog.csdn.net/tototuzuoquan/article/details/73437890

http://www.jianshu.com/p/eba0067b1e1a
大神作者


_java.nio.channels.FileChannel_
`public abstract void force(boolean metaData) throws java.io.IOException`
Forces any updates to this channel's file to be written to the storage device that contains it.
If this channel's file resides on a local storage device then when this method returns it is guaranteed that all changes made to the file since this channel was created, or since this method was last invoked, will have been written to that device. This is useful for ensuring that critical information is not lost in the event of a system crash.
If the file does not reside on a local device then no such guarantee is made.
The metaData parameter can be used to limit the number of I/O operations that this method is required to perform. Passing false for this parameter indicates that only updates to the file's content need be written to storage; passing true indicates that updates to both the file's content and metadata must be written, which generally requires at least one more I/O operation. Whether this parameter actually has any effect is dependent upon the underlying operating system and is therefore unspecified.
Invoking this method may cause an I/O operation to occur even if the channel was only opened for reading. Some operating systems, for example, maintain a last-access time as part of a file's metadata, and this time is updated whenever the file is read. Whether or not this is actually done is system-dependent and is therefore unspecified.
This method is only guaranteed to force changes that were made to this channel's file via the methods defined in this class. **It may or may not force changes that were made by modifying the content of a mapped byte buffer obtained by invoking the map method. Invoking the force method of the mapped byte buffer will force changes made to the buffer's content to be written.**

_java.nio.MappedByteBuffer_
`public final MappedByteBuffer force()`
Forces any changes made to this buffer's content to be written to the storage device containing the mapped file.
If the file mapped into this buffer resides on a local storage device then when this method returns it is guaranteed that all changes made to the buffer since it was created, or since this method was last invoked, will have been written to that device.
If the file does not reside on a local device then no such guarantee is made.
If this buffer was not mapped in read/write mode (java.nio.channels.FileChannel.MapMode.READ_WRITE) then invoking this method has no effect.
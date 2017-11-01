---
title: Cobar——Reactor设计模式
date: 2017-07-24 18:00:58
tags: Cobar
---



### I/O多路复用技术（multiplexing）是什么？

引用下别人的言论：

作者：郭春阳
链接：https://www.zhihu.com/question/28594409/answer/52835876
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

下面举一个例子，模拟一个tcp服务器处理30个客户socket。假设你是一个老师，让30个学生解答一道题目，然后检查学生做的是否正确，你有下面几个选择：1. 第一种选择：按顺序逐个检查，先检查A，然后是B，之后是C、D。。。这中间如果有一个学生卡主，全班都会被耽误。这种模式就好比，你用循环挨个处理socket，根本不具有并发能力。2. 第二种选择：你创建30个分身，每个分身检查一个学生的答案是否正确。 这种类似于为每一个用户创建一个进程或者线程处理连接。3. 第三种选择，你站在讲台上等，谁解答完谁举手。这时C、D举手，表示他们解答问题完毕，你下去依次检查C、D的答案，然后继续回到讲台上等。此时E、A又举手，然后去处理E和A。。。 这种就是IO复用模型，Linux下的select、poll和epoll就是干这个的。将用户socket对应的fd注册进epoll，然后epoll帮你监听哪些socket上有消息到达，这样就避免了大量的无用操作。此时的socket应该采用非阻塞模式。这样，整个过程只在调用select、poll、epoll这些调用的时候才会阻塞，收发客户消息是不会阻塞的，整个进程或者线程就被充分利用起来，这就是事件驱动，所谓的reactor模式。


### demultiplexing

http://www.dre.vanderbilt.edu/~schmidt/PDF/reactor-siemens.pdf


![](Cobar-Reactor-design-pattern/CobarReactorSign.gif)

### NIO原生API

![](Cobar-Reactor-design-pattern/NioRegister.gif)

### Cobar NIO Server线程模型

![](Cobar-Reactor-design-pattern/CobarReactor.gif)




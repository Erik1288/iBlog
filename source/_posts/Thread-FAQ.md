---
title: Thread-FAQ
date: 2017-10-27 11:07:35
tags: THread
---


### TODO hexo new Thread-Poolize-thread

```
假如有一个工厂，工厂里面有10个工人，每个工人同时只能做一件任务。

　　因此只要当10个工人中有工人是空闲的，来了任务就分配给空闲的工人做；

　　当10个工人都有任务在做时，如果还来了任务，就把任务进行排队等待；

　　如果说新任务数目增长的速度远远大于工人做任务的速度，那么此时工厂主管可能会想补救措施，比如重新招4个临时工人进来；

　　然后就将任务也分配给这4个临时工人做；

　　如果说着14个工人做任务的速度还是不够，此时工厂主管可能就要考虑不再接收新的任务或者抛弃前面的一些任务了。

　　当这14个工人当中有人空闲时，而新任务增长的速度又比较缓慢，工厂主管可能就考虑辞掉4个临时工了，只保持原来的10个工人，毕竟请额外的工人是要花钱的。
```


### Executor里面的线程是怎么被复用的？

``` java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

### 问题，ExecutorService的submit和execute有什么区别








http://blog.csdn.net/zhandoushi1982/article/details/5506597
线程的挂起操作实质上就是使线程进入“非可执行”状态下，在这个状态下CPU不会分给线程时间片，进入这个状态可以用来暂停一个线程的运行。在线程挂起后，可以通过重新唤醒线程来使之恢复运行。

### ThreadLocal


### Future


### Promise

### Difference between ReentrantLock and sync

### join

### yield

### fork


### Java线程面试题

如果用了ReentrantLock，还需要设置自旋锁么？

假如用sync做了线程互斥，A和B两个线程竞争monitor，A拿到了monitor，如果A迟迟不释放，B将永远等待，怎么避免这种情况？

CopyOnWrite

15个Java多线程面试题及回答

1)现在有T1、T2、T3三个线程，你怎样保证T2在T1执行完后执行，T3在T2执行完后执行？

这个线程问题通常会在第一轮或电话面试阶段被问到，目的是检测你对”join”方法是否熟悉。这个多线程问题比较简单，可以用join方法实现。

2)在Java中Lock接口比synchronized块的优势是什么？你需要实现一个高效的缓存，它允许多个用户读，但只允许一个用户写，以此来保持它的完整性，你会怎样去实现它？

lock接口在多线程和并发编程中最大的优势是它们为读和写分别提供了锁，它能满足你写像ConcurrentHashMap这样的高性能数据结构和有条件的阻塞。Java线程面试的问题越来越会根据面试者的回答来提问。我强烈建议在你去参加多线程的面试之前认真读一下Locks，因为当前其大量用于构建电子交易终统的客户端缓存和交易连接空间。

3)在java中wait和sleep方法的不同？

通常会在电话面试中经常被问到的Java线程面试问题。最大的不同是在等待时wait会释放锁，而sleep一直持有锁。Wait通常被用于线程间交互，sleep通常被用于暂停执行。

4）用Java实现阻塞队列。

这是一个相对艰难的多线程面试问题，它能达到很多的目的。第一，它可以检测侯选者是否能实际的用Java线程写程序；第二，可以检测侯选者对并发场景的理解，并且你可以根据这个问很多问题。如果他用wait()和notify()方法来实现阻塞队列，你可以要求他用最新的Java 5中的并发类来再写一次。

5）用Java写代码来解决生产者——消费者问题。

与上面的问题很类似，但这个问题更经典，有些时候面试都会问下面的问题。在Java中怎么解决生产者——消费者问题，当然有很多解决方法，我已经分享了一种用阻塞队列实现的方法。有些时候他们甚至会问怎么实现哲学家进餐问题。

6）用Java编程一个会导致死锁的程序，你将怎么解决？

这是我最喜欢的Java线程面试问题，因为即使死锁问题在写多线程并发程序时非常普遍，但是很多侯选者并不能写deadlock free code（无死锁代码？），他们很挣扎。只要告诉他们，你有N个资源和N个线程，并且你需要所有的资源来完成一个操作。为了简单这里的n可以替换为2，越大的数据会使问题看起来更复杂。通过避免Java中的死锁来得到关于死锁的更多信息。

7) 什么是原子操作，Java中的原子操作是什么？

非常简单的java线程面试问题，接下来的问题是你需要同步一个原子操作。

8) Java中的volatile关键是什么作用？怎样使用它？在Java中它跟synchronized方法有什么不同？

自从Java 5和Java内存模型改变以后，基于volatile关键字的线程问题越来越流行。应该准备好回答关于volatile变量怎样在并发环境中确保可见性。

9) 什么是竞争条件？你怎样发现和解决竞争？

这是一道出现在多线程面试的高级阶段的问题。大多数的面试官会问最近你遇到的竞争条件，以及你是怎么解决的。有些时间他们会写简单的代码，然后让你检测出代码的竞争条件。可以参考我之前发布的关于Java竞争条件的文章。在我看来这是最好的java线程面试问题之一，它可以确切的检测候选者解决竞争条件的经验，or writing code which is free of data race or any other race condition。关于这方面最好的书是《Concurrency practices in Java》。

10) 你将如何使用thread dump？你将如何分析Thread dump？

在UNIX中你可以使用kill -3，然后thread dump将会打印日志，在windows中你可以使用”CTRL+Break”。非常简单和专业的线程面试问题，但是如果他问你怎样分析它，就会很棘手。

11) 为什么我们调用start()方法时会执行run()方法，为什么我们不能直接调用run()方法？

这是另一个非常经典的java多线程面试问题。这也是我刚开始写线程程序时候的困惑。现在这个问题通常在电话面试或者是在初中级Java面试的第一轮被问到。这个问题的回答应该是这样的，当你调用start()方法时你将创建新的线程，并且执行在run()方法里的代码。但是如果你直接调用run()方法，它不会创建新的线程也不会执行调用线程的代码。阅读我之前写的《start与run方法的区别》这篇文章来获得更多信息。

12) Java中你怎样唤醒一个阻塞的线程？

这是个关于线程和阻塞的棘手的问题，它有很多解决方法。如果线程遇到了IO阻塞，我并且不认为有一种方法可以中止线程。如果线程因为调用wait()、sleep()、或者join()方法而导致的阻塞，你可以中断线程，并且通过抛出InterruptedException来唤醒它。我之前写的《How to deal with blocking methods in java》有很多关于处理线程阻塞的信息。

13)在Java中CycliBarriar和CountdownLatch有什么区别？

这个线程问题主要用来检测你是否熟悉JDK5中的并发包。这两个的区别是CyclicBarrier可以重复使用已经通过的障碍，而CountdownLatch不能重复使用。

14) 什么是不可变对象，它对写并发应用有什么帮助？

另一个多线程经典面试问题，并不直接跟线程有关，但间接帮助很多。这个java面试问题可以变的非常棘手，如果他要求你写一个不可变对象，或者问你为什么String是不可变的。

15) 你在多线程环境中遇到的常见的问题是什么？你是怎么解决它的？

多线程和并发程序中常遇到的有Memory-interface、竞争条件、死锁、活锁和饥饿。问题是没有止境的，如果你弄错了，将很难发现和调试。这是大多数基于面试的，而不是基于实际应用的Java线程问题。

http://www.importnew.com/27105.html
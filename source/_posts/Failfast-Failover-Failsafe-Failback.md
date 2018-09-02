---
title: Failfast-Failover-Failsafe-Failback
date: 2018-09-02 21:41:38
tags:
---


https://blog.csdn.net/luojinbai/article/details/53428370?utm_source=itdadao&utm_medium=referral



### Fail-Fast

维基百科地址：http://en.wikipedia.org/wiki/Fail-fast

> Fail-fast is a property of a system or module with respect to its response to failures. A fail-fast system is designed to immediately report at its interface anyfailure or condition that is likely to lead to failure. Fail-fast systems are usually designed to stop normal operation rather than attempt to continue a possibly flawed process. Such designs often check the system’s state at several points in an operation, so any failures can be detected early. A fail-fast module passes the responsibility for handling errors, but not detecting them, to the next-higher system design level.

从字面含义看就是“快速失败”，尽可能的发现系统中的错误，使系统能够按照事先设定好的错误的流程执行，对应的方式是“fault-tolerant（错误容忍）”。以JAVA集合（Collection）的快速失败为例，当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。例如：当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常（发现错误执行设定好的错误的流程），产生fail-fast事件。

### Fail-Over

维基百科地址：http://en.wikipedia.org/wiki/Failover

> In computing, failover is switching to a redundant or standby computer server, system, hardware component or network upon the failure or abnormal termination of the previously active application,[1] server, system, hardware component, or network. Failover and switchover are essentially the same operation, except that failover is automatic and usually operates without warning, while switchover requires human intervention.

Fail-Over的含义为“失效转移”，是一种备份操作模式，当主要组件异常时，其功能转移到备份组件。其要点在于有主有备，且主故障时备可启用，并设置为主。如Mysql的双Master模式，当正在使用的Master出现故障时，可以拿备Master做主使用。

### Faile-Safe

维基百科地址：http://en.wikipedia.org/wiki/Fail-safe

> A fail-safe or fail-secure device is one that, in the event of failure, responds in a way that will cause no harm, or at least a minimum of harm, to other devices or danger to personnel.

Fail-Safe的含义为“失效安全”，即使在故障的情况下也不会造成伤害或者尽量减少伤害。维基百科上一个形象的例子是红绿灯的“冲突监测模块”当监测到错误或者冲突的信号时会将十字路口的红绿灯变为闪烁错误模式，而不是全部显示为绿灯。

另外就是我们误用的“自动功能降级”翻译做“Auto-Degrade”会更好一些。

### Fail-Back

Fail-over之后的自动恢复，在簇网络系统（有两台或多台服务器互联的网络）中，由于要某台服务器进行维修，需要网络资源和服务暂时重定向到备用系统。在此之后将网络资源和服务器恢复为由原始主机提供的过程，称为自动恢复。
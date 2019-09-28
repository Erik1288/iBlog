---
title: Linux-Disk-io-scheduler
date: 2019-02-26 22:13:09
tags:
---


https://www.cnblogs.com/bamanzi/p/linux-disk-io-scheduler.html
```
On GNU/Linux, the queue scheduler determines the order in which requests to a block
device are actually sent to the underlying device.

The default is Completely Fair Queueing, or cfq. It’s okay for casual use on laptops and desktops, where it helps prevent
I/O starvation, but it’s terrible for servers. It causes very poor response times under the types of workload that MySQL generates, because it stalls some requests in the queue
needlessly.

You can see which schedulers are available, and which one is active, with the following
command:

$ cat /sys/block/sda/queue/scheduler
noop deadline [cfq]
You should replace sda with the device name of the disk you’re interested in. In our
example, the square brackets indicate which scheduler is in use for this device.

The
other two choices are suitable for server-class hardware, and in most cases they work
about equally well. The noop scheduler is appropriate for devices that do their own
scheduling behind the scenes, such as hardware RAID controllers and SANs, and deadline is fine both for RAID controllers and disks that are directly attached. Our benchmarks show very little difference between these two. The main thing is to use anything
but cfq, which can cause severe performance problems.

Take this advice with a grain of salt, though, because the disk schedulers actually come
in many variations in different kernels, and there is no indication of that in their names.
```
---
title: Linux-OOM-killer
date: 2018-09-22 22:12:32
tags:
---


https://blog.csdn.net/zgrjkflmkyc/article/details/77645570
https://blog.csdn.net/liukuan73/article/details/43238623


# 立即重新启动计算机    （Reboots the kernel without first unmounting file systems or syncing disks attached to the system）
echo "b" > /proc/sysrq-trigger     
# 立即关闭计算机（shuts off the system）
echo "o" > /proc/sysrq-trigger
# 导出内存分配的信息 （可以用/var/log/message 查看）（Outputs memory statistics to the console）   

echo "m" > /proc/sysrq-trigger     
# 导出当前CPU寄存器信息和标志位的信息（Outputs all flags and registers to the console）
echo "p" > /proc/sysrq-trigger     
# 导出线程状态信息    （Outputs a list of processes to the console）
echo "t" > /proc/sysrq-trigger
# 故意让系统崩溃    （ Crashes the system without first unmounting file systems or syncing disks attached to the system）
echo "c" > /proc/sysrq-trigger

# 立即重新挂载所有的文件系统 （Attempts to sync disks attached to the system）
echo "s" > /proc/sysrq-trigger     
# 立即重新挂载所有的文件系统为只读 （Attempts to unmount and remount all file systems as read-only）
echo "u" > /proc/sysrq-trigger
# 手动触发oom-killer
echo "f" > /proc/sysrq-trigger

---------------------

本文来自 njuitjf 的CSDN 博客 ，全文地址请点击：https://blog.csdn.net/njuitjf/article/details/12045527?utm_source=copy 
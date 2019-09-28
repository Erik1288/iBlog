---
title: Linux-NUMA
date: 2019-02-18 17:33:32
tags: Linux
---

### 极佳
https://blog.csdn.net/liguangxianbin/article/details/80797400

### 为什么要有NUMA
在NUMA架构出现前，CPU欢快的朝着频率越来越高的方向发展。受到物理极限的挑战，又转为核数越来越多的方向发展。如果每个core的工作性质 都是share-nothing（类似于map-reduce的node节点的作业属性），那么也许就不会有NUMA。由于所有CPU Core都是通过共享一个北桥来读取内存，随着核数如何的发展，北桥在响应时间上的性能瓶颈越来越明显。于是，聪明的硬件设计师们，先到了把内存控制器 （原本北桥中读取内存的部分）也做个拆分，平分到了每个die上。于是NUMA就出现了！

### NUMA是什么
NUMA中，虽然内存直接attach在CPU上，但是由于内存被平均分配在了各个die上。只有当CPU访问自身直接attach内存对应的物理地址时，才会有较短的响应时间（后称Local Access）。而如果需要访问其他CPU attach的内存的数据时，就需要通过inter-connect通道访问，响应时间就相比之前变慢了（后称Remote Access）。所以NUMA（Non-Uniform Memory Access）就此得名。
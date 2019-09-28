---
title: Linux-Cacheline
date: 2018-08-29 09:31:23
tags:
---

在一个高级语言中编程，理论上你只要把两个变量A和B定义在相邻的位置，就有可能公用同一个Cacheline。
如果变量A的值被修改，OS会将整块Cacheline都置为失效，变量B在Cache中的值自然也失效了。
如果要告诉JVM，定义在相邻位置的变量不要放在一个Cacheline中，那么需要使用Padding方式将变量相隔开。


### linux cacheline
cat /sys/devices/system/cpu/cpu1/cache/index0/coherency_line_size 


### 怎么写一个程序证明Cacheline在起作用
http://processon.com/chart_image/5b85fd4ce4b06fc64ad9ebf5.png

https://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247484179&idx=1&sn=4251523d0fa59a53b4a4261818978f30&chksm=fb0be987cc7c6091a58fa0bc5278b99ff79c4054bebee4dabd34a62993e0f72c7703d0354f64&mpshare=1&scene=1&srcid=08295eOSDzz41jjz3kt8l2q6&key=ef1a4b05b697803cb8f54afe4edc2443783e0e02110f86a4ef8eb30b96b36f3e256cfba9721a9cca57fca60ca6d3b571c4555f2d5da6bf98f233876799e65900cdb847bee4f39152f77db07280bc0948&ascene=0&uin=MjcyMjMxNTA0Mg%3D%3D&devicetype=iMac+MacBookPro11%2C4+OSX+OSX+10.12.6+build(16G1212)&version=12010210&nettype=WIFI&lang=zh_CN&fontScale=100&pass_ticket=9bTk3VRopZ%2FeeIIwYm7T5ZB9ac0zDWCk4nquX%2BTk%2B4982wcuo%2FydTBHDBJyIR2t2

这篇文章从一个非常好的角度切入


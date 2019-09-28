---
title: JVM-JMH
date: 2018-11-07 15:03:25
tags: JVM
---


https://blog.csdn.net/lxbjkben/article/details/79410740

R大
在HotSpot VM上跑microbenchmark切记不要在main()里跑循环计时就完事。这是典型错误。重要的事情重复三遍：请用JMH，请用JMH，请用JMH。除非非常了解HotSpot的实现细节，在main里这样跑循环计时得到的结果其实对一般程序员来说根本没有任何意义，因为无法解释。




https://blog.csdn.net/lxbjkben/article/details/79410740
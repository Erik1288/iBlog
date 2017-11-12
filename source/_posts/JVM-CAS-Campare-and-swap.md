---
title: JVM-CAS-Campare-and-swap
date: 2017-11-10 19:48:55
tags: JVM
---
> 仅供学习交流,如有错误请指出,如要转载请加上出处,谢谢

### CAS

CAS固然好，但会出现ABA问题

对于某些系统，ABA不会产生问题，但也有不能容忍ABA的系统
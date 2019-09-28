---
title: JVM-GC-G1-FAQ
date: 2018-12-21 17:13:48
tags: JVM
---

### 这个需要看源码验证下
When the aforementioned young collection takes place, dead objects are collected and any remaining live objects are evacuated and compacted into the Survivor space. G1 has an explicit hard-margin, defined by the G1ReservePercent (default 10%), that results in a percentage of the heap always being available for the Survivor space during evacuation.
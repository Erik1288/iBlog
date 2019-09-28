---
title: JVM-GC-Write-barrier
date: 2018-12-20 19:01:43
tags: JVM
---


### 在不同的场景中，Write Barrier有着不太一样的语义
A write barrier in a garbage collector is a fragment of code emitted by the compiler immediately before every store operation to ensure that (e.g.) generational invariants are maintained. A write barrier in a memory system, also known as a memory barrier, is a hardware-specific compiler intrinsic that ensures that all preceding memory operations "happen before" all subsequent ones.[citation needed]
---
title: JVM-GC-because-of-allocation-failed
date: 2018-11-17 11:06:31
tags: JVM
---


class VM_CollectForAllocation : public VM_GC_Operation

### 所有由于内存分配而引起的GC的操作，都是继承于`CollectForAllocation`。

#### G1由于内存分配而引起GC
class VM_G1OperationWithAllocRequest : public VM_CollectForAllocation

#### CMS由于内存分配而引起GC（可能是cms，也可能是fullgc）
class VM_GenCollectForAllocation : public VM_CollectForAllocation
---
title: JVM-Oop-Klass
date: 2018-12-07 09:37:45
tags: JVM
---


```
class oopDesc {
  friend class VMStructs;
  friend class JVMCIVMStructs;
 private:
  volatile markOop _mark; // typedef class markOopDesc* markOop
  union _metadata {
    Klass*      _klass;
    narrowKlass _compressed_klass;
  } _metadata;

  // Fast access to barrier set. Must be initialized.
  static BarrierSet* _bs;
}
```
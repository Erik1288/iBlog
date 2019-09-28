---
title: JVM-SoftReference
date: 2018-11-30 09:54:43
tags: JVM
---





```
// NOTE: process_phase*() are largely similar, and at a high level
// merely iterate over the extant list applying a predicate to
// each of its elements and possibly removing that element from the
// list and applying some further closures to that element.
// We should consider the possibility of replacing these
// process_phase*() methods by abstracting them into
// a single general iterator invocation that receives appropriate
// closures that accomplish this work.

// (SoftReferences only) Traverse the list and remove any SoftReferences whose
// referents are not alive, but that should be kept alive for policy reasons.
// Keep alive the transitive closure of all such referents.
void
ReferenceProcessor::process_phase1(DiscoveredList&    refs_list,
                                   ReferencePolicy*   policy,
                                   BoolObjectClosure* is_alive,
                                   OopClosure*        keep_alive,
                                   VoidClosure*       complete_gc) {
  assert(policy != NULL, "Must have a non-NULL policy");
  DiscoveredListIterator iter(refs_list, keep_alive, is_alive);
  // Decide which softly reachable refs should be kept alive.
  while (iter.has_next()) {
    iter.load_ptrs(DEBUG_ONLY(!discovery_is_atomic() /* allow_null_referent */));
    bool referent_is_dead = (iter.referent() != NULL) && !iter.is_referent_alive();
    if (referent_is_dead &&
        !policy->should_clear_reference(iter.obj(), _soft_ref_timestamp_clock)) {
      log_develop_trace(gc, ref)("Dropping reference (" INTPTR_FORMAT ": %s"  ") by policy",
                                 p2i(iter.obj()), iter.obj()->klass()->internal_name());
      // Remove Reference object from list
      iter.remove();
      // keep the referent around
      iter.make_referent_alive();
      iter.move_to_next();
    } else {
      iter.next();
    }
  }
  // Close the reachable set
  complete_gc->do_void();
  log_develop_trace(gc, ref)(" Dropped " SIZE_FORMAT " dead Refs out of " SIZE_FORMAT " discovered Refs by policy, from list " INTPTR_FORMAT,
                             iter.removed(), iter.processed(), p2i(&refs_list));
}
```



相关JVM参数
-XX:SoftRefLRUPolicyMSPerMB=0
-XX:+ParallelRefProcEnabled
-XX:+PrintReferenceGC

-XX:SoftRefLRUPolicyMSPerMB=N 这个参数比较有用的，官方解释是：Soft reference在虚拟机中比在客户集中存活的更长一些。其清除频率可以用命令行参数 -XX:SoftRefLRUPolicyMSPerMB=<N>来控制，这可以指定每兆堆空闲空间的 soft reference 保持存活（一旦它不强可达了）的毫秒数，这意味着每兆堆中的空闲空间中的 soft reference 会（在最后一个强引用被回收之后）存活1秒钟。注意，这是一个近似的值，因为  soft reference 只会在垃圾回收时才会被清除，而垃圾回收并不总在发生。系统默认为一秒，我觉得没必要等1秒，客户集中不用就立刻清除，改为 -XX:SoftRefLRUPolicyMSPerMB=0；

作者：MangoDai
链接：https://www.jianshu.com/p/3eb976e03457
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

### 疑问
ReferenceQueue在哪里，如果有ReferenceQueue，就可以查看到SoftReference的清理情况
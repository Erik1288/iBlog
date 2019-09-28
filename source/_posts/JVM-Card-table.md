---
title: JVM-Card-table
date: 2018-11-21 20:18:31
tags: JVM
---


```
DirtyCardToOopClosure::do_MemRegion space.cpp:116
ClearNoncleanCardWrapper::do_MemRegion cardTableRS.cpp:243
CardTableModRefBSForCTRS::process_stride parCardTableModRefBS.cpp:157
CardTableModRefBSForCTRS::non_clean_card_iterate_parallel_work parCardTableModRefBS.cpp:65
CardTableModRefBSForCTRS::non_clean_card_iterate_possibly_parallel cardTableModRefBSForCTRS.cpp:106
CardTableRS::younger_refs_in_space_iterate cardTableRS.cpp:343
Generation::younger_refs_in_space_iterate generation.cpp:280
CardGeneration::younger_refs_iterate cardGeneration.cpp:314
CardTableRS::younger_refs_iterate cardTableRS.cpp:152
GenCollectedHeap::young_process_roots genCollectedHeap.cpp:671
ParNewGenTask::work parNewGeneration.cpp:608
GangWorker::run_task workgroup.cpp:332
GangWorker::loop workgroup.cpp:342
AbstractGangWorker::run workgroup.cpp:291
thread_native_entry os_bsd.cpp:720
_pthread_body 0x00007fff942c793b
_pthread_start 0x00007fff942c7887
thread_start 0x00007fff942c708d
<unknown> 0x0000000000000000
```

### 讲得超赞
https://segmentfault.com/a/1190000004682407


### 
https://www.jianshu.com/p/5037459097ee

```
class CardTableModRefBSForCTRS: public CardTableModRefBS {
  // This is an array, one element per covered region of the card table.
  // Each entry is itself an array, with one element per chunk in the
  // covered region.  Each entry of these arrays is the lowest non-clean
  // card of the corresponding chunk containing part of an object from the
  // previous chunk, or else NULL.
  typedef jbyte*  CardPtr;
  typedef CardPtr* CardArr;
  CardArr* _lowest_non_clean;
  size_t*  _lowest_non_clean_chunk_size;
  uintptr_t* _lowest_non_clean_base_chunk_index;
  volatile int* _last_LNC_resizing_collection;
}
```
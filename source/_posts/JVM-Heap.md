---
title: JVM-Heap
date: 2018-10-07 16:38:55
tags:
---


Universe::initialize_heap() universe.cpp:755
universe_init() universe.cpp:672
init_globals() init.cpp:110
Threads::create_vm(JavaVMInitArgs*, bool*) thread.cpp:3630
JNI_CreateJavaVM_inner(JavaVM_**, void**, void*) jni.cpp:3937
::JNI_CreateJavaVM(JavaVM **, void **, void *) jni.cpp:4032
InitializeJVM 0x0000000104767e50
JavaMain 0x000000010476702c
_pthread_body 0x00007fffc9ad493b
_pthread_start 0x00007fffc9ad4887
thread_start 0x00007fffc9ad408d

``` c++
CollectedHeap* Universe::create_heap() {
  assert(_collectedHeap == NULL, "Heap already created");
#if !INCLUDE_ALL_GCS
  if (UseParallelGC) {
    fatal("UseParallelGC not supported in this VM.");
  } else if (UseG1GC) {
    fatal("UseG1GC not supported in this VM.");
  } else if (UseConcMarkSweepGC) {
    fatal("UseConcMarkSweepGC not supported in this VM.");
#else
  if (UseParallelGC) {
    return Universe::create_heap_with_policy<ParallelScavengeHeap, GenerationSizer>();
  } else if (UseG1GC) {
    return Universe::create_heap_with_policy<G1CollectedHeap, G1CollectorPolicy>();
  } else if (UseConcMarkSweepGC) {
    return Universe::create_heap_with_policy<GenCollectedHeap, ConcurrentMarkSweepPolicy>();
#endif
  } else if (UseSerialGC) {
    return Universe::create_heap_with_policy<GenCollectedHeap, MarkSweepPolicy>();
  }

  ShouldNotReachHere();
  return NULL;
}
```

```
jint GenCollectedHeap::initialize() {
  CollectedHeap::pre_initialize();

  // While there are no constraints in the GC code that HeapWordSize
  // be any particular value, there are multiple other areas in the
  // system which believe this to be true (e.g. oop->object_size in some
  // cases incorrectly returns the size in wordSize units rather than
  // HeapWordSize).
  guarantee(HeapWordSize == wordSize, "HeapWordSize must equal wordSize");

  // Allocate space for the heap.

  char* heap_address;
  ReservedSpace heap_rs;

  size_t heap_alignment = collector_policy()->heap_alignment();

  heap_address = allocate(heap_alignment, &heap_rs);

  if (!heap_rs.is_reserved()) {
    vm_shutdown_during_initialization(
      "Could not reserve enough space for object heap");
    return JNI_ENOMEM;
  }

  initialize_reserved_region((HeapWord*)heap_rs.base(), (HeapWord*)(heap_rs.base() + heap_rs.size()));

  _rem_set = collector_policy()->create_rem_set(reserved_region());
  set_barrier_set(rem_set()->bs());

  ReservedSpace young_rs = heap_rs.first_part(gen_policy()->young_gen_spec()->max_size(), false, false);
  _young_gen = gen_policy()->young_gen_spec()->init(young_rs, rem_set());
  heap_rs = heap_rs.last_part(gen_policy()->young_gen_spec()->max_size());

  ReservedSpace old_rs = heap_rs.first_part(gen_policy()->old_gen_spec()->max_size(), false, false);
  _old_gen = gen_policy()->old_gen_spec()->init(old_rs, rem_set());
  clear_incremental_collection_failed();

#if INCLUDE_ALL_GCS
  // If we are running CMS, create the collector responsible
  // for collecting the CMS generations.
  if (collector_policy()->is_concurrent_mark_sweep_policy()) {
    bool success = create_cms_collector();
    if (!success) return JNI_ENOMEM;
  }
#endif // INCLUDE_ALL_GCS

  return JNI_OK;
}
```
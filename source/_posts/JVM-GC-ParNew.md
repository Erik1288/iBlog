---
title: JVM-GC-ParNew
date: 2018-11-20 10:37:47
tags: JVM
---


### 动态年龄阈值计算
https://blog.csdn.net/FoolishAndStupid/article/details/77596050

### 疑问
young gc只是会复制存活对象，也就是说需要找到所有活的对象，而不是去找到所有死的对象?

### young gc 过程
```
void ParNewGeneration::collect(bool   full,
                               bool   clear_all_soft_refs,
                               size_t size,
                               bool   is_tlab) {
  assert(full || size > 0, "otherwise we don't want to collect");

  GenCollectedHeap* gch = GenCollectedHeap::heap();

  _gc_timer->register_gc_start();

  AdaptiveSizePolicy* size_policy = gch->gen_policy()->size_policy();
  WorkGang* workers = gch->workers();
  assert(workers != NULL, "Need workgang for parallel work");
  uint active_workers =
       AdaptiveSizePolicy::calc_active_workers(workers->total_workers(),
                                               workers->active_workers(),
                                               Threads::number_of_non_daemon_threads());
  active_workers = workers->update_active_workers(active_workers);
  log_info(gc,task)("Using %u workers of %u for evacuation", active_workers, workers->total_workers());

  _old_gen = gch->old_gen();

  // If the next generation is too full to accommodate worst-case promotion
  // from this generation, pass on collection; let the next generation
  // do it.
  if (!collection_attempt_is_safe()) {
    gch->set_incremental_collection_failed();  // slight lie, in that we did not even attempt one
    return;
  }
  assert(to()->is_empty(), "Else not collection_attempt_is_safe");

  _gc_tracer.report_gc_start(gch->gc_cause(), _gc_timer->gc_start());
  gch->trace_heap_before_gc(gc_tracer());

  init_assuming_no_promotion_failure();

  if (UseAdaptiveSizePolicy) {
    set_survivor_overflow(false);
    size_policy->minor_collection_begin();
  }

  GCTraceTime(Trace, gc, phases) t1("ParNew", NULL, gch->gc_cause());

  age_table()->clear();
  to()->clear(SpaceDecorator::Mangle);

  gch->save_marks();

  // Set the correct parallelism (number of queues) in the reference processor
  ref_processor()->set_active_mt_degree(active_workers);

  // Need to initialize the preserved marks before the ThreadStateSet c'tor.
  _preserved_marks_set.init(active_workers);

  // Always set the terminator for the active number of workers
  // because only those workers go through the termination protocol.
  ParallelTaskTerminator _term(active_workers, task_queues());
  ParScanThreadStateSet thread_state_set(active_workers,
                                         *to(), *this, *_old_gen, *task_queues(),
                                         _overflow_stacks, _preserved_marks_set,
                                         desired_plab_sz(), _term);

  thread_state_set.reset(active_workers, promotion_failed());

  {
    StrongRootsScope srs(active_workers);

    ParNewGenTask tsk(this, _old_gen, reserved().end(), &thread_state_set, &srs);
    gch->rem_set()->prepare_for_younger_refs_iterate(true);
    // It turns out that even when we're using 1 thread, doing the work in a
    // separate thread causes wide variance in run times.  We can't help this
    // in the multi-threaded case, but we special-case n=1 here to get
    // repeatable measurements of the 1-thread overhead of the parallel code.
    // Might multiple workers ever be used?  If yes, initialization
    // has been done such that the single threaded path should not be used.
    if (workers->total_workers() > 1) {
      workers->run_task(&tsk);
    } else {
      tsk.work(0);
    }
  }

  thread_state_set.reset(0 /* Bad value in debug if not reset */,
                         promotion_failed());

  // Trace and reset failed promotion info.
  if (promotion_failed()) {
    thread_state_set.trace_promotion_failed(gc_tracer());
  }

  // Process (weak) reference objects found during scavenge.
  ReferenceProcessor* rp = ref_processor();
  IsAliveClosure is_alive(this);
  ScanWeakRefClosure scan_weak_ref(this);
  KeepAliveClosure keep_alive(&scan_weak_ref);
  ScanClosure               scan_without_gc_barrier(this, false);
  ScanClosureWithParBarrier scan_with_gc_barrier(this, true);
  set_promo_failure_scan_stack_closure(&scan_without_gc_barrier);
  EvacuateFollowersClosureGeneral evacuate_followers(gch,
    &scan_without_gc_barrier, &scan_with_gc_barrier);
  rp->setup_policy(clear_all_soft_refs);
  // Can  the mt_degree be set later (at run_task() time would be best)?
  rp->set_active_mt_degree(active_workers);
  ReferenceProcessorStats stats;
  if (rp->processing_is_mt()) {
    ParNewRefProcTaskExecutor task_executor(*this, *_old_gen, thread_state_set);
    stats = rp->process_discovered_references(&is_alive, &keep_alive,
                                              &evacuate_followers, &task_executor,
                                              _gc_timer);
  } else {
    thread_state_set.flush();
    gch->save_marks();
    stats = rp->process_discovered_references(&is_alive, &keep_alive,
                                              &evacuate_followers, NULL,
                                              _gc_timer);
  }
  _gc_tracer.report_gc_reference_stats(stats);
  _gc_tracer.report_tenuring_threshold(tenuring_threshold());

  if (!promotion_failed()) {
    // Swap the survivor spaces.
    eden()->clear(SpaceDecorator::Mangle);
    from()->clear(SpaceDecorator::Mangle);
    if (ZapUnusedHeapArea) {
      // This is now done here because of the piece-meal mangling which
      // can check for valid mangling at intermediate points in the
      // collection(s).  When a young collection fails to collect
      // sufficient space resizing of the young generation can occur
      // and redistribute the spaces in the young generation.  Mangle
      // here so that unzapped regions don't get distributed to
      // other spaces.
      to()->mangle_unused_area();
    }
    swap_spaces();

    // A successful scavenge should restart the GC time limit count which is
    // for full GC's.
    size_policy->reset_gc_overhead_limit_count();

    assert(to()->is_empty(), "to space should be empty now");

    adjust_desired_tenuring_threshold();
  } else {
    handle_promotion_failed(gch, thread_state_set);
  }
  _preserved_marks_set.reclaim();
  // set new iteration safe limit for the survivor spaces
  from()->set_concurrent_iteration_safe_limit(from()->top());
  to()->set_concurrent_iteration_safe_limit(to()->top());

  plab_stats()->adjust_desired_plab_sz();

  TASKQUEUE_STATS_ONLY(thread_state_set.print_termination_stats());
  TASKQUEUE_STATS_ONLY(thread_state_set.print_taskqueue_stats());

  if (UseAdaptiveSizePolicy) {
    size_policy->minor_collection_end(gch->gc_cause());
    size_policy->avg_survived()->sample(from()->used());
  }

  // We need to use a monotonically non-decreasing time in ms
  // or we will see time-warp warnings and os::javaTimeMillis()
  // does not guarantee monotonicity.
  jlong now = os::javaTimeNanos() / NANOSECS_PER_MILLISEC;
  update_time_of_last_gc(now);

  rp->set_enqueuing_is_done(true);
  if (rp->processing_is_mt()) {
    ParNewRefProcTaskExecutor task_executor(*this, *_old_gen, thread_state_set);
    rp->enqueue_discovered_references(&task_executor);
  } else {
    rp->enqueue_discovered_references(NULL);
  }
  rp->verify_no_references_recorded();

  gch->trace_heap_after_gc(gc_tracer());

  _gc_timer->register_gc_end();

  _gc_tracer.report_gc_end(_gc_timer->gc_end(), _gc_timer->time_partitions());
}

```
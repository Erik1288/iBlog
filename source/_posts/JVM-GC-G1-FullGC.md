---
title: JVM-GC-G1-FullGC
date: 2018-12-13 19:43:11
tags: JVM
---


> The G1 garbage collector is designed to avoid full collections, but when the concurrent collections can't reclaim memory fast enough a fall back full GC will occur. The current implementation of the full GC for G1 uses a single threaded mark-sweep-compact algorithm. We intend to parallelize the mark-sweep-compact algorithm and use the same number of threads as the Young and Mixed collections do. The number of threads can be controlled by the -XX:ParallelGCThreads option, but this will also affect the number of threads used for Young and Mixed collections. http://openjdk.java.net/jeps/307

从以上描述来看，以前G1的FullGC是用单线程去做MSC，

```
bool G1CollectedHeap::do_full_collection(bool explicit_gc,
                                         bool clear_all_soft_refs) {
  assert_at_safepoint(true /* should_be_vm_thread */);

  if (GCLocker::check_active_before_gc()) {
    return false;
  }

  STWGCTimer* gc_timer = G1MarkSweep::gc_timer();
  gc_timer->register_gc_start();

  SerialOldTracer* gc_tracer = G1MarkSweep::gc_tracer();
  GCIdMark gc_id_mark;
  gc_tracer->report_gc_start(gc_cause(), gc_timer->gc_start());

  SvcGCMarker sgcm(SvcGCMarker::FULL);
  ResourceMark rm;

  print_heap_before_gc();
  print_heap_regions();
  trace_heap_before_gc(gc_tracer);

  size_t metadata_prev_used = MetaspaceAux::used_bytes();

  _verifier->verify_region_sets_optional();

  const bool do_clear_all_soft_refs = clear_all_soft_refs ||
                           collector_policy()->should_clear_all_soft_refs();

  ClearedAllSoftRefs casr(do_clear_all_soft_refs, collector_policy());

  {
    IsGCActiveMark x;

    // Timing
    assert(!GCCause::is_user_requested_gc(gc_cause()) || explicit_gc, "invariant");
    GCTraceCPUTime tcpu;

    {
      GCTraceTime(Info, gc) tm("Pause Full", NULL, gc_cause(), true);
      TraceCollectorStats tcs(g1mm()->full_collection_counters());
      TraceMemoryManagerStats tms(true /* fullGC */, gc_cause());

      G1HeapTransition heap_transition(this);
      g1_policy()->record_full_collection_start();

      // Note: When we have a more flexible GC logging framework that
      // allows us to add optional attributes to a GC log record we
      // could consider timing and reporting how long we wait in the
      // following two methods.
      wait_while_free_regions_coming();
      // If we start the compaction before the CM threads finish
      // scanning the root regions we might trip them over as we'll
      // be moving objects / updating references. So let's wait until
      // they are done. By telling them to abort, they should complete
      // early.
      _cm->root_regions()->abort();
      _cm->root_regions()->wait_until_scan_finished();
      append_secondary_free_list_if_not_empty_with_lock();

      gc_prologue(true);
      increment_total_collections(true /* full gc */);
      increment_old_marking_cycles_started();

      assert(used() == recalculate_used(), "Should be equal");

      _verifier->verify_before_gc();

      _verifier->check_bitmaps("Full GC Start");
      pre_full_gc_dump(gc_timer);

#if defined(COMPILER2) || INCLUDE_JVMCI
      DerivedPointerTable::clear();
#endif

      // Disable discovery and empty the discovered lists
      // for the CM ref processor.
      ref_processor_cm()->disable_discovery();
      ref_processor_cm()->abandon_partial_discovery();
      ref_processor_cm()->verify_no_references_recorded();

      // Abandon current iterations of concurrent marking and concurrent
      // refinement, if any are in progress.
      concurrent_mark()->abort();

      // Make sure we'll choose a new allocation region afterwards.
      _allocator->release_mutator_alloc_region();
      _allocator->abandon_gc_alloc_regions();
      g1_rem_set()->cleanupHRRS();

      // We may have added regions to the current incremental collection
      // set between the last GC or pause and now. We need to clear the
      // incremental collection set and then start rebuilding it afresh
      // after this full GC.
      abandon_collection_set(collection_set());

      tear_down_region_sets(false /* free_list_only */);
      collector_state()->set_gcs_are_young(true);

      // See the comments in g1CollectedHeap.hpp and
      // G1CollectedHeap::ref_processing_init() about
      // how reference processing currently works in G1.

      // Temporarily make discovery by the STW ref processor single threaded (non-MT).
      ReferenceProcessorMTDiscoveryMutator stw_rp_disc_ser(ref_processor_stw(), false);

      // Temporarily clear the STW ref processor's _is_alive_non_header field.
      ReferenceProcessorIsAliveMutator stw_rp_is_alive_null(ref_processor_stw(), NULL);

      ref_processor_stw()->enable_discovery();
      ref_processor_stw()->setup_policy(do_clear_all_soft_refs);

      // Do collection work
      {
        HandleMark hm;  // Discard invalid handles created during gc
        G1MarkSweep::invoke_at_safepoint(ref_processor_stw(), do_clear_all_soft_refs);
      }

      assert(num_free_regions() == 0, "we should not have added any free regions");
      rebuild_region_sets(false /* free_list_only */);

      // Enqueue any discovered reference objects that have
      // not been removed from the discovered lists.
      ref_processor_stw()->enqueue_discovered_references();

#if defined(COMPILER2) || INCLUDE_JVMCI
      DerivedPointerTable::update_pointers();
#endif

      MemoryService::track_memory_usage();

      assert(!ref_processor_stw()->discovery_enabled(), "Postcondition");
      ref_processor_stw()->verify_no_references_recorded();

      // Delete metaspaces for unloaded class loaders and clean up loader_data graph
      ClassLoaderDataGraph::purge();
      MetaspaceAux::verify_metrics();

      // Note: since we've just done a full GC, concurrent
      // marking is no longer active. Therefore we need not
      // re-enable reference discovery for the CM ref processor.
      // That will be done at the start of the next marking cycle.
      assert(!ref_processor_cm()->discovery_enabled(), "Postcondition");
      ref_processor_cm()->verify_no_references_recorded();

      reset_gc_time_stamp();
      // Since everything potentially moved, we will clear all remembered
      // sets, and clear all cards.  Later we will rebuild remembered
      // sets. We will also reset the GC time stamps of the regions.
      clear_rsets_post_compaction();
      check_gc_time_stamps();

      resize_if_necessary_after_full_collection();

      // We should do this after we potentially resize the heap so
      // that all the COMMIT / UNCOMMIT events are generated before
      // the compaction events.
      print_hrm_post_compaction();

      if (_hot_card_cache->use_cache()) {
        _hot_card_cache->reset_card_counts();
        _hot_card_cache->reset_hot_cache();
      }

      // Rebuild remembered sets of all regions.
      uint n_workers =
        AdaptiveSizePolicy::calc_active_workers(workers()->total_workers(),
                                                workers()->active_workers(),
                                                Threads::number_of_non_daemon_threads());
      workers()->update_active_workers(n_workers);
      log_info(gc,task)("Using %u workers of %u to rebuild remembered set", n_workers, workers()->total_workers());

      ParRebuildRSTask rebuild_rs_task(this);
      workers()->run_task(&rebuild_rs_task);

      // Rebuild the strong code root lists for each region
      rebuild_strong_code_roots();

      if (true) { // FIXME
        MetaspaceGC::compute_new_size();
      }

#ifdef TRACESPINNING
      ParallelTaskTerminator::print_termination_counts();
#endif

      // Discard all rset updates
      JavaThread::dirty_card_queue_set().abandon_logs();
      assert(dirty_card_queue_set().completed_buffers_num() == 0, "DCQS should be empty");

      // At this point there should be no regions in the
      // entire heap tagged as young.
      assert(check_young_list_empty(), "young list should be empty at this point");

      // Update the number of full collections that have been completed.
      increment_old_marking_cycles_completed(false /* concurrent */);

      _hrm.verify_optional();
      _verifier->verify_region_sets_optional();

      _verifier->verify_after_gc();

      // Clear the previous marking bitmap, if needed for bitmap verification.
      // Note we cannot do this when we clear the next marking bitmap in
      // G1ConcurrentMark::abort() above since VerifyDuringGC verifies the
      // objects marked during a full GC against the previous bitmap.
      // But we need to clear it before calling check_bitmaps below since
      // the full GC has compacted objects and updated TAMS but not updated
      // the prev bitmap.
      if (G1VerifyBitmaps) {
        GCTraceTime(Debug, gc)("Clear Bitmap for Verification");
        _cm->clear_prev_bitmap(workers());
      }
      _verifier->check_bitmaps("Full GC End");

      // Start a new incremental collection set for the next pause
      collection_set()->start_incremental_building();

      clear_cset_fast_test();

      _allocator->init_mutator_alloc_region();

      g1_policy()->record_full_collection_end();

      // We must call G1MonitoringSupport::update_sizes() in the same scoping level
      // as an active TraceMemoryManagerStats object (i.e. before the destructor for the
      // TraceMemoryManagerStats is called) so that the G1 memory pools are updated
      // before any GC notifications are raised.
      g1mm()->update_sizes();

      gc_epilogue(true);

      heap_transition.print();

      print_heap_after_gc();
      print_heap_regions();
      trace_heap_after_gc(gc_tracer);

      post_full_gc_dump(gc_timer);
    }

    gc_timer->register_gc_end();
    gc_tracer->report_gc_end(gc_timer->gc_end(), gc_timer->time_partitions());
  }

  return true;
}
```

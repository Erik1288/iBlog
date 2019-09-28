---
title: JVM-GC-G1-SATB
date: 2018-12-13 18:46:32
tags: JVM
---

class HeapRegion: public G1ContiguousSpace {

  // The start of the unmarked area. The unmarked area extends from this
  // word until the top and/or end of the region, and is the part
  // of the region for which no marking was done, i.e. objects may
  // have been allocated in this part since the last mark phase.
  // "prev" is the top at the start of the last completed marking.
  // "next" is the top at the start of the in-progress marking (if any.)
  HeapWord* _prev_top_at_mark_start;
  HeapWord* _next_top_at_mark_start;

}
---
title: Linux-Madvise
date: 2018-09-26 10:05:09
tags:
---


madvise, posix_madvise -- give advice about use of memory

```
DESCRIPTION
     The madvise() system call allows a process that has knowledge of its memory behavior to describe it to the system.  The advice passed in
     may be used by the system to alter its virtual memory paging strategy.  This advice may improve application and system performance.  The
     behavior specified in advice can only be one of the following values:

     MADV_NORMAL      Indicates that the application has no advice to give on its behavior in the specified address range.  This is the sys-
                      tem default behavior.  This is used with madvise() system call.

     POSIX_MADV_NORMAL
                      Same as MADV_NORMAL but used with posix_madvise() system call.

     MADV_SEQUENTIAL  Indicates that the application expects to access this address range in a sequential manner.  This is used with
                      madvise() system call.

     POSIX_MADV_SEQUENTIAL
                      Same as MADV_SEQUENTIAL but used with posix_madvise() system call.

     MADV_RANDOM      Indicates that the application expects to access this address range in a random manner.  This is used with madvise()
                      system call.

     POSIX_MADV_RANDOM
                      Same as MADV_RANDOM but used with posix_madvise() system call.

     MADV_WILLNEED    Indicates that the application expects to access this address range soon.  This is used with madvise() system call.

     POSIX_MADV_WILLNEED
                      Same as MADV_WILLNEED but used with posix_madvise() system call.

     MADV_DONTNEED    Indicates that the application is not expecting to access this address range soon.  This is used with madvise() system
                      call.

     POSIX_MADV_DONTNEED
                      Same as MADV_DONTNEED but used with posix_madvise() system call.

     MADV_FREE        Indicates that the application will not need the information contained in this address range, so the pages may be
                      reused right away.  The address range will remain valid.  This is used with madvise() system call.

     MADV_ZERO_WIRED_PAGES
                      Indicates that the application would like the wired pages in this address range to be zeroed out if the address range
                      is deallocated without first unwiring the pages (i.e. a munmap(2) without a preceding munlock(2) or the application
                      quits).  This is used with madvise() system call.

```
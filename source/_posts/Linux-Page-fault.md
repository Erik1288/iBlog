---
title: Linux-Page-fault
date: 2018-09-21 23:36:36
tags:
---


https://liam0205.me/2017/09/01/page-fault/


当进程在进行一些计算时，CPU 会请求内存中存储的数据。在这个请求过程中，CPU 发出的地址是逻辑地址（虚拟地址），然后交由 CPU 当中的 MMU 单元进行内存寻址，找到实际物理内存上的内容。若是目标虚存空间中的内存页（因为某种原因），在物理内存中没有对应的页帧，那么 CPU 就无法获取数据。这种情况下，CPU 是无法进行计算的，于是它就会报告一个缺页错误（Page Fault）。

因为 CPU 无法继续进行进程请求的计算，并报告了缺页错误，用户进程必然就中断了。这样的中断称之为缺页中断。在报告 Page Fault 之后，进程会从用户态切换到系统态，交由操作系统内核的 Page Fault Handler 处理缺页错误。


### minor 和 major page fault
https://www.quora.com/What-is-the-difference-between-minor-and-major-page-fault-in-Linux



https://processon.com/diagraming/5bfb6518e4b018141e80aecd

JVM启动参数-XX:+AlwaysPreTouch
```
virtualspace.cpp
static bool commit_expanded(char* start, size_t size, size_t alignment, bool pre_touch, bool executable) {
  if (os::commit_memory(start, size, alignment, executable)) {
    if (pre_touch || AlwaysPreTouch) {
      pretouch_expanded_memory(start, start + size);
    }
    return true;
  }

  debug_only(warning(
      "INFO: os::commit_memory(" PTR_FORMAT ", " PTR_FORMAT
      " size=" SIZE_FORMAT ", executable=%d) failed",
      p2i(start), p2i(start + size), size, executable);)

  return false;
}

static void pretouch_expanded_memory(void* start, void* end) {
  assert(is_ptr_aligned(start, os::vm_page_size()), "Unexpected alignment");
  assert(is_ptr_aligned(end,   os::vm_page_size()), "Unexpected alignment");

  os::pretouch_memory(start, end);
}

os.cpp
void os::pretouch_memory(void* start, void* end, size_t page_size) {
  for (volatile char *p = (char*)start; p < (char*)end; p += page_size) {
    *p = 0;
  }
}
```
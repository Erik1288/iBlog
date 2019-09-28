---
title: JVM-JIT
date: 2018-09-29 15:27:11
tags: JVM
---


### 查看JIT
jstat -printcompilation [pid] 1000


### jvm调优之分层编译
https://www.jianshu.com/p/318617435789
```
除了纯编译和默认的mixed之外，jvm 从jdk6u25 之后，引入了分层编译。HotSpot 内置两种编译器，分别是client启动时的c1编译器和server启动时的c2编译器，c2在将代码编译成机器代码的时候需要搜集大量的统计信息以便在编译的时候进行优化，因此编译出来的代码执行效率比较高，代价是程序启动时间比较长，而且需要执行比较长的时间，才能达到最高性能；与之相反， c1的目标是使程序尽快进入编译执行的阶段，所以在编译前需要搜集的信息比c2要少，编译速度因此提高很多，但是付出的代价是编译之后的代码执行效率比较低，但尽管如此，c1编译出来的代码在性能上比解释执行的性能已经有很大的提升，所以所谓的分层编译，就是一种折中方式，在系统执行初期，执行频率比较高的代码先被c1编译器编译，以便尽快进入编译执行，然后随着时间的推移，执行频率较高的代码再被c2编译器编译，以达到最高的性能。
```



``` src/share/vm/runtime/globals.hpp
  develop(intx, HugeMethodLimit,  8000,                                     \
          "Don't compile methods larger than this if "                      \
          "+DontCompileHugeMethods") 
```

``` src/share/vm/runtime/compilationPolicy.cpp
// Returns true if m is allowed to be compiled
bool CompilationPolicy::can_be_compiled(methodHandle m, int comp_level) {
  // allow any levels for WhiteBox
  assert(WhiteBoxAPI || comp_level == CompLevel_all || is_compile(comp_level), "illegal compilation level");

  if (m->is_abstract()) return false;
  if (DontCompileHugeMethods && m->code_size() > HugeMethodLimit) return false;

  // Math intrinsics should never be compiled as this can lead to
  // monotonicity problems because the interpreter will prefer the
  // compiled code to the intrinsic version.  This can't happen in
  // production because the invocation counter can't be incremented
  // but we shouldn't expose the system to this problem in testing
  // modes.
  if (!AbstractInterpreter::can_be_compiled(m)) {
    return false;
  }
  if (comp_level == CompLevel_all) {
    if (TieredCompilation) {
      // enough to be compilable at any level for tiered
      return !m->is_not_compilable(CompLevel_simple) || !m->is_not_compilable(CompLevel_full_optimization);
    } else {
      // must be compilable at available level for non-tiered
      return !m->is_not_compilable(CompLevel_highest_tier);
    }
  } else if (is_compile(comp_level)) {
    return !m->is_not_compilable(comp_level);
  }
  return false;
}
```


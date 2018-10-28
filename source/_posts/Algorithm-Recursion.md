---
title: Algorithm-Recursion
date: 2018-10-27 22:37:31
tags:
---


### 递归模式代码

```
result recursion(level, param...) {
    # recursion terminator
    if 
        return ;
    
    processData(param);
    result = recursion(level + 1, param...);
    // 作为本level层，已经处理完了所有level大于本层的事情
    processResult(result);
}
```
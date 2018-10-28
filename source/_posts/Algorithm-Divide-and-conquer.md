---
title: Algorithm-Divide-and-conquer
date: 2018-10-27 23:01:35
tags:
---

### 分治模式代码
```
result divideConquer(problem, param...) {
    
    #recursion terminator
    if problem is none {
        return ;
    }

    subProblems[] = splitProblem(problem);
    
    subResult1 = divideConquer(subProblems[0], p1...);
    subResult2 = divideConquer(subProblems[1], p2...);
    subResult3 = divideConquer(subProblems[2], p3...);
    
    result = processResult(subResult1, subResult2, subResult3 ...);
}
```


### 作者还是总结能力很强的
https://blog.csdn.net/cyfcsd/article/details/49924291
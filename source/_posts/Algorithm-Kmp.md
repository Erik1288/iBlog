---
title: Algorithm-Kmp
date: 2018-10-24 20:09:48
tags:
---


http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html

1.当红色部分相同（即S[k]==S[q]）时，则当前 next 数组的值为上一次 next 的值加一（即next[q] = k++），如上图所示。

2.当红色部分不等的时候，则需要对绿色部分递推求解 k’ = next[k-1]，然后再对新的 k’ 位置字符与 q 位置字符进行匹配，如果相等，则 next[q] = k’+1，否则，执行递推匹配，直到k’=0时递推结束。比如，模式串“ABCABXABCABC”，最后一个字符C的next数组值为3。（因为C之前的最长公共前后缀为“ABCAB”，而“ABCAB”的最长公共前后缀为“AB”，其长度为2，又源于第三个字符C与最后一个字符C匹配，所以最后一个字符C的next数组值为3）

作者：knowalker
链接：https://www.jianshu.com/p/53a0c6ffbf77
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

String : S1 A S2 B S1 A S2 D
Pattern: S1 A S2 B S1 A S2 C
这时发现D和C不一样，需要找到Pattern在最后的C位置的Next值。这里有一层递归的关系。

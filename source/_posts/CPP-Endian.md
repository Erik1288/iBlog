---
title: CPP-Endian
date: 2019-03-18 17:06:57
tags:
---


https://www.jianshu.com/p/befb3a7de297

```
// 在 ubuntu12.04 系统下，运行的结果是“Little Endian”；
// 在 iOS 系统下，运行的结果也是“Little Endian”。
void main() {
    int i = 0x12345678;
    char* pc = (char*)&i;
    if (*pc == 0x12) {
        printf("Big Endian\n");
    } else if (*pc == 0x78) {
        printf("Little Endian\n");
    }
}
```
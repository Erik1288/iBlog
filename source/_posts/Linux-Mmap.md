---
title: Linux-Mmap
date: 2017-11-20 17:22:52
tags: Linux
---

http://xiaoz5919.iteye.com/blog/2093323

### 问题
mmap中map和unmap的细节是什么样的？

```
public static void main(String[] args) throws Exception {
    int len = 1024 * 1024 * 1024;
    File file = new File("/Users/eric/Code/kick-off/MappedByteBuffer-kick-off/src/main/resources/bigFile.cc");

    RandomAccessFile accessFile = new RandomAccessFile(file, "rw");

    MappedByteBuffer mappedByteBuffer = accessFile.getChannel().map(FileChannel.MapMode.READ_WRITE, 0, len);

    for (int i = 0; i < len; i++) {
        mappedByteBuffer.put((byte) (i % 128));
    }
    System.out.println(mappedByteBuffer.limit());
    System.out.println("done");

    Thread.sleep(10000000);

//        file.delete();
}
```

eric@ubuntu:~/Software/pcstat$ pmap -X 4690
4690:   /home/eric/Software/jdk1.7.0_79/bin/java -Didea.launcher.port=7532 -Didea.launcher.bin.path=/home/eric/Software/idea-IU-162.1121.32/bin -Dfile.encoding=UTF-8 -classpath /home/eric/Software/jdk1.7.0_79/jre/lib/charsets.jar:/home/eric/Software/jdk1.7.0_79/jre/lib/deploy.jar:/home/eric/Software/jdk1.7.0_79/jre/lib/ext/dnsns.jar:/home/eric/Software/jdk1.7.0_79/jre/lib/ext/localedata.jar:/home/eric/Software/jdk1.7.0_79/jre/lib/ext/sunec.jar:/home/eric/Software/jdk1.7.0_79/jre/lib/ext/sunjce_provider.jar:/home/
         Address Perm   Offset Device   Inode    Size    Rss    Pss Referenced Anonymous Shared_Hugetlb Private_Hugetlb Swap SwapPss Locked Mapping
        00400000 r-xp 00000000  08:01  801959       4      4      2          4         0              0               0    0       0      0 java
        00600000 rw-p 00000000  08:01  801959       4      4      4          4         4              0               0    0       0      0 java
        0122b000 rw-p 00000000  00:00       0     132     12     12         12        12              0               0    0       0      0 [heap]
        dbe00000 rw-p 00000000  00:00       0   10560   2048   2048       2048      2048              0               0    0       0      0 
        dc850000 rw-p 00000000  00:00       0  158720      0      0          0         0              0               0    0       0      0 
        e6350000 rw-p 00000000  00:00       0   21184      0      0          0         0              0               0    0       0      0 
        e7800000 rw-p 00000000  00:00       0  317440      0      0          0         0              0               0    0       0      0 
        fae00000 rw-p 00000000  00:00       0   21248   4096   4096       4096      4096              0               0    0       0      0 
        fc2c0000 rw-p 00000000  00:00       0   62720      0      0          0         0              0               0    0       0      0 
    7f55bcfff000 rw-s 00000000  08:01 1053783 1048576 140544 140544     140544         0              0               0    0       0      0 bigFile.cc
    7f55fcfff000 rw-p 00000000  00:00       0   49156      4      4          4         4              0               0    0       0      0 
    7f5600000000 rw-p 00000000  00:00       0     168    168    168        168       168              0               0    0       0      0 

pmap Rss PageCache: 140544

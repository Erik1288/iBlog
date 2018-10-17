---
title: Linux-Pmap
date: 2018-09-25 13:04:31
tags:
---


http://man.linuxde.net/pmap

用pmap来查看mmap堆外内存

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

运行程序前，
eric@ubuntu:~/Software/pcstat$ free -m
              total        used        free      shared  buff/cache   available
Mem:           1982        1551         213           8         217         238
Swap:          1021         437         584

运行sleep后:
eric@ubuntu:~/Software/pcstat$ free -m
              total        used        free      shared  buff/cache   available
Mem:           1982        1566          81           8         335         221
Swap:          1021         437         584


eric@ubuntu:~/Software/pcstat$ ./pcstat /home/eric/IdeaProjects/memmap/src/main/resources/bigFile.cc
|--------------------------------------------------------------+----------------+------------+-----------+---------|
| Name                                                         | Size           | Pages      | Cached    | Percent |
|--------------------------------------------------------------+----------------+------------+-----------+---------|
| /home/eric/IdeaProjects/memmap/src/main/resources/bigFile.cc | 1073741824     | 262144     | 35136     | 013.403 |
|--------------------------------------------------------------+----------------+------------+-----------+---------|


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

pcstat PageCache: 35136 * 4 = 140544
pmap Rss PageCache: 140544


作者：in nek
链接：https://www.zhihu.com/question/48161206/answer/110418693
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

你的脑子是晕的，被听到的信息搞晕了头脑，我帮你洗一次脑吧。1. 请放弃虚拟内存这个概念，那个是广告性的概念，在开发中没有意义。开发中只有虚拟空间的概念，进程看到的所有地址组成的空间，就是虚拟空间。虚拟空间是某个进程对分配给它的所有物理地址（已经分配的和将会分配的）的重新映射。2. mmap的作用，在应用这一层，是让你把文件的某一段，当作内存一样来访问。内核和驱动如何实现的，性能高不高这些问题，这层语义上没有承诺。你基于功能决定怎么用它就好了，少胡思乱想。有了以上两个，你就可以写好程序了。下面介绍一下Linux的实现细节，权当好玩，如果你搞不清楚前面两条，后面的就不要看，否则你又乱掉了。3. mmap的工作原理，当你发起这个调用的时候，它只是在你的虚拟空间中分配了一段空间，连真实的物理地址都不会分配的，当你访问这段空间，CPU陷入OS内核执行异常处理，然后异常处理会在这个时间分配物理内存，并用文件的内容填充这片内存，然后才返回你进程的上下文，这时你的程序才会感知到这片内存里有数据4. 驱动每次读入多少页面，页面的分配算法等等，都不是系统的承诺，不能作为你编程的依赖。这就是前面说的：不要胡思乱想5. 至于swap分区的作用，参考这里：Linux 是怎样使用内存的？ - in nek 的回答基于题主继续的问题，我们接着来解释一下为什么我建议你放弃虚拟内存而使用虚拟空间的概念。
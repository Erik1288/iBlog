---
title: JVM-Crash-Log-Analysis
date: 2018-01-11 14:39:37
tags: JVM
---

http://www.javaranger.com/archives/1636

工具：
https://github.com/xpbob/CrashAnalysis

http://www.raychase.net/1459

https://www.jianshu.com/p/7652f931cafd

致命错误出现的时候，JVM生成了hs_err_pid<pid>.log这样的文件，其中往往包含了虚拟机崩溃原因的重要信息。因为经常遇到，在这篇文章里，我挑选了一个，并且逐段分析它包含的内容（文件可以在文章最后下载）。默认情况下文件是创建在工作目录下的（如果没权限创建的话JVM会尝试把文件写到/tmp这样的临时目录下面去），当然，文件格式和路径也可以通过参数指定，比如：

1
java -XX:ErrorFile=/var/log/java/java_error%p.log
这个文件将包括：

触发致命错误的操作异常或者信号；
版本和配置信息；
触发致命异常的线程详细信息和线程栈；
当前运行的线程列表和它们的状态；
堆的总括信息；
加载的本地库；
命令行参数；
环境变量；
操作系统CPU的详细信息。
首先，看到的是对问题的概要介绍：

1
#  SIGSEGV (0xb) at pc=0x03568cf4, pid=16819, tid=3073346448
一个非预期的错误被JRE检测到，其中：

SIGSEGV是信号名称
0xb是信号码
pc=0x03568cf4指的是程序计数器的值
pid=16819是进程号
tid=3073346448是线程号
如果你对JVM有了解，应该不会对这些东西陌生。


```
public class OutOfMemory {
    public static void main(String[] args) {
        ArrayList<StubClass> list = new ArrayList<StubClass>();
        int count = 1000000000;
        for (int i = 0; i < count; i++) {
            StubClass bj = new StubClass();
            list.add(obj);
        }
    }
}
```


```
class StubClass {
    private Integer n = new Integer(123334);
    private Integer m = new Integer(123334);
    
    public void print() {
        System.out.println("m="+m);
        System.out.println("n="+n);
    }
}
```

---
title: JVM多态
date: 2017-11-01 15:26:24
tags: JVM
---


http://blog.csdn.net/huangrunqing/article/details/51996424

``` java
package com.eric;

public class Computer {
    public void program() {
        System.out.println("program");
    }
}

```
``` 
// javap -v Computer
public class com.eric.Computer
  minor version: 0
  major version: 49
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#17         // java/lang/Object."<init>":()V
   #2 = Fieldref           #18.#19        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #14            // program
   #4 = Methodref          #20.#21        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #22            // com/eric/Computer
   #6 = Class              #23            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lcom/eric/Computer;
  #14 = Utf8               program
  #15 = Utf8               SourceFile
  #16 = Utf8               Computer.java
  #17 = NameAndType        #7:#8          // "<init>":()V
  #18 = Class              #24            // java/lang/System
  #19 = NameAndType        #25:#26        // out:Ljava/io/PrintStream;
  #20 = Class              #27            // java/io/PrintStream
  #21 = NameAndType        #28:#29        // println:(Ljava/lang/String;)V
  #22 = Utf8               com/eric/Computer
  #23 = Utf8               java/lang/Object
  #24 = Utf8               java/lang/System
  #25 = Utf8               out
  #26 = Utf8               Ljava/io/PrintStream;
  #27 = Utf8               java/io/PrintStream
  #28 = Utf8               println
  #29 = Utf8               (Ljava/lang/String;)V
{
  public com.eric.Computer();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/eric/Computer;

  public void program();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String program
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 9: 0
        line 10: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/eric/Computer;
}

```

package com.eric;

``` java
public class Mac extends Computer {
    @Override
    public void program() {
        System.out.println("mac program");
    }

    public void showOff() {
        System.out.println("show off");
    }
}


public class com.eric.Mac extends com.eric.Computer
  minor version: 0
  major version: 49
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #7.#19         // com/eric/Computer."<init>":()V
   #2 = Fieldref           #20.#21        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #22            // mac program
   #4 = Methodref          #23.#24        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = String             #25            // show off
   #6 = Class              #26            // com/eric/Mac
   #7 = Class              #27            // com/eric/Computer
   #8 = Utf8               <init>
   #9 = Utf8               ()V
  #10 = Utf8               Code
  #11 = Utf8               LineNumberTable
  #12 = Utf8               LocalVariableTable
  #13 = Utf8               this
  #14 = Utf8               Lcom/eric/Mac;
  #15 = Utf8               program
  #16 = Utf8               showOff
  #17 = Utf8               SourceFile
  #18 = Utf8               Mac.java
  #19 = NameAndType        #8:#9          // "<init>":()V
  #20 = Class              #28            // java/lang/System
  #21 = NameAndType        #29:#30        // out:Ljava/io/PrintStream;
  #22 = Utf8               mac program
  #23 = Class              #31            // java/io/PrintStream
  #24 = NameAndType        #32:#33        // println:(Ljava/lang/String;)V
  #25 = Utf8               show off
  #26 = Utf8               com/eric/Mac
  #27 = Utf8               com/eric/Computer
  #28 = Utf8               java/lang/System
  #29 = Utf8               out
  #30 = Utf8               Ljava/io/PrintStream;
  #31 = Utf8               java/io/PrintStream
  #32 = Utf8               println
  #33 = Utf8               (Ljava/lang/String;)V
{
  public com.eric.Mac();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method com/eric/Computer."<init>":()V
         4: return
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/eric/Mac;

  public void program();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String mac program
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 10: 0
        line 11: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/eric/Mac;

  public void showOff();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #5                  // String show off
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 14: 0
        line 15: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/eric/Mac;
}
```

``` java
package com.eric;

public class PC extends Computer {
    @Override
    public void program() {
        System.out.println("pc program");
    }

    public void playGame() {
        System.out.println("play game");
    }
}

public class com.eric.PC extends com.eric.Computer
  minor version: 0
  major version: 49
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #7.#19         // com/eric/Computer."<init>":()V
   #2 = Fieldref           #20.#21        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #22            // pc program
   #4 = Methodref          #23.#24        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = String             #25            // play game
   #6 = Class              #26            // com/eric/PC
   #7 = Class              #27            // com/eric/Computer
   #8 = Utf8               <init>
   #9 = Utf8               ()V
  #10 = Utf8               Code
  #11 = Utf8               LineNumberTable
  #12 = Utf8               LocalVariableTable
  #13 = Utf8               this
  #14 = Utf8               Lcom/eric/PC;
  #15 = Utf8               program
  #16 = Utf8               playGame
  #17 = Utf8               SourceFile
  #18 = Utf8               PC.java
  #19 = NameAndType        #8:#9          // "<init>":()V
  #20 = Class              #28            // java/lang/System
  #21 = NameAndType        #29:#30        // out:Ljava/io/PrintStream;
  #22 = Utf8               pc program
  #23 = Class              #31            // java/io/PrintStream
  #24 = NameAndType        #32:#33        // println:(Ljava/lang/String;)V
  #25 = Utf8               play game
  #26 = Utf8               com/eric/PC
  #27 = Utf8               com/eric/Computer
  #28 = Utf8               java/lang/System
  #29 = Utf8               out
  #30 = Utf8               Ljava/io/PrintStream;
  #31 = Utf8               java/io/PrintStream
  #32 = Utf8               println
  #33 = Utf8               (Ljava/lang/String;)V
{
  public com.eric.PC();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method com/eric/Computer."<init>":()V
         4: return
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/eric/PC;

  public void program();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String pc program
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 10: 0
        line 11: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/eric/PC;

  public void playGame();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #5                  // String play game
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 14: 0
        line 15: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/eric/PC;
}
```

``` java
package com.eric;

public class Main {
    public static void main(String[] args) {
        Computer computer = new Mac();
        computer.program();

        computer = new PC();
        computer.program();

        Mac mac = new Mac();
        mac.showOff();
    }
}

public class com.eric.Main
  minor version: 0
  major version: 49
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #10.#26        // java/lang/Object."<init>":()V
   #2 = Class              #27            // com/eric/Computer
   #3 = Methodref          #2.#26         // com/eric/Computer."<init>":()V
   #4 = Methodref          #2.#28         // com/eric/Computer.program:()V
   #5 = Class              #29            // com/eric/PC
   #6 = Methodref          #5.#26         // com/eric/PC."<init>":()V
   #7 = Class              #30            // com/eric/Mac
   #8 = Methodref          #7.#26         // com/eric/Mac."<init>":()V
   #9 = Class              #31            // com/eric/Main
  #10 = Class              #32            // java/lang/Object
  #11 = Utf8               <init>
  #12 = Utf8               ()V
  #13 = Utf8               Code
  #14 = Utf8               LineNumberTable
  #15 = Utf8               LocalVariableTable
  #16 = Utf8               this
  #17 = Utf8               Lcom/eric/Main;
  #18 = Utf8               main
  #19 = Utf8               ([Ljava/lang/String;)V
  #20 = Utf8               args
  #21 = Utf8               [Ljava/lang/String;
  #22 = Utf8               computer
  #23 = Utf8               Lcom/eric/Computer;
  #24 = Utf8               SourceFile
  #25 = Utf8               Main.java
  #26 = NameAndType        #11:#12        // "<init>":()V
  #27 = Utf8               com/eric/Computer
  #28 = NameAndType        #33:#12        // program:()V
  #29 = Utf8               com/eric/PC
  #30 = Utf8               com/eric/Mac
  #31 = Utf8               com/eric/Main
  #32 = Utf8               java/lang/Object
  #33 = Utf8               program
{
  public com.eric.Main();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/eric/Main;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #2                  // class com/eric/Computer
         3: dup
         4: invokespecial #3                  // Method com/eric/Computer."<init>":()V
         7: astore_1
         8: aload_1
         9: invokevirtual #4                  // Method com/eric/Computer.program:()V
        12: new           #5                  // class com/eric/PC
        15: dup
        16: invokespecial #6                  // Method com/eric/PC."<init>":()V
        19: astore_1
        20: aload_1
        21: invokevirtual #4                  // Method com/eric/Computer.program:()V
        24: new           #7                  // class com/eric/Mac
        27: dup
        28: invokespecial #8                  // Method com/eric/Mac."<init>":()V
        31: astore_1
        32: aload_1
        33: invokevirtual #4                  // Method com/eric/Computer.program:()V
        36: return
      LineNumberTable:
        line 9: 0
        line 10: 8
        line 12: 12
        line 13: 20
        line 15: 24
        line 16: 32
        line 17: 36
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      37     0  args   [Ljava/lang/String;
            8      29     1 computer   Lcom/eric/Computer;
}
SourceFile: "Main.java"
```

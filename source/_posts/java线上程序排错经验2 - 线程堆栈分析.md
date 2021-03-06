---
title: java线上程序排错经验2 - 线程堆栈分析
date: 2018-09-04 21:25:21
tags: [jvm,java]
categories: [java]
author: kaishun
id: 207
permalink: jvm02
---

## 1.前言  
在线上的程序中，我们可能经常会碰到程序卡死或者执行很慢的情况，这时候我们希望知道是代码哪里的问题，我们或许迫切希望得到代码运行到哪里了，是哪一步很慢，是否是进入了死循环，或者是否哪一段代码有问题导致程序很慢，或者出现了线程不安全的情况，或者是某些连接数或者打开文件数太多等问题，总之我们想知道程序卡在哪里了，哪块占用了大量的资源。  
此时，或许通过线程堆栈的分析就能定位出问题。  
**如果能深入掌握堆栈分析的技术，很多问题都能迎刃而解，但是线程堆栈分析并不简单**，设计到线上的排错问题，需要有一定的知识的广度，对某些知识也要有一定的深度，本人不才，无论是广度还是深度，都感觉略有欠缺，工作两年多来，碰到了很多问题，在解决问题时，并不是一帆风顺的，其中可没少折腾，但我相信，正确的理论可以指导实践，本文也算是现学现卖。

注： 本文主要针对于Linux的，不过大多数在windows上也适用

## 2. 什么是线程堆栈
**线程栈**： 
Java线程堆栈是某个时间对所有线程的一个快照，其中主要记录了如下信息  
– 线程的名称    
- 线程的ID   
- 原生线程ID  
- 线程的运行状态  
- 锁的状态  
- 调用堆栈,也就是每个线程在各个方法调用的栈  
上面描述的信息，接下来我们会具体的介绍与分析  

## 3. 如何分析

首先我们得知道如何去得到线程堆栈  
工具有很多，我这里只介绍最原始的
先得到当前运行java程序的PID  
这时候就要用到jps命令  
jps常用的几个命令  
```
jps #显示所有java程序和线程ID  
jps -m #输出main method的参数 
jps -l #输出完全的包名，应用主类名，jar的完全路径名 
jps -v #输出jvm参数
```
一般直接使用jps即可，如果分不清哪个pid是哪个程序，可以直接  
```
jps -mlv
```
下面举个简单的线程堆栈分析的例子  
```
package me.zks.jvm.troubleshoot.threadhump;
public class FirstSample {
    public static void main(String[] args) {
        FirstSample firstSample = new FirstSample();
        firstSample.fun1();
    }
    public void fun1(){
        //执行某些不耗时操作，然后直接进入fun2方法
        fun2();
    }
    public void fun2(){
        //执行某些不耗时的操作，然后直接进入fun3方法
        fun3();
    }
    public void fun3(){
        //一个死循环，或者耗时特别大的操作
        while (true){
            System.out.println("");
        }
    }
}
```
上面程序打成jar包，名为jvm-troubleshoot-1.0-SNAPSHOT.jar，在linux上运行  
```
java -cp jvm-troubleshoot-1.0-SNAPSHOT.jar me.zks.jvm.troubleshoot.threadhump.FirstSample
```
另外开一个session  
输入jps显示如下  
```
ubuntu@VM-0-12-ubuntu:~$ jps
27831 Jps
27819 FirstSample
ubuntu@VM-0-12-ubuntu:~$
``` 
输入jps -mlv显示如下  
```
ubuntu@VM-0-12-ubuntu:~$ jps -mlv
27874 sun.tools.jps.Jps -mlv -Dapplication.home=/usr/lib/jvm/java-8-oracle -Xms8m
27819 me.zks.jvm.troubleshoot.threadhump.FirstSample
ubuntu@VM-0-12-ubuntu:~$
```

jstack 27819显示如下  
```
ubuntu@VM-0-12-ubuntu:~$ jstack 27819
2018-09-02 13:56:14
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.171-b11 mixed mode):

"Attach Listener" #8 daemon prio=9 os_prio=0 tid=0x00007fd350001000 nid=0x6d27 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #7 daemon prio=9 os_prio=0 tid=0x00007fd3700c6800 nid=0x6cb3 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread1" #6 daemon prio=9 os_prio=0 tid=0x00007fd3700b1800 nid=0x6cb2 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007fd3700af800 nid=0x6cb1 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0x00007fd3700ae000 nid=0x6cb0 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=0 tid=0x00007fd37007b000 nid=0x6caf in Object.wait() [0x00007fd35befd000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000ec6b2c98> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
        - locked <0x00000000ec6b2c98> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:212)

"Reference Handler" #2 daemon prio=10 os_prio=0 tid=0x00007fd370076800 nid=0x6cae in Object.wait() [0x00007fd35bffe000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000ec6b2e50> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x00000000ec6b2e50> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"main" #1 prio=5 os_prio=0 tid=0x00007fd37000a800 nid=0x6cac runnable [0x00007fd377121000]
   java.lang.Thread.State: RUNNABLE
        at java.io.FileOutputStream.writeBytes(Native Method)
        at java.io.FileOutputStream.write(FileOutputStream.java:326)
        at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
        at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
        - locked <0x00000000ec6bfe58> (a java.io.BufferedOutputStream)
        at java.io.PrintStream.write(PrintStream.java:482)
        - locked <0x00000000ec6b6ee0> (a java.io.PrintStream)
        at sun.nio.cs.StreamEncoder.writeBytes(StreamEncoder.java:221)
        at sun.nio.cs.StreamEncoder.implFlushBuffer(StreamEncoder.java:291)
        at sun.nio.cs.StreamEncoder.flushBuffer(StreamEncoder.java:104)
        - locked <0x00000000ec6b6e98> (a java.io.OutputStreamWriter)
        at java.io.OutputStreamWriter.flushBuffer(OutputStreamWriter.java:185)
        at java.io.PrintStream.newLine(PrintStream.java:546)
        - eliminated <0x00000000ec6b6ee0> (a java.io.PrintStream)
        at java.io.PrintStream.println(PrintStream.java:807)
        - locked <0x00000000ec6b6ee0> (a java.io.PrintStream)
        at me.zks.jvm.troubleshoot.threadhump.FirstSample.fun3(FirstSample.java:22)
        at me.zks.jvm.troubleshoot.threadhump.FirstSample.fun2(FirstSample.java:17)
        at me.zks.jvm.troubleshoot.threadhump.FirstSample.fun1(FirstSample.java:13)
        at me.zks.jvm.troubleshoot.threadhump.FirstSample.main(FirstSample.java:9)

"VM Thread" os_prio=0 tid=0x00007fd37006f000 nid=0x6cad runnable

"VM Periodic Task Thread" os_prio=0 tid=0x00007fd3700cc000 nid=0x6cb4 waiting on condition

JNI global references: 5

```
通过上面的jstack我们可以指导，我们的这个简单的程序，竟然有这么多线程，分别是
```
"Attach Listener","Service Thread","C1 CompilerThread1","C2 CompilerThread0","Signal Dispatcher","Finalizer","Reference Handler","main","VM Thread","VM Periodic Task Thread",
```
其中，只有main线程是我们写的，其他的线程是jvm启动的时候，自己会创建一些辅助的线程，我们一般分析我们所写的线程（也不一定，有时候也是需要分析其他线程的），我们来仔细分析一下我们得这个main线程,我给每一行标了个号  
```
01 "main" #1 prio=5 os_prio=0 tid=0x00007fd37000a800 nid=0x6cac runnable [0x00007fd377121000]
02   java.lang.Thread.State: RUNNABLE
03        at java.io.FileOutputStream.writeBytes(Native Method)
04        at java.io.FileOutputStream.write(FileOutputStream.java:326)
05        at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
06        at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
07        - locked <0x00000000ec6bfe58> (a java.io.BufferedOutputStream)
08        at java.io.PrintStream.write(PrintStream.java:482)
09        - locked <0x00000000ec6b6ee0> (a java.io.PrintStream)
10        at sun.nio.cs.StreamEncoder.writeBytes(StreamEncoder.java:221)
11        at sun.nio.cs.StreamEncoder.implFlushBuffer(StreamEncoder.java:291)
12        at sun.nio.cs.StreamEncoder.flushBuffer(StreamEncoder.java:104)
13        - locked <0x00000000ec6b6e98> (a java.io.OutputStreamWriter)
14        at java.io.OutputStreamWriter.flushBuffer(OutputStreamWriter.java:185)
15        at java.io.PrintStream.newLine(PrintStream.java:546)
16        - eliminated <0x00000000ec6b6ee0> (a java.io.PrintStream)
17        at java.io.PrintStream.println(PrintStream.java:807)
18        - locked <0x00000000ec6b6ee0> (a java.io.PrintStream)
19        at me.zks.jvm.troubleshoot.threadhump.FirstSample.fun3(FirstSample.java:22)
20        at me.zks.jvm.troubleshoot.threadhump.FirstSample.fun2(FirstSample.java:17)
21        at me.zks.jvm.troubleshoot.threadhump.FirstSample.fun1(FirstSample.java:13)
22        at me.zks.jvm.troubleshoot.threadhump.FirstSample.main(FirstSample.java:9)
```
先分析第一行  
```
"main" #1 prio=5 os_prio=0 tid=0x00007fd37000a800 nid=0x6cac runnable [0x00007fd377121000]
```
- "main" 即为线程的名称，所以给线程去个好的名字很重要，方便于我们得分析
- #1 我也不知道  
- prio=5 线程优先级，java 有1--10个优先级，当有多个线程处于可运行状态时，运行系统总是挑选一个优先级最高的线程执行，两个优先级相同的线程同时等待执行时，那么运行系统会以round-robin的方式选择一个线程执行，Java的优先级策略是抢占式调度。
- tid=0x00007fd37000a800 这个是java虚拟机内部的ID，与什么操作系统没关系，我们可以通过 java.lang.Thread.getId() 或者 java.lang.management.ThreadMXBean.getAllThreadIds()获取到,在Oracle / Sun的JDK实现中，此ID只是一个自动递增的long类型的变量。  
- nid=0x6cac linux或者windows等操作系统的线程ID，jvm作为一个虚拟机，里面的所有线程ID，其实都是映射到linux上面，这个ID就是linux机器实际运行的线程ID，这个很有用，我们可以通过这个来linux上查一些比较重要的信息，这个以后会有详细说明。    
- runnable  线程的状态，线程的状态，这是从java虚拟机的角度来看的，表示线程正在运行，**但是处于Runnable状态的线程不一定真地消耗CPU**，这个以后会说，线程状态以后也会详细说明。  
分析第二行  
java.lang.Thread.State: RUNNABLE， 描述了线程的状态  
分析第3-22行。  
是方法的调用站的关系，我们一般从下往上看  
例如先看第22行  
```
at me.zks.jvm.troubleshoot.threadhump.FirstSample.main(FirstSample.java:9)
```
这行显示了正在调用的包名，类名，方法名称，以及代码中的第几行  
从上可知，main方法调用了fun1,fun1调用了fun2,fun2调用了fun3,fun3调用了java.io.PrintStream.println .....  

使用top命令，看看是哪个线程占用的cpu最多,一般使用top -Hp 27819
top -Hp 27819  
[top-hp pid](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/jvm-thread-dump/top-hp-pid.png)!

## 线程锁分析  
代码  

jps  
```
ubuntu@VM-0-12-ubuntu:~$ jps
21786 LockAnalysics
21804 Jps

```
jstack  
```

ubuntu@VM-0-12-ubuntu:~$ jstack 21786
2018-09-02 22:06:31
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.171-b11 mixed mode):

"Attach Listener" #12 daemon prio=9 os_prio=0 tid=0x00007f0bac001000 nid=0x5563 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"DestroyJavaVM" #11 prio=5 os_prio=0 tid=0x00007f0bd000a800 nid=0x551b waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"ThreadWaitTo" #10 prio=5 os_prio=0 tid=0x00007f0bd00ef800 nid=0x5526 waiting for monitor entry [0x00007f0bd4d17000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at me.zks.jvm.troubleshoot.threadhump.LockAnalysics$ThreadWaitTo.funWaitTo(LockAnalysics.java:76)
        - waiting to lock <0x00000000e2a78dc0> (a java.lang.Object)
        at me.zks.jvm.troubleshoot.threadhump.LockAnalysics$ThreadWaitTo.run(LockAnalysics.java:70)
        at java.lang.Thread.run(Thread.java:748)

"ThreadWaitOn" #9 prio=5 os_prio=0 tid=0x00007f0bd00ee000 nid=0x5525 in Object.wait() [0x00007f0bd4e18000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000e2a7eda8> (a java.lang.Object)
        at java.lang.Object.wait(Object.java:502)
        at me.zks.jvm.troubleshoot.threadhump.LockAnalysics$ThreadWaitOn.funWaitOn(LockAnalysics.java:59)
        - locked <0x00000000e2a7eda8> (a java.lang.Object)
        at me.zks.jvm.troubleshoot.threadhump.LockAnalysics$ThreadWaitOn.run(LockAnalysics.java:52)
        at java.lang.Thread.run(Thread.java:748)

"ThreadLocked" #8 prio=5 os_prio=0 tid=0x00007f0bd00e5800 nid=0x5524 waiting on condition [0x00007f0bd4f19000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at me.zks.jvm.troubleshoot.threadhump.LockAnalysics$ThreadLocked.fun1(LockAnalysics.java:40)
        at me.zks.jvm.troubleshoot.threadhump.LockAnalysics$ThreadLocked.run(LockAnalysics.java:34)
        - locked <0x00000000e2a78dc0> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:748)

"Service Thread" #7 daemon prio=9 os_prio=0 tid=0x00007f0bd00be800 nid=0x5522 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread1" #6 daemon prio=9 os_prio=0 tid=0x00007f0bd00b1800 nid=0x5521 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007f0bd00af800 nid=0x5520 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0x00007f0bd00ae000 nid=0x551f runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=0 tid=0x00007f0bd007b000 nid=0x551e in Object.wait() [0x00007f0bd56ff000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000e2a08ed0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
        - locked <0x00000000e2a08ed0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:212)

"Reference Handler" #2 daemon prio=10 os_prio=0 tid=0x00007f0bd0076800 nid=0x551d in Object.wait() [0x00007f0bd5800000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000e2a06bf8> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x00000000e2a06bf8> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"VM Thread" os_prio=0 tid=0x00007f0bd006f000 nid=0x551c runnable

"VM Periodic Task Thread" os_prio=0 tid=0x00007f0bd00c4000 nid=0x5523 waiting on condition

JNI global references: 5

```
先看一下ThreadLocked线程, 线程状态处于TIMED_WAITING(sleep)，锁特征为waiting on condition。由这个堆栈可知该线程抢占了锁0x00000000e2a78dc0  
```
"ThreadLocked" #8 prio=5 os_prio=0 tid=0x00007f0bd00e5800 nid=0x5524 waiting on condition [0x00007f0bd4f19000] //!! 锁特征为waiting on condition的
   java.lang.Thread.State: TIMED_WAITING (sleeping)  //线程状态是TIMED_WAITING(sleeping)
        at java.lang.Thread.sleep(Native Method)
        at me.zks.jvm.troubleshoot.threadhump.LockAnalysics$ThreadLocked.fun1(LockAnalysics.java:40)
        at me.zks.jvm.troubleshoot.threadhump.LockAnalysics$ThreadLocked.run(LockAnalysics.java:34)
        - locked <0x00000000e2a78dc0> (a java.lang.Object)  // 该线程占有了锁0x00000000e2a78dc0, 进入了同步代码块的方法
        at java.lang.Thread.run(Thread.java:748)
```
再看一下ThreadWaitTo线程,可以发现线程状态为BLOCKED，锁特征为waiting for monitor entry, waiting to lock <0x00000000e2a78dc0>说明锁0x00000000e2a78dc0被其他线程占用了，本线程正在等待抢占这把锁。 
```
"ThreadWaitTo" #10 prio=5 os_prio=0 tid=0x00007f0bd00ef800 nid=0x5526 waiting for monitor entry [0x00007f0bd4d17000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at me.zks.jvm.troubleshoot.threadhump.LockAnalysics$ThreadWaitTo.funWaitTo(LockAnalysics.java:76)
        - waiting to lock <0x00000000e2a78dc0> (a java.lang.Object)
        at me.zks.jvm.troubleshoot.threadhump.LockAnalysics$ThreadWaitTo.run(LockAnalysics.java:70)
        at java.lang.Thread.run(Thread.java:748)
```
看一下ThreadWaitOn线程, 线程处于WAITING状态，锁特征为Object.wait()，其中waiting on <0x00000000e2a7eda8>会释放自己锁定的锁0x00000000e2a7eda8，其他线程可以占有这把锁，并且自己处于等待唤醒的状态  
```
"ThreadWaitOn" #9 prio=5 os_prio=0 tid=0x00007f0bd00ee000 nid=0x5525 in Object.wait() [0x00007f0bd4e18000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000e2a7eda8> (a java.lang.Object) //此时wait方法会导致该锁被释放,其它线程又可以占有该锁
        at java.lang.Object.wait(Object.java:502)
        at me.zks.jvm.troubleshoot.threadhump.LockAnalysics$ThreadWaitOn.funWaitOn(LockAnalysics.java:59)
        - locked <0x00000000e2a7eda8> (a java.lang.Object)
        at me.zks.jvm.troubleshoot.threadhump.LockAnalysics$ThreadWaitOn.run(LockAnalysics.java:52)
        at java.lang.Thread.run(Thread.java:748)
```
通过上面这个例子，我们看到了有locked,waiting to lock,waiting on这三个锁特征  
- locked表示已经占有了锁  
- waiting to lock表示这把锁目前还没抢到（可能别别的线程抢占了），正在等待这把锁
- waiting on 表示线程处于Object.lock()状态，已经释放了锁，其他线程可以占用这把锁，同事本线程等待被唤醒.  

### 3. 线程状态的解读  
从上面的例子中，我们看到了线程状态有BLOCKED,WAITING,TIMED_WAITING。 实际上线程状态有如下几种  
![线程状态](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/jvm-thread-dump/thread-states.png)  
1. 新建状态（New）：新创建了一个线程对象。
2. 就绪状态（Runnable）：线程对象创建后，其他线程调用了该对象的start()方法。该状态的线程位于可运行线程池中，变得可运行，等待获取CPU的使用权。
3. 运行状态（Running）：就绪状态的线程获取了CPU，执行程序代码。
4. 阻塞状态（Blocked）：阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：
（一）、WAITING (on object monitor) 等待阻塞：运行的线程执行wait()方法，JVM会把该线程放入等待池中。
（二）、BLOCKED (on object monitor) 同步阻塞：运行的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池中。区分同步阻塞和等待阻塞也可以看锁的特征，例如同步阻塞锁的特征是waiting for monitor, 等待阻塞锁的特征是object.wait()
（三）、TIMED_WAITING(sleeping) 其他阻塞：运行的线程执行sleep()或join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。  
5. 死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期。

**实际上我们jvm线程栈中，几乎是不会出现NEW,RUNNING,DEAD 这些状态，其中Runnable就算是正在运行了**
**处于WAITING, BLOCKED, TIME_WAITING的是不消耗CPU的，处于Runnable状态的线程，是否消耗cpu要看具体的上下文情况**  
- 如果是纯Java运算代码， 则消耗CPU.  
- 如果是网络IO,很少消耗CPU,这点在分布式程序中经常碰到  
- 如果是本地代码， 结合本地代码的性质判断(可以通过pstack/gstack获取本地线程堆栈)，
如果是纯运算代码， 则消耗CPU, 如果被挂起， 则不消耗CPU,如果是IO,则不怎么消
耗CPU

### 4. 线程死锁  
![死锁](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/jvm-thread-dump/%E6%AD%BB%E9%94%81.jpg)  
死锁比较少见，而且难于调试：
所谓死锁： 是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。
其实很久之前学习数字电路，经常会遇到一些锁，这也是自动化的一些常见的问题，在计算机中，也有类似的东西，请看下图  
R1 和R2，都只能被一个进程使用
T1在使用R1，同时没有使用完R1的情况下，想使用R2
T2在使用R2，同时在没有使用完R2的情况下，想使用R1
这时，T1等待T2放弃使用R2，同时T2等待T1放弃使用R1，他们都不会放弃自己所使用的，于是产生了等待，将会一直僵持下去。  
代码如下  
```
package me.zks.jvm.troubleshoot.threadhump;

public class DeadLock implements Runnable{
    private int flag = 1;
    // 两个static的对象，静态变量
    public static Object obj1=new Object();
    public static Object obj2=new Object();
    @Override
    public void run() {
        System.out.println("flag="+flag);
        if(flag==1){
            fun1();
        }
        if(flag==0){
            fun2();
        }
    }

    private void fun1() {
        synchronized (obj1){
            System.out.println("我已经锁定了obj1,休息0.5秒后再锁定obj2,但是估计进不了obj2，因为obj2估计也被锁定了");
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (obj2){
                System.out.println("进入了obj2");
            }
        }
    }
    private void fun2() {
        synchronized (obj2){
            System.out.println("我已经锁定了obj2,休息0.5秒后再锁定obj1,但是估计进不了obj1，因为obj1估计也被锁定了");
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (obj1){
                System.out.println("进入了obj1");
            }
        }
    }

    public static void main(String[] args) {
        DeadLock deadLock1 = new DeadLock();
        DeadLock deadLock2 = new DeadLock();
        deadLock1.flag=1;
        deadLock2.flag=0;
        System.out.println("线程开始了");
        new Thread(deadLock1).start();
        new Thread(deadLock2).start();

    }
}
```
运行后显示  
```
线程开始了
flag=1
我已经锁定了obj1,休息0.5秒后再锁定obj2,但是估计进不了obj2，因为obj2估计也被锁定了
flag=0
我已经锁定了obj2,休息0.5秒后再锁定obj1,但是估计进不了obj1，因为obj1估计也被锁定了
```
jstack显示(删除了一些信息，只留了用户线程的)  
```
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007f37680062c8 (object 0x00000000e2a794f8, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x00007f3768004e28 (object 0x00000000e2a79508, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
        at me.zks.jvm.troubleshoot.threadhump.DeadLock.fun2(DeadLock.java:41)
        - waiting to lock <0x00000000e2a794f8> (a java.lang.Object)
        - locked <0x00000000e2a79508> (a java.lang.Object)
        at me.zks.jvm.troubleshoot.threadhump.DeadLock.run(DeadLock.java:15)
        at java.lang.Thread.run(Thread.java:748)
"Thread-0":
        at me.zks.jvm.troubleshoot.threadhump.DeadLock.fun1(DeadLock.java:28)
        - waiting to lock <0x00000000e2a79508> (a java.lang.Object)
        - locked <0x00000000e2a794f8> (a java.lang.Object)
        at me.zks.jvm.troubleshoot.threadhump.DeadLock.run(DeadLock.java:12)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```
从线程中我们可以看到Found one Java-level deadlock:  
这就说明进入了死锁，分析Thread-1和Thread-2  
Thread-1锁定了0x00000000e2a79508, waiting to lock 0x00000000e2a794f8
Thread-0锁定了0x00000000e2a794f8, waiting to lock 0x00000000e2a79508，于是这两个线程就进入了死锁。  
如果通过这种方式发现了死锁，那没办法，只有该代码了，将并发做的更安全点才是王道。  

## 5. 大数据中死循环或者复杂度高的方法判断  
首先，如果是进入了死循环，程序肯定是在某个方法直接卡死，我们将线程堆栈分析下来，多分析几次，比如20秒一次（具体情况而定），如果多次都在同一个方法栈的调用，但是根据我们得预估，这个代码并不需要这么多的时间，并且这个方法的线程占用的cpu很高(前面提到的TOP -Hp pid, 然后根据这个方法的nid 转成10进制就可以找到对应的线程)，那么我们就怀疑是这个cpu高引起的问题了。  
这时候就需要分析代码了。  
**需要注意**的是,我们分析还是要根据代码客观的时间来分析，特别是在spark等大数据处理中，有的方法是需要很久的时间，合理的判断，找出死循环或者复杂度高的方法，然后再对代码进行修改。  

## 6. IO或者网络问题  
有时候，大数据在shuffle过程中，或者web程序在传输数据过程中，可能会由于网络等问题，导致程序会很慢，这个分析方法也同样是通过看线程堆栈，查看到是java.io.，或者java.nio等问题，那就可能就是网络问题了，网络问题可以参考使用iotop 和iostat 来进行分析  

## 7. 连接数瓶颈问题  
有时候，linux机器的频繁连接，同时连接数过多，或者IO的频繁打开而不关闭，连接数过多，又或者是socketRead的并发太多，就容易发生连接数瓶颈问题。  
最比较常见的就是在web做压力测试（例如ab test），再并发数一多的时候，就容易发现瓶颈，像mysql有类似于MYTOP的工具，同样，我们也可以通过jstack获取方法栈来进行，例如下面这个例子  
代码参考自《java 问题定位技术》  
```
"Thread-248" prio=1 tid=0xa58f2048 nid=0x7ac2 runnable
[0xaeedb000..0xaeedc480]
at java.net.SocketInputStream.socketRead0(Native Method)
at java.net.SocketInputStream.read(SocketInputStream.java:129)
at oracle.net.ns.Packet.receive(Unknown Source)
... ...
at oracle.jdbc.driver.LongRawAccessor.getBytes()
"Thread-429" prio=1 tid=0xa58f2048 nid=0x7ac2 runnable
[0xaeedb000..0xaeedc480]
at java.net.SocketInputStream.socketRead0(Native Method)
at java.net.SocketInputStream.read(SocketInputStream.java:129)
at oracle.net.ns.Packet.receive(Unknown Source)
... ...
at oracle.jdbc.driver.LongRawAccessor.getBytes()
"Thread-250" prio=1 tid=0xa58f2048 nid=0x7ac2 runnable
[0xaeedb000..0xaeedc480]
at java.net.SocketInputStream.socketRead0(Native Method)
at java.net.SocketInputStream.read(SocketInputStream.java:129)
at oracle.net.ns.Packet.receive(Unknown Source)
... ...
at oracle.jdbc.driver.LongRawAccessor.getBytes()
.....
```
假如有太多的这类线程，那么就可以得出是jdbc访问过多，这时候就要优化资源和优化程序了。   

## 7. 频繁GC导致的cpu速度慢的问题  
下回讲解， 内存堆栈分析。  


参考文献：  
[HotSpotVM故障排除指南](https://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/preface.html)  




















---
title: java线上程序排错经验3 - jvm内存分析
date: 2018-09-04 21:25:21
tags: [jvm,java]
categories: [java]
author: kaishun
id: 207
permalink: jvm03
---

## 前言  
堆分析工具很多，这里只介绍一种分析的方法，也是最原始的一种，以后会在这篇文字里面慢慢补充
<!-- more -->
### 补充： 最简单的方法，比较直观  
```
jmap -histo：live <PID>
```

## 1. 先得到堆  
### 1.1 jmap得到堆
直接jmap查看使用方法  
```
ubuntu@VM-0-12-ubuntu:~$ jmap
Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system

```
上面的说法中有个例子，我也一般使用这个例子  
```
jmap -dump:live,format=b,file=heap.bin <pid>
```
上述命令会生成一个heap.bin文件，是个二进制的（当然也可以不输出二进制的，不过输出二进制的是为了jhat等堆分析工具的查看）  

## 1.2 堆溢出时自动堆转存  
线上的程序什么时候堆溢出很难捕捉，我们可以设置在堆内存时自动进行堆转存，这样方便我们进行查看，操作方法
```
-XX:+HeapDumpOnOutOfMemoryError
```

举个例子  
代码  
```
package me.zks.jvm.troubleshoot.heaphump;

import java.util.ArrayList;

public class FirstHeapSample {
    public static void main(String[] args) {
        ArrayList<Sample> samples = new ArrayList<>();
        while (true){
            for (int i = 0; i <1000 ; i++) {
                samples.add(new Sample(i,String.valueOf(i)));
            }
        }
    }
}
```
运行时VM options中添加-XX:+HeapDumpOnOutOfMemoryError
```
-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
```
运行后，不一会儿就报OOM错误  
```
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid3048.hprof ...
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:261)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:235)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:227)
	at java.util.ArrayList.add(ArrayList.java:458)
	at me.zks.jvm.troubleshoot.heaphump.FirstHeapSample.main(FirstHeapSample.java:13)
Heap dump file created [26835640 bytes in 0.188 secs]
```
并且程序生成了一个堆文件  
```
java_pid3048.hprof
```
分析jvm dump工具的有很多，比如eclipse或者idea的一些插件，jprofile,jvisualVM等。这里先不介绍  
，这里只介绍一种最原始的工具jhat **jhat貌似在jdk1.9后就没了或者不支持了**


jhat 先查看使用方法  
jhat -help  
```
ubuntu@VM-0-12-ubuntu:~$ jhat -help
Usage:  jhat [-stack <bool>] [-refs <bool>] [-port <port>] [-baseline <file>] [-debug <int>] [-version] [-h|-help] <file>

        -J<flag>          Pass <flag> directly to the runtime system. For
                          example, -J-mx512m to use a maximum heap size of 512MB
        -stack false:     Turn off tracking object allocation call stack.
        -refs false:      Turn off tracking of references to objects
        -port <port>:     Set the port for the HTTP server.  Defaults to 7000
        -exclude <file>:  Specify a file that lists data members that should
                          be excluded from the reachableFrom query.
        -baseline <file>: Specify a baseline object dump.  Objects in
                          both heap dumps with the same ID and same class will
                          be marked as not being "new".
        -debug <int>:     Set debug level.
                            0:  No debug output
                            1:  Debug hprof file parsing
                            2:  Debug hprof file parsing, no server
        -version          Report version number
        -h|-help          Print this help and exit
        <file>            The file to read

For a dump file that contains multiple heap dumps,
you may specify which dump in the file
by appending "#<number>" to the file name, i.e. "foo.hprof#3".

All boolean options default to "true"
```
一般这么多我也用不上，我一般只用  
```
jhat -port <port> jmap生成的文件
```
有时候文件比较大，需要比较大的内存可以这样使用  
```
 jhat -J-Xmx2G jmap生成的文件
```
一段时间后会提示server开启成功，
```
Reading from java_pid3048.hprof...
Dump file created Wed Sep 05 21:25:13 CST 2018
Snapshot read, resolving...
Resolving 729071 objects...
Chasing references, expect 145 dots.............................................
................................................................................
....................
Eliminating duplicate references................................................
................................................................................
.................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```
这时候就可以在7000端口上进行查看了  
![jhat](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/jvm-heap-dump/jhat.png)  
比较重要的是： Show instance counts for all classes (excluding platform) 和 Show heap histogram，显示堆实例的列表 
这个可以看到所有的除了平台（jdk内部的）的所有的class的实例，例如如图  
![jhat1.png](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/jvm-heap-dump/jhat1.png)  
这有240098个me.zks.troubleshoot.heaphump.Sample的实例
![jhat2](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/jvm-heap-dump/jhat2.png)  
当然，如果对象种类太多，这样看起来比较麻烦的话，可以使用查询  
![jhatsql](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/jvm-heap-dump/jhatsql.png)    
![jhatsql2](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/jvm-heap-dump/jhatsql2.png)  

当我们点击class me.zks.jvm.troubleshoot.heaphump.Sample 时，会进入到更详细的说明  
![jhat4](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/jvm-heap-dump/jhat4.png)  

我们可以通过对程序的理解，对各个对象的大小做一个比较客观的预估，如果显示的各个堆远大于

## 2. 频繁GC的分析  
TODO 20180916 以后会专门分析相关

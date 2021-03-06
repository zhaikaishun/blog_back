
---
title: java线上程序排错经验4 -Btrace了解一下
date: 2018-10-01 21:25:21
tags: [jvm,java]
categories: [java]
author: kaishun
id: 210
permalink: jvm04
---


## 简介  
在生产环境中经常遇到格式各样的问题，如OOM或者莫名其妙的进程死掉。一般情况下是通过修改程序，添加打印日志；然后重新发布程序来完成。然而，这不仅麻烦，而且带来很多不可控的因素。有没有一种方式，在不修改原有运行程序的情况下获取运行时的数据信息呢？如方法参数、返回值、全局变量、堆栈信息等。Btrace就是这样一个工具，它可以在不修改原有代码的情况下动态地追踪java运行程序，通过hotswap技术，动态将跟踪字节码注入到运行类中，进行监控甚至到达修改程序的目的
<!-- more -->
## 1. 下载  
https://github.com/btraceio/btrace/releases/latest  
## 2. 安装  
配置环境变量  
3. 本地jar包依赖  

## 本地写脚本的准备  
Btrace是脚本，不过也是.java的格式，写脚本前建议引入以下几个jar包，方便于代码编写的提示
jar包引用： 安装目录下build中有三个jar包
```
btrace-agent.jar
btrace-boot.jar
btrace-client.jar
```
三个核心jar直接拷贝到工程中临时使用即可  
```
<dependency>
	<groupId>com.sun.tools.btrace</groupId>
	<artifactId>btrace-agent</artifactId>
	<version>1.3.11</version>
	<scope>system</scope>
	<systemPath>D:/programmesoft/btrace/build/btrace-agent.jar</systemPath>
<dependency>
<dependency>
    <groupId>com.sun.tools.btrace</groupId>
    <artifactId>btrace-boot</artifactId>
    <version>1.3.11</version>
    <scope>system</scope>
    <systemPath>D:/programmesoft/btrace/build/btrace-boot.jar</systemPath>
</dependency>
<dependency>
    <groupId>com.sun.tools.btrace</groupId>
    <artifactId>btrace-client</artifactId>
    <version>1.3.11</version>
    <scope>system</scope>
    <systemPath>D:/programmesoft/btrace/build/btrace-client.jar</systemPath>
</dependency>
```
## 4. 简单入门  
4.1 待测试代码如下  
FirstSample  
该程序有三个方法，main, fun1, fun2  
main方法一个while循环不断的调用fun1方法，传入了两个字符串"aa","bb"，fun1调用fun2方法，传入一个字符串"bb"，然后休眠3秒钟，fun2方法会返回一个"fun2"+传入的参数
```
import java.util.concurrent.TimeUnit;
public class FirstSample {
    public static void main(String[] args) {
        FirstSample firstSample = new FirstSample();
        while (true){
            firstSample.fun1("aa","bb");
        }
    }
    public void fun1(String str1,String str2){
        try {
            String result = fun2(str2);
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
    private String fun2(String str2) {
        System.out.println("fun2 参数为:"+str2);
        return "fun2"+str2;
    }
}
```

4.2 Btrace入门案例   
假如我们想要知道每次调用fun1都是一些什么参数，我们得脚本如下 
```
import com.sun.btrace.BTraceUtils;
import com.sun.btrace.annotations.*;
@BTrace
public class FirstSampleBtrace {

    @OnMethod(
            clazz = "me.zks.jvm.troubleshoot.btrace.FirstSample"
            ,method = "fun1"
            ,location = @Location(value = Kind.ENTRY))
    public static void m(@Self Object self,String str1,String str2){
        BTraceUtils.print("fun1 str1:"+str1+" str2: "+str2+" ");
    }
}
```
其中第4行@BTrace注解表示使用Btrace来监控  
@OnMethod里面定义了内容如下
- clazz = "me.zks.jvm.troubleshoot.btrace.FirstSample"定义了哪个对象  
- method = "fun1"定义了哪个方法  
- location = @Location(value = Kind.ENTRY))定义了拦截的时机，我这里定义的是进入程序的时候进行拦截。更多拦截的请参考:   
m方法： m取名是任意取得  
里面的参数  
- @Self Object  
- String str1, String str2  这个是传入的参数，这个参数必须要和我们定义的方法参数一致，如果方法有重载，我们会根据这个参数的来判断去拦截哪个方法  
- BTraceUtils.print("fun1 str1:"+str1+" str2: "+str2); BtraceUtil的输出方法，用来输出结果  

将上面的FIrstSample运行起来  
程序每三秒执行一次  
```
fun1参数为:aa,bb
fun2参数为:bb
fun1参数为:aa,bb
fun2参数为:bb
fun1参数为:aa,bb
fun2参数为:bb
fun1参数为:aa,bb
fun2参数为:bb
...
```
用jps查看此程序的PID,发现PID为5044
```
C:\Users\Administrator>jps
2404
2420 RemoteMavenServer
3540 Launcher
5044 FirstSample
3372 Jps
```
启动脚本  
```
btrace -v 5044 FirstSampleBtrace.java
```
其中
- -v 代表可以输出到控制台，我们方便控制台查看  
- 5044是PID  
- FirstSampleBtrace.java是我们得脚本名  
其中输入-h 可以看使用方法
```
C:\Users\Administrator>btrace -h
Usage: btrace <options> <pid> <btrace source or .class file> <btrace arguments>
where possible options include:
  --version             Show the version
  -v                    Run in verbose mode
  -o <file>             The path to store the probe output (will disable showing the output in console)
-u                    Run in trusted mode
  -d <path>             Dump the instrumented classes to the specified path
  -pd <path>            The search path for the probe XML descriptors
  -classpath <path>     Specify where to find user class files and annotation processors
  -cp <path>            Specify where to find user class files and annotation processors
  -I <path>             Specify where to find include files
  -p <port>             Specify port to which the btrace agent listens for clients
  -statsd <host[:port]> Specify the statsd server, if any

C:\Users\Administrator>


```

显示  
```
DEBUG: loading D:\programmesoft\btrace\build\btrace-agent.jar
DEBUG: agent args: port=2020,statsd=,debug=true,bootClassPath=.,systemClassPath=
C:\Program Files\Java\jdk1.8.0_92\jre/../lib/tools.jar,probeDescPath=.
DEBUG: loaded D:\programmesoft\btrace\build\btrace-agent.jar
DEBUG: registering shutdown hook
DEBUG: registering signal handler for SIGINT
DEBUG: submitting the BTrace program
DEBUG: opening socket to 2020
DEBUG: setting up client settings
DEBUG: sending instrument command: []
DEBUG: entering into command loop
DEBUG: received com.sun.btrace.comm.OkayCommand@74a10858
DEBUG: received com.sun.btrace.comm.RetransformationStartNotification@23fe1d71
DEBUG: received com.sun.btrace.comm.OkayCommand@28ac3dc3
DEBUG: received com.sun.btrace.comm.MessageCommand@1d371b2d
fun1 str1:aa str2: bb DEBUG: received com.sun.btrace.comm.MessageCommand@543c6f6d

fun1 str1:aa str2: bb DEBUG: received com.sun.btrace.comm.MessageCommand@13eb8acf

fun1 str1:aa str2: bb DEBUG: received com.sun.btrace.comm.MessageCommand@51c8530f

fun1 str1:aa str2: bb DEBUG: received com.sun.btrace.comm.MessageCommand@7403c468

fun1 str1:aa str2: bb DEBUG: received com.sun.btrace.comm.MessageCommand@43738a82

fun1 str1:aa str2: bb
```
我们通过打印的输出可以看到程序开启了2020端口，fun1 str1:aa str2: bb  达到了我们监控的目的  

## 4.3 Btrace案例2-返回时拦截得到执行的时间  
假如我们想要知道每次调用fun2都调用了什么参数，并且fun1使用了多长的时间，代码如下  
FirstSampleBtrace  
```
import com.sun.btrace.BTraceUtils;
import com.sun.btrace.annotations.*;
@BTrace
public class FirstSampleBtrace {
//    @OnMethod(
//            clazz = "me.zks.jvm.troubleshoot.btrace.FirstSample"
//            ,method = "fun1"
//            ,location = @Location(value = Kind.ENTRY))
//    public static void m(@Self Object self,String str1,String str2){//,String str2
//        BTraceUtils.print("fun1 str1:"+str1+" str2: "+str2);
//    }

    @OnMethod(clazz = "me.zks.jvm.troubleshoot.btrace.FirstSample",
            method = "fun2",
            location = @Location(value = Kind.ENTRY))
    public static void m1(@Self Object self,String str1){
        BTraceUtils.print("fun2 str1:"+str1+" ");
    }

    @OnMethod(clazz = "me.zks.jvm.troubleshoot.btrace.FirstSample",
            method = "fun1",
            location = @Location(value = Kind.RETURN))
    public static void m2(@Self Object self,String str1,String str2,@Return Void a,@Duration long time){
        BTraceUtils.print("fun1 str1:"+str1+" str2:"+str2+" Duration is: "+time+" ");
    }

}
```
运行后即可知道打印出拦截的时间
```
D:\git\jvmtroubleshoot\src\main\java\me\zks\jvm\troubleshoot\btrace>btrace -v 6392 FirstSampleBtrace.java
DEBUG: assuming default port 2020
DEBUG: assuming default classpath '.'
DEBUG: compiling FirstSampleBtrace.java
DEBUG: compiled FirstSampleBtrace.java
DEBUG: attaching to 6392
DEBUG: checking port availability: 2020
DEBUG: attached to 6392
DEBUG: loading D:\programmesoft\btrace\build\btrace-agent.jar
DEBUG: agent args: port=2020,statsd=,debug=true,bootClassPath=.,systemClassPath=C:\Program Files\Java\jdk1.8.0_92\jre/../lib/tools.jar,probeDescPat
DEBUG: loaded D:\programmesoft\btrace\build\btrace-agent.jar
DEBUG: registering shutdown hook
DEBUG: registering signal handler for SIGINT
DEBUG: submitting the BTrace program
DEBUG: opening socket to 2020
DEBUG: setting up client settings
DEBUG: sending instrument command: []
DEBUG: entering into command loop
DEBUG: received com.sun.btrace.comm.OkayCommand@fcd6521
DEBUG: received com.sun.btrace.comm.RetransformationStartNotification@27d415d9
DEBUG: received com.sun.btrace.comm.OkayCommand@5c18298f
DEBUG: received com.sun.btrace.comm.MessageCommand@5204062d
fun2 str1:bb DEBUG: received com.sun.btrace.comm.MessageCommand@4fcd19b3
fun1 str1:aa str2:bb Duration is: 3001048056 DEBUG: received com.sun.btrace.comm.MessageCommand@376b4233
fun2 str1:bb DEBUG: received com.sun.btrace.comm.MessageCommand@2fd66ad3
fun1 str1:aa str2:bb Duration is: 2999950327 DEBUG: received com.sun.btrace.comm.MessageCommand@5d11346a
fun2 str1:bb DEBUG: received com.sun.btrace.comm.MessageCommand@7a36aefa
```
## 4.4 Btrace退出方法  
btrace退出时按ctrl+c 然后选择1进行退出，然后确认Y
```
fun1 str1:aa str2:bb Duration is: 2999939243 Please enter your option:
        1. exit
        2. send an event
        3. send a named event
        4. flush console output
1
DEBUG: sending exit command
终止批处理操作吗(Y/N)? y
```

**发现Btrace的一个bug，版本1.3.09~1.3.11都有**，return的时候，当ctrl+c 然后选择1时会导致入侵的程序crash，具体报错Btrace java.lang.NoSuchMethodError
```
Exception in thread "main" java.lang.NoSuchMethodError: me.zks.jvm.troubleshoot.btrace.FirstSample.$btrace$me$zks$jvm$troubleshoot$btrace$FirstSampleBtrace$m2(Ljava/lang/Object;Ljava/lang/String;Ljava/lang/String;Ljava/lang/Void;)V
	at me.zks.jvm.troubleshoot.btrace.FirstSample.fun1(Unknown Source)
	at me.zks.jvm.troubleshoot.btrace.FirstSample.main(Unknown Source)
```
这个应该是Btrace的一个bug，我已经向作者提了issues  
原因可能是由于TimeUnit.SECONDS.sleep(3)和Thread.sleep(3)导致，我猜测对于线程执行sleep()或join()方法时，Btrace有bug导致  
**所以说，我们得平时学习的时候，用Object.wait的等待阻塞来测试，而不应用Thread.sleep来测试，并且注意，如果我们拦截的方法中有Thread.sleep，这几个版本最好是做好jvm crash的打算，这点需要注意**  
```
Object o = new Object();
synchronized (o){
    o.wait(1000);
}
```
以后的测试，我将会使用上面的方法进行停顿。  

## 4.5 Btrace案例4 异常抛出和异常捕获
**Kind.ERROR**  
**待测试代码如下** 
代码在不断的调用divide方法，随机传一个数字进去, divide
```
package me.zks.jvm.troubleshoot.btrace;
import java.util.Random;
public class BtraceThrow {
    public static void main(String[] args) {
        Random random = new Random();
        BtraceThrow btraceThrow = new BtraceThrow();
        while (true){
            int randomNum =random.nextInt(4);
            try {
                int result = btraceThrow.divide(randomNum);
                System.out.println("result: "+result);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public int divide(int a) throws Exception{

        Object object = new Object();
		//每次暂停1秒
        synchronized (object){
            object.wait(1000);
        }
        return 100/a;
    }
}
```
btrace脚本如下  
其中需要有一个@TargetInstance Throwable参数
```
@BTrace
public class BtraceThrowScript {
    @OnMethod(
            clazz = "me.zks.jvm.troubleshoot.btrace.BtraceThrow",
            method = "divide",
            location = @Location(Kind.ERROR)
    )
    public static void testThrow(@Self Object self, @TargetInstance Throwable err , int a){
        BTraceUtils.print("a: ");
        BTraceUtils.Threads.jstack(err);
    }
}
```
监听后输出
```
DEBUG: received com.sun.btrace.comm.OkayCommand@3ecd23d9
DEBUG: received com.sun.btrace.comm.RetransformationStartNotification@569cfc36
DEBUG: received com.sun.btrace.comm.OkayCommand@43bd930a
DEBUG: received com.sun.btrace.comm.MessageCommand@553a3d88
a: DEBUG: received com.sun.btrace.comm.MessageCommand@7a30d1e6
java.lang.ArithmeticException: / by zero
DEBUG: received com.sun.btrace.comm.MessageCommand@5891e32e
        me.zks.jvm.troubleshoot.btrace.BtraceThrow.divide(BtraceThrow.java:24)
        me.zks.jvm.troubleshoot.btrace.BtraceThrow.main(BtraceThrow.java:10)
DEBUG: received com.sun.btrace.comm.MessageCommand@cb0ed20
a: DEBUG: received com.sun.btrace.comm.MessageCommand@8e24743
java.lang.ArithmeticException: / by zero
DEBUG: received com.sun.btrace.comm.MessageCommand@74a10858
        me.zks.jvm.troubleshoot.btrace.BtraceThrow.divide(BtraceThrow.java:24)
        me.zks.jvm.troubleshoot.btrace.BtraceThrow.main(BtraceThrow.java:10)
DEBUG: received com.sun.btrace.comm.MessageCommand@23fe1d71
a: DEBUG: received com.sun.btrace.comm.MessageCommand@28ac3dc3
java.lang.ArithmeticException: / by zero
DEBUG: received com.sun.btrace.comm.MessageCommand@32eebfca
        me.zks.jvm.troubleshoot.btrace.BtraceThrow.divide(BtraceThrow.java:24)
        me.zks.jvm.troubleshoot.btrace.BtraceThrow.main(BtraceThrow.java:10)
DEBUG: received com.sun.btrace.comm.MessageCommand@4e718207
a: DEBUG: received com.sun.btrace.comm.MessageCommand@1d371b2d
java.lang.ArithmeticException: / by zero
Please enter your option:
        1. exit
        2. send an event
        3. send a named event
        4. flush console output
DEBUG: received com.sun.btrace.comm.MessageCommand@543c6f6d
```
**Kind.Cache Kind.Throw**  
catch感觉有bug了，Throw感觉用不上，一般用ERROR代替Throw即可，所以这里不说了。具体读者自己去尝试。  

未完待续。。。  

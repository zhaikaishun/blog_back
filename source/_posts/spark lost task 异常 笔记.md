---
title: spark lost task 异常笔记
date: 2017-02-05 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 33
permalink: spark-exception
---

# Lost task java.lang.NullPointerException
Exception in thread "main" org.apache.spark.SparkException: Job aborted due to stage failure: Task 0 in stage 0.0 failed 1 times, most recent failure: Lost task 0.0 in stage 0.0 (TID 0, localhost): java.lang.NullPointerException  
<!-- more -->
我之前的代码类似于
```java
public Iterable<Tuple2<String, String>> call(String s) throws Exception{
    Tuple2<String, String> tp=null;
    try{
        if(XXX){
            tp=new Tuple2<>(xxx, xxx);
        }
        if(xxx){
          tp=new Tuple2<>(xxx, xxx);  
        }
        return Arrays.asList(tp);
    }
    catch(Exception e){
        e.printStackTrace();
        return new ArrayList<Tuple2<String, String>>();
    }

}
```
发现没有进入catch，那应该没问题啊，最后才发现，原来是tp对于两个if都不满足，return Arrays.asList(tp);中，tp为null，导致程序出错,**更加要注意的是，对于容错，如果某条数据不要，千万不能返回null,可以返回类似于new ArrayList<Tuple2<String, String>>();的list即可,返回null同样会报如上的错误**  

**特别是传过来的参数有可能导致空指针**  
一个对象并不是空的，但是在spark的内部，却报空指针，即使实现了序列化也有可能出现这种情况，这种情况下，应该像mapreduce中的setUp函数,或者使用广播传进来，这两种方法该如何实现呢？  参考 TODO


## Lost task java.lang.OutOfMemoryError: Java heap space   
java.lang.OutOfMemoryError: Java heap space
        at java.io.BufferedOutputStream.<init>(BufferedOutputStream.java:76)
        at java.io.BufferedOutputStream.<init>(BufferedOutputStream.java:59)
        at org.apache.spark.sql.execution.UnsafeRowSerializerInstance$$anon$2.<init>(UnsafeRowSerializer.scala:55)  
        
解决办法：

     由于我们在执行Spark任务是，读取所需要的原数据，数据量太大，导致在Worker上面分配的任务执行数据时所需要的内存不够，直接导致内存溢出了，所以我们有必要增加Worker上面的内存来满足程序运行需要。

在Spark Streaming或者其他spark任务中，会遇到在Spark中常见的问题，典型如Executor Lost 相关的问题(shuffle fetch 失败，Task失败重试等)。这就意味着发生了内存不足或者数据倾斜的问题。这个目前需要考虑如下几个点以获得解决方案：

A、相同资源下，增加partition数可以减少内存问题。 原因如下：通过增加partition数，每个task要处理的数据少了，同一时间内，所有正在运行的task要处理的数量少了很多，所有Executor占用的内存也变小了。这可以缓解数据倾斜以及内存不足的压力。

B、关注shuffle read 阶段的并行数。例如reduce,group 之类的函数，其实他们都有第二个参数，并行度(partition数)，只是大家一般都不设置。不过出了问题再设置一下，也不错。

C、给一个Executor 核数设置的太多，也就意味着同一时刻，在该Executor 的内存压力会更大，GC也会更频繁。我一般会控制在3个左右。然后通过提高Executor数量来保持资源的总量不变。  

# Lost task FileNotFoundException  
```
WARN TaskSetManager: Lost task 0.0 in stage 10.0 (TID 17, xx.xx.xx.xx): java.io.FileNotFoundException: File file:/usr/spark/test.txt does not exist
at org.apache.hadoop.fs.RawLocalFileSystem.deprecatedGetFileStatus(RawLocalFileSystem.java:534)
at org.apache.hadoop.fs.RawLocalFileSystem.getFileLinkStatusInternal(RawLocalFileSystem.java:747)
at org.apache.hadoop.fs.RawLocalFileSystem.getFileStatus(RawLocalFileSystem.java:524)
at org.apache.hadoop.fs.FilterFileSystem.getFileStatus(FilterFileSystem.java:409)
at org.apache.hadoop.fs.ChecksumFileSystem$ChecksumFSInputChecker.<init>(ChecksumFileSystem.java:140)
at org.apache.hadoop.fs.ChecksumFileSystem.open(ChecksumFileSystem.java:341)
at org.apache.hadoop.fs.FileSystem.open(FileSystem.java:766)
at org.apache.hadoop.mapred.LineRecordReader.<init>(LineRecordReader.java:108)
at org.apache.hadoop.mapred.TextInputFormat.getRecordReader(TextInputFormat.java:67)
at org.apache.spark.rdd.HadoopRDD$$anon$1.<init>(HadoopRDD.scala:239)
at org.apache.spark.rdd.HadoopRDD.compute(HadoopRDD.scala:216)
at org.apache.spark.rdd.HadoopRDD.compute(HadoopRDD.scala:101)
at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:277)
at org.apache.spark.rdd.RDD.iterator(RDD.scala:244)
at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:35)
at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:277)
at org.apache.spark.rdd.RDD.iterator(RDD.scala:244)
at org.apache.spark.scheduler.ResultTask.runTask(ResultTask.scala:63)
at org.apache.spark.scheduler.Task.run(Task.scala:70)
at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:213)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
at java.lang.Thread.run(Thread.java:745)
```  
解决办法
1. 确保文件路径没有错误
2. 确保文件没有损坏

# Lost task 0.0 in stage 0.0 (TID 0, localhost): java.lang.ArrayIndexOutOfBoundsException:  
数组越界,这种问题最好是本地先运行小数据可以成功，再尝试在集群运行大数据，一般会有提示，按照提示去看是为什么数组越界了   











# 异常  Exception in thread "main" org.apache.spark.SparkException: Job cancelled because SparkContext was shut down  
```
Exception in thread "main" org.apache.spark.SparkException: Job cancelled because SparkContext was shut down
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$cleanUpAfterSchedulerStop$1.apply(DAGScheduler.scala:736)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$cleanUpAfterSchedulerStop$1.apply(DAGScheduler.scala:735)
	at scala.collection.mutable.HashSet.foreach(HashSet.scala:79)
	at org.apache.spark.scheduler.DAGScheduler.cleanUpAfterSchedulerStop(DAGScheduler.scala:735)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onStop(DAGScheduler.scala:1468)
	at org.apache.spark.util.EventLoop.stop(EventLoop.scala:84)
	at org.apache.spark.scheduler.DAGScheduler.stop(DAGScheduler.scala:1403)
	at org.apache.spark.SparkContext.stop(SparkContext.scala:1642)
	at org.apache.spark.SparkContext$$anonfun$3.apply$mcV$sp(SparkContext.scala:559)
	at org.apache.spark.util.SparkShutdownHook.run(Utils.scala:2292)
	at org.apache.spark.util.SparkShutdownHookManager$$anonfun$runAll$1$$anonfun$apply$mcV$sp$1.apply$mcV$sp(Utils.scala:2262)
	at org.apache.spark.util.SparkShutdownHookManager$$anonfun$runAll$1$$anonfun$apply$mcV$sp$1.apply(Utils.scala:2262)
	at org.apache.spark.util.SparkShutdownHookManager$$anonfun$runAll$1$$anonfun$apply$mcV$sp$1.apply(Utils.scala:2262)
	at org.apache.spark.util.Utils$.logUncaughtExceptions(Utils.scala:1772)
	at org.apache.spark.util.SparkShutdownHookManager$$anonfun$runAll$1.apply$mcV$sp(Utils.scala:2262)
	at org.apache.spark.util.SparkShutdownHookManager$$anonfun$runAll$1.apply(Utils.scala:2262)
	at org.apache.spark.util.SparkShutdownHookManager$$anonfun$runAll$1.apply(Utils.scala:2262)
	at scala.util.Try$.apply(Try.scala:161)
	at org.apache.spark.util.SparkShutdownHookManager.runAll(Utils.scala:2262)
	at org.apache.spark.util.SparkShutdownHookManager$$anon$6.run(Utils.scala:2244)
	at org.apache.hadoop.util.ShutdownHookManager$1.run(ShutdownHookManager.java:54)
```
原因是我当时测试的时候 ，在flatmap的iterator的方法中使用了System.exit(0),sparkcontext关闭取消,所以导致报此错误  


##  Lost task 5.0 in stage 1.0 (TID 32, node003): java.lang.OutOfMemoryError: unable to create new native thread 

一般是内存溢出，内存溢出情况很多种，单个execute 负载太高等都可能导致，如果是yarn模式，execute-memory设置多一些


##  Lost task 1.0 in stage 6.0 (TID 17, 192.168.xx.xx): org.apache.hadoop.ipc.RemoteException(java.io.IOException)  File xxx  could only be replicated to 0 nodes instead of minReplication (=1).  There are 4 datanode(s) running and 4 node(s) are excluded in this operation
```
at org.apache.hadoop.hdfs.server.blockmanagement.BlockManager.chooseTarget4NewBlock(BlockManager.java:1550)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.getNewBlockTargets(FSNamesystem.java:3110)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.getAdditionalBlock(FSNamesystem.java:3034)
        at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.addBlock(NameNodeRpcServer.java:723)
        at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.addBlock(ClientNamenodeProtocolServerSideTranslatorPB.java:492)
        at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:616)
        at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:969)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2049)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:2045)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:415)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1657)
        at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2043)

        at org.apache.hadoop.ipc.Client.call(Client.java:1468)
        at org.apache.hadoop.ipc.Client.call(Client.java:1399)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:232)
        at com.sun.proxy.$Proxy13.addBlock(Unknown Source)
        at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolTranslatorPB.addBlock(ClientNamenodeProtocolTranslatorPB.java:399)
        at sun.reflect.GeneratedMethodAccessor7.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:606)
        at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:187)
        at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:102)
        at com.sun.proxy.$Proxy14.addBlock(Unknown Source)
        at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.locateFollowingBlock(DFSOutputStream.java:1532)
        at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.nextBlockOutputStream(DFSOutputStream.java:1349)

```

可能的情况：参考 http://blog.csdn.net/zuiaituantuan/article/details/6533867


##  WARN TaskSetManager: Lost task 0.0 in stage 2.0 (TID 4, 192.168.x.xx): java.net.SocketException: Too many open files
是否是在spark中，有大量打开文件操作，打开文件操作太多，可能会报这个错误








---
title: spark碰到的问题
date: 2017-05-07 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 60
permalink: spark-fqa
---

####  java.io.FileNotFoundException: File does not exist 
``` 
java.io.FileNotFoundException: File does not exist: hdfs://master:9000/user/hmaster/.sparkStaging/application_1498791665418_0041/spark-assembly-1.4.0-hadoop2.6.0.jar
        at org.apache.hadoop.hdfs.DistributedFileSystem$22.doCall(DistributedFileSystem.java:1309)
        at org.apache.hadoop.hdfs.DistributedFileSystem$22.doCall(DistributedFileSystem.java:1301)
        at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
        at org.apache.hadoop.hdfs.DistributedFileSystem.getFileStatus(DistributedFileSystem.java:1301)
        at org.apache.hadoop.yarn.util.FSDownload.copy(FSDownload.java:253)
        at org.apache.hadoop.yarn.util.FSDownload.access$000(FSDownload.java:63)
        at org.apache.hadoop.yarn.util.FSDownload$2.run(FSDownload.java:361)
        at org.apache.hadoop.yarn.util.FSDownload$2.run(FSDownload.java:359)
        at java.lang.Thread.run(Thread.java:745)
        ...

Failing this attempt. Failing the application.
         ApplicationMaster host: N/A
         ApplicationMaster RPC port: -1
         queue: root.hmaster
         start time: 1499311838058
         final status: FAILED
         tracking URL: http://master:8088/cluster/app/application_1498791665418_0041
         user: hmaster
Exception in thread "main" org.apache.spark.SparkException: Application application_1498791665418_0041 finished with failed status
        at org.apache.spark.deploy.yarn.Client.run(Client.scala:841)
        at org.apache.spark.deploy.yarn.Client$.main(Client.scala:867)
        at org.apache.spark.deploy.yarn.Client.main(Client.scala)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:606)
        at org.apache.spark.deploy.SparkSubmit$.org$apache$spark$deploy$SparkSubmit$$runMain(SparkSubmit.scala:664)
...
``` 
<!-- more -->
原因： 目前主要碰到这两种
1. 代码中本地测试setMaster("local")，集群中运行应该去掉这个 
2. 编译打包用的jdk版本和集群上配置不一样

#### spark yarn 模式下 一直 运行不报错

#### spark yarn模式下，打开节点很少

#### spark standalone 模式下，不报错，停在某个statge额某个任务
个人目前觉得standalone模式在一些复杂的计算中不太使用，之前测试一份数据，计算比较复杂，standalone为了追求速度，所有节点火力全开，导致cpu阻塞


#### 不断增加
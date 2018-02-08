---
title: yarn资源管理最佳实践
date: 2017-10-11 21:25:21
tags: [大数据,hadoop,yarn]
categories: [大数据,hadoop]
author: kaishun
id: 116
permalink: yarn-resource-best_practice
---

原文翻译自 https://mapr.com/blog/best-practices-yarn-resource-management/#.Ve5bLdOqoVU 有改动  -- 翻译以及记录的目的是对yarn进行合理的资源配置，以及yarn平台出错后的分析


这篇文章主要是讨论YARN资源管理的最佳实践，YARN的基本理论是将资源管理和任务调度分离，
所以设计了一个全局的资源管理器 ResourceManager（RM）和每个应用程序的ApplicationMaster（AM）。  

- **ResourceManager（RM）**：有两个主要组件：Scheduler和ApplicationsManager。调度程序负责将资源分配给各种正在运行的应用程序，这些应用程序受到容量，队列等熟悉的约束。==调度程序是纯调度程序，因为它不会监视或跟踪应用程序的状态==。此外，由于应用程序故障或硬件故障，它不提供有关重新启动失败任务的保证。调度器根据应用程序的资源需求执行调度功能; 它是基于资源容器的抽象概念来实现的，它包含了内存，CPU，磁盘，网络等元素。

<!-- more -->

- **ApplicationsManager：** 负责接受作业提交。协商执行特定于应用程序的ApplicationMaster的第一个容器，并提供失败时重新启动ApplicationMaster容器的服务。**每个应用程序的ApplicationMaster负责从调度程序中申请适当的资源容器，跟踪他们的状态并监视进度**。

在阅读本文之前，请先阅读[官方的YARN文档](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html)。  
![yarn架构](https://mapr.com/blog/best-practices-yarn-resource-management/assets/otherpageimages/YarnResourceMang-Fig1.png)  



本博客文章涵盖了有关YARN资源管理的以下主题，并为每个主题提供了最佳实践：

1. 监护人如何计算和分配资源给YARN？
2. YARN的最小和最大分配单位
3. 虚拟/物理内存检查器
4. Mapper，Reducer和AM的资源请求
5. 瓶颈资源

## 如何计算和分配资源给YARN？  
在hadoop集群中，各种资源都是默认分配的，我们可以对其进行修改，具体的各个参数是什么意思和各个参数的默认值，参考[mapr的文档](http://doc.mapr.com/display/MapR/Resource+Allocation+for+Jobs+and+Applications?_ga=2.152775354.2005604119.1513390596-784399482.1513390596) -- ps:  个人也准备仔细研究一番   

通过在NM节点的yarn-site.xml中设置以下三个配置并重新启动NM, 将可以配置每个节点的资源  
```
yarn.nodemanager.resource.memory-mb
yarn.nodemanager.resource.cpu-vcores
yarn.nodemanager.resource.io-spindles   不知道是什么
```  

那么可以在网页上看到每个节点自己所配置的内存，cpu核数

![集群参数](https://mapr.com/blog/best-practices-yarn-resource-management/assets/otherpageimages/YarnResourceMang-Fig2.png)

上面这几个参数应该适当的设置，如果设置太大可能会有问题，例如如果yarn.nodemanager.resource.memory-mb 设置过大，内存过度分配给YARN，可能会使用巨大的swap，并可能触发内核OOM杀手来终止容器进程  
下面的错误是OS OOM的一个标志，可能是内存过量分配了  
```
os::commit_memory(0x0000000000000000, xxxxxxxxx, 0) failed; 
error=’Cannot allocate memory’ (errno=12)
```  
这时就仔细检查各个节点，降低分配给yarn使用的内存  

## YARN中的最小和最大分配单位  
![yarn最大最小分配](https://mapr.com/blog/best-practices-yarn-resource-management/assets/otherpageimages/YarnResourceMang-Fig3.png)

基本上，这意味着RM只能以“yarn.scheduler.minimum-allocation-mb”为增量对容器分配内存，不能超过“yarn.scheduler.maximum-allocation-mb”。

它只能以“yarn.scheduler.minimum-allocation-vcores”为增量分配CPU vcore到容器，不能超过“yarn.scheduler.maximum-allocation-vcores”。

如果需要更改，请在RM节点的yarn-site.xml中设置以上配置，然后重新启动RM。

例如，如果一个作业要求每个映射容器1025 MB的内存（set mapreduce.map.memory.mb = 1025），RM会给它一个2048 MB（2 * yarn.scheduler.minimum-allocation-mb）容器。

如果您有一个需要9999 MB地图容器的巨大MR作业，则作业将在AM日志中显示以下错误消息：
```
MAP capability required is more than the supported max container capability in the cluster. 
Killing the Job. mapResourceRequest: 9999 maxContainerCapability:8192
```

如果YARN作业上的Spark要求容量大于“yarn.scheduler.maximum-allocation-mb”的大容器，则会显示以下错误：  
```
Exception in thread "main" java.lang.IllegalArgumentException: 
Required executor memory (99999+6886 MB) is above the max threshold (8192 MB) of this cluster!
```  
**在以上两种情况下，您可以增加yarn-site.xml中的“yarn.scheduler.maximum-allocation-mb”并重新启动RM**。

<font color=#be1a21 size=3 face="黑体"> 所以在这一步中，需要熟悉每个Mapper和Reducer的资源需求的上下界，并据此设置最小和最大分配单位。</font>

## 虚拟/物理内存检查器  
NodeManager可以监视容器的内存使用情况（虚拟的和物理的）。如果它的虚拟内存超过了“yarn.nodemanager.vmem-pmem-ratio”乘以“mapreduce.reduce.memory.mb”或者“mapreduce.map.memory.mb”，并且如果“yarn.nodemanager。 vmem-check-enabled“ 设置的是true;  
容器将会被kill掉  

同理。如果它的物理内存超过了“mapreduce.reduce.memory.mb”或者“mapreduce.map.memory.mb”，并且“yarn.nodemanager.pmem-check-enabled”设置的为true，容器也将被终止。  

下面的参数可以在每个NM节点的yarn-site.xml中设置，以覆盖默认行为。  
![yarn设置](https://mapr.com/blog/best-practices-yarn-resource-management/assets/otherpageimages/YarnResourceMang-Fig4.png)  

**这是由虚拟内存检查器杀死的容器的示例错误：**  
```
Current usage: 347.3 MB of 1 GB physical memory used; 
2.2 GB of 2.1 GB virtual memory used
```  

**这是物理内存检查器的示例错误：**
```
Current usage: 2.1gb of 2.0gb physical memory used  
1.1gb of 3.15gb virtual memory used. Killing 
```  
解决办法参考我的一篇文章： http://blog.csdn.net/t1dmzks/article/details/78818874  


## Mapper，Reducer和AM的资源请求  
MapReduce v2作业有3种不同的容器类型 - Mapper，Reducer和AM。 Mapper和Reducer可以要求资源 - 内存，CPU和磁盘，而AM只能要求内存和CPU。以下是三种容器类型的资源请求的配置摘要. 默认值来自MapR 4.1的Hadoop 2.5.1，它们可以在客户端节点的mapred-site.xml中被覆盖，也可以在MapReduce java代码，Pig和Hive Cli等应用程序中设置。 **程序设置优先于命令行的配置优先于节点的配置**  


- **Mapper:**    
![mapper配置](https://mapr.com/blog/best-practices-yarn-resource-management/assets/otherpageimages/YarnResourceMang-Fig5.png)  




- **Reducer:**
 ![reducer配置](https://mapr.com/blog/best-practices-yarn-resource-management/assets/otherpageimages/YarnResourceMang-Fig6.png)  



- **AM:**  
![AM配置](https://mapr.com/blog/best-practices-yarn-resource-management/assets/otherpageimages/YarnResourceMang-Fig7.png)  


<font color=#be1a21 size=3 face="黑体"> 每个容器实际上是一个JVM进程，在java-opts的“-Xmx”之上应该适合分配的内存大小。一个最佳的做法是将其设置为0.8 *（容器内存分配）</font>。例如，如果请求的mapper容器的mapreduce.map.memory.mb = 4096，我们可以设置mapreduce.map.java.opts = -Xmx3277m。  

有很多因素会影响每个容器的内存要求。这些因素包括Mappers / Reducers的数量，文件类型（纯文本文件，Parquet，ORC），数据压缩算法，操作类型（排序，分组，聚合，连接），数据歪斜等。熟悉MapReduce作业的性质，找出Mapper，Reducer和AM的最低要求。任何类型的容器如果不满足最小内存要求，可能会耗尽内存，并被物理/虚拟内存检查器杀死。如果是这样，你需要检查AM日志和失败的容器日志来找出原因。  

例如，如果MapReduce作业排序parquet文件，则Mapper需要将整个Parquet行组缓存在内存中。我已经做了测试，证明镶木地板文件的行组大小越大，需要更大的Mapper内存。在这种情况下，确保Mapper内存足够大而不触发OOM。  

另外一个例子是AM内存不足。通常情况下AM的1G Java堆大小足够用于许多作业， 但是如果工程中需要写大量的parquet文件， 在此期间commit提交了一个job, AM 将调用ParquetOutputCommitter.commitJob(), 这个方法会将首先读取**所有**输出parquet文件的页脚，并在输出目录中写入名为“_metadata”的元数据文件  
在AM日志中，此步骤可能会导致AM的内存不足  
```
Caused by:  java.lang.OutOfMemoryError   GC overhead limit exceeded
        at java.lang.StringCoding$StringEncoder.encode(StringCoding.java:300)
        at java.lang.StringCoding.encode(StringCoding.java:344)
        at java.lang.String.getBytes(String.java:916)
        at parquet.org.apache.thrift.protocol.TCompactProtocol.writeString(TCompactProtocol.java:298)
        at parquet.format.ColumnChunk.write(ColumnChunk.java:512)
        at parquet.format.RowGroup.write(RowGroup.java:521)
        at parquet.format.FileMetaData.write(FileMetaData.java:923)
        at parquet.format.Util.write(Util.java:56)
        at parquet.format.Util.writeFileMetaData(Util.java:30)
        at parquet.hadoop.ParquetFileWriter.serializeFooter(ParquetFileWriter.java:322)
        at parquet.hadoop.<font color="red">ParquetFileWriter.writeMetadataFile</font>(ParquetFileWriter.java:342)
        at parquet.hadoop.<font color="red">ParquetOutputCommitter.commitJob</font>(ParquetOutputCommitter.java:51)
        ... 10 more

```  
**解决方法是增加AM的内存需求**，并通过“set parquet.enable.summary-metadata false”来禁用parquet功能。  

除了找出每个容器的最小内存需求之外，有时我们需要平衡工作性能和资源容量。例如，进行排序的作业可能需要相对较大的“mapreduce.task.io.sort.mb”以避免或减少溢出文件的数量。如果整个系统有足够的内存容量，我们可以增加“mapreduce.task.io.sort.mb”和容器内存，以获得更好的工作性能。  

<font color=#be1a21 size=3 face="黑体"> 在这一步中，我们需要确保每种类型的容器都满足适当的资源需求。如果发生OOM，请首先检查AM日志以确定哪个容器以及每个堆栈跟踪的原因是。</font>


## 瓶颈资源  
由于有三种类型的资源，来自不同工作的不同容器可能要求不同的资源量。这可能导致其中一个资源成为瓶颈。假设我们有一个容量为1000G RAM，16个核心，16个磁盘的集群，每个Mapper容器需要10G RAM，1个核心，0.5个磁盘，最多可以有16个Mapper并行运行，因为CPU核心成了这里的瓶颈。

因此，（840G RAM，8个磁盘）资源不被任何人使用。如果你遇到这种情况，只需检查RM UI（http：//：8088 /集群/节点）来确定哪个资源是瓶颈。您可以将剩余的资源分配给可以提高性能的作业。例如，您可以分配更多的内存来排序作业溢出到磁盘。  


## 关键点  

- 确保分配给集群的内存大小和cpu核心数要合适  
- 熟悉mapper和reducer的资源需求的上下界。  
- 请注意虚拟和物理内存检查器。  
- 将每个容器的java-opts的-Xmx设置为0.8 *（容器内存分配）。
- 确保每种类型的容器都符合适当的资源要求。  
- 充分利用瓶颈资源。



















































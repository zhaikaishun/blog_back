---
title: Spark Shuffle FetchFailedException异常
date: 2017-05-09 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 61
permalink: spark-fetchfailedException
---

翻译 
http://stackoverflow.com/questions/34941410/fetchfailedexception-or-metadatafetchfailedexception-when-processing-big-data-se  

```
org.apache.spark.shuffle.MetadataFetchFailedException: Missing an output location for shuffle 0

org.apache.spark.shuffle.FetchFailedException: Failed to connect to ip-xxxxxxxx

org.apache.spark.shuffle.FetchFailedException: Error in opening FileSegmentManagedBuffer{file=/mnt/yarn/nm/usercache/xxxx/appcache/application_1450751731124_8446/blockm
gr-8a7b17b8-f4c3-45e7-aea8-8b0a7481be55/08/shuffle_0_224_0.data, offset=12329181, length=2104094}
```
<!-- more -->
This error is almost guaranteed to be caused by memory issues on your executors. I can think of a couple of ways to address these types of problems.

1) You could try to run with more partitions (do a repartition on your dataframe). Memory issues typically arise when one or more partitions contain more data than will fit in memory.

2) I'm noticing that you have not explicitly set spark.yarn.executor.memoryOverhead, so it will default to max(386, 0.10* executorMemory) which in your case will be 400MB. That sounds low to me. I would try to increase it to say 1GB (note that if you increase memoryOverhead to 1GB, you need to lower --executor-memory to 3GB)

3) Look in the log files on the failing nodes. You want to look for the text "Killing container". If you see the text "running beyond physical memory limits", increasing memoryOverhead will - in my experience - solve the problem.

大概的翻译：
这个错误几乎肯定会被记忆的问题在你的遗嘱执行人造成的。我能想到几种方法来解决这些类型的问题。

1）你可以尝试更多的分区上运行。存储器问题通常出现在一个或多个分区包含比将适合存储器更多的数据。

2）我注意到，你还没有明确设置spark.yarn.executor.memoryOverhead，所以它会默认max(386, 0.10* executorMemory)而你的情况将是400MB。这可能太低了。我会尽量增加到1GB（注意，如果你增加memoryOverhead为1GB，你需要降低--executor-memory至3GB）

3）看在日志文件中失败的节点上。你想查找文本“杀容器”。如果你看到文本“运行超出物理内存限制”，增加memoryOverhead   这是我解决的经验  



额外补充一些： 如果存在数据倾斜，那么可以从这方面来解决，使用更好的分区方式，例如修改分区的数量，方式，或者自定义分区的方式

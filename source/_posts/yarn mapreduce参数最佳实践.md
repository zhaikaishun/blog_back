---
title: yarn mapreduce参数最佳实践
date: 2018-5-05 13:46:21
tags: [hadoop,大数据]
categories: [大数据,hadoop]
author: kaishun
id: 181
permalink: yarn-mapreduce-best-practice
blogexcerpt: 小文件整合,mapreduce中map的个数和两个有关，一个是文件的个数，一个是大小，默认split是128M，推测功能，mapreduce.reduce.speculative，调优就是尽可能的让集群资源充分利用，这里需要根据具体的需求和集群资源情况来定
---

## mapreduce.job.queuename  
设置队列名
## 小文件整合  
- mapreduce.input.fileinputformat.split.minsize  
- mapreduce.input.fileinputformat.split.maxsize  
- mapreduce.input.fileinputformat.split.minsize.per.node  
- mapreduce.input.fileinputformat.split.minsize.per.rack  

mapreduce中map的个数和两个有关，一个是文件的个数，一个是大小，默认split是128M， 如果一个文件大于128M，例如129M，那么会有两个map，一个是128M，一个是1M。  
又例如有10个文件，每个都是1M，那么map是会有10个的。  
对于这种小文件太多，或者是我们想讲每一个map处理的数据量大一些，就应该设置上面的几个参数，上面几个参数是byte的单位。  
例如我们想设置一次处理1G，那么就设置成
```
mapreduce.input.fileinputformat.split.minsize = 1024*1024*1024*1024
mapreduce.input.fileinputformat.split.maxsize = 2048*1024*1024*1024
mapreduce.input.fileinputformat.split.minsize.per.node = 512*1024*1024*1024
mapreduce.input.fileinputformat.split.minsize.per.rack = 512*1024*1024*1024
```
## 推测功能  
mapreduce.reduce.speculative  
默认是true，有时候需要设置成false。  
参考： http://itfish.net/article/60389.html或者搜索

## container大小设置最佳实践 
**mapreduce.map.memory.mb** 和 **mapreduce.reduce.memory.mb**  和**mapreduce.map.cpu.vcores**和 **mapreduce.reduce.cpu.vcores**

mapreduce.map.memory.mb 默认1024M，一个map启动的container是1024M  
mapreduce.reduce.memory.mb 默认3072M，一个map启动的container是3072  
mapreduce.map.cpu.vcores默认1个vcore，一个map任务默认使用一个虚拟核运行
mapreduce.reduce.cpu.vcores默认1个vcore，一个reduce任务默认使用一个虚拟核运行。   

**调优就是尽可能的让集群资源充分利用，这里需要根据具体的需求和集群资源情况来定**。  
例如不考虑内存溢出等情况, 集群资源如下

Memory Total | VCores Total
---|---
320G | 80

如果数据比较均匀，应该尽可能的设置成如下：  

mapreduce.map.memory.mb | mapreduce.reduce.memory.mb | mapreduce.map.cpu.vcores |mapreduce.reduce.cpu.vcores
---|---|---|---|
4096 | 4096 | 1 |1

这样并发数能到

max(320G/4G，80vcore/1vcore)=80
上面是map核reduce都到了最大的80的并发，集群利用最充分。  

一般来说，我们默认mapreduce.map.cpu.vcores和mapreduce.reduce.cpu.vcores为1个就好了，**但是对于一个map和一个reduce的container的内存大小设置成了4G，如果一个map和一个reduce处理的任务很小，那又会很浪费资源，这时，对于map来说，可以用前面说的小文件整合，设置mapreduce.input.fileinputformat.split来解决map的大小，尽可能接近4G，但是又要注意可能出现的内存溢出的情况。  
对于reduce，1个container启动用了4G内存，那这4G内存也应尽可能的充分使用，这时候，我们尽量的评估进入到reduce的数据大小有多少，合理的设置reduceTask数，这一步是比较麻烦的，因为这里如果出现数据倾斜将会导致oom内存溢出错误**。  

前面说到了，并发数受到集群总内存/container的限制，同时，并发数也会受到集群vcore的限制，还是上面那个例子，例如集群资源为320G，80vcore，我一个map任务为2G，由于受到cpu的限制，最多同时80个vcore的限制，那么内存使用只能使用160G。这显然是浪费资源了。  

对于mapreduce.map.cpu.vcores和mapreduce.reduce.cpu.vcores，为什么默认是1呢，在集群的内存/cpu很小的情况下，能否一个map端将这两个值设置成2或者更大呢。这是当然可以的，但是，即使我们将这个设置成2，**任务的并发并不会是 Vcores Total/2的关系，并发仍然将是上面两条决定的**。举个例子，还是320G，80vcore集群。我们设置mapreduce.map.memory.mb为4G，mapreduce.map.cpu.vcores为2， 很多人以为我一个map需要两个核，那么80vcore/2vcore=40，那么我们并发最大只能用到40*4=160G的内存，其实这是错误的，这种情况，我们任然基本上能将内存占满，并发数任然能到80个。这个时候， mapreduce.map.cpu.vcores基本就失效了。后来仔细想了想，一个map或者reduce任务，里面的数据应该并不可能会有多线程并发，但是mapreduce.map.cpu.vcores为什么会有这个参数呢，后来想了一下，一个map或者reduce任务，代码的执行肯定是一个线程，但是任务的状态等监控，垃圾回收等，是可以使用另外一个线程来运行的，这也是将mapreduce.map.cpu.vcores设置成2可能会快一点的效果。  
我曾经碰到一个cpu十分充足的集群，vcore和内存比例是1比1，但是为了让数据不倾斜，我们的mapreduce.reduce.memory.mb至少要到4G，那么这时候，其实cpu就只能利用1/4了，这时候cpu很充足，我便尝试将mapreduce.map.cpu.vcores设置成2.其实这样也并不是说我一定每个map都能使用到2个vcore，只不过有时候，有的任务状态监控，jvm垃圾回收等，就有了另外一个vcore来运行了。  




## mapreduce.task.io.sort.mb  
这个参数理解需要理解mapreduce的shuffle过程，mapreduce的shuffle中，有一个环形缓冲区（就是一个带有前后两个指针的数组，shuffle过程自行搜索），这个值默认是100兆，配合上有个参数mapreduce.task.io.sort.spill.percent，一般这个参数默认为0.8，那么就意味着，这个数组到了80M，我就要开始进行排序了，然后要往磁盘写数据了。所以这个值越大，就不用导致频繁的溢出了。  
按照经验，一般这个值和map的输出，reduce的输入的大小相关比较好，但是这个值最好别超过2046，假如一个reduce处理的数据量为1G，那么这个值可以设置成200M， 一般的经验是reduce处理的数据量/5的关系比较好。  


## mapreduce.map.java.opts  
就是一个map container中jvm虚拟机的内存
一般设置成mapreduce.map.memory.mb的0.8倍比较合适
例如mapreduce.map.memory.mb=4096
mapreduce.map.java.opts 设置成 -Xmx3276M



## mapreduce.reduce.java.opts  
就是一个reduce container中jvm虚拟机的内存
一般设置成mapreduce.reduce.memory.mb的0.8倍比较合适
例如mapreduce.reduce.memory.mb=4096
mapreduce.reduce.java.opts 设置成 -Xmx3276M


## yarn.app.mapreduce.am.resource.mb  
MR ApplicationMaster占用的内存量，具体设置TODO，记得有时候小文件太多，超过多少万，这个太小了任务不会运行  


## mapreduce.task.timeout  
mapreduce任务默认超时时间，有时候抢队列的时候，这个会用上，默认值600000就好，不用管

## mapred.max.map.failures.percent
允许map失败的比例，默认是0，可以根据自己需求，合理设置

## mapred.max.reduce.failures.percent  
允许reduce失败的比例，默认是0，可以根据自己需求，合理设置

## mapreduce.job.reduce.slowstart.completedmaps  
map不用跑完就可以开始reduce了的比例，默认是0.95（网上说的0.05感觉不对啊），也就是map完成到百分之95时就可以开始reduce了，这样的好处是到了map最后几个，其实大多数资源都空闲了，这时候就先进行reduce吧，不然等全部跑完map有点浪费资源了。  
但是我之前碰到过一次资源死锁饿死的情况，就是map还有几个没跑完，reduce已经起来了，然而reduce需要等待map跑完的数据，reduce端拉不到，然后map端也没完成，并且整个集群的资源都被利用完了，这样map跑不完，reduce也跑不完，就这样相互等待卡着
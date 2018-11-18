---
title: mapreduce的map阶段中某几个task非常慢的一次排错过程与总结
date: 2018-08-03 21:25:21
tags: [hadoop]
categories: [大数据]
author: kaishun
id: 205
permalink: mapreduce-trouble-shotting
---


### 发现问题： 
在家里的测试集群测试数据，发现如下问题：  
程序map阶段很慢，然后通过hadoop的集群界面，几乎大多数的task都是在几分钟就执行完，看到有几个task非常慢，执行了4个多小时还不到一半。  
![task](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/map%E4%B8%AD%E6%9F%90%E4%B8%AAtask%E5%8D%A1%E4%BD%8F%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90/task慢开始.png) 

### 分析原因  
要么数据和代码问题，要么测试集群问题  
## 初步查看测试集群问题  
通过hadoop UI 和hdfs UI 并没有发现有任何问题，测试集群11个节点都是Active。 
![health](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/map%E4%B8%AD%E6%9F%90%E4%B8%AAtask%E5%8D%A1%E4%BD%8F%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90/health.png)  

### 初步检查代码和数据问题   

**排除数据倾斜问题**： map阶段是不会数据倾斜的，而且我们一个map是设置了跑多少数据的，我们所设置的是6G，也就是一个map跑6G的数据
查看代码问题： 因为我们的程序之前都运行过，并没发现什么问题，但是也是有点担心最近有改动或者某些异常数据导致程序出问题了。  
**代码问题可能的情况**：  
1. 某个地方try cache了，try cache肯定会导致很慢，但是也不至于这么慢，基本排除这个问题。
2. 进入了死循环或者某个大循环（复杂度很高的运算），通过task查看，确实好几分钟都没有变化了
3. 程序等原因等可能导致MapContainer内存不足，或许大量GC  


### 进一步分析问题  
通过界面这几个慢的task可以分析出，并不是说某一台机器慢，他们分别分散在不同的节点中，那么我就随便取了一个慢的节点进行查看。   
jvm分析，我还是建议使用命令行的摸索，例如jmap，jstack等命令，占用的内存会比较少，我这里是比较懒，直接用的界面程序查看。
**1. 使用xstart远程桌面连接**  因为我对jvm的界面工具比较熟悉
**2. 找到jdk按照目录，先打开jvisualVM, 监控某个container的cpu和内存 ** 
如下所示  
![jvisualVM](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/map%E4%B8%AD%E6%9F%90%E4%B8%AAtask%E5%8D%A1%E4%BD%8F%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90/jvisualvm.png)  

**3. 找到我们的这个container**：  
如何找是哪个container呢？ 我们其实可以通过每个container运行的时间去找，一个一个找，找到一个时间和我们hadoop ui上这个task运行时间一样的，那么就可以确定是这个container了，然后我找到的是这个container 。（当然，在linux界面上，查看ps -ef |grep pid，可以看到全名，其中有个attmpt_task_id， 这个也可以确定是哪个container） 
![pid](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/map%E4%B8%AD%E6%9F%90%E4%B8%AAtask%E5%8D%A1%E4%BD%8F%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90/pid.png) 

**4. 监控container**    
![cpuAndMemmonitor](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/map%E4%B8%AD%E6%9F%90%E4%B8%AAtask%E5%8D%A1%E4%BD%8F%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90/cpuAndMemMonitor.png) 

**分析问题**：  
从图中可以看出，cpu占用率很低： 那么排除是死循环或者大循环问题
右边堆内存最大3个G，我们程序占用1G不到，而且右边和左边都可以看出，没有发生GC， 说明内存充足。  

这时候我就暂时卡住了，感觉很诡异，不是cpu，也不是内存，到底是什么问题呢？   
看看线程栈，将线程dump下来查看：  
![threaddump](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/map%E4%B8%AD%E6%9F%90%E4%B8%AAtask%E5%8D%A1%E4%BD%8F%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90/thread-dump.png)   
通过分析，发现是maptask再等待拉的操作，我们主线程是runnable状态，但是cpu利用很低。是在线程的wait阶段。 OK，这里很容易就猜到是在进行IO等操作导致的了，由于这个稍微比较卡，我又用回jconsole了，因为为了给同事看，我就没用jstack工具。  


打开jconsole工具具体查看线程栈  
![jconsole线程Main锁定](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/map%E4%B8%AD%E6%9F%90%E4%B8%AAtask%E5%8D%A1%E4%BD%8F%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90/jconsole线程Main锁定.png)   
发现是在读取hdfs的时候，由于是同步IO，main方法一直在等待读取hdfs的数据  
然后同时查看TCP传输的方法， 也同样卡在io流的地方，占用了一个锁，一直不释放（占用锁不释放是正常的，但是这个读取的操作，每个文件都会是一个锁，我dump多次都发现是一样的锁，那么就发现可能是io问题了）  
![tcp已经锁定](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/map%E4%B8%AD%E6%9F%90%E4%B8%AAtask%E5%8D%A1%E4%BD%8F%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90/tcp已经锁定.png)  

**IO问题**： 我们可以基本猜测（确定）是IO问题了。  

登录到刚才有问题的这个container所在的节点，查看一下IO负载，因为担心是不是说某些程序在占用IO，发现后并不是，并且读取hdfs的速度很慢，一秒钟才1M，后来想了想那一小时才3个多G的读取速度，肯定慢啊。  
![iotop](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/map%E4%B8%AD%E6%9F%90%E4%B8%AAtask%E5%8D%A1%E4%BD%8F%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90/iotop.png)    

查看是哪个节点比较慢  
**界面中看log知道这个mapTask跑的是这些数据**
```
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00235:805306368+130232698,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00239:0+134217728,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00239:134217728+134217728,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00239:268435456+134217728,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00239:402653184+134217728,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00239:536870912+134217728,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00239:671088640+134217728,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00239:805306368+110006092,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00277:0+134217728
,/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00277:134217728+134217728
,/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00277:268435456+134217728,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00277:402653184+134217728,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00277:536870912+134217728,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00277:671088640+134217728,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00277:805306368+111362066
,/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00278:0+134217728,
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00278:134217728+134217728


``` 
**在界面中可以知道跑到了百分之16**， ok，这时候估计是在第二个文件（因为每个文件的大小都差不多）  
查看第二个文件在哪个节点
 hadoop fsck /mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00239  -files -locations -blocks  
 
 显示  
```
 DEPRECATED: Use of this script to execute hdfs command is deprecated.
Instead use the hdfs command for it.

SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hmaster/MyCloudera/APP/hadoop/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hmaster/MyCloudera/APP/hbase/hbase-1.2.5/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
Connecting to namenode via http://master:50070/fsck?ugi=hmaster&files=1&locations=1&blocks=1&path=%2Fmt_wlyh%2FData%2FBeiJing%2Fmroxdrmerge%2Fxdr_loc%2Fdata_01_180125%2FXDR_LOCATION_01_180125%2FxdrLocation-r-00239
FSCK started by hmaster (auth:SIMPLE) from /192.168.1.31 for path /mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00239 at Thu Aug 02 19:10:29 CST 2018
/mt_wlyh/Data/BeiJing/mroxdrmerge/xdr_loc/data_01_180125/XDR_LOCATION_01_180125/xdrLocation-r-00239 915312460 bytes, 7 block(s):  OK
0. BP-369656756-192.168.1.65-1441938871593:blk_1091923684_18278024 len=134217728 repl=1 [DatanodeInfoWithStorage[192.168.1.72:50010,DS-5ce4cd2a-bda2-4c02-9fd6-cc008f589945,DISK]]
1. BP-369656756-192.168.1.65-1441938871593:blk_1091923705_18278045 len=134217728 repl=1 [DatanodeInfoWithStorage[192.168.1.72:50010,DS-5ce4cd2a-bda2-4c02-9fd6-cc008f589945,DISK]]
2. BP-369656756-192.168.1.65-1441938871593:blk_1091923724_18278064 len=134217728 repl=1 [DatanodeInfoWithStorage[192.168.1.72:50010,DS-5ce4cd2a-bda2-4c02-9fd6-cc008f589945,DISK]]
3. BP-369656756-192.168.1.65-1441938871593:blk_1091923736_18278076 len=134217728 repl=1 [DatanodeInfoWithStorage[192.168.1.72:50010,DS-5ce4cd2a-bda2-4c02-9fd6-cc008f589945,DISK]]
4. BP-369656756-192.168.1.65-1441938871593:blk_1091923744_18278084 len=134217728 repl=1 [DatanodeInfoWithStorage[192.168.1.72:50010,DS-5ce4cd2a-bda2-4c02-9fd6-cc008f589945,DISK]]
5. BP-369656756-192.168.1.65-1441938871593:blk_1091923751_18278091 len=134217728 repl=1 [DatanodeInfoWithStorage[192.168.1.72:50010,DS-5ce4cd2a-bda2-4c02-9fd6-cc008f589945,DISK]]
6. BP-369656756-192.168.1.65-1441938871593:blk_1091923762_18278102 len=110006092 repl=1 [DatanodeInfoWithStorage[192.168.1.72:50010,DS-5ce4cd2a-bda2-4c02-9fd6-cc008f589945,DISK]]

Status: HEALTHY
 Total size:    915312460 B
 Total dirs:    0
 Total files:   1
 Total symlinks:                0
 Total blocks (validated):      7 (avg. block size 130758922 B)
 Minimally replicated blocks:   7 (100.0 %)
 Over-replicated blocks:        0 (0.0 %)
 Under-replicated blocks:       0 (0.0 %)
 Mis-replicated blocks:         0 (0.0 %)
 Default replication factor:    1
 Average block replication:     1.0
 Corrupt blocks:                0
 Missing replicas:              0 (0.0 %)
 Number of data-nodes:          11
 Number of racks:               1
FSCK ended at Thu Aug 02 19:10:29 CST 2018 in 0 milliseconds
```
 
 **说明这个数据在65和72节点**  


**进一步查看**，确实发现72节点（node002）有问题，ping很慢  
![ping速度](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/map%E4%B8%AD%E6%9F%90%E4%B8%AAtask%E5%8D%A1%E4%BD%8F%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90/ping速度.png)    
ping node002发现很慢，但是ping其他的发现还好

再测试一下node002的网络传输性能  
```
dd if=/dev/zero of=/dev/null bs=50 count=100M
```
SCP到master节点的时候，发现速度很慢，平均就是1.2M每秒。  


**找来一位运维的同事，然后他发现了问题**  
![网卡问题](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/map%E4%B8%AD%E6%9F%90%E4%B8%AAtask%E5%8D%A1%E4%BD%8F%E9%97%AE%E9%A2%98%E5%88%86%E6%9E%90/网卡.png)   
其他的节点都是1000M的网卡，而这个节点确实10M的网卡，肯定慢啊，也证实了前面的1.2M的scp和1M的hdfs读取速度就是这个问题引起的。    

后来发现可能是网线还是什么的问题，后来解决后，集群也就恢复了正常，程序也能正常运行了。  


### 排错总结：  
1. 冷静，这是测试集群，操作起来还比较方便，如果是正式环境，操作起来会比较麻烦，这时候更要冷静思考，不要放弃  
2. 理论指导实践，需要对jvm，hadoop等原理有所了解，需要对分布式，IO流，网络，linux等有所了解。即使没有经验，也能依靠自己的理论知识来指导实践  
3. 合理猜测中进行，最后验证猜测，除非很有经验，否则还是要在猜测中不断的证实自己的猜测，要大胆的猜测，有规划的验证  
4. 总结排错经验总结，对以后很有帮助
5. jvm定位技巧还是需要点火候的，堆内存溢出还比较简单，堆栈dump转存是比较考验一个人能力的。
6. 感觉在spark中，界面显示更友好一些，mapreduce中的排错，可能是经常会用上述到这种方法的

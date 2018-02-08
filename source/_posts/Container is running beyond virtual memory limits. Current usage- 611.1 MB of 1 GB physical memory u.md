---
title: Container is running beyond virtual memory limits. Current usage- 611.1 MB of 1 GB physical memory u
date: 2017-10-25 21:25:21
tags: [大数据,hadoop,yarn]
categories: [大数据,hadoop]
author: kaishun
id: 118
permalink: yarn-vmemory-over
---

hadoop 在 yarn 上运行时报的虚拟内存错误，或者是物理内存不够错误
## 异常显示
```shell

Container [pid=100287,containerID=container_1513249052998_0007_01_000009] is 
running beyond virtual memory limits. Current usage: 611.1 MB of 1 GB physical
memory used; 4.9 GB of 3 GB virtual memory used. Killing container.

```
但是后面任务可能还是可以正常的运行的
<!-- more -->
## 异常分析

611.1MB:  任务所占的物理内存

1GB 是mapreduce.map.memory.mb 设置的  

4.9G 是程序占用的虚拟内存：  [什么是虚拟内存以及和物理内存的关系](http://blog.csdn.net/moshenglv/article/details/52242153 )

3GB 是mapreduce.map.memory.db  乘以 yarn.nodemanager.vmem-pmem-ratio  得到的

其中yarn.nodemanager.vmem-pmem-ratio 是 **虚拟内存和物理内存比例**，在yarn-site.xml中设置，默认是2.1, 由于我本地设置的是3， 所以 1*3 = 3GB  

很明显，container占用了4.9G的虚拟内存，但是分配给container的却只有3GB。所以kill掉了这个container  

<font color=#be1a21 size=3 face="黑体">上面只是map中产生的报错，当然也有可能在reduce中报错</font>，如果是reduce中，那么就是mapreduce.reduce.memory.db * yarn.nodemanager.vmem-pmem-ratio
## 解决办法
主要是以下四个方面考虑  

- 取消虚拟内存的检查：  
在yarn-site.xml或者程序中中设置yarn.nodemanager.vmem-check-enabled为false  
```
<property>
  <name>yarn.nodemanager.vmem-check-enabled</name>
  <value>false</value>
  <description>Whether virtual memory limits will be enforced for containers.</description>
</property>
```
除了虚拟内存超了，也有可能是物理内存超了，同样也可以设置物理内存的检查为false： yarn.nodemanager.pmem-check-enabled 
个人认为这种办法并不太好，**如果程序有内存泄漏等问题，取消这个检查，可能会导致集群崩溃**。


- 增大mapreduce.map.memory.mb 或者 mapreduce.map.memory.mb  
个人觉得是一个办法，<font color=#be1a21 size=3 face="黑体">应该优先考虑这种办法，这种办法不仅仅可以解决虚拟内存，或许大多时候都是物理内存不够了，这个办法正好适用</font>
- 适当增大yarn.nodemanager.vmem-pmem-ratio的大小，一个物理内存增大多个虚拟内存， 但是这个参数也不能太离谱
- 如果任务所占用的内存太过离谱，<font color=#be1a21 size=3 face="黑体">更多考虑的应该是程序是否有内存泄漏</font>，是否存在数据倾斜等，优先程序解决此类问题

 
参考资料：  
https://stackoverflow.com/questions/14110428/am-container-is-running-beyond-virtual-memory-limits  
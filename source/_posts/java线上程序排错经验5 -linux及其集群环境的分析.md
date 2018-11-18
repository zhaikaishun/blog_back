
---
title: java线上程序排错经验5 -linux及其集群环境的分析
date: 2018-10-04 21:25:21
tags: [jvm,java]
categories: [java]
author: kaishun
id: 211
permalink: jvm06
---

## 1. top命令查看整体情况  
top命令和灵活，具体可自行搜索   
图转自： www.codesheep.cn
![top命令](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/linux-monitor/top-description.png)


## 2. 查看内存  
free -m
分析系统内存，看是否足够程序运行  


## 3. 磁盘占用情况  

### 3.1. 查看文件夹中各文件（夹）的大小
举例  
```
du -h --max-depth=1 /home/ubuntu/
```

### 3.2. 查看磁盘占用情况  
```
df -h
```
## 4. 查看磁盘IO  
```
dd if=/dev/zero of=dd.file bs=100M count=10 conv=fdatasync
```
上述命令会在当前目录生成一个dd.file文件，每次往这个文件写100M，一共写10次，conv=fdatasync表示实际写盘而不是用缓存，测试磁盘IO的时候，必须加上这个才准  


## 5 网络能否ping通  
ping ip  
如果是同一个集群系统，查看是否ping通，是否丢包情况，ping速度如何

## 6. 查看网卡以及测试集群中SCP复制速度  
ifconfig找到本机Ip，对应的即为本IP网卡  
假如你看到的网卡是 eth0， 查看网卡速度的命令为  
```
ethtool eth0  
```
有时候网卡会有问题，导致网络有问题，排除后，我们可以测试节点间数据复制情况  
scp 命令将上面生成的文件复制到另外一个机器，查看其速度

## 7 查看cpu报告与设备利用率报告  
iostat, linux需要先安装  
使用方法：  
iostat  
```
ubuntu@VM-0-12-ubuntu:~/sample/testio$ iostat
Linux 4.4.0-91-generic (VM-0-12-ubuntu)         09/16/2018      _x86_64_        (1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.63    0.01    0.41    0.34    0.00   98.62

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               1.86         5.31        15.11   27627271   78590297
scd0              0.00         0.00         0.00        208          0
```
**第一部分包含了CPU报告**  
- %user : 显示了在执行用户(应用)层时的CPU利用率    
- %nice : 显示了在以nice优先级运行用户层的CPU利用率    
- %system : 显示了在执行系统(内核)层时的CPU利用率   
- %iowait : 显示了CPU在I/O请求挂起时空闲时间的百分比    
- %steal : 显示了当hypervisor正服务于另外一个虚拟处理器时无意识地等待虚拟CPU所占有的时间百分比。  
- %idle : 显示了CPU在I/O没有挂起请求时空闲时间的百分比  
**第二部分包含了设备利用率报告**  
- Device : 列出的/dev 目录下的设备/分区名称    
- tps : 显示每秒传输给设备的数量。更高的tps意味着处理器更忙。  
- Blk_read/s : 显示了每秒从设备上读取的块的数量(KB,MB)  
- Blk_wrtn/s : 显示了每秒写入设备上块的数量(KB,MB)  
- Blk_read : 显示所有已读取的块  
- Blk_wrtn : 显示所有已写入的块  


## 8. 查看目前程序的IO情况  
iotop  
需要先安装  
使用方法  
(如果没设置权限，可能需要在root权限才能使用)
```
iotop
```
会显示
```
TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
```
其中  
-  DISK READ  DISK WRITE 读写速度  
- IO io占比  
- COMMAND 运行的命令  

未完待续： 以后会逐渐补充







  

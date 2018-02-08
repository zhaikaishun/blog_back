---
title: centos虚拟机网络配置NAT模式和桥接模式总结
date: 2017-08-29 21:25:21
tags: [linux]
categories: [linux]
author: kaishun
id: 101
permalink: linux-NAT-Bridge
---

每次装虚拟机，都需要配置网络IP，或者是装hadoop，zookeeper等各种集群服务器的时候，都需要配置网络，每次都还挺麻烦的，需要网上各种经验，本人也配置过很多次了，但是还是需要去网上找资料，还各种容易出问题，这次重装电脑，就做个笔记吧
<!-- more -->
## NAT模式
如果想玩NAT模式：  
先把该虚拟机设置成NAT模式： 虚拟机设置->网络适配器->右边选择NAT模式  

修改/etc/sysconfig/network-scripts/ifcfg-eth0的内容为:
```
DEVICE=eth0
HWADDR=00:0C:29:3D:7C:7A
TYPE=Ethernet
UUID=e204d9f9-c272-4bc2-80e7-6b0687aaad45
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=dhcp

NETMASK=255.255.255.0
IPV$_FALLURE_FATAL=yes
DNS1=8.8.8.8
IPADDR=192.168.221.128
PREFIX=24
GATEWAY=192.168.221.2
```  
上面的参数怎么设置的呢？ 
BOOTPROTO=dhcp  说明是动态的分配ip了  
DNS1=8.8.8.8  是必须这样设置的，这是一台位于美国的DNS服务器地址  
GATEWAY=192.168.221.2 这是怎么知道的呢？ VNware->编辑->虚拟网络编辑器->选择NAT模式这个->选择下面NAT设置就可以看到网关IP了   
![查看网关](http://or49tneld.bkt.clouddn.com/17-10-7/65903862.jpg)  

IPADDR怎么设置呢？ 上面那个图，还有个DHCP设置，点击可以看到开始IP地址和结束IP地址，在这两个IP中间随便选一个就好了  
![虚拟网络编辑2](http://or49tneld.bkt.clouddn.com/17-10-7/34561050.jpg)    

## /etc/resolv.conf的修改  
```
nameserver 8.8.8.8
```    
service network restart  尝试看看是否成功了呢？ 不成功再继续看其他文章怎么设置吧，可能自己本地也要设置IP共享之类的，具体网上找吧  

## 桥接模式  
桥接模式可以设置静态IP了，好处就是另外一台服务器，也可以通过这个ip连接到虚拟机中，在玩分布式的时候，多台电脑之间一起玩的时候还是很有用的。  
方法就是将NAT设置成桥接模式。  
先在本地cmd看一下自己的ip是多少，然后设置Linux的ifcfg-eth0的内容  
vim /etc/sysconfig/network-scripts/ifcfg-eth0

```
DEVICE=eth0
HWADDR=00:0C:29:3D:7C:7A
TYPE=Ethernet
UUID=e204d9f9-c272-4bc2-80e7-6b0687aaad45
ONBOOT=yes
NM_CONTROLLED=yes
#BOOTPROTO=dhcp
BOOTPROTO=static

NETMASK=255.255.255.0
BROADCAST=192.168.1.255
IPV4_FALLURE_FATAL=yes
DNS1=8.8.8.8
IPADDR=192.168.1.31
PREFIX=24
GATEWAY=192.168.1.1
```  

GATEWAY一般都是本地cmd，然后ipconfig查看的  
```
以太网适配器 本地连接:

   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::38a6:2
   IPv4 地址 . . . . . . . . . . . . : 192.168.1.10
   子网掩码  . . . . . . . . . . . . : 255.255.255.
   默认网关. . . . . . . . . . . . . : 192.168.1.1

```  
NETMASK=255.255.255.0和DNS1=8.8.8.8必须这样填了  
BOOTPROTO=static 说明是静态的获取ip地址  
IPADDR=192.168.1.31 这个是自己设置的，但要注意前三位和网关的前3位一样，并且这个不能和本地的ip重复  
BROADCAST=192.168.1.255 前3位和网关一样，最后一位设置成255  
HWADDR  这个就是mac地址了，一般不用改变，这个也可以这样找到：虚拟机设置->网络适配器->右下角高级，然后就可以看到MAC地址了 **注意： ** 克隆一个新的虚拟机的时候，ifcfg-eth0的这个HWADDR是要修改的哦，否则可能会有问题。  
  
然后 service network restart 就OK了。  
ping www.baidu.com 试一试? 若还不行，就再google或者百度吧  

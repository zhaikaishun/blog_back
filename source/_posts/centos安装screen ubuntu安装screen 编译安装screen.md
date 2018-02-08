---
title: centos安装screen ubuntu安装screen 编译安装screen
date: 2017-08-28 21:25:21
tags: [linux]
categories: [linux]
author: kaishun
id: 100
permalink: linux-screen-install
---

## yun安装：  
```
yum install screen  
```
## ubuntu 的 apt-get安装  
```
sudo apt-get update
sudo apt-get install screen
```
<!-- more -->
## 编译安装 
为什么我需要编译安装，因为我所操作的集群不能连外网  

tar.gz 下载地址：  
https://ftp.gnu.org/gnu/screen/  

解压：  
略

编译   
```
cd screen-4.6.2

./configure  

make  

```
安装, 这一步需要root权限  
```
make install  
install -m 644 etc/etcscreenrc /etc/screenrc
```
查看是否安装成功  
```
which screen  
出现  
/bin/screen 则成功
```

## 碰到的坑  
刚开始不敢直接上真实环境操作，所以先使用了一个VPS自己来使用编译安装的方式安装  

./config报错  
```
no acceptable C compiler found in $PATH
```
网上说是需要安装gcc套件（qqtt: 别问我怎么联网的）  
```
yum -y install gcc
```

后来还是另外一个错  
```
configure: error: !!! no tgetent - no screen  
```
真是日狗了，这时需要安装ncurses-devel  
```
yum -y install ncurses-devel
```
可能是VPS上面什么都没有安装吧，后来去集群环境上，上面的依赖都是有的，安装的很顺利。  



参考了官方的安装文档：  
http://www.gnu.org/software/screen/manual/screen.html#Compiling-Screen  
参考了国外的一篇文章： http://www.linuxfromscratch.org/blfs/view/cvs/general/screen.html  
强烈推荐看看第二篇文章，源码编译指定自己想要放置的路径，我这里设置的都是默认的路径，没这么复杂。  



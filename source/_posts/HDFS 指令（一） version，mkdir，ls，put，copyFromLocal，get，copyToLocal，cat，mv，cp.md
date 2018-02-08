---
title: HDFS 指令（一）version，mkdir，ls，put，copyFromLocal，get，copyToLocal，cat，mv，cp
date: 2016-05-17 23:13:21
tags: [hdfs]
categories: [大数据,hdfs]
author: kaishun
id: 3
permalink: hdfs-operator-1
---
# **1. 简介**  
本文主要简单介绍一些常用的hdfs，包括复制文件，修改文件权限，查看文件内容，改变文件的所有权，创建目录，等HDFS文件操作  
所有的Hadoop文件系统shell命令是由在bin / HDFS脚本调用  
![hdfs-commands](http://or49tneld.bkt.clouddn.com/18-1-23/63075597.jpg)
<!-- more -->
# **2. version  查看版本**
**指令用法**
```
hdfs version
```
例子
```
[hadoop@master ~]$ hdfs version
```
输出hadoop版本号  Hadoop 2.7.1

# **3. mkdir 创建目录**
**指令用法**
```
mkdir <path>
```
例子 , 创建testdir目录
```
[hadoop@master ~]$ hdfs dfs -mkdir /testdir
```


# **4. ls  查看目录文件**
能显示指定路径的文件或文件夹列表，显示每个条目 的名称，权限，拥有者，大小和修改日期，路径。  

**指令用法**
```
ls <path>

```
例子，查看 /test目录 (我之前创建了test目录，并且放了一个文件在里面)
```
[hadoop@master ~]$ hdfs dfs -ls  /test
```
输出
```
Found 1 items
-rw-r--r--   2 hadoop supergroup         14 2017-04-03 14:45 /test/slaves
```
# **5. put 复制文件到hdfs**  
可以复制本地文件到hdfs，也可以复制hdfs上的文件，到hdfs的另外一个地方
**指令用法**
```
put <localSrc> <dest>

```
例子，把hdfs.site文件放到 /test目录下
```
[hadoop@master hadoop]$ hdfs dfs -put /home/hadoop/MyCloudera/APP/hadoop/hadoop/etc/hadoop/hdfs-site.xml /test
```


# **6. copyFromLocal**  
和put很像，但是指定了只能是从本地复制到hdfs  
**指令用法**
```
copyFromLocal <localSrc> <dest>
```
例子，把hadoop-env.sh复制到 /test中
```
[hadoop@master hadoop]$ hdfs dfs -copyFromLocal /home/hadoop/MyCloudera/APP/hadoop/hadoop/etc/hadoop/hadoop-env.sh  /test
```

# **7. get 复制hdfs文件到本地**  
指令用法
```
get [-crc] <src> <localDest>

```
例如我将hdfs 上的文件拷贝到本地
```
[hadoop@master Downloads]$ hdfs dfs -get /test/slaves /home/hadoop/Downloads/
```

# **8. copyToLocal  复制hdfs文件到本地**
类似于get命令，唯一的区别是，在这个目的路径被限制到本地

**指令用法**
```
copyToLocal <src> <localDest>

```
例子, 复制slaves到本地,Downloads目录
```
[hadoop@master Downloads]$ hdfs dfs -copyToLocal /test/slaves /home/hadoop/Downloads/
```

# **9. cat 查看文件内容**
将文件的内容输出到console或者stdout控制台
**指令用法**
```
cat <file-name>

```
例如，查看/test/slaves文件
```
[hadoop@master Downloads]$ hdfs dfs -cat /test/slaves
```
控制台打印
```
master
node01
```

# **10. mv 移动**  
类似于linux的mv命令，移动hdfs中的一个地方到另外一个地方
**指定用法**
```
mv <src> <dest>

```
把/test/slaves 复制到 /testdir 
```
[hadoop@master Downloads]$ hdfs dfs -mv /test/slaves /testdir 
```

# **11. copy复制**
类似于linux中的cp方法，复制hdfs中的一个文件到另外一个文件目录

**指令用法**
```
cp <src> <dest>
```
例子
```
[hadoop@master Downloads]$ hdfs dfs -cp /testdir/slaves /test
```  
翻译原文：http://data-flair.training/blogs/top-hadoop-hdfs-commands-tutorial/

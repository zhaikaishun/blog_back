---
title: HDFS 指令（三）touchz，test，text，stat，appendToFile，checksum，count，chmod
date: 2016-05-19 23:13:21
tags: [hdfs]
categories: [大数据,hdfs]
author: kaishun
id: 7
permalink: hdfs-operator-3
blogexcerpt: 关键词：hdfs命令，touchz，test，text，stat，appendToFile，checksum，count，chmod. 本章目的. 在这个Hadoop HDFS命令教程中，我们将学习剩下一些重要并且经常使用的HDFS命令，借助这些命令，我们将能够执行HDFS文件操作，如复制文件，更改文件权限，查看文件内容，更改文件所有权，创建目录等。要了解有关世界上最可靠的存储层的更多信息，请参阅HDFS入门指南。  
---

关键词：hdfs命令，touchz，test，text，stat，appendToFile，checksum，count，chmod
# **本章目的**
在这个Hadoop HDFS命令教程中，我们将学习剩下一些重要并且经常使用的HDFS命令，借助这些命令，我们将能够执行HDFS文件操作，如复制文件，更改文件权限，查看文件内容，更改文件所有权，创建目录等。要了解有关世界上最可靠的存储层的更多信息，请参阅HDFS入门指南。  
![hdfs-commands](http://or49tneld.bkt.clouddn.com/18-1-23/63075597.jpg)

# **1. touchz**  
在指定目录创建一个新文件，如果文件存在，则创建失败
**命令用法**
```
touchz <path>
```
**例子**
```
hdfs dfs -touchz /user/dataflair/dir2
```

#  **2. test**  
测试hdfs中的某个文件或者目录是否存在  
-d: 如果测试的路径是一个文件夹, 则返回0，否则返回1。    
-e: 如果测试的路径存在, 则返回0，否则返回1。   
-f: 如果测试的路径是一个文件, 则返回0，否则返回1。    
-s: if 如果测试的路径不是空(文件夹下有文件或者文件夹), 则返回0，否则返回1。    
-z: 如果测试的是一个文件，并且这个文件不为空, 则返回0，否则返回1。
**命令用法**
```
hdfs dfs -test -[ezd] URI
```
**例子**
```
"hdfs dfs -test -e /test/file/sample
hdfs dfs -test -z /test/file/sample
hdfs dfs -test -d /test/file/sample
```

# **3. text**  
格式化输出文件的内容，允许的格式化包括zip,和 TextRecordInputStream  
**命令用法**
```
hdfs dfs -text <source>
```
**例子**
```
hdfs dfs -text /user/dataflair/dir1/sample
```

#  **4. stat**  
打印有关路径的信息，可以加下面的格式化输出      
%b: 文件大小  

%n: 文件名  

%o: 块大小  

%r: 副本个数  

%y, %Y: 修改日期.  
**命令用法**
```
hdfs dfs -stat [format] path

```
**例子**
```
hdfs dfs -stat /user/dataflair/dir1
hdfs fs -stat "%o %r" /user/dataflair/dir1
```



# **5. appendToFile**  
appendToFile命令是将一个或者多个文件添加到HDFS系统中，他也是从标准输入中读取，然后添加到目标文件系统汇总  
**命令用法**
```
hadoop fs -appendToFile <localsource> ... <dst>

```
**例子**
```
hadoop fs -appendToFile /home/dataflair/Desktop/sample /user/dataflair/dir1
```

# **6. checksum**  
Datanode在把数据实际存储之前会验证数据的校验和（checksum的初始值？）如果某个client在读取数据时检测到数据错误, 在抛出ChecksumException  
参考： http://blog.csdn.net/oh_mourinho/article/details/52524442
**命令用法**
```
hadoop fs -checksum URI
```
**例子**
```
hadoop fs -checksum /user/dataflair/dir1/sample
```

# **7. count**  
统计hdfs对应路径下的目录个数，文件个数，文件总计大小   
显示为目录个数，文件个数，文件总计大小，输入路径
**命令用法**
```
hdfs dfs -count [-q] <paths>
```
**例子**
```
hdfs dfs -count /user/dataflair
```

# **8. chmod**  
类似于linux，用来修改权限
**命令用法**
```
chmod [-R] mode,mode,... <path>...
```
**例子**
```
hdfs dfs -chmod 777 /user/dataflair/dir1/sample
```
翻译原文：http://data-flair.training/blogs/hadoop-hdfs-commands-tutorial/
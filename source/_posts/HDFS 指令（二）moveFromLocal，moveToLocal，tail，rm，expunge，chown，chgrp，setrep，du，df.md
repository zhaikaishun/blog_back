---
title: HDFS 指令（二）moveFromLocal，moveToLocal，tail，rm，expunge，chown，chgrp，setrep，du，df
date: 2016-05-17 23:13:21
tags: [hdfs]
categories: [大数据,hdfs]
author: kaishun
id: 6
permalink: hdfs-operator-2
blogexcerpt: 本文主要学习hadoop hdfs, 从hdfs移动到本地，从本地移动到hdfs，tail查看最后，rm删除文件，expunge清空 trash,chown, 改变拥有者，setrep 改变文件副本数，chgrp改变所属组，，du,df磁盘占用情况
---

# **目的**  
本文主要学习hadoop hdfs 从hdfs移动到本地，从本地移动到hdfs，tail查看最后，rm删除文件，expunge清空 trash,chown 改变拥有者，setrep 改变文件副本数，chgrp改变所属组，，du,df磁盘占用情况

![hdfs-commands](http://or49tneld.bkt.clouddn.com/18-1-23/63075597.jpg)

# **moveFromLocal**  
复制一份本地文件到hdfs，当成功后，删除本地文件
**指令用法**
```
moveFromLocal <localSrc> <dest>
```
**例子**
```
hdfs dfs -moveFromLocal /home/dataflair/Desktop/sample /user/dataflair/dir1
```

# **moveToLocal**  
类似于-get，但是当复制完成后，会删除hdfs上的文件
**指令用法**
```
moveToLocal <src> <localDest>
```
**例子**
```
hdfs dfs -moveToLocal /user/dataflair/dir2/sample /user/dataflair/Desktop
```

# **tail**  
类似于linux中的tail，把文件最后1kb给打印到console 或者 stdout.
**指令用法**
```
hdfs dfs -tail [-f] <filename>
```
**例子**
```
hdfs dfs -tail /user/dataflair/dir2/purchases.txt
hdfs dfs -tail -f /user/dataflair/dir2/purchases.txt
```

#  **rm**  
移除文件或者清空路径下的数据
**指令用法**  

```
rm <path>
```
**例子**

```java
hdfs dfs -rm /user/dataflair/dir2/sample
```
**递归删除**

```
hdfs dfs -rm -r /user/dataflair/dir2

```

#   **expunge**  
清空 trash  
HDFS中的数据删除也是比较有特点的，并不是直接删除，而是先放在一个类似回收站的地方（/trash），可供恢复  
对于用户或者应用程序想要删除的文件，HDFS会将它重命名并移动到/trash中  ，当被覆盖或者到了一定的生命周期（现在默认为6小时），hdfs才会从文件中删除，并且只有这个时候，Datanode上相关的磁盘空间才能节省出  
而 expunge 就类会清空/trash，类似于清空回收站
**指令用法**
```
hdfs dfs -expunge
```
**例子**
```
hdfs dfs -expunge

```

#   **chown**  
类似于linux中的chown，用户应该是超级用户才能做次操作  
改变文件拥有者
**指令用法**
```
hdfs dfs -chown [-R] [OWNER][:[GROUP]] URI [URI ]
```
**例子**
```
hdfs dfs -chown -R dataflair /opt/hadoop/logs
```

#   **chgrp**  
类似于linux中的chgrp，用户应该是超级用户才能做次操作  
改变文件所有组
**指令用法**
```
hdfs dfs -chgrp [-R] <NewGroupName> <file or directory name>
```
**例子**
```
hdfs dfs -chgrp [-R] New Group sample
```

#  **setrep** 
**指令用法** 
用来改变文件的副本数，如果是文件夹，那么次命令会针对该文件夹下的所有文件都会改变副本数  
-w 表示副本数改为多少
使用-R选项可以对一个目录下的所有目录+文件递归执行改变副本个数的操作  
```
setrep [-R] [-w] rep <path>
```
**例子**
```
hdfs dfs -setrep -w 3 /user/dataflair/dir1

```

#  **du** 
类似于linux中的du ， 统计个目录下各个文件大小，可以加-h 提高文件可读性
**指令用法**
```
du <path>
```
**例子**
```
hdfs dfs -du /user/dataflair/dir1/sample
```

#  **df** 
类似于linux中的du ，查询某个目录的空间大小，可以加-h 提高文件可读性
**指令用法**
```
hdfs dfs -df [-h] URI [URI ...]
```
**例子**
```
hdfs dfs -df -h
```

翻译原文：http://data-flair.training/blogs/most-used-hdfs-commands-tutorial-examples/
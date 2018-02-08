---
title: HDFS 指令（四）find,help,setfatter,truncate,usage
date: 2016-05-19 23:20:21
tags: [hdfs]
categories: [大数据,hdfs]
author: kaishun
id: 9
permalink: hdfs-operator-4
blogexcerpt: 关键字：hdfs命令,find,help,setfatter,truncate,usage 的用法,前言,在这个HDFS教程中，主要介绍了如何查看命令帮助，如何设置文件扩展属性，文件的截断 . 1. find, 找出能匹配上的所有文件, 不区分大小写，对大小写不敏感
---

关键字：hdfs命令,find,help,setfatter,truncate,usage 的用法
# **前言**
在这个HDFS教程中，主要介绍了 如何查看命令帮助，如何设置文件扩展属性，文件的截断
![hdfs-commands](http://or49tneld.bkt.clouddn.com/18-1-23/63075597.jpg)

# **1. find**
找出能匹配上的所有文件  
-name pattern 不区分大小写，对大小写不敏感
-iname pattern 对大小写敏感。
-print 打印。
-print0 打印在一行
**命令用法**
```
hadoop fs -find <path> ... <expression> ...
```
**例子**
```
hadoop fs -find /user/dataflair/dir1/ -name sample -print
```

# **2. help**
查看帮助文档
**命令用法**
```
hadoop fs -help
```
**例子**
```
hadoop fs -help
```

# **3. setfattr**
为一个文件或者文件夹设置一个扩展属性和值  
操作  
-b: 除了ACL的文件不移除，他移除所有的扩展属性。所有文件信息都保留给用户，组和其他用户
-n name: 显示扩展的属性的名.
-v value: 显示扩展的属性的值。
-x name: 移除所有的扩展属性
path: 文件或文件夹.
**命令用法**
```
hadoop fs -setfattr -n name [-v value] | -x name <path>
```
**例子**
```
hdfs dfs -setfattr -n user.myAttr -v myValue /user/dataflair/dir2/purchases.txt
hdfs dfs -setfattr -n user.noValue /user/dataflair/dir2/purchases.txt
hdfs dfs -setfattr -x user.myAttr /user/dataflair/dir2/purchases.txt
```

# **4. truncate ** 
参考文章：http://blog.csdn.net/androidlushangderen/article/details/52651995
类似于linux的truncate
**命令用法**
```
hadoop fs -truncate [-w] <length> <paths>
```
**例子**
```
hadoop fs -truncate 55  /user/dataflair/dir2/purchases.txt  /user/dataflair/dir1/purchases.txt
hadoop fs -truncate -w 127 /user/dataflair/dir2/purchases.txt
```

# **5. usage**
返回某个命令的帮助
**命令用法**
```
hadoop fs -usage command
```
**例子**
```
hadoop fs -usage mkdir
```

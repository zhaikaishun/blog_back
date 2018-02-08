---
title: Linux 常见的查找相关笔记
date: 2017-05-05 21:25:21
tags: [linux]
categories: [Linux]
author: kaishun
id: 56
permalink: linux-find-xargs
---
 
## 查找文件名  
 find -name filename 
 
 1. 按照文件名查找
```
find / -name httpd.conf　　#在根目录下查找文件httpd.conf，表示在整个硬盘查找
find /etc -name httpd.conf　　#在/etc目录下文件httpd.conf
find /etc -name '*srm*'　　#使用通配符*(0或者任意多个)。表示在/etc目录下查找文件名中含有字符串‘srm’的文件
find . -name 'srm*' 　　#表示当前目录下查找文件名开头是字符串‘srm’的文件
```
<!-- more -->
2. 按照文件特征查找 
```
find / -amin -10 　　# 查找在系统中最后10分钟访问的文件(access time)
find / -atime -2　　 # 查找在系统中最后48小时访问的文件
find / -empty 　　# 查找在系统中为空的文件或者文件夹
find / -group cat 　　# 查找在系统中属于 group为cat的文件
find / -mmin -5 　　# 查找在系统中最后5分钟里修改过的文件(modify time)
find / -mtime -1 　　#查找在系统中最后24小时里修改过的文件
find / -user fred 　　#查找在系统中属于fred这个用户的文件
find / -size +10000c　　#查找出大于10000000字节的文件(c:字节，w:双字，k:KB，M:MB，G:GB)
find / -size -1000k 　　#查找出小于1000KB的文件
```
3. 使用混合查找方式查找文件  
**参数有： ！，-and(-a)，-or(-o)。**
```
find /tmp -size +10000c -and -mtime +2 　　#在/tmp目录下查找大于10000字节并在最后2分钟内修改的文件
find / -user fred -or -user george 　　#在/目录下查找用户是fred或者george的文件文件
find /tmp ! -user panda　　#在/tmp目录中查找所有不属于panda用户的文件
```

## 查找包含字符串的文件    
 
 grep  "string"  ./*  
 “string"为待查找串  ， ./* 表示当前目录下所有文件
**二、grep命令**

　　　  基本格式：find  expression

 　　　 1.主要参数

　　　　[options]主要参数：  
　　　　－c：只输出匹配行的计数。  
　　　　－i：不区分大小写  
　　　　－h：查询多文件时不显示文件名。  
　　　　－l：查询多文件时只输出包含匹配字符的文件名。  
　　　　－n：显示匹配行及行号。  
　　　　－s：不显示不存在或无匹配文本的错误信息。  
　　　　－v：显示不包含匹配文本的所有行。  

　　　　pattern正则表达式主要参数：  
　　　　\： 忽略正则表达式中特殊字符的原有含义。  
　　　　^：匹配正则表达式的开始行。  
　　　　$: 匹配正则表达式的结束行。  
　　　　\<：从匹配正则表达 式的行开始。  
　　　　\>：到匹配正则表达式的行结束。  
　　　　[ ]：单个字符，如[A]即A符合要求 。  
　　　　[ - ]：范围，如[A-Z]，即A、B、C一直到Z都符合要求 。  
　　　　.：所有的单个字符。  
　　　　* ：有字符，长度可以为0。  

**2.实例**　   

　　(1)grep 'test' d*　　#显示所有以d开头的文件中包含 test的行  
　　(2)grep ‘test’ aa bb cc 　　 #显示在aa，bb，cc文件中包含test的行  
　　(3)grep ‘[a-z]\{5\}’ aa   　　#显示所有包含每行字符串至少有5个连续小写字符的字符串的行  
　　(4)grep magic /usr/src　　#显示/usr/src目录下的文件(不含子目录)包含magic的行  
　　(5)grep -r magic   /usr/src　　#显示/usr/src目录下的文件(包含子目录)包含magic的行  

　　(6)grep -w pattern files   ：只匹配整个单词，而不是字符串的一部分(如匹配’magic’，而不是’magical’)，  
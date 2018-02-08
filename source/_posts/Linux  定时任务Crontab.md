---
title: Linux  定时任务Crontab
date: 2017-05-06 21:25:21
tags: [linux,crontab]
categories: [Linux]
author: kaishun
id: 58
permalink: linux-crontab
blogexcerpt: Crontab是Linux系统中在固定时间执行某一个程序的工具，类似于Windows系统中的任务计划程序，在一些自动化的程序或者运维中，起到很大的作用，在很多数据处理方面，例如定时备份数据库，定时指定某些任务等等，用linux的Crontab往往比较方便，下面介绍如何安装与使用crontab

---

# Crontab 介绍
Crontab是Linux系统中在固定时间执行某一个程序的工具，类似于Windows系统中的任务计划程序，在一些自动化的程序或者运维中，起到很大的作用，在很多数据处理方面，例如定时备份数据库，定时指定某些任务等等，用linux的Crontab往往比较方便，下面介绍如何安装与使用crontab
# crontab安装
## 检查是否已经安装，若已经安装那么不需要重复安装
先查看crontab有没有安装，有的话就不需要安装了,==root用户下==
```
[root@master crontest]# service crond status  
若没有出现status说明已经安装
```  
若没有安装，则按以下步骤进行
## 安装
```shell
yum -y install vixie-cron
yum -y install crontabs
```
说明：
vixie-cron 软件包是 cron 的主程序；
crontabs 软件包是用来安装、卸装、或列举用来驱动 cron 守护进程的表格的程序。  
## 配置
```linux
service crond start     //启动服务
service crond stop      //关闭服务
service crond restart   //重启服务
service crond reload    //重新载入配置
service crond status    //查看crontab服务状态
chkconfig --level 345 crond on  //在CentOS系统中加入开机自动启动: 
```
## 添加定时器任务  
```
crontab -e
```
如果是root用户，可以直接编写下面的  
```
/etc/crontab
```

## cron文件语法:  
下面是打开/etc/crontab显示的内容  
```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
# */1 * * * * hadoop /home/hadoop/crontest/testcronmkdir.sh
```

分     时    日       月       周    用户  命令  
注意最后一个是周，不是年！！！  ,/etc/crontab 需要指定用户, crontable -e就没有  
 0-59   0-23   1-31   1-12     0-6     command     (取值范围,0表示周日一般一行对应一个任务)  
 
下面有一些特殊符号需要记住，通过这些特殊符号，可以很灵活的设置定时任务   
```
“*”代表取值范围内的数字,
“/”代表”每”,
“-”代表从某个数字到某个数字,
“,”分开几个离散的数字
```
如果把最后的命令换做成脚本，例如我上面的那个，记得必须得是.sh脚本，那就需要在最前面引入  
```
#!/bin/bash
```
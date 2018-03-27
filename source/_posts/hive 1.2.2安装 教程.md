---
title: hive 1.2.2安装 教程
date: 2017-07-1 21:25:21
tags: [hive,大数据]
categories: [大数据]
author: kaishun
id: 76
permalink: hive-1.2.2-install
blogexcerpt: 关键字hive 1.2.2 安装，hive配置.安装好了hadoop，hadoop的集群搭建, http://blog.csdn.net/t1dmzks/article/details/68958562 , 安装好了mysql,mysql的编译安装（yum安装） http://blog.csdn.net/t1dmzks/article/details/71374740 .下载并且配置环境
---

关键字：hive 1.2.2 安装，hive配置
# 安装前提：
1.  安装好了hadoop，hadoop的集群搭建：http://blog.csdn.net/t1dmzks/article/details/68958562  
2.  安装好了mysql： mysql的编译安装（yum安装） http://blog.csdn.net/t1dmzks/article/details/71374740

<!-- more -->
# 下载并且配置环境
http://mirrors.hust.edu.cn/apache/hive/   进入hive-1.2.2/  下载 apache-hive-1.2.2-bin.tar.gz

```
[root@master Downloads]# cp apache-hive-1.2.2-bin.tar.gz /usr/local/

[root@master Downloads]# cd /usr/local

[root@master local]# tar zxvf apache-hive-1.2.2-bin.tar.gz

[root@master hadoop]# vim /etc/profile

# set hive environment
export HIVE_HOME=/usr/local/apache-hive-1.2.2-bin
export PATH=$PATH:$HIVE_HOME/bin
```

# mysql 的配置以及权限
mysql安装： http://blog.csdn.net/t1dmzks/article/details/71374740   
mysql配置hive表
```
mysql> CREATE DATABASE hive;   # 创建hive表

mysql> grant all on hive.* to hive@'%' identified by 'shun';   #给hive的权限，shun是我数据库的密码
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> grant all on hive.* to hive@localhost identified by 'shun';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

```

# 配置hive
## 先复制配置文件
```
[root@master apache-hive-1.2.2-bin]# cd $HIVE_HOME/conf/
[root@master conf]# cp hive-default.xml.template hive-site.xml

[root@master conf]# cp hive-default.xml.template hive-site.xml
[root@master conf]# cp hive-env.sh.template hive-env.sh
[root@master conf]# cp hive-exec-log4j.properties.template  hive-exec-log4j.properties
[root@master conf]# cp hive-log4j.properties.template hive-log4j.properties


```
## 创建一些目录 hdfs上的目录  
下面的配置需要用到这些目录
```
[hadoop@node01 ~]$ hadoop fs -mkdir -p /hive/warehouse
[hadoop@master ~]$ hadoop fs -mkdir -p /hive/logs
[hadoop@master ~]$ hadoop fs -mkdir -p /hive/tmp
```
## 创建本地的目录 
下面的配置需要用到这些目录

```
[root@master bin]# mkdir -p /hive/logs
[root@master bin]# mkdir -p /hive/exec
[root@master bin]# mkdir -p /hive/downloadedsource
[root@master /]# chown hadoop:hadoop /hive/ -R
```


## 配置hive-site.xml
hive-ite.xml中的配置很多，我们需要修改的是下面的这些，下面的description就能看出来配置的作用
```
<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
  <description>Driver class name for a JDBC metastore</description>
</property>
<property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hive</value>
    <description>Username to use against metastore database</description>
</property>

 <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>shun</value>
    <description>password to use against metastore database</description>
  </property>
  
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://master:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false</value>
    <description>JDBC connect string for a JDBC metastore</description>
  </property>
  
   <property>
    <name>hive.metastore.uris</name>
    <value/>  //这里我是默认的，没变
    <description>Thrift URI for the remote metastore. Used by metastore client to connect to remote metastore.</description>  
  </property>
  
  <property>
    <name>hive.metastore.warehouse.dir</name>
    <value>/hive/warehouse</value>   //到时候需要在hdfs上建立想要的目录
    <description>location of default database for the warehouse，Hive在HDFS上的根目录</description>
  </property>
  
    <property>
    <name>hive.exec.local.scratchdir</name>
    <value>/hive/exec</value>
    <description>Local scratch space for Hive jobs</description>
  </property>
  
    <property>
    <name>hive.downloaded.resources.dir</name>
    <value>/hive/downloadedsource</value>
    <description>Temporary local directory for added resources in the remote file system.</description>
  </property>
```

## 配置log4j.property
修改hive.log.dir为
```
hive.log.dir=/hive/logs
```

# 下载mysql驱动包
```
[root@master conf]# cd $HIVE_HOME/lib
[root@master lib]# wget http://central.maven.org/maven2/mysql/mysql-connector-java/5.1.38/mysql-connector-java-5.1.38.jar
```  

# 启动  
进入到 bin 目录下，用hadoop用户启动

先启动  
```
./hive --service metastore &
```
再另外开一个终端启动hive

```
[hadoop@master bin]$ hive
```
显示
```
[hadoop@master bin]$ hive

Logging initialized using configuration in file:/usr/local/apache-hive-1.2.2-bin/conf/hive-log4j.properties
hive> 
```
若有问题就尽量按照提示，可以参考google，baidu 等
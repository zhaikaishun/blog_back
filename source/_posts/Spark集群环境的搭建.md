---
title: Spark集群环境的搭建
date: 2016-09-05 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 31
permalink: spark-cluster-install
---


前提：已经安装好了hadoop, 说明，这次安装的是spark-1.4.0-bin-hadoop2.6.tgz版本，这个版本是兼容hadoop2.7的hdfs和yarn的
## 软件准备:   
scala-2.10.4 ,[下载地址](http://www.scala-lang.org/)  
spark-1.4.0-bin-hadoop2.6.tgz [下载地址](http://spark.apache.org/downloads.html)
## 1. 安装scala-2.10.4
将下载好的scala-2.11.4.tgz复制到/usr/lib  
<!-- more -->
### 1.1 解压安装包
```shell
sudo tar -zxf scala-2.11.4.tgz
```  
### 1.2 设置scala环境变量
```shell
sudo vim /etc/profile
```   
打开之后在末尾添加
```shell
#scale
export SCALA_HOME=/usr/lib/scala-2.10.4
export PATH=$SCALA_HOME/bin:$PATH
```
使配置生效
```shell
source /etc/profile
```
### 1.3 检查是否安装成功
```shell
scala -version
```
出现Scala code runner version 2.10.4 -- Copyright 2002-2013, LAMP/EPFL说明安装成功  

## 2. 安装spark-1.4.0-bin-hadoop2.6
将下载的 spark-1.4.0-bin-hadoop2.6.tar.gz 复制到/usr目录  
### 2.1 解压并且改名
```shell
sudo tar zxvf spark-1.4.0-bin-hadoop2.6.tgz
sudo mv spark-1.4.0-bin-hadoop2.6/ spark-1.4.0-hadoop2.6
```
### 2.2 配置环境
```shell
sudo vi /etc/profile
``` 
打开之后在末尾添加
```shell
#spark
export SPARK_HOME=/usr/spark-1.4.0-hadoop2.6
export PATH=$SPARK_HOME/bin:$PATH
```  
使资源生效
```
source /etc/profile
```  
### 2.3 配置Spark环境
#### 配置 spark-env.sh文件
进入到 spark的conf目录, 拷贝一份spark-env.sh.template  并且改名为 spark-env.sh
```shell
cd $SPARK_HOME/conf
sudo cp spark-env.sh.template spark-env.sh
```
sudo vi spark-env.sh 添加以下内容：
```shell
export JAVA_HOME=/usr/local/java/jdk1.7.0_79
export HADOOP_HOME=/home/hadoop/MyCloudera/APP/hadoop/hadoop
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export SCALA_HOME==/usr/lib/scala-2.10.4
export SPARK_HOME=/usr/spark-1.4.0-hadoop2.6
export SPARK_MASTER_IP=master
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_WEBUI_PORT=8099

#每个Worker使用的CPU核数
export SPARK_WORKER_CORES=1
#每个Slave中启动几个Worker实例
export SPARK_WORKER_INSTANCES=1
#每个Worker使用多大的内存
export SPARK_WORKER_MEMORY=2G
#Worker的WebUI端口号
export SPARK_WORKER_WEBUI_PORT=8081
#每个Executor使用使用的核数
export SPARK_EXECUTOR_CORES=1
#每个Executor使用的内存
export SPARK_EXECUTOR_MEMORY=1G

export SPARK_CLASSPATH=$SPARK_HOME/conf/:$SPARK_HOME/lib/*
export SPARK_CLASSPATH=$SPARK_CLASSPATH:$CLASSPATH
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:$HADOOP_HOME/lib/nativ
```
#### 配置Slave文件  
拷贝一份slaves.template并且改名为slaves  
```shell
sudo cp slaves.template slaves
```  
sudo vim slaves，然后根据自己对节点的设置，按照下面类似的方法添加内容
```shell
master
node01
node02
```  
### 2.4 把配置好的，所有文件，复制一份到node节点的服务器上
```shell
建议先打包，然后使用scp传到所有的node节点，然后在所有的node的节点的/usr/目录下解压
```

### 2.5 启动Spark并且测试
启动Spark(前提是先启动hadoop，以后会用到其hdfs的系统) ,先启动master,再启动slave， 操作全部在主节点进行, 或者直接全部启动  
如果启动失败，根据提示进行处理
```shell
#方法一:
cd $SPARK_HOME/sbin/
./start-master.sh
./start-slaves.sh
#方法二
./start-all.sh
```
启动 master机器jps有以下显示,相对于hadoop, 多了一个master,一个worker

```
14930 ResourceManager
16437 Jps
15033 NodeManager
16324 Worker
14510 DataNode
16166 Master
14753 SecondaryNameNode
14409 NameNode
```
slave机器jps有如下显示，多了一个worker
```
2610 NodeManager
2816 Worker
2888 Jps
2466 DataNode
```


使用spark shell进行测试
在hadoop用户下，进入spark 的bin目录，打开spark-shell命令编程
```shell 
cd $SPARK_HOME/bin
spark-shell 
打开后可以看到一个大大的Spark的欢迎图和版本，说明启动成功
```  
开始测试
```
val text= sc.parallelize(Seq("aa","aa","bb","cc"))
#回车可以看到 text: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[0] at parallelize at <console>:21
```
我们看一看其第一个元素
```
text.take(1)
#回车后最后出现一堆info信息，最后一行出现 res0: Array[String] = Array(aa)
```
我们统计其元组数量
```
text.count
#回车后出现 res1: Long = 4
```
过滤，统计不含aa的元组数量
```
# 注意这条命令的 > 和！之间要有一个空格
text.filter(r => !r.contains("aa")).count
#回车后出现
res3: Long = 2
```  

如果想自己写相应的spark测试，可以参考这篇文章，这篇文章使用的是idea的编译器，eclipse的我没用过，估计除了打包这里不一样，其他都是一样的  
[Spark Idea搭建实战](http://www.cnblogs.com/shishanyuan/p/4721120.html)

官网给了很多例子，均在 spark-1.4.0-bin-hadoop2.7\examples\src\main 文件夹下，可做参考

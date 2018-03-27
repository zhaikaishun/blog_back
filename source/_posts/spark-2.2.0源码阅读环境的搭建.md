---
title: spark-2.2.0源码阅读环境的搭建
date: 2017-05-28 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 64
permalink: spark-sourcecode-install
blogexcerpt: 下载源码 去[官网](http://spark.apache.org/downloads.html). 解压后，用idea打开, pom maven需要下载很多包，并且加载，需要等待一下，最好建议改成阿里云的依赖仓库，这样比较快。不改也可以,找到example包中的SparkPi类，setMaster后运行这个类.运行的时候肯定会报错，很多情况下就是找不到类。spark源码放在idea中还是比较坑的啊，真是麻烦，但是一般也就下面两种情况，安装下面的方法就可以解决了
---


## 下载源码  
去[官网](http://spark.apache.org/downloads.html) 下载 [spark-2.2.0.tgz](https://d3kbcqa49mib13.cloudfront.net/spark-2.2.0.tgz)

## 解压后，用idea打开
pom maven需要下载很多包，并且加载，需要等待一下，最好建议改成阿里云的依赖仓库，这样比较快。不改也可以
<!-- more -->
## 找到example包中的SparkPi类，setMaster后运行这个类
运行的时候肯定会报错，很多情况下就是找不到类。spark源码放在idea中还是比较坑的啊，真是麻烦，但是一般也就下面两种情况，安装下面的方法就可以解决了
## 报错SparkFlumeProtocol 找不到

在intellij ieda里面： 
- 打开View -> Tool Windows -> Maven Projects 
- 右击Spark Project External Flume Sink 
- 点击Generate Sources and Update Folders 
随后，Intellij IDEA会自动下载Flume Sink相关的包

然后重新build -> Make Project，一切ok!!

This should generate source code from sparkflume.avdl. 
Generate Sources and Update Folders do can resolve type SparkFlumeProtocol not found issue.
来源： http://apache-spark-developers-list.1001551.n3.nabble.com/A-Spark-Compilation-Question-td8402.html  

## 报错 SparkSqlParser.包找不到
  
- IntelliJ 下载并且安装 antlr4插件
- 打开View -> Tool Windows -> Maven Projects 
- 右击park Project Catalyst
- 点击Generate sources and update folders
- 


## 运行java的sparkpi的时候报这个错 

```
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/spark/SparkConfat org.apache.spark.examples.JavaSparkPi.main(JavaSparkPi.java:41)  
```
实在找不到问题，编译不报错，运行的时候就报错，肯定是少了某个包，估计是少了scala的包，然后去运行scala的LocalPi这个scala类，哇，这个都运行不了,报scala错误，搞了好久，难道在玩我。
后来自己写个scala的类，也报错，OK ，那确定是scala没引入。最后解决办法就是给加上scala的包了: 
file->Project Structure->对这个example的项目加上scala的lib，然后再次运行~, OK 没问题，环境也就搭建好了


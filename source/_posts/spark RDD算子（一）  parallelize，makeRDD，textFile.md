---
title: spark RDD算子（一）  parallelize，makeRDD，textFile
date: 2017-03-01 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 34
permalink: spark-rdd-1
---


## parallelize  
调用SparkContext 的 parallelize()，将一个存在的集合，变成一个RDD，这种方式试用于学习spark和做一些spark的测试
**scala版本**    
def parallelize[T](seq: Seq[T], numSlices: Int = defaultParallelism)(implicit arg0: ClassTag[T]): RDD[T]  
- 第一个参数一是一个 Seq集合
- 第二个参数是分区数
- 返回的是RDD[T]
```scala
scala> sc.parallelize(List("shenzhen", "is a beautiful city"))
res1: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[1] at parallelize at <console>:22
```  
<!-- more -->
**java版本**     
def parallelize[T](list : java.util.List[T], numSlices : scala.Int) : org.apache.spark.api.java.JavaRDD[T] = { /* compiled code */ } 
- 第一个参数是一个List集合
- 第二个参数是一个分区，可以默认
- 返回的是一个JavaRDD[T]
java版本只能接收List的集合
```java
JavaRDD<String> javaStringRDD = sc.parallelize(Arrays.asList("shenzhen", "is a beautiful city"));
```  

## makeRDD
只有scala版本的才有makeRDD  
def makeRDD[T](seq : scala.Seq[T], numSlices : scala.Int = { /* compiled code */ })  
跟parallelize类似
```scala
sc.makeRDD(List("shenzhen", "is a beautiful city"))
```

## textFile  
调用SparkContext.textFile()方法，从外部存储中读取数据来创建 RDD  
例如在我本地F:\dataexample\wordcount\input下有个sample.txt文件，文件随便写了点内容，我需要将里面的内容读取出来创建RDD  
**scala版本**
```scala
var lines = sc.textFile("F:\\dataexample\\wordcount\\input") 

```
**java版本**
```java
 JavaRDD<String> lines = sc.textFile("F:\\dataexample\\wordcount\\input");
```  
注: textFile支持分区，支持模式匹配，例如把F:\\dataexample\\wordcount\\目录下inp开头的给转换成RDD
```scala
var lines = sc.textFile("F:\\dataexample\\wordcount\\inp*")
```
多个路径可以使用逗号分隔，例如
```scala
var lines = sc.textFile("dir1,dir2",3)
```
---
title: spark RDD算子（六）之键值对聚合操作reduceByKey，foldByKey，排序操作sortByKey
date: 2017-03-06 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 39
permalink: spark-rdd-6
---

# **reduceByKey**
```
def reduceByKey(func: (V, V) => V): RDD[(K, V)]

def reduceByKey(func: (V, V) => V, numPartitions: Int): RDD[(K, V)]

def reduceByKey(partitioner: Partitioner, func: (V, V) => V): RDD[(K, V)]
```
接收一个函数，按照相同的key进行reduce操作，类似于scala的reduce的操作  
例如RDD {(1, 2), (3, 4), (3, 6)}进行reduce  
<!-- more -->
**scala版本**
```scala
    var mapRDD = sc.parallelize(List((1,2),(3,4),(3,6)))
    var reduceRDD = mapRDD.reduceByKey((x,y)=>x+y)
    reduceRDD.foreach(x=>println(x))
------输出---------
(1,2)
(3,10)
```
再举例  
**单词计数**  
F:\sparktest\sample.txt中的内容如下
```
aa bb cc aa aa aa dd dd ee ee ee ee 
ff aa bb zks
ee kks
ee  zz zks

```
**scala版本**

```scala
    val lines = sc.textFile("F:\\sparktest\\sample.txt")
    val wordsRDD = lines.flatMap(x=>x.split("\\s+")).map(x=>(x,1))
    val wordCountRDD = wordsRDD.reduceByKey((x,y)=>x+y)
    wordCountRDD.foreach(x=>println(x))
---------输出-----------
(ee,6)
(aa,5)
(dd,2)
(zz,1)
(zks,2)
(kks,1)
(ff,1)
(bb,2)
(cc,1)
```
**java版本**
```java
 JavaRDD<String> lines = sc.textFile("F:\\sparktest\\sample.txt");

        JavaPairRDD<String, Integer> wordPairRDD = lines.flatMapToPair(new PairFlatMapFunction<String, String, Integer>() {
            @Override
            public Iterable<Tuple2<String, Integer>> call(String s) throws Exception {
                ArrayList<Tuple2<String, Integer>> tpLists = new ArrayList<Tuple2<String, Integer>>();
                String[] split = s.split("\\s+");
                for (int i = 0; i <split.length ; i++) {
                    Tuple2 tp = new Tuple2<String,Integer>(split[i], 1);
                    tpLists.add(tp);
                }
            return tpLists;
            }
        });

        JavaPairRDD<String, Integer> wordCountRDD = wordPairRDD.reduceByKey(new Function2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer i1, Integer i2) throws Exception {
                return i1 + i2;
            }
        });
        Map<String, Integer> collectAsMap = wordCountRDD.collectAsMap();
        for (String key:collectAsMap.keySet()) {
            System.out.println("("+key+","+collectAsMap.get(key)+")");
        }
----------输出-------------------------------
(kks,1)
(ee,6)
(bb,2)
(zz,1)
(ff,1)
(cc,1)
(zks,2)
(dd,2)
(aa,5)
```
# **foldByKey**  
```scala
def foldByKey(zeroValue: V)(func: (V, V) => V): RDD[(K, V)]

def foldByKey(zeroValue: V, numPartitions: Int)(func: (V, V) => V): RDD[(K, V)]

def foldByKey(zeroValue: V, partitioner: Partitioner)(func: (V, V) => V): RDD[(K, V)]

```
该函数用于RDD[K,V]根据K将V做折叠、合并处理，其中的参数zeroValue表示先根据映射函数将zeroValue应用于V,进行初始化V,再将映射函数应用于初始化后的V.  
foldByKey可以参考我之前的[scala的fold的介绍](http://blog.csdn.net/t1dmzks/article/details/69858060#t27)  
与reduce不同的是 foldByKey开始折叠的第一个元素不是集合中的第一个元素，而是传入的一个元素  
[参考LXW的博客  scala的例子](http://lxw1234.com/archives/2015/07/358.htm)  
**直接看例子**
```scala
scala> var rdd1 = sc.makeRDD(Array(("A",0),("A",2),("B",1),("B",2),("C",1)))
scala> rdd1.foldByKey(0)(_+_).collect
res75: Array[(String, Int)] = Array((A,2), (B,3), (C,1)) 
//将rdd1中每个key对应的V进行累加，注意zeroValue=0,需要先初始化V,映射函数为+操
//作，比如("A",0), ("A",2)，先将zeroValue应用于每个V,得到：("A",0+0), ("A",2+0)，即：
//("A",0), ("A",2)，再将映射函数应用于初始化后的V，最后得到(A,0+2),即(A,2)
```
**再看：**
```scala
scala> rdd1.foldByKey(2)(_+_).collect
res76: Array[(String, Int)] = Array((A,6), (B,7), (C,3))
//先将zeroValue=2应用于每个V,得到：("A",0+2), ("A",2+2)，即：("A",2), ("A",4)，再将映射函
//数应用于初始化后的V，最后得到：(A,2+4)，即：(A,6)
```
**再看乘法操作：**  
```scala
scala> rdd1.foldByKey(0)(_*_).collect
res77: Array[(String, Int)] = Array((A,0), (B,0), (C,0))
//先将zeroValue=0应用于每个V,注意，这次映射函数为乘法，得到：("A",0*0), ("A",2*0)，
//即：("A",0), ("A",0)，再将映射函//数应用于初始化后的V，最后得到：(A,0*0)，即：(A,0)
//其他K也一样，最终都得到了V=0
 
scala> rdd1.foldByKey(1)(_*_).collect
res78: Array[(String, Int)] = Array((A,0), (B,2), (C,1))
//映射函数为乘法时，需要将zeroValue设为1，才能得到我们想要的结果。
```

# **SortByKey**
```
  def sortByKey(ascending : scala.Boolean = { /* compiled code */ }, numPartitions : scala.Int = { /* compiled code */ }) : org.apache.spark.rdd.RDD[scala.Tuple2[K, V]] = { /* compiled code */ }

```
SortByKey用于对pairRDD按照key进行排序，第一个参数可以设置true或者false，默认是true
**scala例子**
```scala
scala> val rdd = sc.parallelize(Array((3, 4),(1, 2),(4,4),(2,5), (6,5), (5, 6)))  

// sortByKey不是Action操作，只能算是转换操作
scala> rdd.sortByKey()
res9: org.apache.spark.rdd.RDD[(Int, Int)] = ShuffledRDD[28] at sortByKey at <console>:24 

//看看sortByKey后是什么类型
scala> rdd.sortByKey().collect() 
res10: Array[(Int, Int)] = Array((1,2), (2,5), (3,4), (4,4), (5,6), (6,5)) 

//降序排序
scala> rdd.sortByKey(false).collect() 
res12: Array[(Int, Int)] = Array((6,5), (5,6), (4,4), (3,4), (2,5), (1,2)) 
```  
java例子也是一样的，这里就不写了



参考文章: Spark快速大数据分析,[LXW的大数据田地](http://lxw1234.com/archives/2015/07/358.htm),spark官网  

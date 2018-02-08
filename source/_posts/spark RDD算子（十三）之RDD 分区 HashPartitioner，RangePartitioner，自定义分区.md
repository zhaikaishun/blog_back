---
title: spark RDD算子（十三）之RDD 分区 HashPartitioner，RangePartitioner，自定义分区
date: 2017-04-22 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 48
permalink: spark-rdd-13
---

**关键字**: spark分区方式，java HashPartitioner分区，scala HashPartitioner分区， java RangePartitioner 分区，scala RangePartitioner分区， java 自定义分区，scala自定义分区 
<!-- more -->
#  **默认分区和HashPartitioner分区**   
默认的分区就是HashPartition分区,默认分区不再介绍，下面介绍HashPartition的使用

通过上一章 [mapPartitionsWithIndex的例子](http://blog.csdn.net/t1dmzks/article/details/71336119#t1)，我们可以构建一个方法，用来查看RDD的分区  
```scala
    def mapPartIndexFunc(i1:Int,iter: Iterator[(Int,Int)]):Iterator[(Int,(Int,Int))]={
      var res = List[(Int,(Int,Int))]()
      while(iter.hasNext){
        var next = iter.next()
        res=res.::(i1,next)
      }
      res.iterator
    }
    def printRDDPart(rdd:RDD[(Int,Int)]): Unit ={
      var mapPartIndexRDDs = rdd.mapPartitionsWithIndex(mapPartIndexFunc)
      mapPartIndexRDDs.foreach(println( _))
    }
```  
**HashPartitioner分区  scala**  
使用pairRdd.partitionBy(new spark.HashPartitioner(n)), 可以分为n个区
```scala
    var pairRdd = sc.parallelize(List((1,1), (1,2), (2,3), (2,4), (3,5), (3,6),(4,7), (4,8),(5,9), (5,10)))
    //未分区的输出
    printRDDPart(pairRdd)
    println("=========================")
    val partitioned = pairRdd.partitionBy(new spark.HashPartitioner(3))
    //分区后的输出
    printRDDPart(partitioned)
-----------输出------------
(0,(5,10))
(0,(5,9))
(0,(4,8))
(0,(4,7))
(0,(3,6))
(0,(3,5))
(0,(2,4))
(0,(2,3))
(0,(1,2))
(0,(1,1))
=========================
(0,(3,6))
(0,(3,5))
(1,(4,8))
(1,(4,7))
(1,(1,2))
(1,(1,1))
(2,(5,10))
(2,(5,9))
(2,(2,4))
(2,(2,3))
```  
**HashPartitioner是如何分区的**： 国内很多说法都是有问题的，参考国外的一个说法 Uses Java’s Object.hashCodemethod to determine the partition as partition = key.hashCode() % numPartitions.  翻译过来就是使用java对象的hashCode来决定是哪个分区，对于piarRDD, 分区就是key.hashCode() % numPartitions, 3%3=0，所以 (3,6) 这个元素在0 分区， 4%3=1，所以元素(4,8) 在1 分区。 下面参考一张图  
![spark分区](https://cdn.edureka.co/blog/wp-content/uploads/2016/03/11.png)  


# **RangePartitioner**  
我理解成范围分区器
使用一个范围，将范围内的键分配给相应的分区。这种方法适用于键中有自然排序，键不为负。本文主要介绍如何使用，原理以后再仔细研究,以下代码片段显示了RangePartitioner的用法

**RangePartitioner 分区 scala **
```scala
    var pairRdd = sc.parallelize(List((1,1), (5,10), (5,9), (2,4), (3,5), (3,6),(4,7), (4,8),(2,3), (1,2)))
    printRDDPart(pairRdd)
    println("=========================")
    val partitioned = pairRdd.partitionBy(new RangePartitioner(3,pairRdd))
    printRDDPart(partitioned)
  }
-------------------输出------------------
(0,(1,2))
(0,(2,3))
(0,(4,8))
(0,(4,7))
(0,(3,6))
(0,(3,5))
(0,(2,4))
(0,(5,9))
(0,(5,10))
(0,(1,1))
=========================
(0,(1,2))
(0,(2,3))
(0,(2,4))
(0,(1,1))
(1,(4,8))
(1,(4,7))
(1,(3,6))
(1,(3,5))
(2,(5,9))
(2,(5,10))

```  
上面的RDD生成的时候是乱的，但是我们让他分成三个范围，按照范围，key值为1,2的划分到第一个分区，key值为3，4的划分到第二个分区，key值为5的划分到第三个分区  



# **自定义分区**  
要实现自定义的分区器，你需要继承 org.apache.spark.Partitioner 类并实现下面三个方法  
- **numPartitions: Int**：返回创建出来的分区数。  
- **getPartition(key: Any): Int**：返回给定键的分区编号（ 0 到 numPartitions-1）。
下面我自定义一个分区，让key大于等于4的落在第一个分区，key>=2并且key<4的落在第二个分区，其余的落在第一个分区。  
**scala版本**
自定义分区器
```scala
class CustomPartitioner(numParts: Int) extends Partitioner{
  override def numPartitions: Int = numParts

  override def getPartition(key: Any): Int = {
    if(key.toString.toInt>=4){
       0
    }else if(key.toString.toInt>=2&&key.toString.toInt<4){
      1
    }else{
      2
    }
  }
}
```
分区, 然后调用前面我们写的printRDDPart方法把各个分区中的RDD打印出来
```scala
    var pairRdd = sc.parallelize(List((1,1), (5,10), (5,9), (2,4), (3,5), (3,6),(4,7), (4,8),(2,3), (1,2)))
    val partitionedRdd = pairRdd.partitionBy(new CustomPartitioner(3))
    printRDDPart(partitionedRdd)
----------输出-----------------
(0,(4,8))
(0,(4,7))
(0,(5,9))
(0,(5,10))
(1,(2,3))
(1,(3,6))
(1,(3,5))
(1,(2,4))
(2,(1,2))
(2,(1,1))
```


# ==**java 分区的用法**== 

同样，写个方法，该方法能打印RDD下的每个分区下的各个元素

**打印每个分区下的各个元素的printPartRDD函数**
```java
    private static void printPartRDD(JavaPairRDD<Integer, Integer> pairRDD) {
        JavaRDD<Tuple2<Integer, Tuple2<Integer, Integer>>> mapPartitionIndexRDD = pairRDD.mapPartitionsWithIndex(new Function2<Integer, Iterator<Tuple2<Integer, Integer>>, Iterator<Tuple2<Integer, Tuple2<Integer, Integer>>>>() {
            @Override
            public Iterator<Tuple2<Integer, Tuple2<Integer, Integer>>> call(Integer partIndex, Iterator<Tuple2<Integer, Integer>> tuple2Iterator) {
                ArrayList<Tuple2<Integer, Tuple2<Integer, Integer>>> tuple2s = new ArrayList<>();

                while (tuple2Iterator.hasNext()) {
                    Tuple2<Integer, Integer> next = tuple2Iterator.next();
                    tuple2s.add(new Tuple2<Integer, Tuple2<Integer, Integer>>(partIndex, next));
                }
                return tuple2s.iterator();
            }
        }, false);

        mapPartitionIndexRDD.foreach(new VoidFunction<Tuple2<Integer, Tuple2<Integer, Integer>>>() {
            @Override
            public void call(Tuple2<Integer, Tuple2<Integer, Integer>> integerTuple2Tuple2) throws Exception {
                System.out.println(integerTuple2Tuple2);
            }
        });
    }
```  
**java HashPartitioner 分区**  

```java

		JavaRDD<Tuple2<Integer, Integer>> tupRdd = sc.parallelize(Arrays.asList(new Tuple2<Integer, Integer>(1, 1), new Tuple2<Integer, Integer>(1, 2)
                , new Tuple2<Integer, Integer>(2, 3), new Tuple2<Integer, Integer>(2, 4)
                , new Tuple2<Integer, Integer>(3, 5), new Tuple2<Integer, Integer>(3, 6)
                , new Tuple2<Integer, Integer>(4, 7), new Tuple2<Integer, Integer>(4, 8)
                , new Tuple2<Integer, Integer>(5, 9), new Tuple2<Integer, Integer>(5, 10)
        ), 3);
        JavaPairRDD<Integer, Integer> pairRDD = JavaPairRDD.fromJavaRDD(tupRdd);

        JavaPairRDD<Integer, Integer> partitioned = pairRDD.partitionBy(new HashPartitioner(3));
        System.out.println("============HashPartitioner==================");
        printPartRDD(partitioned);
--------------打印--------------------
============HashPartitioner==================
(0,(3,5))
(0,(3,6))
(1,(1,2))
(1,(4,8))
(1,(4,7))
(1,(1,1))
(2,(5,10))
(2,(2,4))
(2,(2,3))
(2,(5,9))
```
**java 自定义分区**
**自定义分区器** ，key大于4的落在第一个分区，[2,4)之间的落在第二个分区，其余的落在第三个分区
```
public class JavaCustomPart  extends Partitioner {
    int i = 1;
    public JavaCustomPart(int i){
        this.i=i;
    }
    public JavaCustomPart(){}
    @Override
    public int numPartitions() {
        return i;
    }

    @Override
    public int getPartition(Object key) {
        int keyCode = Integer.parseInt(key.toString());
        if(keyCode>=4){
            return 0;
        }else if(keyCode>=2&&keyCode<4){
            return 1;
        }else {
            return 2;
        }
    }
}

```
**分区并打印**  
```java
        System.out.println("============CustomPartition==================");
        JavaPairRDD<Integer, Integer> customPart = pairRDD.partitionBy(new JavaCustomPart(3));
        printPartRDD(customPart);
--------------打印---------------
============CustomPartition==================
(0,(5,10))
(0,(4,8))
(0,(4,7))
(0,(5,9))
(1,(2,4))
(1,(3,5))
(1,(3,6))
(1,(2,3))
(2,(1,2))
(2,(1,1))
```  
**java RangePartitioner** 
TODO
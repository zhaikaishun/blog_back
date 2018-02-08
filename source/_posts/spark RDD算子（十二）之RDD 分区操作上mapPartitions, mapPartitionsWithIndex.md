---
title: spark RDD算子（十二）之RDD 分区操作上mapPartitions, mapPartitionsWithIndex保存操作saveAsTextFile,saveAsSequenceFile,saveAsObjectFile,saveAsHadoopFile 等
date: 2017-04-18 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 46
permalink: spark-rdd-12
---

# **mapPartitions** 
mapPartition可以倒过来理解，先partition，再把每个partition进行map函数，
**适用场景**   
如果在映射的过程中需要频繁创建额外的对象，使用mapPartitions要比map高效的过。

比如，将RDD中的所有数据通过JDBC连接写入数据库，如果使用map函数，可能要为每一个元素都创建一个connection，这样开销很大，如果使用mapPartitions，那么只需要针对每一个分区建立一个connection。  
下面的例子，**把每一个元素平方**
<!-- more -->
**java 每一个元素平方**
```java
        JavaRDD<Integer> rdd = sc.parallelize(
                Arrays.asList(1,2,3,4,5,6,7,8,9,10));
        JavaRDD<Integer> mapPartitionRDD = rdd.mapPartitions(new FlatMapFunction<Iterator<Integer>, Integer>() {
            @Override
            public Iterable<Integer> call(Iterator<Integer> it) throws Exception {
                ArrayList<Integer> results = new ArrayList<>();
                while (it.hasNext()) {
                    int i = it.next();
                    results.add(i*i);
                }
                return results;
            }
        });
        mapPartitionRDD.foreach(new VoidFunction<Integer>() {
            @Override
            public void call(Integer integer) throws Exception {
                System.out.println(integer);
            }
        });
----------输出-------------
1
4
9
16
25
36
49
64
81
100
```  
**把每一个数字i变成一个map(i,i*i)的形式**

**java，把每一个元素变成map(i,i*i)**
```java
        JavaRDD<Tuple2<Integer, Integer>> tuple2JavaRDD = rdd.mapPartitions(new FlatMapFunction<Iterator<Integer>, Tuple2<Integer, Integer>>() {
            @Override
            public Iterable<Tuple2<Integer, Integer>> call(Iterator<Integer> it) throws Exception {
                ArrayList<Tuple2<Integer, Integer>> tuple2s = new ArrayList<>();
                while (it.hasNext()) {
                    Integer next = it.next();
                    tuple2s.add(new Tuple2<Integer, Integer>(next, next * next));
                }
                return tuple2s;
            }
        });
        tuple2JavaRDD.foreach(new VoidFunction<Tuple2<Integer, Integer>>() {
            @Override
            public void call(Tuple2<Integer, Integer> tp2) throws Exception {
                System.out.println(tp2);
            }
        });
---------输出---------------
(1,1)
(2,4)
(3,9)
(4,16)
(5,25)
(6,36)
(7,49)
(8,64)
(9,81)
(10,100)
```
**scala 把每一个元素变成map(i,i*i)**  
```
    val rdd = sc.parallelize(List(1,2,3,4,5,6,7,8,9,10),3)
    def mapPartFunc(iter: Iterator[Int]):Iterator[(Int,Int)]={
      var res = List[(Int,Int)]()
      while (iter.hasNext){
        val cur = iter.next
        res=res.::(cur,cur*cur)
      }
       res.iterator
    }
    val mapPartRDD = rdd.mapPartitions(mapPartFunc)
    mapPartRDD.foreach(maps=>println(maps))
----------输出-----------
(3,9)
(2,4)
(1,1)
(6,36)
(5,25)
(4,16)
(10,100)
(9,81)
(8,64)
(7,49)
``` 
**mapPartitions操作键值对 把(i,j) 变成(i,j*j)**
**scala版本**
```scala
    var rdd = sc.parallelize(List((1,1), (1,2), (1,3), (2,1), (2,2), (2,3)))

    def mapPartFunc(iter: Iterator[(Int,Int)]):Iterator[(Int,Int)]={
      var res = List[(Int,Int)]()
      while (iter.hasNext){
        val cur = iter.next
        res=res.::(cur._1,cur._2*cur._2)
      }
      res.iterator
    }
    val mapPartionsRDD = rdd.mapPartitions(mapPartFunc)
    mapPartionsRDD.foreach(println( _))
```  
**java版本**
```java
        JavaRDD<Tuple2<Integer, Integer>> rdd = sc.parallelize(Arrays.asList(new Tuple2<Integer, Integer>(1, 1), new Tuple2<Integer, Integer>(1, 2)
                , new Tuple2<Integer, Integer>(1, 3), new Tuple2<Integer, Integer>(2, 1)
                , new Tuple2<Integer, Integer>(2, 2), new Tuple2<Integer, Integer>(2, 3)), 3);
        JavaPairRDD<Integer, Integer> pairRDD = JavaPairRDD.fromJavaRDD(rdd);
        JavaRDD<Tuple2<Integer, Integer>> tuple2JavaRDD = pairRDD.mapPartitions(new FlatMapFunction<Iterator<Tuple2<Integer, Integer>>, Tuple2<Integer, Integer>>() {
            @Override
            public Iterable<Tuple2<Integer, Integer>> call(Iterator<Tuple2<Integer, Integer>> tp2It) throws Exception {
                ArrayList<Tuple2<Integer, Integer>> tuple2s = new ArrayList<>();
                while (tp2It.hasNext()){
                    Tuple2<Integer, Integer> next = tp2It.next();
                    tuple2s.add(new Tuple2<Integer, Integer>(next._1,next._2*next._2));
                }
                return tuple2s;
            }
        });
        tuple2JavaRDD.foreach(new VoidFunction<Tuple2<Integer, Integer>>() {
            @Override
            public void call(Tuple2<Integer, Integer> tp2) throws Exception {
                System.out.println(tp2);
            }
        });
-----------------输出---------------
(1,1)
(1,4)
(1,9)
(2,1)
(2,4)
(2,9)
```  



# **mapPartitionsWithIndex**  
与mapPartitionWithIndex类似，也是按照分区进行的map操作，不过mapPartitionsWithIndex传入的参数多了一个分区的值，下面举个例子,为统计各个分区中的元素 (稍加修改可以做统计各个分区的数量)

**java**
```java
JavaRDD<Integer> rdd = sc.parallelize(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10), 3);
        JavaRDD<Tuple2<Integer, Integer>> tuple2JavaRDD = rdd.mapPartitionsWithIndex(new Function2<Integer, Iterator<Integer>, Iterator<Tuple2<Integer, Integer>>>() {
            @Override
            public Iterator<Tuple2<Integer, Integer>> call(Integer partIndex, Iterator<Integer> it) throws Exception {
                ArrayList<Tuple2<Integer, Integer>> tuple2s = new ArrayList<>();
                while (it.hasNext()) {
                    int next = it.next();
                    tuple2s.add(new Tuple2<>(partIndex, next));
                }
                return tuple2s.iterator();
            }
        }, false);

        tuple2JavaRDD.foreach(new VoidFunction<Tuple2<Integer, Integer>>() {
            @Override
            public void call(Tuple2<Integer, Integer> tp2) throws Exception {
                System.out.println(tp2);
            }
        });
-------输出-------------
(0,1)
(0,2)
(0,3)
(1,4)
(1,5)
(1,6)
(2,7)
(2,8)
(2,9)
(2,10)
``` 
**scala**  
```
    val rdd = sc.parallelize(List(1,2,3,4,5,6,7,8,9,10),3)

    def mapPartIndexFunc(i1:Int,iter: Iterator[Int]):Iterator[(Int,Int)]={
      var res = List[(Int,Int)]()
      while(iter.hasNext){
        var next = iter.next()
        res=res.::(i1,next)
      }
      res.iterator
    }
    var mapPartIndexRDDs = rdd.mapPartitionsWithIndex(mapPartIndexFunc)

    mapPartIndexRDDs.foreach(println( _))
------------输出-------
(0,3)
(0,2)
(0,1)
(1,6)
(1,5)
(1,4)
(2,10)
(2,9)
(2,8)
(2,7)
```  

**mapPartitionsWithIndex 统计键值对中的各个分区的元素**  
**scala版本**
```scala
var rdd = sc.parallelize(List((1,1), (1,2), (2,3), (2,4), (3,5), (3,6),(4,7), (4,8),(5,9), (5,10)),3)

    def mapPartIndexFunc(i1:Int,iter: Iterator[(Int,Int)]):Iterator[(Int,(Int,Int))]={
      var res = List[(Int,(Int,Int))]()
      while(iter.hasNext){
        var next = iter.next()
        res=res.::(i1,next)
      }
      res.iterator
    }
    val mapPartIndexRDD = rdd.mapPartitionsWithIndex(mapPartIndexFunc)

    mapPartIndexRDD.foreach(println( _))
-----------输出---------
(0,(1,1))
(0,(1,2))
(0,(2,3))
(1,(2,4))
(1,(3,5))
(1,(3,6))
(2,(4,7))
(2,(4,8))
(2,(5,9))
(2,(5,10))
```
**java版本**
```
        JavaRDD<Tuple2<Integer, Integer>> rdd = sc.parallelize(Arrays.asList(new Tuple2<Integer, Integer>(1, 1), new Tuple2<Integer, Integer>(1, 2)
                , new Tuple2<Integer, Integer>(2, 3), new Tuple2<Integer, Integer>(2, 4)
                , new Tuple2<Integer, Integer>(3, 5), new Tuple2<Integer, Integer>(3, 6)
                , new Tuple2<Integer, Integer>(4, 7), new Tuple2<Integer, Integer>(4, 8)
                , new Tuple2<Integer, Integer>(5, 9), new Tuple2<Integer, Integer>(5, 10)
                ), 3);
        JavaPairRDD<Integer, Integer> pairRDD = JavaPairRDD.fromJavaRDD(rdd);

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
--------------输出---------
(0,(1,1))
(0,(1,2))
(0,(2,3))
(1,(2,4))
(1,(3,5))
(1,(3,6))
(2,(4,7))
(2,(4,8))
(2,(5,9))
(2,(5,10))
```

mapPartitionsWithIndex 中 第二个参数，true还是false  
这篇文章有些探讨,http://stackoverflow.com/questions/38048904/how-to-use-function-mappartitionswithindex-in-spark/38049239，个人还未理解 TODO  


补充： 打印各个分区的操作，可以使用 glom 的方法
```java
JavaRDD<Tuple2<Integer, Integer>> rdd1 = sc.parallelize(Arrays.asList(new Tuple2<Integer, Integer>(1, 1), new Tuple2<Integer, Integer>(1, 2)
                , new Tuple2<Integer, Integer>(2, 3), new Tuple2<Integer, Integer>(2, 4)
                , new Tuple2<Integer, Integer>(3, 5), new Tuple2<Integer, Integer>(3, 6)
                , new Tuple2<Integer, Integer>(4, 7), new Tuple2<Integer, Integer>(4, 8)
                , new Tuple2<Integer, Integer>(5, 9), new Tuple2<Integer, Integer>(5, 10)
        ), 3);
        JavaPairRDD<Integer, Integer> pairRDD = JavaPairRDD.fromJavaRDD(rdd1);
        /*补充：打印各个分区的操作，可以使用 glom 的方法*/
        System.out.println("打印各个分区的操作，可以使用 glom 的方法");
        JavaRDD<List<Tuple2<Integer, Integer>>> glom = pairRDD.glom();
        glom.foreach(new VoidFunction<List<Tuple2<Integer, Integer>>>() {
            @Override
            public void call(List<Tuple2<Integer, Integer>> tuple2s) throws Exception {
                System.out.println(tuple2s);
            }
        });

//************************* 输出 
打印各个分区的操作，可以使用 glom 的方法
[(1,1), (1,2), (2,3)]
[(2,4), (3,5), (3,6)]
[(4,7), (4,8), (5,9), (5,10)]
```
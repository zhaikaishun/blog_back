---
title: spark RDD算子（八）之键值对关联操作 subtractByKey, join, rightOuterJoin, leftOuterJoin
date: 2017-03-18 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 42
permalink: spark-rdd-8
---

先从spark-learning中的一张图大致了解其功能  
![](http://i1.piimg.com/567571/1ccee438414f9c6f.png)

# **subtractByKey**
**函数定义**
```
def subtractByKey[W](other: RDD[(K, W)])(implicit arg0: ClassTag[W]): RDD[(K, V)]

def subtractByKey[W](other: RDD[(K, W)], numPartitions: Int)(implicit arg0: ClassTag[W]): RDD[(K, V)]

def subtractByKey[W](other: RDD[(K, W)], p: Partitioner)(implicit arg0: ClassTag[W]): RDD[(K, V)]
```
类似于subtrac，删掉 RDD 中键与 other RDD 中的键相同的元素
<!-- more -->
# **join**  
**函数定义**
```
def join[W](other: RDD[(K, W)]): RDD[(K, (V, W))]

def join[W](other: RDD[(K, W)], numPartitions: Int): RDD[(K, (V, W))]

def join[W](other: RDD[(K, W)], partitioner: Partitioner): RDD[(K, (V, W))]
```
RDD1.join(RDD2)  
可以把RDD1,RDD2中的相同的key给连接起来，类似于sql中的join操作  

# **leftOuterJoin**
```java
def leftOuterJoin[W](other: RDD[(K, W)]): RDD[(K, (V, Option[W]))]

def leftOuterJoin[W](other: RDD[(K, W)], numPartitions: Int): RDD[(K, (V, Option[W]))]

def leftOuterJoin[W](other: RDD[(K, W)], partitioner: Partitioner): RDD[(K, (V, Option[W]))]
```  
直接看图即可 
对两个 RDD 进行连接操作，类似于sql中的左外连接
# **rightOuterJoin**
对两个 RDD 进行连接操作，类似于sql中的右外连接，存在的话，value用的Some, 不存在用的None,具体的看上面的图和下面的代码即可  

# **代码示例**

**scala语言**
```scala  
    scala> val rdd = sc.makeRDD(Array((1,2),(3,4),(3,6)))
    scala> val other = sc.makeRDD(Array((3,9)))
    
    scala>  rdd.subtractByKey(other).collect()
    res0: Array[(Int, Int)] = Array((1,2))
    
    scala> rdd.join(other).collect()
    res1: Array[(Int, (Int, Int))] = Array((3,(4,9)), (3,(6,9)))
    
    scala> rdd.leftOuterJoin(other).collect()
    res2: Array[(Int, (Int, Option[Int]))] = Array((1,(2,None)), (3,(4,Some(9))), (3,(6,Some(9))))
    
    scala> rdd.rightOuterJoin(other).collect()
    res3: Array[(Int, (Option[Int], Int))] = Array((3,(Some(4),9)), (3,(Some(6),9)))
```


**java语言**
```java
    JavaRDD<Tuple2<Integer,Integer>> rddPre = sc.parallelize(Arrays.asList(new Tuple2(1,2)
            , new Tuple2(3,4)
            , new Tuple2(3,6)));
    JavaRDD<Tuple2<Integer,Integer>> otherPre = sc.parallelize(Arrays.asList(new Tuple2(3,10)));

	//JavaRDD转换成JavaPairRDD
    JavaPairRDD<Integer, Integer> rdd = JavaPairRDD.fromJavaRDD(rddPre);
    JavaPairRDD<Integer, Integer> other = JavaPairRDD.fromJavaRDD(otherPre);
    //subtractByKey
    JavaPairRDD<Integer, Integer> subRDD = rdd.subtractByKey(other);
    
    //join
    JavaPairRDD<Integer, Tuple2<Float, Integer>> joinRDD =  rdd.join(other);
    
    //leftOuterJoin
    JavaPairRDD<Integer, Tuple2<Integer, Optional<Integer>>> integerTuple2JavaPairRDD = rdd.leftOuterJoin(other);
    
    //rightOutJoin
    JavaPairRDD<Integer, Tuple2<Optional<Integer>, Integer>> rightOutJoin = rdd.rightOuterJoin(other);
```


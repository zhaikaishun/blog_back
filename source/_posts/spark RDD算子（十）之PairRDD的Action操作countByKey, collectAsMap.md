---
title: spark RDD算子（十）之PairRDD的Action操作countByKey, collectAsMap
date: 2017-04-05 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 44
permalink: spark-rdd-10
---

# **countByKey**
def countByKey(): Map[K, Long]  
以RDD{(1, 2),(2,4),(2,5), (3, 4),(3,5), (3, 6)}为例  rdd.countByKey会返回{(1,1),(2,2),(3,3)}
**scala例子**
```scala
scala> val rdd = sc.parallelize(Array((1, 2),(2,4),(2,5), (3, 4),(3,5), (3, 6)))

scala> val countbyKeyRDD = rdd.countByKey()
countbyKeyRDD: scala.collection.Map[Int,Long] = Map(1 -> 1, 2 -> 2, 3 -> 3)
```
<!-- more -->
**java例子**  
```java
    JavaRDD<Tuple2<Integer, Integer>> tupleRDD =
            sc.parallelize(Arrays.asList(new Tuple2<>(1, 2),
            new Tuple2<>(2, 4),
            new Tuple2<>(2, 5),
            new Tuple2<>(3, 4),
            new Tuple2<>(3, 5),
            new Tuple2<>(3, 6)));
    JavaPairRDD<Integer, Integer> mapRDD = JavaPairRDD.fromJavaRDD(tupleRDD);
    //countByKey
    Map<Integer, Object> countByKeyRDD = mapRDD.countByKey();
    for (Integer i:countByKeyRDD.keySet()) {
        System.out.println("("+i+", "+countByKeyRDD.get(i)+")");
    }
/*
输出  
(1, 1)
(3, 3)
(2, 2)

*/
```

# **collectAsMap**
将pair类型(键值对类型)的RDD转换成map, 还是上面的例子

**scala例子**
```scala
    scala> val rdd = sc.parallelize(Array((1, 2),(2,4),(2,5), (3, 4),(3,5), (3, 6)))
    
    scala> rdd.collectAsMap()
    res1: scala.collection.Map[Int,Int] = Map(2 -> 5, 1 -> 2, 3 -> 6)
```

**java例子**  
```java
    JavaRDD<Tuple2<Integer, Integer>> tupleRDD =
            sc.parallelize(Arrays.asList(new Tuple2<>(1, 2),
            new Tuple2<>(2, 4),
            new Tuple2<>(2, 5),
            new Tuple2<>(3, 4),
            new Tuple2<>(3, 5),
            new Tuple2<>(3, 6)));
    JavaPairRDD<Integer, Integer> mapRDD = JavaPairRDD.fromJavaRDD(tupleRDD);

    Map<Integer, Integer> collectMap = mapRDD.collectAsMap();
```
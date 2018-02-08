---
title: spark RDD算子（九）之基本的Action操作 first, take, collect, count, countByValue, reduce, aggregate, fold,top
date: 2017-03-28 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 43
permalink: spark-rdd-9
toc: true
---

# **first**  
返回第一个元素  
**scala**
```scala
scala> val rdd = sc.parallelize(List(1,2,3,3))

scala> rdd.first()
res1: Int = 1
```
<!-- more -->
**java**
```java
    JavaRDD<Integer> rdd = sc.parallelize(Arrays.asList(1, 2, 3, 3));
    Integer first = rdd.first();
```

# **take**
rdd.take(n)返回第n个元素  
**scala**
```scala
scala> val rdd = sc.parallelize(List(1,2,3,3))

scala> rdd.take(2)
res3: Array[Int] = Array(1, 2)
```
**java**
```java
    JavaRDD<Integer> rdd = sc.parallelize(Arrays.asList(1, 2, 3, 3));
    List<Integer> take = rdd.take(2);
```

# **collect**  
rdd.collect() 返回 RDD 中的所有元素  
**scala**
```scala
scala> val rdd = sc.parallelize(List(1,2,3,3))

scala> rdd.collect()
res4: Array[Int] = Array(1, 2, 3, 3)
```
**java**
```java
    JavaRDD<Integer> rdd = sc.parallelize(Arrays.asList(1, 2, 3, 3));
    List<Integer> collect = rdd.collect();
```


# **count**
rdd.count() 返回 RDD 中的元素个数  
**scala**
```scala
scala> val rdd = sc.parallelize(List(1,2,3,3))

scala> rdd.count()
res5: Long = 4
```
**java**
```java
    JavaRDD<Integer> rdd = sc.parallelize(Arrays.asList(1, 2, 3, 3));
    long count = rdd.count();
```
# **countByValue**
各元素在 RDD 中出现的次数 返回{(key1,次数),(key2,次数),...(keyn,次数)}  
**scala**
```scala
scala> val rdd = sc.parallelize(List(1,2,3,3))

scala> rdd.countByValue()
res6: scala.collection.Map[Int,Long] = Map(1 -> 1, 2 -> 1, 3 -> 2)
```
**java**
```java
    JavaRDD<Integer> rdd = sc.parallelize(Arrays.asList(1, 2, 3, 3));
    Map<Integer, Long> integerLongMap = rdd.countByValue();
```

# **reduce**
rdd.reduce(func)
并行整合RDD中所有数据， 类似于是scala中集合的reduce  
**scala**
```scala
scala> val rdd = sc.parallelize(List(1,2,3,3))

scala> rdd.reduce((x,y)=>x+y)
res7: Int = 9
```
**java**
```java
    Integer reduce = rdd.reduce(new Function2<Integer, Integer, Integer>() {
        @Override
        public Integer call(Integer integer, Integer integer2) throws Exception {
            return integer + integer2;
        }
    });

```

# **aggregate**
和 reduce() 相 似， 但 是 通 常
返回不同类型的函数  一般不用这个函数
 
**scala**
```scala
scala> val rdd = sc.parallelize(List(1,2,3,3))
TODO
```
**java**
```java

```

# **fold**
rdd.fold(num)(func) 一般不用这个函数
和 reduce() 一 样， 但是提供了初始值num,每个元素计算时，先要合这个初始值进行折叠, 注意，这里会按照每个分区进行fold，然后分区之间还会再次进行fold
提供初始值  
**scala**
```scala
// 解释 TODO 
scala> val rdd = sc.parallelize(List(1,2,3,3)，2)

scala> rdd.fold(1)((x,y)=>x+y)
res8: Int = 12
```
**java**
```java
    JavaRDD<Integer> rdd = sc.parallelize(Arrays.asList(1, 2, 3, 3),2);
    Integer fold = rdd.fold(1, new Function2<Integer, Integer, Integer>() {
        @Override
        public Integer call(Integer integer, Integer integer2) throws Exception {
            return integer + integer2;
        }
    });
    System.out.println(fold);
-------输出-----
12
```


# **top**
rdd.top(n)  
按照降序的或者指定的排序规则，返回前n个元素  
**scala**
```scala
scala> val rdd = sc.parallelize(List(1,2,3,3))

scala> rdd.top(2)
res9: Array[Int] = Array(3, 3)
```
**java**
```java
    JavaRDD<Integer> rdd = sc.parallelize(Arrays.asList(1, 2, 3, 3),2);
    List<Integer> top = rdd.top(2);
```

# **takeOrdered**
rdd.take(n)  
对RDD元素进行升序排序,取出前n个元素并返回，也可以自定义比较器（这里不介绍），类似于top的相反的方法  
**scala**
```scala
scala> val rdd = sc.parallelize(List(1,2,3,3))

scala> rdd.takeOrdered(2)
res10: Array[Int] = Array(1, 2)
```
**java**
```java
    JavaRDD<Integer> rdd = sc.parallelize(Arrays.asList(1, 2, 3, 3),2);
    List<Integer> integers = rdd.takeOrdered(2);
```

# **foreach**  
对 RDD 中的每个元素使用给
定的函数  
**scala**
```scala
    val rdd = sc.parallelize(List(1,2,3,3))
    rdd.foreach(print(_))
-----输出-----------
1233

```
**java**
```java
    rdd.foreach(new VoidFunction<Integer>() {
       @Override
       public void call(Integer integer) throws Exception {
           System.out.println(integer);
       }
    });
```



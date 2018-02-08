---
title: spark RDD算子（三） distinct，union，intersection，subtract，cartesian
date: 2017-03-03 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 36
permalink: spark-rdd-3
---

**spark伪集合**  
尽管 RDD 本身不是严格意义上的集合，但它也支持许多数学上的集合操作，比如合并和相交操作, 下图展示了这四种操作  
![RDD伪集合](http://i2.muimg.com/567571/cf473cf5e7f9f270.png)    
<!-- more -->
# **distinct**
distinct用于去重， 我们生成的RDD可能有重复的元素，使用distinct方法可以去掉重复的元素, 不过此方法涉及到混洗，操作开销很大  
**scala版本**
```scala
    scala> var RDD1 = sc.parallelize(List("aa","aa","bb","cc","dd"))
    
    scala> RDD1.collect
    res3: Array[String] = Array(aa, aa, bb, cc, dd)
    
    scala> var distinctRDD = RDD1.distinct
    
    scala> distinctRDD.collect
    res5: Array[String] = Array(aa, dd, bb, cc)
```  
**java版本**
```java
    JavaRDD<String> RDD1 = sc.parallelize(Arrays.asList("aa", "aa", "bb", "cc", "dd"));
    JavaRDD<String> distinctRDD = RDD1.distinct();
    List<String> collect = distinctRDD.collect();
    for (String str:collect) {
        System.out.print(str+", ");
    }
---------输出----------
aa, dd, bb, cc,
```  
# **union**  
两个RDD进行合并
**scala版本**
```
    scala> var RDD1 = sc.parallelize(List("aa","aa","bb","cc","dd"))
    scala> var RDD2 = sc.parallelize(List("aa","dd","ff"))
    
    scala> RDD1.collect
    res6: Array[String] = Array(aa, aa, bb, cc, dd)
    
    scala> RDD2.collect
    res7: Array[String] = Array(aa, dd, ff)
    
    scala> RDD1.union(RDD2).collect
    res8: Array[String] = Array(aa, aa, bb, cc, dd, aa, dd, ff)

```
**java版本**
```java
    JavaRDD<String> RDD1 = sc.parallelize(Arrays.asList("aa", "aa", "bb", "cc", "dd"));
    JavaRDD<String> RDD2 = sc.parallelize(Arrays.asList("aa","dd","ff"));
    JavaRDD<String> unionRDD = RDD1.union(RDD2);
    List<String> collect = unionRDD.collect();
    for (String str:collect) {
        System.out.print(str+", ");
    }
-----------输出---------
aa, aa, bb, cc, dd, aa, dd, ff,
``` 
# **intersection**
RDD1.intersection(RDD2)  返回两个RDD的交集，并且去重  
intersection 需要混洗数据，比较浪费性能  
**scala版本**  
```scala
    scala> var RDD1 = sc.parallelize(List("aa","aa","bb","cc","dd"))
    scala> var RDD2 = sc.parallelize(List("aa","dd","ff"))
    
    scala> RDD1.collect
    res6: Array[String] = Array(aa, aa, bb, cc, dd)
    
    scala> RDD2.collect
    res7: Array[String] = Array(aa, dd, ff)
    
    scala> var insertsectionRDD = RDD1.intersection(RDD2)
    scala> insertsectionRDD.collect
    
    res9: Array[String] = Array(aa, dd)
```  
**java版本**  
```java
    JavaRDD<String> RDD1 = sc.parallelize(Arrays.asList("aa", "aa", "bb", "cc", "dd"));
    JavaRDD<String> RDD2 = sc.parallelize(Arrays.asList("aa","dd","ff"));
    JavaRDD<String> intersectionRDD = RDD1.intersection(RDD2);
    List<String> collect = intersectionRDD.collect();
    for (String str:collect) {
        System.out.print(str+" ");
    }
-------------输出-----------
aa dd
```  
# **subtract**
RDD1.subtract(RDD2),返回在RDD1中出现，但是不在RDD2中出现的元素，不去重  
**scala版本**
```scala
    JavaRDD<String> RDD1 = sc.parallelize(Arrays.asList("aa", "aa","bb", "cc", "dd"));
    
    JavaRDD<String> RDD2 = sc.parallelize(Arrays.asList("aa","dd","ff"));
    
    scala> var substractRDD =RDD1.subtract(RDD2)
    
    scala>  substractRDD.collect
    res10: Array[String] = Array(bb, cc)
```
**java版本**
```java
    JavaRDD<String> RDD1 = sc.parallelize(Arrays.asList("aa", "aa", "bb","cc", "dd"));
    JavaRDD<String> RDD2 = sc.parallelize(Arrays.asList("aa","dd","ff"));
    JavaRDD<String> subtractRDD = RDD1.subtract(RDD2);
    List<String> collect = subtractRDD.collect();
    for (String str:collect) {
        System.out.print(str+" ");
    }
------------输出-----------------
bb  cc 
```  
# **cartesian**  
RDD1.cartesian(RDD2)  返回RDD1和RDD2的笛卡儿积，这个开销非常大

**scala版本**
```scala
    scala>  var RDD1 = sc.parallelize(List("1","2","3"))
    
    scala> var RDD2 = sc.parallelize(List("a","b","c"))
    
    scala> var cartesianRDD = RDD1.cartesian(RDD2)
    
    scala> cartesianRDD.collect
    res11: Array[(String, String)] = Array((1,a), (1,b), (1,c), (2,a), (2,b), (2,c), (3,a), (3,b), (3,c))
```  

**java版本**
```java
    JavaRDD<String> RDD1 = sc.parallelize(Arrays.asList("1", "2", "3"));
    JavaRDD<String> RDD2 = sc.parallelize(Arrays.asList("a","b","c"));
    JavaPairRDD<String, String> cartesian = RDD1.cartesian(RDD2);

    List<Tuple2<String, String>> collect1 = cartesian.collect();
    for (Tuple2<String, String> tp:collect1) {
        System.out.println("("+tp._1+" "+tp._2+")");
    }
------------输出-----------------
(1 a)
(1 b)
(1 c)
(2 a)
(2 b)
(2 c)
(3 a)
(3 b)
(3 c)
```  

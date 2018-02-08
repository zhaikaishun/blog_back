---
title: spark RDD算子（七）之键值对分组操作 groupByKey，cogroup
date: 2017-03-07 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 41
permalink: spark-rdd-7
---

# **groupByKey**
```
def groupByKey(): RDD[(K, Iterable[V])]

def groupByKey(numPartitions: Int): RDD[(K, Iterable[V])]

def groupByKey(partitioner: Partitioner): RDD[(K, Iterable[V])]
```
groupByKey会将RDD[key,value] 按照相同的key进行分组，形成RDD[key,Iterable[value]]的形式， 有点类似于sql中的groupby，例如类似于mysql中的group_concat  
例如这个例子， 我们对学生的成绩进行分组
<!-- more -->
**scala版本**
```scala
    val scoreDetail = sc.parallelize(List(("xiaoming",75),("xiaoming",90),("lihua",95),("lihua",100),("xiaofeng",85)))
    scoreDetail.groupByKey().collect().foreach(println(_));
    /*输出
(lihua,CompactBuffer(95, 100))
(xiaoming,CompactBuffer(75, 90))
(xiaofeng,CompactBuffer(85))
    */
```
**java版本**
```java
        JavaRDD<Tuple2<String,Float>> scoreDetails = sc.parallelize(Arrays.asList(new Tuple2("xiaoming", 75)
                , new Tuple2("xiaoming", 90)
                , new Tuple2("lihua", 95)
                , new Tuple2("lihua", 188)));
        //将JavaRDD<Tuple2<String,Float>> 类型转换为 JavaPairRDD<String, Float>
        JavaPairRDD<String, Float> scoreMapRDD = JavaPairRDD.fromJavaRDD(scoreDetails);
        Map<String, Iterable<Float>> resultMap = scoreMapRDD.groupByKey().collectAsMap();
        for (String key:resultMap.keySet()) {
            System.out.println("("+key+", "+resultMap.get(key)+")");
        }
```

# **cogroup**
groupByKey是对单个 RDD 的数据进行分组，还可以使用一个叫作 cogroup() 的函数对多个共享同一个键的 RDD 进行分组
例如  
RDD1.cogroup(RDD2) 会将RDD1和RDD2按照相同的key进行分组，得到(key,RDD[key,Iterable[value1],Iterable[value2]])的形式
cogroup也可以多个进行分组  
例如RDD1.cogroup(RDD2,RDD3,...RDDN), 可以得到(key,Iterable[value1],Iterable[value2],Iterable[value3],...,Iterable[valueN])  
案例,scoreDetail存放的是学生的优秀学科的分数，scoreDetai2存放的是刚刚及格的分数，scoreDetai3存放的是没有及格的科目的分数，我们要对每一个学生的优秀学科，刚及格和不及格的分数给分组统计出来     
**scala版本**
```scala
scala> val scoreDetail = sc.parallelize(List(("xiaoming",95),("xiaoming",90),("lihua",95),("lihua",98),("xiaofeng",97)))
scala> val scoreDetai2 = sc.parallelize(List(("xiaoming",65),("lihua",63),("lihua",62),("xiaofeng",67)))
scala> val scoreDetai3 = sc.parallelize(List(("xiaoming",25),("xiaoming",15),("lihua",35),("lihua",28),("xiaofeng",36)))
scala> scoreDetail.cogroup(scoreDetai2,scoreDetai3)

//输出
res1: Array[(String, (Iterable[Int], Iterable[Int], Iterable[Int]))] = Array((xiaoming,(CompactBuffer(95, 90),CompactBuffer(65),CompactBuffer(25, 15))), (lihua,(CompactBuffer(95, 98),CompactBuffer(63, 62),CompactBuffer(35, 28))), (xiaofeng,(CompactBuffer(97),CompactBuffer(67),CompactBuffer(36))))

```  
**java版本**  
```java
        JavaRDD<Tuple2<String,Float>> scoreDetails1 = sc.parallelize(Arrays.asList(new Tuple2("xiaoming", 75)
                , new Tuple2("xiaoming", 90)
                , new Tuple2("lihua", 95)
                , new Tuple2("lihua", 96)));
        JavaRDD<Tuple2<String,Float>> scoreDetails2 = sc.parallelize(Arrays.asList(new Tuple2("xiaoming", 75)
                , new Tuple2("lihua", 60)
                , new Tuple2("lihua", 62)));
        JavaRDD<Tuple2<String,Float>> scoreDetails3 = sc.parallelize(Arrays.asList(new Tuple2("xiaoming", 75)
                , new Tuple2("xiaoming", 45)
                , new Tuple2("lihua", 24)
                , new Tuple2("lihua", 57)));
        
        JavaPairRDD<String, Float> scoreMapRDD1 = JavaPairRDD.fromJavaRDD(scoreDetails1);
        JavaPairRDD<String, Float> scoreMapRDD2 = JavaPairRDD.fromJavaRDD(scoreDetails2);
        JavaPairRDD<String, Float> scoreMapRDD3 = JavaPairRDD.fromJavaRDD(scoreDetails2);
        
        JavaPairRDD<String, Tuple3<Iterable<Float>, Iterable<Float>, Iterable<Float>>> cogroupRDD = (JavaPairRDD<String, Tuple3<Iterable<Float>, Iterable<Float>, Iterable<Float>>>) scoreMapRDD1.cogroup(scoreMapRDD2, scoreMapRDD3);
        Map<String, Tuple3<Iterable<Float>, Iterable<Float>, Iterable<Float>>> tuple3 = cogroupRDD.collectAsMap();
        for (String key:tuple3.keySet()) {
            System.out.println("("+key+", "+tuple3.get(key)+")");
        }
        
-----输出----------
(lihua, ([95, 96],[60, 62],[60, 62]))
(xiaoming, ([75, 90],[75],[75]))

```
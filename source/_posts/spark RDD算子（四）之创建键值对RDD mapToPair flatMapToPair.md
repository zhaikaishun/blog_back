---
title: spark RDD算子（四）之创建键值对RDD mapToPair flatMapToPair
date: 2017-03-04 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 37
permalink: spark-rdd-4
---

# **mapToPair**
举例，在F:\sparktest\sample.txt 文件的内容如下  
```
aa bb cc aa aa aa dd dd ee ee ee ee 
ff aa bb zks
ee kks
ee  zz zks
```
将每一行的第一个单词作为键，1 作为value创建pairRDD
<!-- more -->
**scala版本**  
scala是没有mapToPair函数的，scala版本只需要map就可以了
```scala
    scala> val lines = sc.textFile("F:\\sparktest\\sample.txt")
    
    scala> val pairs = lines.map(x => (x.split("\\s+")(0), 1))
    
    scala> pairs.collect
    res0: Array[(String, Int)] = Array((aa,1), (ff,1), (ee,1), (ee,1))
```
**java版本**
```java
    JavaRDD<String> lines = sc.textFile("F:\\sparktest\\sample.txt");
    //输入的是一个string的字符串，输出的是一个(String, Integer) 的map
    JavaPairRDD<String, Integer> pairRDD = lines.mapToPair(new PairFunction<String, String, Integer>() {
        @Override
        public Tuple2<String, Integer> call(String s) throws Exception {
            return new Tuple2<String, Integer>(s.split("\\s+")[0], 1);
        }
    });
```
# **flatMapToPair**
类似于xxx连接  mapToPair是一对一，一个元素返回一个元素，而flatMapToPair可以一个元素返回多个，相当于先flatMap,在mapToPair
例子: 将每一个单词都分成键为
**scala版本**
```scala
    val lines = sc.textFile("F:\\sparktest\\sample.txt")
    val flatRDD = lines.flatMap(x => (x.split("\\s+")))
    val pairs = flatRDD.map(x=>(x,1))
    
    scala> pairs.collect
    res1: Array[(String, Int)] = Array((aa,1), (bb,1), (cc,1), (aa,1), (aa,1), (aa,1), (dd,1), (dd,1), (ee,1), (ee,1), (ee,1), (ee,1), (ff,1), (aa,1), (bb,1), (zks,1), (ee,1), (kks,1), (ee,1), (zz,1), (zks,1))
```
**java版本 spark2.0以下**
```java
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
```  
**java版本 spark2.0以上**
主要是iterator和iteratable的一些区别
```java
        JavaPairRDD<String, Integer> wordPairRDD = lines.flatMapToPair(new PairFlatMapFunction<String, String, Integer>() {
            @Override
            public Iterator<Tuple2<String, Integer>> call(String s) throws Exception {
                ArrayList<Tuple2<String, Integer>> tpLists = new ArrayList<Tuple2<String, Integer>>();
                String[] split = s.split("\\s+");
                for (int i = 0; i <split.length ; i++) {
                    Tuple2 tp = new Tuple2<String,Integer>(split[i], 1);
                    tpLists.add(tp);
                }
                return tpLists.iterator();
            }
        });
```
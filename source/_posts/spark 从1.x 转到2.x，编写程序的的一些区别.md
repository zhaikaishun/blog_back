---
title: spark 从1.x 转到2.x，编写程序的的一些区别
date: 2017-02-05 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 32
permalink: spark-v1-v2
---

spark 2.x 版本相对于1.x版本，有挺多地方的修改，一是类似于flatMapRDD 中 iteator iteatable之类的区别  
2是类似于dataset的一些问题

下面是2.x版本的iteatable和iteartor之类的区别，只举例了两个，其实只要和iteartor有关的都有了修改
<!-- more -->
## flatMap
```java
        JavaRDD<String> flatMapRDD = lines.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public Iterator<String> call(String s) throws Exception {
                String[] split = s.split("\\s+");
                return Arrays.asList(split).iterator();
            }
        });
```

## flatMapToPair  java
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

## spark中初始化driver的区别
spark2.0中，可以使用session来创建一个sparkContext作为一个新的入口，具体参考例子就可以了

## jar包的区别
spark2.x版本中不再有spark-assembly-xxx jar包，jar包全都在.jars 中 

## scala的版本
spark2.x版本的，对scala的版本最低要求是2.11
## 下面是sql中的区别
2.x 版本的 sparkSql中  
1.x 版本的 DataFrame与Dataset 统一化了，只剩下DataSet了，具体的也可以直接参看官方给的spark sql 的例子即可 
具体 todo

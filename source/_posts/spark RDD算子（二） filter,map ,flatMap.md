---
title: spark RDD算子（二） filter,map ,flatMap
date: 2017-03-02 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 35
permalink: spark-rdd-2
---


先来一张spark快速大数据中的图片进行快速入门，后面有更详细的例子  
![spark-map](http://i4.buimg.com/567571/9bf09aff04368678.png)
<!-- more -->

## **filter**
举例，在F:\sparktest\sample.txt 文件的内容如下  
```
aa bb cc aa aa aa dd dd ee ee ee ee 
ff aa bb zks
ee kks
ee  zz zks
```  
我要将包含zks的行的内容给找出来
**scala版本**
```scala
    val lines = sc.textFile("F:\\sparktest\\sample.txt").filter(line=>line.contains("zks"))
    //打印内容
    lines.collect().foreach(println(_));
-------------输出------------------
ff aa bb zks
ee  zz zks

```  
**java版本**
```java
        JavaRDD<String> lines = sc.textFile("F:\\sparktest\\sample.txt");
        JavaRDD<String> zksRDD = lines.filter(new Function<String, Boolean>() {
            @Override
            public Boolean call(String s) throws Exception {
                return s.contains("zks");
            }
        });
        //打印内容
        List<String> zksCollect = zksRDD.collect();
        for (String str:zksCollect) {
            System.out.println(str);
        }
----------------输出-------------------
ff aa bb zks
ee  zz zks
```  

## **map**  
map() 接收一个函数，把这个函数用于 RDD 中的每个元素，将函数的返回结果作为结果RDD编程 ｜ 31
RDD 中对应元素的值 map是一对一的关系
举例，在F:\sparktest\sample.txt 文件的内容如下  
```
aa bb cc aa aa aa dd dd ee ee ee ee 
ff aa bb zks
ee kks
ee  zz zks
```    
把每一行变成一个数组
**scala版本**
```scala
//读取数据
scala> val lines = sc.textFile("F:\\sparktest\\sample.txt")
//用map，对于每一行数据，按照空格分割成一个一个数组，然后返回的是一对一的关系
scala> var mapRDD = lines.map(line => line.split("\\s+"))
---------------输出-----------
res0: Array[Array[String]] = Array(Array(aa, bb, cc, aa, aa, aa, dd, dd, ee, ee, ee, ee), Array(ff, aa, bb, zks), Array(ee, kks), Array(ee, zz, zks))

//读取第一个元素
scala> mapRDD.first
---输出----
res1: Array[String] = Array(aa, bb, cc, aa, aa, aa, dd, dd, ee, ee, ee, ee)
```  
**java版本**
```java
        JavaRDD<Iterable<String>> mapRDD = lines.map(new Function<String, Iterable<String>>() {
            @Override
            public Iterable<String> call(String s) throws Exception {
                String[] split = s.split("\\s+");
                return Arrays.asList(split);
            }
        });
        //读取第一个元素
        System.out.println(mapRDD.first());
    ---------------输出-------------
    [aa, bb, cc, aa, aa, aa, dd, dd, ee, ee, ee, ee]
```


## **flatMap**  
有时候，我们希望对某个元素生成多个元素，实现该功能的操作叫作 flatMap()  
faltMap的函数应用于每一个元素，对于每一个元素返回的是多个元素组成的迭代器(想要了解更多，请参考[scala的flatMap和map用法](http://blog.csdn.net/t1dmzks/article/details/69858060#t23))    
例如我们将数据切分为单词  
**scala版本**
```scala
    scala>  val lines = sc.textFile("F:\\sparktest\\sample.txt")
    scala> val flatMapRDD = lines.flatMap(line=>line.split("\\s"))
    scala> flatMapRDD.first() 
---输出----
res0: String = aa
```
**java版本，spark2.0以下**
```java
    JavaRDD<String> lines = sc.textFile("F:\\sparktest\\sample.txt");
    JavaRDD<String> flatMapRDD = lines.flatMap(new FlatMapFunction<String, String>() {
        @Override
        public Iterable<String> call(String s) throws Exception {
            String[] split = s.split("\\s+");
            return Arrays.asList(split);
        }
    });
    //输出第一个
    System.out.println(flatMapRDD.first());
------------输出----------
aa
```

**java版本，spark2.0以上**
spark2.0以上，对flatMap的方法有所修改，就是flatMap中的Iterator和Iteratable的小区别  
```java
        JavaRDD<String> flatMapRDD = lines.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public Iterator<String> call(String s) throws Exception {
                String[] split = s.split("\\s+");
                return Arrays.asList(split).iterator();
            }
        });
```


---
title: spark RDD算子（五）之键值对聚合操作 combineByKey
date: 2017-03-05 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 38
permalink: spark-rdd-5
---

# **combineByKey**
聚合数据一般在集中式数据比较方便，如果涉及到分布式的数据集，该如何去实现呢。这里介绍一下combineByKey, 这个是各种聚集操作的鼻祖，应该要好好了解一下,[参考scala API](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.PairRDDFunctions)  
<!-- more -->
## **简要介绍**
```scala
def combineByKey[C](createCombiner: (V) => C,  
                    mergeValue: (C, V) => C,   
                    mergeCombiners: (C, C) => C): RD
```

- **createCombiner**: combineByKey() 会遍历分区中的所有元素，因此每个元素的键要么还没有遇到过，要么就
和之前的某个元素的键相同。如果这是一个新的元素， combineByKey() 会使用一个叫作 createCombiner() 的函数来创建
那个键对应的累加器的初始值
-  **mergeValue:** 如果这是一个在处理当前分区之前已经遇到的键， 它会使用 mergeValue() 方法将该键的累加器对应的当前值与这个新的值进行合并  
-  **mergeCombiners:** 由于每个分区都是独立处理的， 因此对于同一个键可以有多个累加器。如果有两个或者更
多的分区都有对应同一个键的累加器， 就需要使用用户提供的 mergeCombiners() 方法将各
个分区的结果进行合并。

## **计算学生平均成绩例子**

这里举一个计算学生平均成绩的例子,例子参考至https://www.edureka.co/blog/apache-spark-combinebykey-explained, [github源码](https://github.com/prithvirajbose/spark-dev/blob/master/src/main/scala/examples/TestCombineByKey.scala)  我对此进行了解析
**创建一个学生成绩说明的类**
```scala
case class ScoreDetail(studentName: String, subject: String, score: Float)
```
**下面是一些测试数据**，加载测试数据集合  key = Students name and value = ScoreDetail instance
```scala
    val scores = List(
      ScoreDetail("xiaoming", "Math", 98),
      ScoreDetail("xiaoming", "English", 88),
      ScoreDetail("wangwu", "Math", 75),
      ScoreDetail("wangwu", "English", 78),
      ScoreDetail("lihua", "Math", 90),
      ScoreDetail("lihua", "English", 80),
      ScoreDetail("zhangsan", "Math", 91),
      ScoreDetail("zhangsan", "English", 80))
```
**将集合转换成二元组**， 也可以理解成转换成一个map, 利用了for 和 yield的组合
```scala
val scoresWithKey = for { i <- scores } yield (i.studentName, i)
```
**创建RDD, 并且指定三个分区**
```scala
    val scoresWithKeyRDD = sc.parallelize(scoresWithKey).partitionBy(new HashPartitioner(3)).cache
```
**输出打印一下各个分区的长度和各个分区的一些数据**
```scala
    println(">>>> Elements in each partition")

    scoresWithKeyRDD.foreachPartition(partition => println(partition.length))

    // explore each partition...
    println(">>>> Exploring partitions' data...")

    scoresWithKeyRDD.foreachPartition(
      partition => partition.foreach(
        item => println(item._2)))
/*
会输出 
>>>> Elements in each partition
6
2
0
>>>> Exploring partitions' data...
ScoreDetail(xiaoming,Math,98.0)
ScoreDetail(xiaoming,English,88.0)
ScoreDetail(lihua,Math,90.0)
ScoreDetail(lihua,English,80.0)
ScoreDetail(zhangsan,Math,91.0)
ScoreDetail(zhangsan,English,80.0)
ScoreDetail(wangwu,Math,75.0)
ScoreDetail(wangwu,English,78.0)

*/
```
**聚合求平均值让后打印**
```scala
      val avgScoresRDD = scoresWithKeyRDD.combineByKey(
      (x: ScoreDetail) => (x.score, 1) /*createCombiner*/,
      (acc: (Float, Int), x: ScoreDetail) => (acc._1 + x.score, acc._2 + 1) /*mergeValue*/,
      (acc1: (Float, Int), acc2: (Float, Int)) => (acc1._1 + acc2._1, acc1._2 + acc2._2) /*mergeCombiners*/
      // calculate the average
    ).map( { case(key, value) => (key, value._1/value._2) })

    avgScoresRDD.collect.foreach(println)
/*输出:
(zhangsan,85.5)
(lihua,85.0)
(xiaoming,93.0)
(wangwu,76.5)
*/
```   
**解释一下scoresWithKeyRDD.combineByKey**  
**createCombiner:**  (x: ScoreDetail) => (x.score, 1)  
这是第一次遇到zhangsan，创建一个函数，把map中的value转成另外一个类型   ，这里是把(zhangsan,(ScoreDetail类))**转换成**(zhangsan,(91,1))  
**mergeValue:** (acc: (Float, Int), x: ScoreDetail) => (acc._1 + x.score, acc._2 + 1)  再次碰到张三， 就把这两个合并, 这里是将(zhangsan,(91,1)) 这种类型 和 (zhangsan,(ScoreDetail类))这种类型合并，合并成了(zhangsan,(171,2))  
**mergeCombiners** (acc1: (Float, Int), acc2: (Float, Int))  这个是将多个分区中的zhangsan的数据进行合并， 我们这里zhansan在同一个分区，这个地方就没有用上  

## java版本的介绍
**ScoreDetail类**
```java

public class ScoreDetail implements Serializable{
    //case class ScoreDetail(studentName: String, subject: String, score: Float)
    public String studentName;
    public String subject;
    public float score;

    public ScoreDetail(String studentName, String subject, float score) {
        this.studentName = studentName;
        this.subject = subject;
        this.score = score;
    }
}
```
CombineByKey的测试类
```java
public class CombineTest {
    public static void main(String[] args) {
        SparkConf sparkConf = new SparkConf().setAppName("JavaWordCount").setMaster("local");
        JavaSparkContext sc = new JavaSparkContext(sparkConf);
        ArrayList<ScoreDetail> scoreDetails = new ArrayList<>();
        scoreDetails.add(new ScoreDetail("xiaoming", "Math", 98));
        scoreDetails.add(new ScoreDetail("xiaoming", "English", 88));
        scoreDetails.add(new ScoreDetail("wangwu", "Math", 75));
        scoreDetails.add(new ScoreDetail("wangwu", "Englist", 78));
        scoreDetails.add(new ScoreDetail("lihua", "Math", 90));
        scoreDetails.add(new ScoreDetail("lihua", "English", 80));
        scoreDetails.add(new ScoreDetail("zhangsan", "Math", 91));
        scoreDetails.add(new ScoreDetail("zhangsan", "English", 80));

        JavaRDD<ScoreDetail> scoreDetailsRDD = sc.parallelize(scoreDetails);

        JavaPairRDD<String, ScoreDetail> pairRDD = scoreDetailsRDD.mapToPair(new PairFunction<ScoreDetail, String, ScoreDetail>() {
            @Override
            public Tuple2<String, ScoreDetail> call(ScoreDetail scoreDetail) throws Exception {

                return new Tuple2<>(scoreDetail.studentName, scoreDetail);
            }
        });
//        new Function<ScoreDetail, Float,Integer>();

        Function<ScoreDetail, Tuple2<Float, Integer>> createCombine = new Function<ScoreDetail, Tuple2<Float, Integer>>() {
            @Override
            public Tuple2<Float, Integer> call(ScoreDetail scoreDetail) throws Exception {
                return new Tuple2<>(scoreDetail.score, 1);
            }
        };

        // Function2传入两个值，返回一个值
        Function2<Tuple2<Float, Integer>, ScoreDetail, Tuple2<Float, Integer>> mergeValue = new Function2<Tuple2<Float, Integer>, ScoreDetail, Tuple2<Float, Integer>>() {
            @Override
            public Tuple2<Float, Integer> call(Tuple2<Float, Integer> tp, ScoreDetail scoreDetail) throws Exception {
                return new Tuple2<>(tp._1 + scoreDetail.score, tp._2 + 1);
            }
        };
        Function2<Tuple2<Float, Integer>, Tuple2<Float, Integer>, Tuple2<Float, Integer>> mergeCombiners = new Function2<Tuple2<Float, Integer>, Tuple2<Float, Integer>, Tuple2<Float, Integer>>() {
            @Override
            public Tuple2<Float, Integer> call(Tuple2<Float, Integer> tp1, Tuple2<Float, Integer> tp2) throws Exception {
                return new Tuple2<>(tp1._1 + tp2._1, tp1._2 + tp2._2);
            }
        };
        JavaPairRDD<String, Tuple2<Float,Integer>> combineByRDD  = pairRDD.combineByKey(createCombine,mergeValue,mergeCombiners);

        //打印平均数
        Map<String, Tuple2<Float, Integer>> stringTuple2Map = combineByRDD.collectAsMap();
        for ( String et:stringTuple2Map.keySet()) {
            System.out.println(et+" "+stringTuple2Map.get(et)._1/stringTuple2Map.get(et)._2);
        }
    }
}
```
**注意有个坑的地方**  createCombine方法必须是这样的
```java
        Function<ScoreDetail, Tuple2<Float, Integer>> createCombine = new Function<ScoreDetail, Tuple2<Float, Integer>>() {
            @Override
            public Tuple2<Float, Integer> call(ScoreDetail scoreDetail) throws Exception {
                return new Tuple2<>(scoreDetail.score, 1);
            }
        };
```
**而不能是这样的**, 即使最后的RDD都类似
```java
        PairFunction<ScoreDetail, Float, Integer> createCombine = new PairFunction<ScoreDetail, Float, Integer>() {
            @Override
            public Tuple2<Float, Integer> call(ScoreDetail scoreDetail) throws Exception {
                return new Tuple2<>(scoreDetail.score, 1);
            }
        };
```

再推荐几篇比较好的文章 [LXW的大数据田地: combineByKey](http://lxw1234.com/archives/2015/07/358.htm)  
 推荐图书快速大数据分析中combineByKey的讲解

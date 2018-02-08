---
title: Hadoop入门案例（一） wordcount
date: 2016-07-16 21:25:21
tags: [hadoop]
categories: [大数据,hadoop]
author: kaishun
id: 13
permalink: hadoop-example-1
---

个人感想： 
虽然现在一直用的是Spark，但是由于目前公司是将Hadoop的程序转移到Spark，这个过程需要用到hadoop，加上现在大多数的大数据平台依旧还是使用的MapReduce的编程模型，所以MapReduce也应该是比较熟悉才行。并且，一般的大数据学习，都是从一些分布式计算框架开始看起，MapReduce一般都是必学的内容，接下来几篇文章，不讲述原理（因为原理网上太多了），只记录一些非常简单，但却非常实用应用案例，以便以后自己查看阅读
<!-- more -->

# **1. 需求说明**
大数据中，经常可能会碰到一些需要单词的出现个数，例如top n 等等。下面介绍一个hadoop的入门案例，对一个或多个文本中的单词进行统计
## 1.1 需求输入
输入一个或者多个文本  测试的文本内容如下
```
aa bb cc aa aa aa dd dd ee ee ee ee 
ff aa bb zks
ee kks
ee  zz zks
```
## **1.2 需求输出**
将文本中的内容按照单词进行计数，并且将各个单词的统计记录到制定的路径下
# **2. 代码如下**  
```java
package com.myhadoop.mapreduce.test;

import java.io.IOException;
import java.util.*;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.conf.*;
import org.apache.hadoop.io.*;
import org.apache.hadoop.mapreduce.lib.input.*;
import org.apache.hadoop.mapreduce.lib.output.*;
import org.apache.hadoop.mapreduce.*;


public class WordCount{

	public static class Map extends Mapper<LongWritable, Text, Text, IntWritable>
	{
		private final static IntWritable one = new IntWritable(1);
		private Text word = new Text();
		public void map(LongWritable key,Text value,Context context) throws IOException,InterruptedException
		{
			String lines = value.toString();
			StringTokenizer tokenizer = new StringTokenizer(lines," ");
			while(tokenizer.hasMoreElements())
			{
				word.set(tokenizer.nextToken());
				context.write(word, one);
			}
		}
	}

	public static class Reduce extends Reducer<Text, IntWritable, Text, IntWritable>
	{
		public void reduce(Text key,Iterable<IntWritable> values,Context context) throws IOException,InterruptedException
		{
			int sum = 0;
			for (IntWritable val : values) {
				sum += val.get();
			}
			context.write(key, new IntWritable(sum));
		}
	}

	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJarByClass(WordCount.class);
		job.setJobName("WordCount");

		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);

		job.setMapperClass(Map.class);
		job.setReducerClass(Reduce.class);

		job.setInputFormatClass(TextInputFormat.class);
		job.setOutputFormatClass(TextOutputFormat.class);
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		System.exit(job.waitForCompletion(true) ? 0 : 1);

	}
}



```  
# **3. 代码输出**
```
aa	5
bb	2
cc	1
dd	2
ee	6
ff	1
kks	1
zks	2
zz	1
```
# **4. 代码解析**  
原理：先将文本每行内容进行分割，每行都得到一些单词，将单词都转变成map，并且key设置成单词，value设置成1，经过shuffle，把相同key的value放在一个list中，reduce过程把  
相同key中的value进行相加，最后输出key为单词，value为总数
**Map类：**用于将每一行的单词变成map，如 (aa,1),(bb,1)...等。    
输入是： LongWritable, Text  
输出是： Text, IntWritable
**Reduce类：**用于将map后，并且经过shuffle的数据，进行整合处理  
输入是：Text，Iterable<IntWritable>  
输出是：Text, IntWritable  

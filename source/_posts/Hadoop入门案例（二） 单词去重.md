---
title: Hadoop入门案例（二） 单词去重
date: 2016-07-22 21:25:21
tags: [hadoop]
categories: [大数据,hadoop]
author: kaishun
id: 15
permalink: hadoop-example-2
blogexcerpt: 前言:单词去重在很多地方都会进行，其实这个就类似于wordcount。 需求说明:对指定的一个或者多个文本进行数据去重,需求输入一个或者多个文本，测试文本内容:输出的内容中单词没有重复,代码如下
---


# **前言**
单词去重在很多地方都会进行，其实这个就类似于wordcount
# **1. 需求说明**
对指定的一个或者多个文本进行数据去重
## **1.1 需求输入**
一个或者多个文本，测试文本内容:  
```
aa bb cc aa aa aa dd dd ee ee ee ee 
ff aa bb zks
ee kks
ee  zz zks
```
## **1.2 需求输出**
输出的内容中单词没有重复
# **2. 代码如下**  
```java
package com.myhadoop.mapreduce.test;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import java.io.IOException;
import java.util.StringTokenizer;


public class Dedup{

	public static class Map extends Mapper<LongWritable, Text, Text, Text>
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
				context.write(word, new Text(""));
			}
		}
	}

	public static class Reduce extends Reducer<Text, Text, Text, Text>
	{
		public void reduce(Text key,Iterable<Text> values,Context context) throws IOException,InterruptedException
		{
			context.write(key, new Text(""));
		}
	}

	public static void main(String[] args) throws Exception {
		Job job = Job.getInstance();
		job.setJarByClass(Dedup.class);
		job.setJobName("Dedup");

		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);

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
aa	
bb	
cc	
dd	
ee	
ff	
kks	
zks	
zz
```
# 4. **代码解析**  
整体非常类似于wordcount，先将文本每行内容进行分割，每行都得到一些单词，将单词都转变成map，并且key设置成单词，value设置成空值，经过shuffle，把相同key的放在一组，在reduce中把  
相同key中的value变成一个空值，然后输出(word,"")的形式
**Map类：**  
输入： LongWritable, Text
输出： Text, Text  
**Reduce类：**  
输入：Text, Text ---> Text, Interable<Text>
输出：Text, Text  
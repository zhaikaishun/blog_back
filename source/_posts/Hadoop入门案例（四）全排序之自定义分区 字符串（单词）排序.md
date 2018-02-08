---
title: Hadoop入门案例（四）全排序之自定义分区 字符串（单词）排序
date: 2016-08-01 21:25:21
tags: [hadoop]
categories: [大数据,hadoop]
author: kaishun
id: 17
permalink: hadoop-example-4
blogexcerpt: 大量文本中有很多单词，需要对这些单词进行排序，排序规则按照字符进行排序。 和上一篇对数字进行排序是一样的 http://blog.csdn.net/T1DMzks/article/details/73028776 ， 只不过是自定义分区有点变化, 利用mapReduce中map到reduce端的shuffle进行排序，MapReduce只能保证各个分区内部有序，但不能保证全局有序，于是我还自定义了分区，在map后、shuffle之前，我先将小于c的放在0分区，c-f的放在1分区，其余的放在2分区，这样，首先保证了分区与分区之间是整体有序，然后各个分区进行各自的shuffle，使其分区内部有序。
---


## 需求  
大量文本中有很多单词，需要对这些单词进行排序，排序规则按照字符进行排序

## 测试文本
```
ba bac
df gh hgg dft dfa dfga df fdaf qqq we fsf aa bb ab
rr
ty ioo zks huawei mingtong jyzt beijing shanghai shenzhen wuhan nanning guilin 
zhejiang hanzhou anhui hefei xiaoshan xiaohao anqian zheli guiyang
```
## 原理分析 
和上一篇对数字进行排序是一样的 http://blog.csdn.net/T1DMzks/article/details/73028776 ， 只不过是自定义分区有点变化, 利用mapReduce中map到reduce端的shuffle进行排序，MapReduce只能保证各个分区内部有序，但不能保证全局有序，于是我还自定义了分区，在map后、shuffle之前，我先将小于c的放在0分区，c-f的放在1分区，其余的放在2分区，这样，首先保证了分区与分区之间是整体有序，然后各个分区进行各自的shuffle，使其分区内部有序。

## 代码
```java
package com.myhadoop.mapreduce.test;

import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.Mapper;
import org.apache.hadoop.mapred.OutputCollector;
import org.apache.hadoop.mapreduce.Partitioner;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapred.Reporter;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;
import java.io.IOException;

/**
 * Created by kaishun on 2017/6/10.
 */
public class TotalSortTest extends Configured implements Tool{

    public static class myMap extends org.apache.hadoop.mapreduce.Mapper<LongWritable, Text, Text, Text>{

        public void map(LongWritable key,Text value,Context context) throws IOException,InterruptedException{
            String[] split = value.toString().split("\\s+");
            for (int i = 0; i <split.length ; i++) {
                Text word = new Text(split[i]);
                context.write(word,new Text(""));
            }
        }
    }
    public static class myReduce extends Reducer<Text,Text,Text,Text>{

        public void reduce(Text key, Iterable<Text> values,Context context) throws IOException,InterruptedException
        {
                context.write(key, new Text(""));

        }
    }
    public static class Partition extends Partitioner<Text,Text>{
        @Override
        public int getPartition(Text value1, Text value2, int i) {

            if(value1.toString().compareTo("c")<0){
                return 0;
            }else if(value1.toString().compareTo("f")<0){
                return 1;
            }
            return 2;
        }
    }



    @Override
    public int run(String[] args) throws Exception {
        Job job = Job.getInstance(getConf());
        job.setJarByClass(TotalSort.class);
        job.setJobName("TotalSortTest");
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        job.setPartitionerClass(Partition.class);
        job.setMapperClass(myMap.class);
        job.setReducerClass(myReduce.class);
        job.setNumReduceTasks(3);

        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        boolean success = job.waitForCompletion(true);
        return success ? 0:1;
    }
    public static void main(String[] args) throws Exception {
        int ret = ToolRunner.run(new TotalSortTest(), args);
        System.exit(ret);
    }
}

```

## 测试结果
生成了三个文件part-r-00000，part-r-00001，part-r-00002
各个分区之间有顺序，分区内部也有顺序，分别为
```
aa	
ab	
anhui	
anqian	
ba	
bac	
bb	
beijing	

```
```
df	
dfa	
dfga	
dft	

```
```
fdaf	
fsf	
gh	
guilin	
guiyang	
hanzhou	
hefei	
hgg	
huawei	
ioo	
jyzt	
mingtong	
nanning	
qqq	
rr	
shanghai	
shenzhen	
ty	
we	
wuhan	
xiaohao	
xiaoshan	
zhejiang	
zheli	
zks	

```  

## 总结
mapreduce的shuffle是对key值得hashcode进行排序的，所以单词的全排序也是一样的，类似于数据库中的order by 一样， 利用自定义分区，保证整体有序，利用mapreduce内部的shuffle，对key进行排序，保证了局部有序，从而实现了全排序

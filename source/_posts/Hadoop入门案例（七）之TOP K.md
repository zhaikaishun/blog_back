---
title: Hadoop入门案例（七）之TOP K
date: 2016-08-05 21:25:21
tags: [hadoop]
categories: [大数据,hadoop]
author: kaishun
id: 20
permalink: hadoop-example-7
blogexcerpt: 目的：找出数据集中的top k，二. 思路1. 最开始是快速排序或者归并排序2. 其次就是wordcount，然后再进行一遍mapReduce 3. 先排序，再取前k个2.2 在mapper阶段，找出本地的top k, 然后所有的独立的top k集合在reduce中运算
---


## **一. 目的：**
找出数据集中的top k，




## **二. 思路**
### 2.1 全排序，取前 k 个
1. 最开始是快速排序或者归并排序
2. 其次就是wordcount，然后再进行一遍mapReduce
3. 先排序，再取前k个

### 2.2 在mapper阶段，找出本地的top k, 然后所有的独立的top k集合在reduce中运算  
因为k 一般比较小，所以我们只需要一个reduce来处理最后的运算
          
### 2.3 流程图如下  
![topk](http://or49tneld.bkt.clouddn.com/17-6-25/82113418.jpg)

### **三. 代码**
代码中利用了 treemap 来获取前k个，Treemap 参考 http://blog.csdn.net/chenssy/article/details/26668941  
其实也可以用其他的一些结构，例如参考这篇文章， 使用的是数组 https://my.oschina.net/u/1378204/blog/343666   

```java
package com.myhadoop.mapreduce.test;
import java.io.IOException;
import java.util.StringTokenizer;
import java.util.TreeMap;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class TopN {
    public static class TopTenMapper extends
            Mapper<Object, Text, NullWritable, IntWritable> {
        private TreeMap<Integer, String> repToRecordMap = new TreeMap<Integer, String>();

        public void map(Object key, Text value, Context context) {
            int N = 10; //默认为Top10
            N = Integer.parseInt(context.getConfiguration().get("N"));
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                repToRecordMap.put(Integer.parseInt(itr.nextToken()), " ");
                if (repToRecordMap.size() > N) {
                    repToRecordMap.remove(repToRecordMap.firstKey());
                }
            }
        }



        protected void cleanup(Context context) {
            for (Integer i : repToRecordMap.keySet()) {
                try {
                    context.write(NullWritable.get(), new IntWritable(i));
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static class TopTenReducer extends
            Reducer<NullWritable, IntWritable, NullWritable, IntWritable> {
        private TreeMap<Integer, String> repToRecordMap = new TreeMap<Integer, String>();

        public void reduce(NullWritable key, Iterable<IntWritable> values,
                           Context context) throws IOException, InterruptedException {
            int N = 10; //默认为Top10
            N = Integer.parseInt(context.getConfiguration().get("N"));
            for (IntWritable value : values) {
                repToRecordMap.put(value.get(), " ");
                if (repToRecordMap.size() > N) {
                    repToRecordMap.remove(repToRecordMap.firstKey());
                }
            }
            for (Integer i : repToRecordMap.descendingMap().keySet()) {
                context.write(NullWritable.get(), new IntWritable(i));
            }
        }

    }

    public static void main(String[] args) throws Exception {
        if (args.length != 3) {
            throw new IllegalArgumentException(
                    "!!!!!!!!!!!!!! Usage!!!!!!!!!!!!!!: hadoop jar <jar-name> "
                            + "TopN.TopN "
                            + "<the value of N>"
                            + "<input-path> "
                            + "<output-path>");
        }
        Configuration conf = new Configuration();
        conf.set("N", args[0]);
        Job job = Job.getInstance(conf, "TopN");
        job.setJobName("TopN");
        Path inputPath = new Path(args[1]);
        Path outputPath = new Path(args[2]);
        FileInputFormat.setInputPaths(job, inputPath);
        FileOutputFormat.setOutputPath(job, outputPath);
        job.setJarByClass(TopN.class);
        job.setMapperClass(TopTenMapper.class);
        job.setReducerClass(TopTenReducer.class);
        job.setNumReduceTasks(1);  //reduce Num 设置成1

        job.setMapOutputKeyClass(NullWritable.class);// map阶段的输出的key
        job.setMapOutputValueClass(IntWritable.class);// map阶段的输出的value

        job.setOutputKeyClass(NullWritable.class);// reduce阶段的输出的key
        job.setOutputValueClass(IntWritable.class);// reduce阶段的输出的value

        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }

}
```


## **四. 结果**
上述代码经过测试，能返回Top k条记录

## **五. 性能分析**

上述代码使用的是第二种思路，避免了第一种思路的全排序，但是注意到，我们只能用一个reduce，如果数据量特别大，k也非常大，单一的reduce可能会出现一些问题    
1. 这个reducer的主机需要通过网络获取大量数据，会造成单一节点工作负荷太大。  
2. reducer的内存中可能会出现java虚拟机内存不足    
3. 写文件不是并行的，当数据规模很大的时候，这种思路会导致效率变得很低


  
参考自 mappreduce 设计模式
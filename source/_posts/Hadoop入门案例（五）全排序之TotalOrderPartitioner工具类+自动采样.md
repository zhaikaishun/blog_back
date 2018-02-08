---
title: Hadoop入门案例（五）全排序之TotalOrderPartitioner工具类+自动采样
date: 2016-08-02 21:25:21
tags: [hadoop]
categories: [大数据,hadoop]
author: kaishun
id: 18
permalink: hadoop-example-5
---



**为什么用这种方法** 
我们之前的是自定义分区的，但是如果我们不知道数据的分布，手动分区不太容易，稍有不慎，会导致数据倾斜较大，这时候，我们应该使用采样点进行排序，本文使用的是 Hadoop内置的名为 TotalOrderPartitioner 的全排序，采样器使用的是 InputSampler.Sampler，关键解释已经存在于代码之中。 文章参考了    
国外  http://blog.ditullio.fr/2016/01/04/hadoop-basics-total-order-sorting-mapreduce/#The_TotalOrderPartitioner    
国内  http://www.cnblogs.com/one--way/p/5931308.html
<!-- more -->
## **代码**
```
package com.myhadoop.mapreduce.test;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.lib.InputSampler;
import org.apache.hadoop.mapred.lib.TotalOrderPartitioner;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.KeyValueTextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;
import java.io.IOException;

/**
 * Created by kaishun on 2017/6/10.
 */
public class TotalOrderSort extends Configured implements Tool{

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

    @Override
    public int run(String[] args) throws Exception {
        Job job = Job.getInstance(getConf());
        job.setJarByClass(TotalSort.class);
        job.setJobName("TotalSortTest");


        job.setInputFormatClass(KeyValueTextInputFormat.class);

        job.setNumReduceTasks(3);

        //因为map和reduce的输出是同样的类型，所以输出一个就可以了
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        job.setMapperClass(myMap.class);
        job.setReducerClass(myReduce.class);

        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 设置分区文件，即采样后放在的文件的文件名，不是完整路径
        TotalOrderPartitioner.setPartitionFile(job.getConfiguration(), new Path(args[2]));
         //采样器：三个参数
        /* 第一个参数 freq: 表示来一个样本，将其作为采样点的概率。如果样本数目很大
         *第二个参数 numSamples：表示采样点最大数目为，我这里设置10代表我的采样点最大为10，如果超过10，那么每次有新的采样点生成时
         * ，会删除原有的一个采样点,此参数大数据的时候尽量设置多一些
         * 第三个参数 maxSplitSampled：表示的是最大的分区数：我这里设置100不会起作用，因为我设置的分区只有4个而已
         */

        InputSampler.Sampler<Text, Text> sampler = new InputSampler.RandomSampler<>(0.01, 10, 100);

        //把分区文件放在hdfs上，对程序没什么效果，方便我们查看而已
        FileInputFormat.addInputPath(job, new Path("/test/sort"));
        //将采样点写入到分区文件中，这个必须要
        InputSampler.writePartitionFile(job, sampler);
        job.setPartitionerClass(TotalOrderPartitioner.class);

        boolean success = job.waitForCompletion(true);
        return success ? 0:1;
    }
    public static void main(String[] args) throws Exception {
        int ret = ToolRunner.run(new TotalSortTest(), args);
        System.exit(ret);
    }
}


```
## **注意的地方**  

```
 InputSampler.Sampler<Text, Text> sampler = new InputSampler.RandomSampler<>(0.01, 10, 100);中三个参数要注意
``` 
```
InputSampler.Sampler<Text, Text> 只能是Text,Text的类型
```
```
3. TotalOrderPartitioner.setPartitionFile(job.getConfiguration(), new Path(args[2]));用来给TotalOrderPartitioner初始化赋值，job.setPartitionerClass(TotalOrderPartitioner.class); 进行分区，就不需要自己写分区函数了  
```
```
4. job.setInputFormatClass(KeyValueTextInputFormat.class); 注意里面是KeyValueTextInputFormat.class，而不是TextInputFormat.class。
```
```
5. 在集群上，次程序才能体现出来
```

6.由于我这里，map的输入和输出都是用的(Text,Text),所以我只需要设置 
```
job.setOutputKeyClass(Text.class); 
job.setOutputValueClass(Text.class);
```
如果不一样，那么 应该设置4个,前两个为map的输出类型，后两个为reduce的输出类型
```
job.setMapOutputKeyClass(Text.class);
job.setMapOutputValueClass(IntWritable.class);
job.setOutputKeyClass(IntWritable.class);
job.setOutputValueClass(NullWritable.class);
```  


更多文章：
自定义分区全排序  
  http://blog.csdn.net/t1dmzks/article/details/73032796    
http://blog.csdn.net/t1dmzks/article/details/73028776

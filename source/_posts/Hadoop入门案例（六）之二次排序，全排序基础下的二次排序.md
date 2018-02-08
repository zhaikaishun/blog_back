---
title: Hadoop入门案例（六）之二次排序，全排序基础下的二次排序
date: 2016-08-03 21:25:21
tags: [hadoop]
categories: [大数据,hadoop]
author: kaishun
id: 19
permalink: hadoop-example-6
blogexcerpt: 对于二次排序，定义是在一个字段排好顺序的前提下，另外一个字段也进行排序。类似于sql中的order by多个字段，然而，网上大多数二次排序，对于partitioner都没有利用，因为网上的partition竟然都是是按照hash分组的，而且也没设置reduceTaskNum，这不能充分利用集群的资源，只有一个reduce，，即使如果有多个reduce，partition按照hash分区也不属于全排序，准确的说属于分组排序。我对网上的代码做了稍加修改，可以满足全排序的情况下，再进行二次排序。测试数和代码都参考自网上，具体哪个不记得了，反正网上都差不多测试数据
---


## **前言**：
对于二次排序，定义是在一个字段排好顺序的前提下，另外一个字段也进行排序。类似于sql中的order by多个字段，然而，网上大多数二次排序，对于partitioner都没有利用，因为网上的partition竟然都是是按照hash分组的，而且也没设置reduceTaskNum，这不能充分利用集群的资源，只有一个reduce，，即使如果有多个reduce，partition按照hash分区也不属于全排序，准确的说属于分组排序。 我对网上的代码做了稍加修改，可以满足全排序的情况下，再进行二次排序。  
测试数和代码都参考自网上，具体哪个不记得了，反正网上都差不多
测试数据
```
20 21 
50 51 
50 52 
50 53 
50 54 
60 51 
60 53 
60 52 
60 56 
60 57 
70 58 
60 61 
70 54 
70 55 
70 56 
70 57 
70 58 
1 2 
3 4 
5 6 
7 82 
203 21 
50 512 
50 522 
50 53 
530 54 
40 511 
20 53 
20 522 
60 56 
60 57 
740 58 
63 61 
730 54 
71 55 
71 56 
73 57 
74 58 
12 211 
31 42 
50 62 
7 8
```

## **代码如下**
```java
package com.myhadoop.mapreduce.test;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.*;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Partitioner;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import java.util.StringTokenizer;

/**
 * Created by Administrator on 2017/6/14.
 */
public class SecondSort {
    public static class IntPair implements WritableComparable<IntPair>
    {
        int first;
        int second;
        /**
         * Set the left and right values.
         */
        public void set(int left, int right)
        {
            first = left;
            second = right;
        }
        public int getFirst()
        {
            return first;
        }
        public int getSecond()
        {
            return second;
        }
        @Override
        //反序列化，从流中的二进制转换成IntPair
        public void readFields(DataInput in) throws IOException
        {
            // TODO Auto-generated method stub
            first = in.readInt();
            second = in.readInt();
        }
        @Override
        //序列化，将IntPair转化成使用流传送的二进制
        public void write(DataOutput out) throws IOException
        {
            // TODO Auto-generated method stub
            out.writeInt(first);
            out.writeInt(second);
        }
        @Override
        //key的比较
        public int compareTo(IntPair o)
        {
            // TODO Auto-generated method stub
            if (first != o.first)
            {
                return first < o.first ? -1 : 1;
            }
            else if (second != o.second)
            {
                return second < o.second ? -1 : 1;
            }
            else
            {
                return 0;
            }
        }

        //新定义类应该重写的两个方法
        @Override
        //The hashCode() method is used by the HashPartitioner (the default partitioner in MapReduce)
        public int hashCode()
        {
            return first * 157 + second;
        }
        @Override
        public boolean equals(Object right)
        {
            if (right == null)
                return false;
            if (this == right)
                return true;
            if (right instanceof IntPair)
            {
                IntPair r = (IntPair) right;
                return r.first == first && r.second == second;
            }
            else
            {
                return false;
            }
        }
    }

    /**
     * 分区函数类。根据first确定Partition。
     */
    public static class FirstPartitioner extends Partitioner<IntPair, IntWritable>
    {
        @Override
        public int getPartition(IntPair key, IntWritable value,int numPartitions)
        {
            int first = key.getFirst();
            if(first<60){
                return 0;
            }else if(first<80){
                return 1;
            }
            return 2;


        }
    }

    /**
     * 分组函数类。只要first相同就属于同一个组。
     */
    public static class GroupingComparator extends WritableComparator
    {
        protected GroupingComparator()
        {
            super(IntPair.class, true);
        }
        @Override
        //Compare two WritableComparables.
        public int compare(WritableComparable w1, WritableComparable w2)
        {
            IntPair ip1 = (IntPair) w1;
            IntPair ip2 = (IntPair) w2;
            int l = ip1.getFirst();
            int r = ip2.getFirst();
            return l == r ? 0 : (l < r ? -1 : 1);
        }
    }
    
    // 自定义map
    public static class Map extends Mapper<LongWritable, Text, IntPair, IntWritable>
    {
        private final IntPair intkey = new IntPair();
        private final IntWritable intvalue = new IntWritable();
        public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
        {
            String line = value.toString();
            StringTokenizer tokenizer = new StringTokenizer(line);
            int left = 0;
            int right = 0;
            if (tokenizer.hasMoreTokens())
            {
                left = Integer.parseInt(tokenizer.nextToken());
                if (tokenizer.hasMoreTokens())
                    right = Integer.parseInt(tokenizer.nextToken());
                intkey.set(left, right);
                intvalue.set(right);
                context.write(intkey, intvalue);
            }
        }
    }
    // 自定义reduce
    //
    public static class Reduce extends Reducer<IntPair, IntWritable, Text, IntWritable>
    {
        private final Text left = new Text();
        private static final Text SEPARATOR = new Text("------------------------------------------------");

        public void reduce(IntPair key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException
        {
            context.write(SEPARATOR, null);
            left.set(Integer.toString(key.getFirst()));
            for (IntWritable val : values)
            {
                context.write(left, val);
            }
        }
    }
    /**
     * @param args
     */
    public static void main(String[] args) throws IOException, InterruptedException, ClassNotFoundException
    {
        // TODO Auto-generated method stub
        // 读取hadoop配置
        Configuration conf = new Configuration();
        // 实例化一道作业
        Job job = new Job(conf, "secondarysort");
        job.setJarByClass(SecondSort.class);
        // Mapper类型
        job.setMapperClass(Map.class);
        // 不再需要Combiner类型，因为Combiner的输出类型<Text, IntWritable>对Reduce的输入类型<IntPair, IntWritable>不适用
        //job.setCombinerClass(Reduce.class);
        // Reducer类型
        job.setReducerClass(Reduce.class);
        //设置NumReduceTasks 的数量为三
        job.setNumReduceTasks(3);
        // 分区函数
        job.setPartitionerClass(FirstPartitioner.class);


        // 分组函数
        job.setGroupingComparatorClass(GroupingComparator.class);

        // map 输出Key的类型
        job.setMapOutputKeyClass(IntPair.class);
        // map输出Value的类型
        job.setMapOutputValueClass(IntWritable.class);
        // rduce输出Key的类型，是Text，因为使用的OutputFormatClass是TextOutputFormat
        job.setOutputKeyClass(Text.class);
        // rduce输出Value的类型
        job.setOutputValueClass(IntWritable.class);

        // 将输入的数据集分割成小数据块splites，同时提供一个RecordReder的实现。
        job.setInputFormatClass(TextInputFormat.class);
        // 提供一个RecordWriter的实现，负责数据输出。
        job.setOutputFormatClass(TextOutputFormat.class);

        // 输入hdfs路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        // 输出hdfs路径
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        // 提交job
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}

```  
## 运行结果
生成三个文件 partition-00000，partition-00001，partition-00002  
每个文件第一个字段都按照进行了全排序，满足上一个条件下对第二个字段进行了排序，并且按照第一个字段进行了分组  
输出的文件分别为 
**partition-00000**
```
------------------------------------------------ 
1       2 
------------------------------------------------ 
3       4 
------------------------------------------------ 
5       6 
------------------------------------------------ 
7       8 
7       82 
------------------------------------------------ 
12      211 
------------------------------------------------ 
20      21 
20      53 
20      522 
------------------------------------------------ 
31      42 
------------------------------------------------ 
40      511 
------------------------------------------------ 
50      51 
50      52 
50      53 
50      53 
50      54 
50      62 
50      512 
50      522 
```
**partition-00001**
```
------------------------------------------------ 
60      51 
60      52 
60      53 
60      56 
60      56 
60      57 
60      57 
60      61 
------------------------------------------------ 
63      61 
------------------------------------------------ 
70      54 
70      55 
70      56 
70      57 
70      58 
70      58 
------------------------------------------------ 
71      55 
71      56 
------------------------------------------------ 
73      57 
------------------------------------------------ 
74      58 
```
**partion-00003**
```
------------------------------------------------ 
203     21 
------------------------------------------------ 
530     54 
------------------------------------------------ 
730     54 
------------------------------------------------ 
740     58
```

## **分析：**
网上说的已经够清楚了，我这代码主要也是参考于网上的，主要有三步  
1. 自定义一个对象，需要实现WritableComparable接口，这个对象包含要排序的字段，并且内部有一些方法需要重写，具体参考我的代码  
2. 自定义分区，网上的hash分区是有小问题的，网上的要么第一个字段不属于全排序，要么只有一个reduce，我们这里参考全排序，自己定义分区，保证整体有序  
3. 自定义分组（没要求就不需要了），比如只按照第一个字段分组，参考代码中
4. 正常的map，reduce，map中的key使用我们自定义的对象

## 感悟：
主要代码还是网上的，但是做程序开发还是要有自己的独立思考，网上的并不是真正的二次排序，或者说即使满足二次排序，但是也是只有一个reduce任务
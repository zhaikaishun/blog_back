---
title: Hadoop入门案例（三）全排序之自定义分区 数字排序
date: 2016-07-25 21:25:21
tags: [hadoop]
categories: [大数据,hadoop]
author: kaishun
id: 16
permalink: hadoop-example-3
blogexcerpt: 需求介绍：大量的文本中有大量数字，需要对数字进行全排序,按照升序排序。 原理分析 利用mapReduce中map到reduce端的shuffle进行排序，MapReduce只能保证各个分区内部有序，但不能保证全局有序，于是我还自定义了分区，在map后、shuffle之前，我先将小于50的放在0分区，50-100的放在1分区，100到150的放在2分区，其余的放在三分区，这样，首先保证了分区与分区之间是整体有序，然后各个分区进行各自的shuffle，使其分区内部有序
---


## **需求介绍**  
大量的文本中有大量数字，需要对数字进行全排序,按照升序排序  
## **测试的文本**
```
145 
95 167 84 6 120 164 195 81 35 63 1 11 89 170 55 
58 88 125 173 2 173 129 74 69 24 107 55 149 83 178 
159 147 178 53 137 53 132 134 154 174 164 122 108 130 184 
28 129 93 157 171 127 192 86 194 41 111 114 190 98 99 
99 5 161 146 120 122 80 1 66 171 47 54 121 130 170 
125 119 8 52 182 112 146 1 198 0 149 72 56 191 48 
172 165 49 73 107 134 179 0 59 16 143 83 92 113 152 
109 118 186 186 97 117 193 67 34 152 92 179 52 51 26 
163 121 115 72 17 61 107 125 115 163 18 76 2 172 39 
190 184 73 108 7 142 68 54 60 169 71 28 141 48 139 
182 140 158 102 99 36 158 55 190 176 45 63 126 179 130 
95 22 120 109 59 78 38 13 5 88 1 87 184 83 198 
47 73 82 94 141 190 184 161 56 141 99 177 107 21 158 
71 149 61 137 
```  
## **原理分析**  
利用mapReduce中map到reduce端的shuffle进行排序，MapReduce只能保证各个分区内部有序，但不能保证全局有序，于是我还自定义了分区，在map后、shuffle之前，我先将小于50的放在0分区，50-100的放在1分区，100到150的放在2分区，其余的放在三分区，这样，首先保证了分区与分区之间是整体有序，然后各个分区进行各自的shuffle，使其分区内部有序
## **代码**  
``` java
package com.myhadoop.mapreduce.test;

import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Partitioner;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

import java.io.IOException;

public class TotalSort extends Configured implements Tool{

    public static class Map extends Mapper<LongWritable, Text, IntWritable, IntWritable>
    {
        public void map(LongWritable key,Text value,Context context) throws IOException,InterruptedException
        {
            String[] split = value.toString().split("\\s+");
            for (int i = 0; i <split.length ; i++) {
                try{
                    IntWritable intWritable = new IntWritable(Integer.parseInt(split[i]));
                    context.write(intWritable, intWritable);
                }catch (Exception e){
                    e.printStackTrace();
                }
            }


        }
    }

    public static class Reduce extends Reducer<IntWritable, IntWritable, IntWritable, Text>
    {
        public void reduce(IntWritable key,Iterable<IntWritable> values,Context context) throws IOException,InterruptedException
        {
            for (IntWritable value:values) {
                context.write(value, new Text(""));
            }
        }
    }

    /*
    ·* 重写Partition的方法
     */
    public static class Partition extends Partitioner<IntWritable, IntWritable>{
        @Override
        public int getPartition(IntWritable key, IntWritable intWritable2, int i) {
            int i1=key.get();
            if(i1<50){
                return 0;
            }else if(i1<100){
                return 1;
            }else if(i1<150){
                return 2;
            }
            return 3;

        }
    }

    @Override
    public int run(String[] args) throws Exception {
        Job job = Job.getInstance(getConf());
        job.setJarByClass(TotalSort.class);
        job.setJobName("TotalSort");

        job.setOutputKeyClass(IntWritable.class);
        job.setOutputValueClass(IntWritable.class);

        job.setPartitionerClass(Partition.class);
        job.setMapperClass(Map.class);
        job.setReducerClass(Reduce.class);
        job.setNumReduceTasks(4);

        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        boolean success = job.waitForCompletion(true);
        return success ? 0:1;
    }

    public static void main(String[] args) throws Exception {
        int ret = ToolRunner.run(new TotalSort(), args);
        System.exit(ret);
    }
}

```  
## **运行结果**
生成了4个文件，part-r-00000，part-r-00001，part-r-00002，part-r-00003，这四个文件内部都有序
part-r-00000内的元素都小于part-r00001的元素，其他的以此类推  

## **总结**
利用自定义分区，保证整体有序，利用mapreduce内部的shuffle，对key进行排序，保证了局部有序，从而实现了全排序
---
title: Hadoop入门案例（八）之 表 关联
date: 2016-08-20 21:25:21
tags: [hadoop]
categories: [大数据,hadoop]
author: kaishun
id: 21
permalink: hadoop-example-8
blogexcerpt: 表关联:两个表关联，代码和以后再贴一起在github上贴出来，mapreduce表关联思路很简单。主要如下 思路： mapper阶段:(key，value) value中加一个字符，用来区别多个表 .reduce时 new 多个数组，例如我们是两个表关联，通过value中的字符，用来判断哪个是哪个表，分别放在不同的数组中，然后数组中进行笛卡尔乘积即可
---



表关联:
两个表关联，代码和以后再贴一起在github上贴出来，mapreduce表关联思路很简单。主要如下  

## 思路
**mapper阶段:** 
(key，value) value中加一个字符，用来区别多个表  
**reduce阶段**:
reduce时，new 多个数组，例如我们是两个表关联，通过value中的字符，用来判断哪个是哪个表，分别放在不同的数组中，然后数组中进行笛卡尔乘积即可 
**如果是单表关联**：
找出单表中哪些字段需要关联，理清楚思路，和sql中的单表join一样，和多表关联相同的思路

## 优化的地方
看具体需求如何，如果是大小表，最好不要用这种方法，因为会经过shuffle，导致大量资源浪费。可以在map段就做关联，而不经过reduce。类似于spark中，用flatmap代替join的方法  
下面是网上常见的方法，我觉得自己的笔记，用这个代码就挺好的了，如果是多表关联，这网上关于爷孙的关联，也是很好的入门


```
public class LeftJoin extends Configured implements Tool{

    public static final String DELIMITER = ",";

    public static class LeftJoinMapper extends
        Mapper<LongWritable,Text,Text,Text> {

        protected void map(LongWritable key, Text value, Context context)
            throws IOException,InterruptedException {
            /*
             * 拿到两个不同文件，区分出到底是哪个文件，然后分别输出
             */
            String filepath = ((FileSplit)context.getInputSplit()).getPath().toString();
            String line = value.toString();
            if (line == null || line.equals("")) return; 

            if (filepath.indexOf("employee") != -1) {
                String[] lines = line.split(DELIMITER);
                if(lines.length < 2) return;

                String company_id = lines[0];
                String employee = lines[1];
                context.write(new Text(company_id),new Text("a:"+employee));
            }

            else if(filepath.indexOf("salary") != -1) {
                String[] lines = line.split(DELIMITER);
                if(lines.length < 2) return;

                String company_id = lines[0];
                String salary = lines[1];
                context.write(new Text(company_id), new Text("b:" + salary));
            }
        }
    }

    public static class LeftJoinReduce extends 
            Reducer<Text, Text, Text, Text> {
        protected void reduce(Text key, Iterable<Text> values,
                Context context) throws IOException, InterruptedException{
            Vector<String> vecA = new Vector<String>();
            Vector<String> vecB = new Vector<String>();

            for(Text each_val:values) {
                String each = each_val.toString();
                if(each.startsWith("a:")) {
                    vecA.add(each.substring(2));
                } else if(each.startsWith("b:")) {
                    vecB.add(each.substring(2));
                }
            }

            for (int i = 0; i < vecA.size(); i++) {
                /*
                 * 如果vecB为空的话，将A里的输出，B的位置补null。
                 */
                if (vecB.size() == 0) {
                    context.write(key, new Text(vecA.get(i) + DELIMITER + "null"));
                } else {
                    for (int j = 0; j < vecB.size(); j++) {
                        context.write(key, new Text(vecA.get(i) + DELIMITER + vecB.get(j)));
                    }
                }
            }
        }
    }

    public int run(String[] args) throws Exception {
        Configuration conf = getConf();
        GenericOptionsParser optionparser = new GenericOptionsParser(conf, args);
        conf = optionparser.getConfiguration();

        Job job = new Job(conf,"leftjoin");
        job.setJarByClass(LeftJoin.class);
        FileInputFormat.addInputPaths(job, conf.get("input_dir"));
        Path out = new Path(conf.get("output_dir"));
        FileOutputFormat.setOutputPath(job, out);
        job.setNumReduceTasks(conf.getInt("reduce_num",2));

        job.setMapperClass(LeftJoinMapper.class);
        job.setReducerClass(LeftJoinReduce.class);
        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        conf.set("mapred.textoutputformat.separator", ",");

        return (job.waitForCompletion(true) ? 0 : 1);
    }

    public static void main(String[] args) throws Exception{
        int res = ToolRunner.run(new Configuration(),new LeftJoin(),args);
        System.exit(res);
    }

}
```
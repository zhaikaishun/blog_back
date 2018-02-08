---
title: HDFS获取目录大小API
date: 2016-05-16 23:13:21
tags: [hdfs]
categories: [大数据,hdfs]
author: kaishun
id: 2
permalink: hdfs-ContentSummary-directory
blogexcerpt: 获取文件大小，在命令行上，使用hadoop fs -du 命令可以，但是通过javaAPI怎么获取呢，最开始我想到的是递归的方法，这个方法很慢，后来发现FileSystem.getContentSummary的方法,最慢的一个方法--递归,使用FileSystem.getContentSummary方法,
---

获取文件大小，在命令行上，使用hadoop fs -du 命令可以，但是通过javaAPI怎么获取呢，
最开始我想到的是递归的方法，这个方法很慢，后来发现FileSystem.getContentSummary的方法

# **最慢的一个方法--递归**

```java
网上很多类似的方法，不建议
```

# **使用FileSystem.getContentSummary方法**
下面是api的一句话： 

The getSpaceConsumed() function in the ContentSummary class will return the actual space the file/directory occupies in the cluster i.e. it takes into account the replication factor set for the cluster.

For instance, if the replication factor in the hadoop cluster is set to 3 and the directory size is 1.5GB, the getSpaceConsumed() function will return the value as 4.5GB.

Using getLength() function in the ContentSummary class will return you the actual file/directory size.  
示例代码如下

```java
 public static void main(String[] args) {
        FileSystem hdfs = null;

        Configuration conf = new Configuration();

        try {
            hdfs = FileSystem.get(new URI("hdfs://192.xxx.xx.xx:9000"),conf,"username");
        } catch (Exception e) {
            e.printStackTrace();
        }

        Path filenamePath = new Path("/test/input");
        try {
            //会根据集群的配置输出，例如我这里输出3G
            System.out.println("SIZE OF THE HDFS DIRECTORY : " + hdfs.getContentSummary(filenamePath).getSpaceConsumed());
           // 显示实际的输出，例如这里显示 1G
            System.out.println("SIZE OF THE HDFS DIRECTORY : " + hdfs.getContentSummary(filenamePath).getLength());
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

```
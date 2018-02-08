---
title: HDFS java操作（二）FileStatus 获取文件属性，globStatus 进行路径过滤
date: 2016-06-28 21:20:21
tags: [hdfs]
categories: [大数据,hdfs]
author: kaishun
id: 11
permalink: hdfs-filesystem-2
blogexcerpt: 前言，本章主要记录了如何使用fileStatus来获取hdfs文件的一些属性，以及如何使用globStatus对路径进行过滤, filestatus 获取文件状态.globStatus 路径过滤等等 
---

# 前言：  
本章主要记录了如何使用fileStatus来获取hdfs文件的一些属性，以及如何使用globStatus对路径进行过滤, 列出某个目录下所有文件的信息

![hdfs-commands](http://or49tneld.bkt.clouddn.com/18-1-23/63075597.jpg)

# filestatus 获取文件状态

```java
package hdfs.tutorial;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;
import java.sql.Timestamp;

/**
 * Created by Administrator on 2017/5/25.
 */
public class ReadHdfs {
    public static void main(String[] args) {

        Path path = new Path("hdfs://192.xxx.xxx.xxx:9000/hdfstest");
        Path filePath = new Path("hdfs://192.xxx.xxx.xxx:9000/hdfstest/people.txt");
        try {

            //得到 FileSystem 类
            FileSystem hdfs = getFileSystem();

            //列出某个目录下所有文件的信息
            listFilesStatus(path, hdfs);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     *
     * @return 得到hdfs的连接 FileSystem类
     * @throws URISyntaxException
     * @throws IOException
     * @throws InterruptedException
     */
    public static FileSystem getFileSystem() throws URISyntaxException, IOException, InterruptedException {
      //获取FileSystem类的方法有很多种，这里只写一种
        Configuration config = new Configuration();
        URI uri = new URI("hdfs://192.xxx.xxx.xxx:9000");
        return FileSystem.get(uri,config,"your_user");// 第一位为uri，第二位为config，第三位是登录的用户
    }

    /**
     * 列出某个目录下所有文件的信息
     * @param path
     * @param hdfs
     * @throws IOException
     */
    public static void listFilesStatus(Path path, FileSystem hdfs) throws IOException {
        // 列出目录下的所有文件
        FileStatus[] files = hdfs.listStatus(path);

        for (int i = 0; i <files.length ; i++) {
            FileStatus file = files[i];
            if(file.isFile()){
                System.out.println("这是文件");
                long len = file.getLen();                   //文件长度
                String pathSource = file.getPath().toString();//文件路径
                String fileName = file.getPath().getName();   // 文件名称
                String parentPath = file.getPath().getParent().toString();//文件父路径
                Timestamp timestamp = new Timestamp(file.getModificationTime());//文件最后修改时间
                long blockSize = file.getBlockSize();   //文件块大小
                String group = file.getGroup();         //文件所属组
                String owner = file.getOwner();          // 文件拥有者
                long accessTime = file.getAccessTime();  //该文件上次访问时间
                short replication = file.getReplication(); //文件副本数
                System.out.println("文件长度: "+len+"\n"+
                        "文件路径: "+pathSource+"\n"+
                        "文件名称: "+fileName+"\n"+
                        "文件父路径: "+parentPath+"\n"+
                        "文件最后修改时间: "+timestamp+"\n"+
                        "文件块大小: "+blockSize+"\n"+
                        "文件所属组: "+group+"\n"+
                        "文件拥有者: "+owner+"\n"+
                        "该文件上次访问时间: "+accessTime+"\n"+
                        "文件副本数: "+replication+"\n"+
                        "==============================");

            }else if(file.isDirectory()){
                System.out.println("这是文件夹");
                System.out.println("文件父路径: "+file.getPath().toString());

                //递归调用
                listFilesStatus(file.getPath(),hdfs);

            }else if(file.isSymlink()){
                System.out.println("这是链接文件");
            }
        }
    }

}


```


##  globStatus 路径过滤  
例如下面的例子，我需要得到/filtertest/*/* 路径中 带有abc的路径
globStatus 很灵活，内部甚至可以写一些正则表达式，有时候在处理大数据的预处理的时候可能很有效

```
public class FileListFilter {

    public static void main(String[] args) {
        FileSystem hdfs = null;
        try{
            Configuration config = new Configuration();

            hdfs = FileSystem.get(new URI("hdfs://192.xxx.xxx.xxx:9000"),
                    config, "hmaster");
            Path allPath = new Path("/filtertest/*/*");

            FileStatus[] fileGlobStatuses = hdfs.globStatus(allPath, new PathFilter() {
                @Override
                public boolean accept(Path path) {
                    String contidion = "abc";
                    // 过滤出路径中包含 abc字符串 的路径
                    return  path.toString().contains(contidion);
                }
            });

            Path[] globPaths = FileUtil.stat2Paths(fileGlobStatuses);
            for (Path p :globPaths){
                System.out.println("globe过滤后的路径"+p);
            }

        }catch(Exception e){
            e.printStackTrace();
        }
    }
}

```
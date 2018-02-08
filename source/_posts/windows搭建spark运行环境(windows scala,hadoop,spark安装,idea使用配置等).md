---
title: windows搭建spark运行环境(windows scala,hadoop,spark安装,idea使用配置等)
date: 2016-09-06 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 32
permalink: spark-windows-install
---

**关键字**: spark windows安装，spark运行环境，idea普通模式构建spark程序，idea maven构建spark程序，idea运行wordcount
<!-- more -->
## 安装scala
 下载地址 http://www.scala-lang.org/download/  
 下载scala-xxx.msi ，然后直接安装好就行了（注意，默认的位置最好别有空格，对于windows10我直接安装没有出现问题，但是对于windows7却出现  此时不应有XXX，这种情况就是安装目录有空格），一般他会默认帮我们配置好环境，当在命令行窗口输出scala -version能出现版本，和输入scala能出现scala的编辑的时候，说明成功了
## 安装spark
直接去官网，下载spark-1.4.0-bin-hadoop2.6.tgz，下载地址，解压后建议改名为spark，然后配置环境变量，把解压后的bin目录，写在path里面，然后尝试运行spark-shell, 一般可以成功运行，但是spark还依赖于hadoop。所以还需要安装hadoop
## 安装hadoop
直接去官网下载，我下的是2.6，我之前测试2.7版本的其实也兼容2.6版本的，为了保险起见，下载和spark对应版本的hadoop，下载地址http://mirrors.hust.edu.cn/apache/hadoop/common/
下载好后解压，然后把bin目录写在path环境里面，这时候，为了防止运行程序的时候出现nullpoint异常，我们需要去github https://github.com/steveloughran/winutils 找到对应的hadoop版本，然后进入bin目录下，下载hadoop.dll和winutils.exe, 然后复制到所安装hadoop目录下

## 安装idea
进入官网 http://www.jetbrains.com/idea/download/#section=windows 下载右边的Community版本的idea, 然后默认一直安装，安装好后，如果是运行scala项目，需要安装scala插件，在File->Settings>Pligins下，点击Install JeBrains plugins, 找到scala，注意要看其版本和自己的idea要对应（一般都是对应的），然后点击install, 需要等待一段时间，然后安装好重启idea就行了

## 运行wordcount
File->New->Project->scala->IDEA  
![](http://i1.piimg.com/567571/b7683db6f586455c.png)


选择next，注意，这里jdk和scala版本一定要选择，如果jdk没有就手动指定jdk安装目录，scala没有就点击create，然后会出现版本进行选择就行了，没有版本就点击左下角download，一般第一次需要点击坐下家download
![](http://i2.muimg.com/567571/ccc5b4c41ae94a6c.png)  
![](http://i1.piimg.com/567571/90080539b88c9c09.png)  

创建一个wordcount, 其中我们需要依赖于spark的spark-assembly-1.4.0-hadoop2.6.0.jar包， idea点击File->Project Structure->Module, 选择右边的Dependencies,点击右边的+号， 选择jars or direc... 引入spark-assembly-1.4.0-hadoop2.6.0.jar包即可
```java
/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed


----------


 on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package cn.kaishun.spark;

import scala.Tuple2;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;

import java.util.Arrays;
import java.util.List;
import java.util.regex.Pattern;

public final class JavaWordCount {
  private static final Pattern SPACE = Pattern.compile(" ");

  public static void main(String[] args) throws Exception {

    if (args.length < 1) {
      System.err.println("Usage: JavaWordCount <file>");
      System.exit(1);
    }

    SparkConf sparkConf = new SparkConf().setAppName("JavaWordCount").setMaster("local");
    JavaSparkContext ctx = new JavaSparkContext(sparkConf);
    JavaRDD<String> lines = ctx.textFile(args[0], 1);

    JavaRDD<String> words = lines.flatMap(new FlatMapFunction<String, String>() {
      @Override
      public Iterable<String> call(String s) {
        return Arrays.asList(SPACE.split(s));
      }
    });

    JavaPairRDD<String, Integer> ones = words.mapToPair(new PairFunction<String, String, Integer>() {
      @Override
      public Tuple2<String, Integer> call(String s) {
        return new Tuple2<String, Integer>(s, 1);
      }
    });

    JavaPairRDD<String, Integer> counts = ones.reduceByKey(new Function2<Integer, Integer, Integer>() {
      @Override
      public Integer call(Integer i1, Integer i2) {
        return i1 + i2;
      }
    });
    counts.saveAsTextFile("G:\\ceshi\\bigdata\\spark\\wordcount\\output\\out1");
    List<Tuple2<String, Integer>> output = counts.collect();
    for (Tuple2<?,?> tuple : output) {
      System.out.println(tuple._1() + ": " + tuple._2());
    }
    ctx.stop();
  }
}


```  
创建测试文件，在args进行配置输入和输出路径，点击运行，即可  
若有空指针问题，大多情况是hadoop下的winutils.exe的问题，github找一份复制就可以了(注： 我记得之前如果第一次出问题了，以后怎么解决都解决不了，一直报空指针，后来我复制winutils.exe和hadoop.dll,然后重启电脑才解决)  


## idea  Maven 构建spark程序
大多情况下，我不想按照上面的方式，各种jar包的导入，我更多的是想用maven来构建spark程序。
### maven下载速度慢的解决办法
在maven安装目录的conf目录下，修改settings.xml文件, 添加阿里云的加速即可，参考我的setting
```
 <?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

<!--
 | This is the configuration file for Maven. It can be specified at two levels:
 |
 |  1. User Level. This settings.xml file provides configuration for a single user,
 |                 and is normally provided in ${user.home}/.m2/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -s /path/to/user/settings.xml
 |
 |  2. Global Level. This settings.xml file provides configuration for all Maven
 |                 users on a machine (assuming they're all using the same Maven
 |                 installation). It's normally provided in
 |                 ${maven.home}/conf/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -gs /path/to/global/settings.xml
 |
 | The sections in this sample file are intended to give you a running start at
 | getting the most out of your Maven installation. Where appropriate, the default
 | values (values used when the setting is not specified) are provided.
 |
 |-->
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <pluginGroups>

  </pluginGroups>

  <proxies>

  </proxies>

  <servers>

  </servers>

<repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>http://maven.aliyun.com/nexus/content/repositories/central</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>


  <mirrors>

    <mirror>  
        <id>nexus-aliyun</id>  
        <mirrorOf>*</mirrorOf>  
        <name>Nexus aliyun</name>  
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>  
    </mirror>  

	 
  </mirrors>

  <profiles>
   
  </profiles>

  
</settings>

```  

pom中增加
```
        <repository>
            <id>central</id>
            <name>Central Repository</name>
            <url>http://maven.aliyun.com/nexus/content/repositories/central</url>
            <layout>default</layout>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
```  

### **spark中的maven配置**  

这是我的本身配置，有的不必要可以删除，其中的groupId，artifactId 根据自己的设置而设置
**spark1.6的配置**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.kaishun</groupId>
    <artifactId>mvntest</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spark.version>1.6.0</spark.version>
        <scala.version>2.10</scala.version>
        <hadoop.version>2.6.0</hadoop.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-hive_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.6.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming-kafka_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-mllib_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.39</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.6.6</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.6.6</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.16</version>
        </dependency>
    </dependencies>

    <!-- maven官方 http://repo1.maven.org/maven2/  或 http://repo2.maven.org/maven2/ （延迟低一些） -->
    <repositories>
               <repository>
            <id>central</id>
            <name>Central Repository</name>
            <url>http://maven.aliyun.com/nexus/content/repositories/central</url>
            <layout>default</layout>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>

        <repository>
            <id>central</id>
            <name>Maven Repository Switchboard</name>
            <layout>default</layout>
            <url>http://repo2.maven.org/maven2</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

    <build>
        <sourceDirectory>src/main/scala</sourceDirectory>
        <testSourceDirectory>src/test/scala</testSourceDirectory>

        <plugins>
            <plugin>
                <!-- MAVEN 编译使用的JDK版本 -->
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```  

**spark2.10**  
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>sparkmaven</groupId>
    <artifactId>com.shuner.mvn</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spark.version>2.11</spark.version>
        <scala.version>2.11.8</scala.version>
        <hadoop.version>2.6.0</hadoop.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.10</artifactId>
            <version>2.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.10</artifactId>
            <version>2.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-hive_2.10</artifactId>
            <version>2.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming_2.10</artifactId>
            <version>2.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.6.0</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.6.6</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.6.6</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.16</version>
        </dependency>
    </dependencies>

    <!-- maven官方 http://repo1.maven.org/maven2/  或 http://repo2.maven.org/maven2/ （延迟低一些） -->
    <repositories>
        <repository>
            <id>central</id>
            <name>Central Repository</name>
            <url>http://maven.aliyun.com/nexus/content/repositories/central</url>
            <layout>default</layout>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>

    </repositories>

    <build>
        <sourceDirectory>src/main/scala</sourceDirectory>
        <testSourceDirectory>src/test/scala</testSourceDirectory>

        <plugins>
            <plugin>
                <!-- MAVEN 编译使用的JDK版本 -->
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```  


最后补充，来一个既可以玩spark，又可以玩hadoop的pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.mingtong</groupId>
    <artifactId>spark16test</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spark.version>1.6.0</spark.version>
        <scala.version>2.10</scala.version>
        <hadoop.version>2.7.3</hadoop.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-hive_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>


        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.39</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.6.6</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.6.6</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.16</version>
        </dependency>


        <!-- 配置hadoop的环境 -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>${hadoop.version}</version>
        </dependency>

    </dependencies>




    <!-- maven官方 http://repo1.maven.org/maven2/  或 http://repo2.maven.org/maven2/ （延迟低一些） -->
    <repositories>
        <repository>
            <id>central</id>
            <name>Central Repository</name>
            <url>http://maven.aliyun.com/nexus/content/repositories/central</url>
            <layout>default</layout>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>

    </repositories>

    <build>
        <sourceDirectory>src/main/scala</sourceDirectory>
        <testSourceDirectory>src/test/scala</testSourceDirectory>

        <plugins>
            <plugin>
                <!-- MAVEN 编译使用的JDK版本 -->
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```
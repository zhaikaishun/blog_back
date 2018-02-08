---
title: hadoop集群搭建教程
date: 2016-07-15 21:25:21
tags: [hadoop]
categories: [大数据,hadoop]
author: kaishun
id: 12
permalink: hadoop-cluster-install
toc: true
---

hadoop的配置参看github https://github.com/zhaikaishun/hadoop_cluster  
作者: 翟开顺
  
关键字: 
集群环境介绍，Hadoop简介，网络配置，所需软件
SSH免密码登陆配置，java环境安装，卸载原有的JDK， 安装jdk17， 配置java环境变量，验证是否成功，Hadoop集群安装，安装Hadoop，验证hadoop
hadoop错误分析
<!-- more -->
# **集群环境介绍**
## **1.  Hadoop简介**  
　Hadoop是Apache软件基金会旗下的一个开源分布式计算平台。以Hadoop分布式文件系统（HDFS，Hadoop Distributed Filesystem）和MapReduce（Google MapReduce的开源实现）为核心的Hadoop为用户提供了系统底层细节透明的分布式基础架构。
　　对于Hadoop的集群来讲，可以分成两大类角色：Master和Salve。一个HDFS集群是由一个NameNode和若干个DataNode组成的。其中NameNode作为主服务器，管理文件系统的命名空间和客户端对文件系统的访问操作；集群中的DataNode管理存储的数据。MapReduce框架是由一个单独运行在主节点上的JobTracker和运行在每个集群从节点的TaskTracker共同组成的。主节点负责调度构成一个作业的所有任务，这些任务分布在不同的从节点上。主节点监控它们的执行情况，并且重新执行之前的失败任务；从节点仅负责由主节点指派的任务。当一个Job被提交时，JobTracker接收到提交作业和配置信息之后，就会将配置信息等分发给从节点，同时调度任务并监控TaskTracker的执行。
　　从上面的介绍可以看出，HDFS和MapReduce共同组成了Hadoop分布式系统体系结构的核心。HDFS在集群上实现分布式文件系统，MapReduce在集群上实现了分布式计算和任务处理。HDFS在MapReduce任务处理过程中提供了文件操作和存储等支持，MapReduce在HDFS的基础上实现了任务的分发、跟踪、执行等工作，并收集结果，二者相互作用，完成了Hadoop分布式集群的主要任务。


## **2.  环境说明**  
本教程为了简单起见只设置两个节点： master为主节点，node01为数据节点，节点之间局域网连接，相互可以ping通，节点IP分布如下

机器名称 | IP地址
---|---
master | 192.168.200.128
node01 | 192.168.200.129

两个节点都是centos6.5系统，都有同一个用户，用户名叫Hadoop  

### **给hadoop用户赋予root权限**  
切换到root用户 赋予etc/sudoers777权限，然后打开
```shell
[root@kaishun etc]# chmod 777 /etc/sudoers
[root@kaishun etc]# vim /etc/sudoers
```  
找到Allows people in group wheel to run all commands，把下面%wheel的#给去掉,在Allow root to run any commands anywhere下，加上hadoop  ALL=(ALL)       ALL，然后保存
```shell
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL     
hadoop  ALL=(ALL)       ALL

## Allows people in group wheel to run all commands
%wheel        ALL=(ALL)       ALL
```
把sudoers的权限改回来成440
```shell
[root@kaishun etc]# chmod 440 /etc/sudoers
```    
测试是否成功  
在普通用户下
```shell
[hadoop@kaishun ~]$ sudo mkdir test
输入密码如果可以成功创建文件夹，说明成功
```
### **网络配置**

**1. 查看当前机器名**
**在root用户下**输入，显示
```shell
[root@kaishun hadoop]# hostname
显示 kaishun， 与我们规划的master不符合
```  
**2. 在root用户下修改当前机器名称**
```shell
[root@kaishun hadoop]# vim /etc/sysconfig/network
```
修改HOSTNAME 为 master
```shell
HOSTNAME=master
```
同理，192.168.200.129这台机器修改成node01
修改之后，可能不会立即生效，我是重启后才生效的  
**3. 在root用户下配置hosts文件, 每台机器都需要配置（必须）**  
```shell
[root@master hadoop]# vim /etc/hosts
```
添加
```shell
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.200.128 master
192.168.200.129 node01
```  
**测试是否成功**，如果能相互使用ping node01   ping master能成功，说明hosts文件配置成功
```shell
[hadoop@master ~]$ ping node01
PING node01 (192.168.200.129) 56(84) bytes of data.
64 bytes from node01 (192.168.200.129): icmp_seq=1 ttl=64 time=0.391 ms
64 bytes from node01 (192.168.200.129): icmp_seq=2 ttl=64 time=0.435 ms
64 bytes from node01 (192.168.200.129): icmp_seq=3 ttl=64 time=0.442 ms

[hadoop@node01 ~]$ ping master
PING master (192.168.200.128) 56(84) bytes of data.
64 bytes from master (192.168.200.128): icmp_seq=1 ttl=64 time=0.379 ms
64 bytes from master (192.168.200.128): icmp_seq=2 ttl=64 time=0.411 ms
64 bytes from master (192.168.200.128): icmp_seq=3 ttl=64 time=0.460 ms
```

## **3.  所需软件**
1.  JDK版本1.7  
2.  hadoop版本hadoop-2.7.1  去官网的华科镜像下载hadoop-2.7.1.tar.gz， [地址](http://mirrors.hust.edu.cn/apache/hadoop/common/hadoop-2.7.1/)  
3. 数据传输工具FileZilla， ssh连接工具 secureCRT  

## **4.  SSH免密码登陆配置**
Hadoop运行过程中需要管理远端Hadoop守护进程，在Hadoop启动以后，NameNode是通过SSH（Secure Shell）来启动和停止各个DataNode上的各种守护进程的。这就必须在节点之间执行指令的时候是不需要输入密码的形式，故我们需要配置SSH运用无密码公钥认证的形式，这样NameNode使用SSH无密码登录并启动DataName进程，同样原理，DataNode上也能使用SSH无密码登录到NameNode。    
安装CentOS6.5时，我们选择了一些基本安装包，所以我们需要两个服务：ssh和rsync已经安装了。可以通过下面命令查看结果显示如下：
```
[hadoop@master ~]$ rpm –qa | grep openssh
[hadoop@master ~]$ rpm –qa | grep rsync
如果有相应的提示，说明这两个是装好了的，我这里是系统自带的
```  
**4.1 配置master无密码登陆所有的node**  
原理请百度
在master节点上执行以下命令 然后按几次回车键：
```shell
[hadoop@master ~]$ ssh-keygen -t rsa
```  
出现下图  
![ssh免密码登陆](http://i4.buimg.com/567571/b22c79e94cbc2297.png)

我们看到这句话  
Your identification has been saved in /home/hadoop/.ssh/id_rsa.  
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub.  
说明默认目录在 /home/hadoop/.ssh/ 下

接着在master节点上做如下配置，把id_rsa.pub追加到授权的key里面去。  
```shell
[hadoop@master ~]$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
现在我们进入~/.ssh目录可以看到
```
[hadoop@master ~]$ cd ~/.ssh/
[hadoop@master .ssh]$ ll
total 12
-rw-rw-r--. 1 hadoop hadoop  395 Apr  2 16:22 authorized_keys
-rw-------. 1 hadoop hadoop 1675 Apr  2 16:17 id_rsa
-rw-r--r--. 1 hadoop hadoop  395 Apr  2 16:17 id_rsa.pub
```
**4.1.1. 修改文件"authorized_keys权限**  
```
[hadoop@master .ssh]$ chmod 600 ~/.ssh/authorized_keys
```
![authorized_keys权限](http://i2.muimg.com/567571/41ecd0b8634028ae.png)

**4.1.2. 设置SSH配置**  
用root用户登录服务器修改SSH配置文件"/etc/ssh/sshd_config"的下列内容。这里找到这些内容，把前面的#去掉即可
```shell
[root@master .ssh]# vim /etc/ssh/sshd_config
RSAAuthentication yes # 启用 RSA 认证
PubkeyAuthentication yes # 启用公钥私钥配对认证方式
AuthorizedKeysFile .ssh/authorized_keys # 公钥文件路径（和上面生成的文件同）
```
**重启SSH服务**
```
[root@master .ssh]# /etc/rc.d/init.d/sshd restart
Stopping sshd:                                             [  OK  ]
Starting sshd:                                             [  OK  ]
[root@master .ssh]# 
```
退出root用户，使用hadoop普通用户验证是否成功, ssh localhost, 如果不需要输入密码，那么验证成功
```shell
[hadoop@master .ssh]$ ssh localhost
The authenticity of host 'localhost (::1)' can't be established.
RSA key fingerprint is 48:0b:ee:9b:67:85:4c:19:35:10:d1:1d:e1:5d:fa:c4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (RSA) to the list of known hosts.
Last login: Sun Apr  2 16:09:39 2017 from 192.168.200.1
```

**4.1.3. 把公钥复制到所有的node机器上**
从上图中得知无密码登录本级已经设置完毕，接下来的事儿是把公钥复制所有的node机器上。使用下面的命令格式进行复制公钥  
scp ~/.ssh/id_rsa.pub 远程用户名@远程服务器IP:~/
我本地这样使用 scp ~/.ssh/id_rsa.pub hadoop@192.168.200.129:~/  ,然后根据提示输入需要复制的远程服务器的密码，最后出现下面的提示说明复制成功
```shell
[hadoop@master ~]$ scp ~/.ssh/id_rsa.pub hadoop@192.168.200.129:~/
hadoop@192.168.200.129's password:  # 这里输入远程密码
id_rsa.pub                                                                                                                        100%  395     0.4KB/s   00:00    
[hadoop@master ~]$ 
```  
**4.1.4. 对节点机器进行配置**
下面就针对IP为"192.168.200.129"的node01的节点进行配置。
4.1  ll -a查看是否有.ssh目录，如果没有，我们需要创建一个.ssh目录，并且赋予这个权限 drwx------.  2 hadoop hadoop 4096 Apr  2 16:40 .ssh  具体权限参照master的机器， centos6.5一般都是默认带有.ssh目录的
```
[hadoop@node01 ~]$ ll -a 
drwx------.  2 hadoop hadoop 4096 Apr  2 16:40 .ssh
```
如果有这个目录了，我们把刚才的文件追加到authorized_keys 中去，然后修改authorized_keys文件权限
```
[hadoop@node01 ~]$ cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
[hadoop@node01 .ssh]$ chmod 600 ~/.ssh/authorized_keys

```
进入到ssh 目录，ll 看到如下所示说明成功，注意权限是否正确
```shell
[hadoop@node01 .ssh]$ ll
total 8
-rw-------. 1 hadoop hadoop 395 Apr  2 16:52 authorized_keys
-rw-r--r--. 1 hadoop hadoop 391 Apr  2 16:40 known_hosts
```
**4.2  用root用户修改/etc/ssh/sshd_config**
参考前面的master的修改/etc/ssh/sshd_config的方法  
设置SSH配置  
用root用户登录服务器修改SSH配置文件"/etc/ssh/sshd_config"的下列内容。这里找到这些内容，把前面的#去掉即可
```shell
[root@master .ssh]# vim /etc/ssh/sshd_config
RSAAuthentication yes # 启用 RSA 认证
PubkeyAuthentication yes # 启用公钥私钥配对认证方式
AuthorizedKeysFile .ssh/authorized_keys # 公钥文件路径（和上面生成的文件同）
```
**重启SSH服务**
```
[root@master .ssh]# /etc/rc.d/init.d/sshd restart
Stopping sshd:                                             [  OK  ]
Starting sshd:                                             [  OK  ]
[root@master .ssh]# 
```
最后记得把"/home/hadoop/"目录下的"id_rsa.pub"文件删除掉 

到此为止，我们经过的步骤已经实现了从"master"到"node01"SSH无密码登录

验证master到node01的无密码登陆,在master机器上，使用hadoop用户 ssh node01或者ssh 192.168.200.129, 下面是成功的的结果
```
[hadoop@master ~]$ ssh node01
Last login: Sun Apr  2 17:25:50 2017 from localhost
[hadoop@node01 ~]$ 

[hadoop@node01 ~]$ ssh master
Last login: Sun Apr  2 17:26:04 2017 from node01
[hadoop@master ~]$ 
```


## **5 java安装环境**
### **5.1 卸载原有的JDK**
因为有的系统自带有JDK, 安装前先卸载  
查看所装的JDK
```shell
[hadoop@master ~]$ rpm -qa | grep jdk
出现
java-1.7.0-openjdk-1.7.0.45-2.4.3.3.el6.x86_64
java-1.6.0-openjdk-1.6.0.0-1.66.1.13.0.el6.x86_64
```
root下卸载前面查出的这两个
```shell
[root@master hadoop]#  yum -y remove java-1.7.0-openjdk-1.7.0.45-2.4.3.3.el6.x86_64  
[root@master hadoop]#  yum -y remove java-1.6.0-openjdk-1.6.0.0-1.66.1.13.0.el6.x86_64  
成功后会出现一个complete

```  
### **5.2 安装jdk1.7**
首先用root身份登录master后在/usr/local下创建java文件夹
```shell
[root@master hadoop]# mkdir -p /usr/local/java
```  
我们把FTP传来的jdk-7u79-linux-x64.tar.gz复制到/usr/local/java 文件夹下
```shell
[root@master Downloads]# cp jdk-7u79-linux-x64.tar.gz /usr/local/java/
```
解压并且
```shell
[root@master java]# tar zxvf jdk-7u79-linux-x64.tar.gz 
解压完成后出现
[root@master java]# ll
total 149920
drwxr-xr-x. 8 uucp  143      4096 Apr 11  2015 jdk1.7.0_79
-rw-r--r--. 1 root root 153512879 Apr  2 18:17 jdk-7u79-linux-x64.tar.gz
```
给所有者权限
```shell
[root@master java]# chown hadoop:hadoop jdk1.7.0_79/ -R
```
### **5.3 配置java环境变量**
编辑"/etc/profile"文件
```shell
[root@master java]#  vim /etc/profile
```
在尾部加入
```shell
# set java environment
export JAVA_HOME=/usr/local/java/jdk1.7.0_79
export JRE_HOME=/usr/local/java/jdk1.7.0_79/jre
export PATH=$PATH:/usr/local/java/jdk1.7.0_79/bin
export CLASSPATH=./:/usr/local/java/jdk1.7.0_79/lib:/usr/local/java/jdk1.7.0_79/jre/lib
```
使配置生效
```shell
[root@master java]# source /etc/profile
```
### **5.4 验证是否成功**
```
java -version  出现 java version "1.7.0_79"
javac  有提示
java  有提示
```
确保是按照我上面的步骤，权限不能有错，否则可能会有问题, 同样，在另外的节点上也安装好jdk


## **6. Hadoop集群安装**  
所有的机器上都要安装hadoop，现在就先在Master服务器安装，然后其他服务器按照步骤重复进行即可。**安装和配置hadoop需要以"root"的身份进行。**
### **6.1 安装Hadoop**
#### **6.1.1 建立一个目录，用来存放hadoop**
```shell
[root@master Downloads]#  mkdir -p /home/hadoop/MyCloudera/APP/hadoop/
```
#### **6.1.2 把下载好得hadoop-2.7.1.tar.gz 复制到这个目录下，解压并且命名为hadoop**
```shell
复制到我们建立得目录
[root@master Downloads]# cp hadoop-2.7.1.tar.gz  /home/hadoop/MyCloudera/APP/hadoop/
进入到我们复制得目录
[root@master Downloads]# cd /home/hadoop/MyCloudera/APP/hadoop  
对此tar.gz解压
[root@master hadoop]# tar zxvf hadoop-2.7.1.tar.gz 
改名为hadoop
[root@master hadoop]# mv hadoop-2.7.1 hadoop
```
#### **6.1.3 将文件夹得读写权限赋予给hadoop用户**
```shell
[root@master APP]# chown -R hadoop:hadoop hadoop  
ll 查看权限，是这样得
[root@master APP]# ll
total 4
drwxr-xr-x. 3 hadoop hadoop 4096 Apr  2 23:29 hadoop
```


#### **6.1.4 配置/etc/profile**
```shell
[root@master APP]# vim /etc/profile
```
在末尾加上如下配置，其中HADOOP_HOME填写前面得hadoop存放得位置
```
# set hadoop path
export HADOOP_HOME=/home/hadoop/MyCloudera/APP/hadoop/hadoop 
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export YARN_CONF_DIR=${HADOOP_HOME}/etc/hadoop

```
让配置生效
```shell
[root@master APP]# source /etc/profile
```

### **6.2 配置hadoop**
Hadoop配置文件在conf目录下，之前的版本的配置文件主要是Hadoop-default.xml和Hadoop-site.xml。由于Hadoop发展迅速，代码量急剧增加，代码开发分为了core，hdfs和map/reduce三部分，配置文件也被分成了三个core-site.xml、hdfs-site.xml、mapred-site.xml。core-site.xml和hdfs-site.xml是站在HDFS角度上配置文件；core-site.xml和mapred-site.xml是站在MapReduce角度上配置文件。

#### **6.2.1 配置hadoop-env.sh**
该hadoop-env.sh文件位于/home/hadoop/MyCloudera/APP/hadoop/hadoop/etc/hadoop目录下
在文件的末尾添加下面内容
```shell
# The java environment
export JAVA_HOME=/usr/local/java/jdk1.7.0_79
```
#### **6.2.2 配置core-site.xml文件**
我们先在本地建立几个目录，用来存放一些hadoop得文件  
在根目录下，建立一个data
```shell
根目录路径
[root@master /]# pwd
/
创建一个data目录
[root@master /]# mkdir data
创建/data/tmpdata/hadoop/data/tmp目录
[root@master /]# mkdir -p /data/tmpdata/hadoop/data/tmp
```
然后对core-site.xml做如下配置, 具体得hadoop.tmp.dir和fs.default.name得功能参看百度google
```xml
<configuration>
  <property>
      <name>hadoop.tmp.dir</name>
      <value>/data/tmpdata/hadoop/data/tmp</value>
  </property>

  <property>
      <name>fs.default.name</name>
      <value>hdfs://master:9000</value>
  </property>
</configuration>
```
#### **6.2.3 配置hdfs-site.xml文件**  
我这里配置的比较完整，如果想简单点，有的其实可以默认设置，具体参看其他文章
**1. 创建namenode和datanode的存放目录,然后对/data目录赋予权限。 注意权限不能有错**
```shell
[root@master /]# mkdir -p /data/hadoop/data/name
[root@master data]# mkdir -p /data/hadoop/data/data

[root@master /]#  chown hadoop:hadoop data/ -R
[root@master /]# chmod 777 data/ -R 

```
**2. 创建SecondaryNameNode的目录**
在根目录下创建hadoop目录，然后创建/hadoop/SecondaryNameNode/目录，最后赋予hadoop目录权限
```shell
[root@master /]# mkdir hadoop  
[root@master /]# mkdir -p /hadoop/SecondaryNameNode/  

[root@master /]# chown hadoop:hadoop hadoop/ -R 
[root@master /]#  chmod 777 hadoop/ -R

```

**hdfs-site.xml配置**
```xml
<configuration>
     <property>
            <name>dfs.namenode.name.dir</name>
            <value>/data/hadoop/data/name/</value>
     </property>
     <property>
            <name>dfs.datanode.data.dir</name>
            <value>/data/hadoop/data/data/</value>
     </property>
     <property>
           <name>dfs.replication</name>
           <value>2</value>
     </property>
     <property>
            <name>dfs.namenode.checkpoint.dir</name>
            <value>/hadoop/SecondaryNameNode/</value>
     </property>
    
     <property>
            <name>dfs.http.address</name>
            <value>master:50070</value>
     </property>
    
     <property>
            <name>dfs.secondary.http.address</name>
            <value>master:50090</value>
     </property>


    <property>
    <name>dfs.datanode.du.reserved</name>
    <value>0</value>
    <description> 每个卷预留的空闲空间数量 </description>
    </property>
    
    <property>
    <name>dfs.datanode.max.xcievers</name>
    <value>32768</value>
    </property>
    
    
    <property>
    <name>dfs.datanode.socket.write.timeout</name>
    <value>0</value>
    </property>
    <property>
    <name>dfs.socket.timeout</name>
    <value>180000</value>
    <description>socket通讯超时时间</description>
    </property>

</configuration> 
```
#### **6.2.3 配置mapred-site.xml文件**
我这里配置的比较完整，网上大多数都是用的默认，具体其中的一些参数可以百度  
这里先建立几个文件
```shell
[root@master /]# mkdir -p /hadoop/mapreduce/jobhistory/history/done
[root@master /]# mkdir -p /hadoop/mapreduce/jobhistory/history/done_intermediate
[root@master /]# mkdir -p /hadoop/hadoop-yarn/staging  

赋予权限
[root@master /]# chown hadoop:hadoop hadoop/ -R 
[root@master /]#  chmod 777 hadoop/ -R
```
复制一份 mapred-site.xml
```shell
[root@master hadoop]# cp mapred-site.xml.template mapred-site.xml
```

```xml
<configuration>

<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>

<property>
<name>mapreduce.jobtracker.address</name>
<value>master:9001</value>
</property>

<property>
<name>mapreduce.jobtracker.http.address</name>
<value>master:50030</value>
</property>

<property>
<name>mapreduce.jobhistory.address</name>
<value>master:10020</value>
</property>


<property>
<name>mapreduce.jobhistory.webapp.address</name>
<value>master:19888</value>
</property>


<property>
<name>mapreduce.jobhistory.done-dir</name>
<value>/hadoop/mapreduce/jobhistory/history/done</value>
</property>

<property>
<name>mapreduce.jobhistory.intermediate-done-dir</name>
<value>/hadoop/mapreduce/jobhistory/history/done_intermediate</value>
</property>

<property>
<name>yarn.app.mapreduce.am.staging-dir</name>
<value>/hadoop/hadoop-yarn/staging</value>
</property>

<property>
<name>mapred.hosts.exclude</name>
<value>/home/hadoop/MyCloudera/APP/hadoop/hadoop/etc/hadoop/excludes</value>
<final>true</final>
</property>




<property>
<name>mapreduce.tasktracker.map.tasks.maximum</name>
<value>32</value>
<description> 同一时间允许运行的最大map任务数 </description>
</property>
<property>
<name>mapreduce.tasktracker.reduce.tasks.maximum</name>
<value>16</value>
<description> 同一时间允许运行的最大reduce任务数 </description>
</property>



<property>
<name>yarn.nodemanager.resource.memory-mb</name>
<value>1000</value>
</property>
<property>
<name>mapreduce.map.memory.mb</name>
<value>512</value>
<description>map阶段申请的container的内存的大小</description>
</property>


<property>
<name>mapreduce.reduce.memory.mb</name>
<value>512</value>
<description>reduce阶段申请的container的内存的大小</description>
</property>


<property>
<name>mapreduce.map.java.opts</name>
<value>-Xmx512M</value>
<description>用户设定的map/reduce阶段申请的container的JVM参数。最大堆设定要比申请的内存少一些，用于JVM的非堆部分使用。 </description>
</property>
<property>
<name>mapreduce.reduce.java.opts</name>
<value>-Xmx1024M</value>
</property>
<property>
<name>mapreduce.task.io.sort.mb</name>
<value>1024</value>
</property>
<property>
<name>mapreduce.reduce.shuffle.parallelcopies</name>
<value>16</value>

</configuration>
```

#### **6.2.3 配置yarn-site.xml文件**
创建一些文件夹，并且赋予权限
```shell
[root@master /]# clear
[root@master /]# mkdir -p /data/nodemanager/tmp/
[root@master /]# mkdir -p /hadoop/nodemanager/remote
[root@master /]# mkdir -p /data/hadoop/data/nodemanager/logs
[root@master /]# chown hadoop:hadoop hadoop/ -R
[root@master /]# chmod 777 hadoop/ -R
[root@master /]# chown hadoop:hadoop data/ -R
[root@master /]# chmod 777 data/ -R
```

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
    <property>
         <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
    </property>

    <property>
          <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
          <value>org.apache.hadoop.mapred.ShuffleHandler</value>
     </property>

    <property>
            <name>yarn.resourcemanager.scheduler.address</name>
            <value>master:8030</value>
     </property>

    <property>
            <name>yarn.resourcemanager.resource-tracker.address</name>
            <value>master:8031</value>
            </property>

    <property>
            <name>yarn.resourcemanager.address</name>
            <value>master:8032</value>
     </property>

    <property>
            <name>yarn.resourcemanager.admin.address</name>
             <value>master:8033</value>
    </property>

    <property>
            <name>yarn.nodemanager.address</name>
            <value>master:9999</value>
    </property>

    <property>
            <name>yarn.nodemanager.webapp.address</name>
            <value>master:8042</value>
    </property>
    
     <property>
            <name>yarn.resourcemanager.webapp.address</name>
            <value>master:8088</value>
    </property>

    <property>
            <name>yarn.nodemanager.local-dirs</name>
            <value>/data/nodemanager/tmp/</value>
            </property>

    <property>
            <name>yarn.nodemanager.remote-app-log-dir</name>
            <value>/hadoop/nodemanager/remote</value>
    </property>

    <property>
          <name>yarn.nodemanager.log-dirs</name>
           <value>/data/hadoop/data/nodemanager/logs</value>
     </property>

    <property>
            <name>yarn.nodemanager.log.retain-seconds</name>
            <value>604800</value>
    </property>

    <property>
            <name>yarn.nodemanager.resource.cpu-vcores</name>
            <value>24</value>
    </property>

    <property>
            <name>yarn.nodemanager.resource.memory-mb</name>
            <value>1024</value>
    </property>

    <property>
            <name>yarn.nodemanager.vmem-pmem-ratio</name>
            <value>2</value>
    </property>

    <property>
            <name>yarn.scheduler.minimum-allocation-mb</name>
            <value>256</value>
    </property>

    <property>
            <name>yarn.scheduler.maximum-allocation-mb</name>
            <value>1024</value>
    </property>

    <property>
            <name>yarn.scheduler.minimum-allocation-vcores</name>
            <value>1</value>
    </property>

    <property>
            <name>yarn.scheduler.maximum-allocation-vcores</name>
            <value>24</value>
    </property>

    <property>
            <name>yarn.log-aggregation-enable</name>
            <value>true</value>
    </property>

</configuration>

```




#### **6.2.3 配置slaves文件**
这个配置主要记录数据节点的列表，假如集群有3个数据节点，如：node001，node002，node003
那么在slave文件里面就可以设置为：  
node001  
node002  
node003  

我这里为两个节点，配置如下
```shell
master
node01
```
到此，master的hadoop的配置已经完成，对于其他节点，我们建立好相关的目录，复制过去，稍作配置即可了

需要建立的目录总结

```shell
[root@node01 /]# mkdir -p /home/hadoop/MyCloudera/APP/hadoop/
[root@node01 /]# mkdir data
[root@node01 /]# mkdir -p /data/tmpdata/hadoop/data/tmp
[root@node01 /]# mkdir -p /data/hadoop/data/name
[root@node01 /]# mkdir -p /data/hadoop/data/data
[root@node01 /]# mkdir hadoop
[root@node01 /]# mkdir -p /hadoop/SecondaryNameNode/  
[root@node01 /]# chown hadoop:hadoop hadoop/ -R 
[root@node01 /]# chmod 777 hadoop/ -R
[root@node01 /]# mkdir -p /hadoop/mapreduce/jobhistory/history/done
[root@node01 /]# mkdir -p /hadoop/mapreduce/jobhistory/history/done_intermediate
[root@node01 /]# mkdir -p /hadoop/hadoop-yarn/staging  
[root@node01 /]# mkdir -p /data/nodemanager/tmp/
[root@node01 /]# mkdir -p /hadoop/nodemanager/remote
[root@node01 /]# mkdir -p /data/hadoop/data/nodemanager/logs
[root@node01 /]# chown hadoop:hadoop hadoop/ -R
[root@node01 /]# chmod 777 hadoop/ -R
[root@node01 /]# chown hadoop:hadoop data/ -R
[root@node01 /]# chmod 777 data/ -R
[root@node01 /]# 

```


为了确保 hadoop目录 权限没有问题，每台机器在hadoop目录下再次执行一下以下命令
```shell
chown -R hadoop:hadoop hadoop
## 为了保险起见，我给了777的权限， 下面的这一步貌似不做也可以
chmod 777 hadoop/ -R
```

### **6.3 启动与验证**
#### **6.3.1 格式化HDFS文件系统**
**在master上使用普通用户hadoop进行操作**
如果第一次启动需要对hadoop平台进行格式化，记得第一次，假如原来有数据就不需要格式化：
```
hdfs namenode -format
```
如果经过多次format之后，**一定要把/data/hadoop/data/data /data/hadoop/data/name目录下的文件删除**

#### **6.3.2 启动hadoop**
在启动前关闭集群中所有机器的防火墙，不然会出现datanode开后又自动关闭。
记得永久的关闭防火墙chkconfig iptables off
```shell
chkconfig iptables off
```
开始启动,在master的普通用户 hadoop下进行操作
```shell
start-all.sh
```
验证hadoop: 输入jps命令，会出现以下进程说明成功
```shell
[hadoop@master hadoop]$ jps
[hadoop@master hadoop]$ jps
4197 ResourceManager
3851 DataNode
4602 Jps
4013 SecondaryNameNode
4308 NodeManager
3739 NameNode
```  
#### **6.3.3 测试以下hdfs**  
创建一个目录
```shell
[hadoop@node01 ~]$ hadoop fs -mkdir -p /hive/warehouse
``` 
传一个文件
```shell
[hadoop@master hadoop]$ hadoop fs -put slaves /hive/warehouse
```
查看文件
```shell
[hadoop@master hadoop]$ hadoop fs -cat /hive/warehouse/slaves
显示
master
node01
```
经过上面的测试，说明我们集群安装成功
### 6.4 网页查看集群
查看hdfs   
http://192.168.200.128:50070
显示  
![hdfs验证](http://i2.muimg.com/567571/7435a4693a8a21b4.png)

验证hadoop   
http://192.168.200.128:8088/cluster/nodes
显示  
![hadoop验证](http://i1.piimg.com/567571/80a651b3fc692958.png)


## **7. hadoop 集群碰到错误的解决办法**
这里的错误，一般都分为几大类，一类是某些文件夹没有创建，一类是某些文件或者文件夹权限不够，一类就是配置错误  
这些错误都可以去logs目录下查看，我的logs目录在  /home/hadoop/MyCloudera/APP/hadoop/hadoop/logs  
哪里有问题就对应哪个文件去查看错误，例如resourcemanager没起来或者出问题，就去yarn-hadoop-resourcemanager-master.log
```shell
-rwxrwxrwx. 1 hadoop hadoop 921348 Apr  3 13:19 hadoop-hadoop-datanode-master.log
-rw-rw-r--. 1 hadoop hadoop   1434 Apr  3 13:18 hadoop-hadoop-datanode-master.out
-rw-rw-r--. 1 hadoop hadoop      0 Apr  3 13:18 hadoop-hadoop-datanode-master.out.1
-rw-rw-r--. 1 hadoop hadoop   1434 Apr  3 12:52 hadoop-hadoop-datanode-master.out.2
-rwxrwxrwx. 1 hadoop hadoop   1434 Apr  3 12:45 hadoop-hadoop-datanode-master.out.3
-rwxrwxrwx. 1 hadoop hadoop   1434 Apr  3 12:42 hadoop-hadoop-datanode-master.out.4
-rwxrwxrwx. 1 hadoop hadoop   1434 Apr  3 12:35 hadoop-hadoop-datanode-master.out.5
-rwxrwxrwx. 1 hadoop hadoop 371773 Apr  3 13:26 hadoop-hadoop-namenode-master.log
-rw-rw-r--. 1 hadoop hadoop    717 Apr  3 13:18 hadoop-hadoop-namenode-master.out
-rw-rw-r--. 1 hadoop hadoop    717 Apr  3 13:09 hadoop-hadoop-namenode-master.out.1
-rw-rw-r--. 1 hadoop hadoop    717 Apr  3 12:52 hadoop-hadoop-namenode-master.out.2
-rwxrwxrwx. 1 hadoop hadoop    717 Apr  3 12:44 hadoop-hadoop-namenode-master.out.3
-rwxrwxrwx. 1 hadoop hadoop    717 Apr  3 12:42 hadoop-hadoop-namenode-master.out.4
-rwxrwxrwx. 1 hadoop hadoop    717 Apr  3 12:35 hadoop-hadoop-namenode-master.out.5
-rwxrwxrwx. 1 hadoop hadoop      0 Apr  3 01:43 SecurityAuth-hadoop.audit
-rwxrwxrwx. 1 hadoop hadoop 618506 Apr  3 13:19 yarn-hadoop-nodemanager-master.log
-rw-rw-r--. 1 hadoop hadoop   1402 Apr  3 13:19 yarn-hadoop-nodemanager-master.out
-rw-rw-r--. 1 hadoop hadoop      0 Apr  3 13:19 yarn-hadoop-nodemanager-master.out.1
-rw-rw-r--. 1 hadoop hadoop   1402 Apr  3 13:09 yarn-hadoop-nodemanager-master.out.2
-rw-rw-r--. 1 hadoop hadoop   1402 Apr  3 12:52 yarn-hadoop-nodemanager-master.out.3
-rwxrwxrwx. 1 hadoop hadoop   1402 Apr  3 12:36 yarn-hadoop-nodemanager-master.out.4
-rwxrwxrwx. 1 hadoop hadoop   1402 Apr  3 12:30 yarn-hadoop-nodemanager-master.out.5
-rwxrwxrwx. 1 hadoop hadoop 343209 Apr  3 13:19 yarn-hadoop-resourcemanager-master.log
-rw-rw-r--. 1 hadoop hadoop    701 Apr  3 13:19 yarn-hadoop-resourcemanager-master.out
-rw-rw-r--. 1 hadoop hadoop    701 Apr  3 13:09 yarn-hadoop-resourcemanager-master.out.1
-rw-rw-r--. 1 hadoop hadoop    701 Apr  3 12:52 yarn-hadoop-resourcemanager-master.out.2
-rwxrwxrwx. 1 hadoop hadoop    701 Apr  3 12:45 yarn-hadoop-resourcemanager-master.out.3
-rwxrwxrwx. 1 hadoop hadoop    701 Apr  3 12:36 yarn-hadoop-resourcemanager-master.out.4
-rwxrwxrwx. 1 hadoop hadoop    701 Apr  3 12:30 yarn-hadoop-resourcemanager-master.out.5
```











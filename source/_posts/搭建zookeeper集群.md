---
title: 搭建zookeeper集群
date: 2017-11-25 21:25:21
tags: [zookeeper]
categories: [架构]
author: kaishun
id: 122
permalink: zookeeper3
---

Zookeeper环境搭建  

前期准备： 
由于Zookeeper需要先安装java

机器： 三台测试机器  
192.168.1.31  
192.168.1.32  
192.168.1.33  

<!-- more -->
----------


1. 上传zookeeper的压缩包


2. 三个节点都解压到usr/local下  
```shell
[root@kaishun local]# tar zxvf zookeeper-3.4.5.tar.gz -C /usr/local  
```

3. 三台节点都重命名 zookeeper-3.4.5 为zookeeper
```
[root@kaishun local]# mv zookeeper-3.4.5/ zookeeper
```  

4. 修改环境变量 path末尾增加一个zookeeper的bin目录  
```
# zookeeper
export ZOOKEEPER_HOME=/usr/local/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```
5. 到zookeeper下修改配置文件
```
cd /usr/local/zookeeper/conf/  
mv zoo_sample.cfg  zoo.cfg
```  
6. 修改conf  
```
1.修改 dataDir=/usr/local/zookeeper/data  

2. 后面添加：
```
server.0=192.168.1.31:2888:3888
server.1=192.168.1.32:2888:3888
server.2=192.168.1.33:2888:3888

```
然后去创建/usr/local/zookeeper/data  这个文件夹
```
cd /usr/local/zookeeper/
mkdir data
```

7. 去data目录下创建myid文件分别每台机器对应server的值  
```
vim myid  
第一台机器写0，第二台机器写1 ， 第三台机器写2.然后保存退出
```   
8. 关闭防火墙  
```
chkconfig iptables off
```

9. 启动zookeeper  
注意了，三台机器都要启动，最好是使用securtCRT批量发送命令  
```
zkServer.sh start

```
过一段时间查看是否启动成功
```
zkServer.sh status

显示下面说明成功：  
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: follower或者leader

jps会有显示  
QuorumPeerMain
```

进入zookeeper客户端  
```
zkCli.sh  
```
根据命令进行操作  
- 查找：ls /  
- 创建并赋值  create /kaishun spark  
- 获取  get /kaishun  
- 设值 set/kaishun 6666
以上操作可以看到zookeeper集群的数据一致性  
- 递归删除节点 rmr /path   
- 删除指定某个节点 delete /path/child  
**创建节点有两种类型： 短暂(ephemeral)  持久(persistent)**  

上面命令操作是经常需要使用的，另外，有人做了一个客户端,可以使用界面进行操作。 下载地址链接：http://pan.baidu.com/s/1c2JCLcW 密码：ncfl  点击左上角，输入ip和端口即可连接。然后操作就不多说了， 当然使用eclipse插件也是可以的，使用命令也当然是可以的。  





10. 报错解决方案
去查看zookeeper.out，什么错误基本都在里面，然后去网上查就好了，如果是首次安装免不了会出错的



## zoo.cfg参数详解  

- **tickTime**：基本事件单元，以毫秒为单位。这个时间是作为 Zookeeper服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每隔 tickTime时间就会发送一个心跳。  

- **dataDir**： 存储内存中数据库快照的位置，顾名思义就是 Zookeeper保存数据的目录，默认情况下， Zookeeper将写数据的日志文件也保存在这个目录里。

- **clientPort**： 这个端口就是客户端连接 Zookeeper 服务器的端口， Zookeeper
会监听这个端口，接受客户端的访问请求。  

- **initLimit**： 这个配置项是用来配置 Zookeeper接受客户端初始化连接时最长能忍受多少个心跳时间间隔数，
当已经超过 10 个心跳的时间（也就是 tickTime）长度后Zookeeper 服务器还没有收到客户端的返回信息，
那么表明这个客户端连接失败。总的时间长度就是10*2000=20 秒。  

- **syncLimit**： 这个配置项标识 Leader 与 Follower之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime
的时间长度，总的时间长度就是 5*2000=10 秒  
- server.A = B:C:D :  
A 表示这个是第几号服务器。      
B 是这个服务器的 ip 地址。      
C 表示的是这个服务器与集群中的 Leader服务器交换信息的端口。  
D 表示的是万一集群中的 Leader  服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader  

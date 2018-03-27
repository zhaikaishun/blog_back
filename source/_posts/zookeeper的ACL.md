---
title: zookeeper的ACL
date: 2017-12-28 21:25:21
tags: [zookeeper]
categories: [架构]
author: kaishun
id: 126
permalink: zookeeper7
blogexcerpt: ACL  叫做Access Control List，ACL（访问控制列表）,例如linux中的文件系统中就有ACL，传统的文件系统中，ACL分为两个维度，一个是属组，一个是权限。 子目录/文件默认继承父目录的ACL。而在Zookeeper中，node的ACL是没有继承关系的，是独立控制的。Zookeeper的ACL，可以从三个维度来理解：一是scheme; 二是user; 三是permission.为什么zookeeper也要有ACL。zookeeper作为一个分布式协调框架，其内部存储的都是一些关乎分布式系统运行时状态的元数据...
---

## 什么是ACL  
ACL  叫做Access Control List，ACL（访问控制列表）,例如linux中的文件系统中就有ACL，传统的文件系统中，ACL分为两个维度，一个是属组，一个是权限。 子目录/文件默认继承父目录的ACL。而在Zookeeper中，node的ACL是没有继承关系的，是独立控制的。Zookeeper的ACL，可以从三个维度来理解：一是scheme; 二是user; 三是permission。  
<!-- more -->
## 为什么zookeeper也要有ACL  
zookeeper作为一个分布式协调框架，其内部存储的都是一些关乎分布式系统运行时状态的元数据，尤其是设计到分布式锁、Master选举和协调等应用场景。我们需要有效地保障Zookeeper中的数据安全，Zookeeper提供一套完善的ACL权限控制机制来保障数据的安全。  
ZK提供了三种模式。权限模式、授权对象、权限。  
1. **授权模式：scheme**  
- world: 它下面只有一个id, 叫anyone, world:anyone代表任何人，zookeeper中对所有人有权限的结点就是属于world:anyone的
- auth: 它不需要id, 只要是通过authentication的user都有权限（zookeeper支持通过kerberos来进行authencation, 也支持username/password形式的authentication)
- digest: 它对应的id为username:BASE64(SHA1(password))，它需要先通过username:password形式的authentication
- ip: 它对应的id为客户机的IP地址，设置的时候可以设置一个ip段，比如ip:192.168.1.0/16, 表示匹配前16个bit的IP段
- super: 在这种scheme情况下，对应的id拥有超级权限，可以做任何事情(cdrwa)  

具体代码可以参考bjsxt.zookeeper.auth包下的内容，或者网上查看相关的内容, 网上的确实比较详细，博主抛砖引玉了。    

github: https://github.com/zhaikaishun/zookeeper_tutorial
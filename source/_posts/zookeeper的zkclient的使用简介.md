---
title: zookeeper的zkclient的使用简介
date: 2017-12-29 21:25:21
tags: [zookeeper]
categories: [架构]
author: kaishun
id: 128
permalink: zookeeper8
blogexcerpt: Zookeeper的原生API，就之前的那一些，用起来还是比较麻烦的，所以，有些工程师对原生的API接口进行了封装，简化了ZK的复杂性。1. 创建客户端的方法： ZKClient(Arguments) 参数1：zkServer zookeeper服务器的地址，用","分割.参数2：sessionTimeout超时回话，为毫秒，默认是30000ms....
---


github： https://github.com/zhaikaishun/zookeeper_tutorial


----------


# 前言  
Zookeeper的原生API，就之前的那一些，用起来还是比较麻烦的，所以，有些工程师对原生的API接口进行了封装，简化了ZK的复杂性。  
1. 创建客户端的方法： ZKClient(Arguments)  
- 参数1：zkServer zookeeper服务器的地址，用","分割
- 参数2：sessionTimeout超时回话，为毫秒，默认是30000ms
- 参数3：connectionTimeOut 连接超时会话  
- 参数4：IZKConnection接口的实现类
- 参数5：zkSerializer 兹定于序列化实现  

需要引入zkclient的jar包，自行百度即可  
<!-- more -->
# ZKClient的基础操作
## ZKClient建立连接  
直接new即可  
```
String CONNECT_ADDR = "192.168.1.171:2181,192.168.1.172:2181,192.168.1.173:2181";  
ZkClient zkc = new ZkClient(new ZkConnection(CONNECT_ADDR), 10000);
```

## ZKClient创建节点  
方法1  和原生的差不多  
```
zkc.create(final String path, Object data, final CreateMode mode)
```
方法2  
![zookeeper创建操作](http://images2015.cnblogs.com/blog/512650/201606/512650-20160611132540105-414970283.png)  
```java
zkc.createEphemeral("/temp");
// 可以支持递归的创建，但是不能递归赋值
zkc.createPersistent("/super/c1", true);
```

## 删除节点的操作
支持递归的删除  
```java
删除 /temp节点
zkc.delete("/temp");
//递归删除/super
zkc.deleteRecursive("/super");
```

## 获取子节点和阅读节点内容操作
```java
List<String> list = zkc.getChildren("/super");
for(String p : list){
	System.out.println(p);
	String rp = "/super/" + p;
	String data = zkc.readData(rp);
	System.out.println("节点为：" + rp + "，内容为: " + data);
}

```  
## 更新节点
```
zkc.writeData("/super/c1", "新内容");
```

## 一个简单的例子 
代码在 bjsxt.zkclient.base
```java
public class ZkClientBase {

	/** zookeeper地址 */
	static final String CONNECT_ADDR = "192.168.1.31:2181,192.168.1.32:2181,192.168.1.33:2181";
	/** session超时时间 */
	static final int SESSION_OUTTIME = 10000;//ms
	
	
	public static void main(String[] args) throws Exception {
		ZkClient zkc = new ZkClient(new ZkConnection(CONNECT_ADDR), SESSION_OUTTIME);
		//1. create and delete方法

		zkc.createEphemeral("/temp");
		zkc.createPersistent("/super/c1", true);
		Thread.sleep(10000);
		zkc.delete("/temp");
		zkc.deleteRecursive("/super");
		
		//2. 设置path和data 并且读取子节点和每个节点的内容
		zkc.createPersistent("/super", "1234");
		zkc.createPersistent("/super/c1", "c1内容");
		zkc.createPersistent("/super/c2", "c2内容");
		List<String> list = zkc.getChildren("/super");
		for(String p : list){
			System.out.println(p);
			String rp = "/super/" + p;
			String data = zkc.readData(rp);
			System.out.println("节点为：" + rp + "，内容为: " + data);
		}
		
		//3. 更新和判断节点是否存在
		zkc.writeData("/super/c1", "新内容");
		System.out.println(zkc.readData("/super/c1").toString());
		System.out.println(zkc.exists("/super/c1"));
		
//		4.递归删除/super内容
		zkc.deleteRecursive("/super");
	}
}
```
输出  
```
c1
节点为：/super/c1，内容为: c1内容
c2
节点为：/super/c2，内容为: c2内容
新内容
true
```
# ZKClient Watch的使用  
我们发现，上述ZKClient里面并没有类似的watcher、watch参数，这也就是说我们开发人员无需关心反复注册watcher的问题，zkclient给我们提供了一套监听方式，我们可以使用监听节点的方式进行操作，剔除了繁琐的反复watcher操作、简化了代码的复杂程度  
## subscribeChildChanges方法 订阅子节点变化  
参数1：path路径  
参数2：实现了IZKChildListener接口的类（如：实例化IZKClientListener类） 只需要重写其handleChildChanges(String parentPath, List<String> currentChild)方法。其中  
- parentPath 为监听节点全路径  
- currentChilds为新的子节点列表  
IZKChildListener事件说明针对于下面三个事件触发：  
- 新增子节点
- 减少子节点  
- 删除节点  
注意： 不监听节点内容的变化  
举例  
```java
public class ZkClientWatcher1 {

	/** zookeeper地址 */
	static final String CONNECT_ADDR = "192.168.1.31:2181,192.168.1.32:2181,192.168.1.33:2181";
	/** session超时时间 */
	static final int SESSION_OUTTIME = 10000;//ms
	
	
	public static void main(String[] args) throws Exception {
		ZkClient zkc = new ZkClient(new ZkConnection(CONNECT_ADDR), SESSION_OUTTIME);
		
		//对父节点添加监听子节点变化。
		zkc.subscribeChildChanges("/super", new IZkChildListener() {
			@Override
			public void handleChildChange(String parentPath, List<String> currentChilds) throws Exception {
				System.out.println("parentPath: " + parentPath);
				System.out.println("currentChilds: " + currentChilds);
			}
		});
		
		Thread.sleep(3000);
		
		zkc.createPersistent("/super");
		Thread.sleep(1000);
		
		zkc.createPersistent("/super" + "/" + "c1", "c1内容");
		Thread.sleep(1000);
		
		zkc.createPersistent("/super" + "/" + "c2", "c2内容");
		Thread.sleep(1000);		
		
		zkc.delete("/super/c2");
		Thread.sleep(1000);	
		
		zkc.deleteRecursive("/super");
		Thread.sleep(Integer.MAX_VALUE);
		
	}
}

```
输出  
```
parentPath: /super
currentChilds: []
parentPath: /super
currentChilds: [c1]
parentPath: /super
currentChilds: [c1, c2]
parentPath: /super
currentChilds: [c1]
parentPath: /super
currentChilds: null
parentPath: /super
currentChilds: null
```

## subscribeDataChanges 订阅内容变化  
和前面的subscribeChildChanges类似  
- 参数1 路径  
- 参数2 IZkDataListener对象，重写handleDataDeleted(String path) 方法，可以得到删除节点的path  重写handleDataChange(String path, Object data)可以得到变更的节点和变更的内容   

举例  
```java
public class ZkClientWatcher2 {

	/** zookeeper地址 */
	static final String CONNECT_ADDR = "192.168.1.31:2181,192.168.1.32:2181,192.168.1.33:2181";
	/** session超时时间 */
	static final int SESSION_OUTTIME = 10000;//ms
	
	
	public static void main(String[] args) throws Exception {
		ZkClient zkc = new ZkClient(new ZkConnection(CONNECT_ADDR), SESSION_OUTTIME);
		
		zkc.createPersistent("/super", "1234");
		
		//对父节点添加监听子节点变化。
		zkc.subscribeDataChanges("/super", new IZkDataListener() {
			@Override
			public void handleDataDeleted(String path) throws Exception {
				System.out.println("删除的节点为:" + path);
			}
			
			@Override
			public void handleDataChange(String path, Object data) throws Exception {
				System.out.println("变更的节点为:" + path + ", 变更内容为:" + data);
			}
		});
		
		Thread.sleep(3000);
		zkc.writeData("/super", "456", -1);
		Thread.sleep(1000);

		zkc.delete("/super");
		Thread.sleep(Integer.MAX_VALUE);
	}
}
```

输出  
```
变更的节点为:/super, 变更内容为:456
删除的节点为:/super
```




















# 前言  
Zookeeper的原生API，就之前的那一些，用起来还是比较麻烦的，所以，有些工程师对原生的API接口进行了封装，简化了ZK的复杂性。  
1. 创建客户端的方法： ZKClient(Arguments)  
- 参数1：zkServer zookeeper服务器的地址，用","分割
- 参数2：sessionTimeout超时回话，为毫秒，默认是30000ms
- 参数3：connectionTimeOut 连接超时会话  
- 参数4：IZKConnection接口的实现类
- 参数5：zkSerializer 兹定于序列化实现  

需要引入zkclient的jar包，自行百度即可  

# ZKClient的基础操作
## ZKClient建立连接  
直接new即可  
```
String CONNECT_ADDR = "192.168.1.171:2181,192.168.1.172:2181,192.168.1.173:2181";  
ZkClient zkc = new ZkClient(new ZkConnection(CONNECT_ADDR), 10000);
```

## ZKClient创建节点  
方法1  和原生的差不多  
```
zkc.create(final String path, Object data, final CreateMode mode)
```
方法2  
![zookeeper创建操作](http://images2015.cnblogs.com/blog/512650/201606/512650-20160611132540105-414970283.png)  
```java
zkc.createEphemeral("/temp");
// 可以支持递归的创建，但是不能递归赋值
zkc.createPersistent("/super/c1", true);
```

## 删除节点的操作
支持递归的删除  
```java
删除 /temp节点
zkc.delete("/temp");
//递归删除/super
zkc.deleteRecursive("/super");
```

## 获取子节点和阅读节点内容操作
```java
List<String> list = zkc.getChildren("/super");
for(String p : list){
	System.out.println(p);
	String rp = "/super/" + p;
	String data = zkc.readData(rp);
	System.out.println("节点为：" + rp + "，内容为: " + data);
}

```  
## 更新节点
```
zkc.writeData("/super/c1", "新内容");
```

## 一个简单的例子 
代码在 bjsxt.zkclient.base
```java
public class ZkClientBase {

	/** zookeeper地址 */
	static final String CONNECT_ADDR = "192.168.1.31:2181,192.168.1.32:2181,192.168.1.33:2181";
	/** session超时时间 */
	static final int SESSION_OUTTIME = 10000;//ms
	
	
	public static void main(String[] args) throws Exception {
		ZkClient zkc = new ZkClient(new ZkConnection(CONNECT_ADDR), SESSION_OUTTIME);
		//1. create and delete方法

		zkc.createEphemeral("/temp");
		zkc.createPersistent("/super/c1", true);
		Thread.sleep(10000);
		zkc.delete("/temp");
		zkc.deleteRecursive("/super");
		
		//2. 设置path和data 并且读取子节点和每个节点的内容
		zkc.createPersistent("/super", "1234");
		zkc.createPersistent("/super/c1", "c1内容");
		zkc.createPersistent("/super/c2", "c2内容");
		List<String> list = zkc.getChildren("/super");
		for(String p : list){
			System.out.println(p);
			String rp = "/super/" + p;
			String data = zkc.readData(rp);
			System.out.println("节点为：" + rp + "，内容为: " + data);
		}
		
		//3. 更新和判断节点是否存在
		zkc.writeData("/super/c1", "新内容");
		System.out.println(zkc.readData("/super/c1").toString());
		System.out.println(zkc.exists("/super/c1"));
		
//		4.递归删除/super内容
		zkc.deleteRecursive("/super");
	}
}
```
输出  
```
c1
节点为：/super/c1，内容为: c1内容
c2
节点为：/super/c2，内容为: c2内容
新内容
true
```
# ZKClient Watch的使用  
我们发现，上述ZKClient里面并没有类似的watcher、watch参数，这也就是说我们开发人员无需关心反复注册watcher的问题，zkclient给我们提供了一套监听方式，我们可以使用监听节点的方式进行操作，剔除了繁琐的反复watcher操作、简化了代码的复杂程度  
## subscribeChildChanges方法 订阅子节点变化  
参数1：path路径  
参数2：实现了IZKChildListener接口的类（如：实例化IZKClientListener类） 只需要重写其handleChildChanges(String parentPath, List<String> currentChild)方法。其中  
- parentPath 为监听节点全路径  
- currentChilds为新的子节点列表  
IZKChildListener事件说明针对于下面三个事件触发：  
- 新增子节点
- 减少子节点  
- 删除节点  
注意： 不监听节点内容的变化  
举例  
```java
public class ZkClientWatcher1 {

	/** zookeeper地址 */
	static final String CONNECT_ADDR = "192.168.1.31:2181,192.168.1.32:2181,192.168.1.33:2181";
	/** session超时时间 */
	static final int SESSION_OUTTIME = 10000;//ms
	
	
	public static void main(String[] args) throws Exception {
		ZkClient zkc = new ZkClient(new ZkConnection(CONNECT_ADDR), SESSION_OUTTIME);
		
		//对父节点添加监听子节点变化。
		zkc.subscribeChildChanges("/super", new IZkChildListener() {
			@Override
			public void handleChildChange(String parentPath, List<String> currentChilds) throws Exception {
				System.out.println("parentPath: " + parentPath);
				System.out.println("currentChilds: " + currentChilds);
			}
		});
		
		Thread.sleep(3000);
		
		zkc.createPersistent("/super");
		Thread.sleep(1000);
		
		zkc.createPersistent("/super" + "/" + "c1", "c1内容");
		Thread.sleep(1000);
		
		zkc.createPersistent("/super" + "/" + "c2", "c2内容");
		Thread.sleep(1000);		
		
		zkc.delete("/super/c2");
		Thread.sleep(1000);	
		
		zkc.deleteRecursive("/super");
		Thread.sleep(Integer.MAX_VALUE);
		
	}
}

```
输出  
```
parentPath: /super
currentChilds: []
parentPath: /super
currentChilds: [c1]
parentPath: /super
currentChilds: [c1, c2]
parentPath: /super
currentChilds: [c1]
parentPath: /super
currentChilds: null
parentPath: /super
currentChilds: null
```

## subscribeDataChanges 订阅内容变化  
和前面的subscribeChildChanges类似  
- 参数1 路径  
- 参数2 IZkDataListener对象，重写handleDataDeleted(String path) 方法，可以得到删除节点的path  重写handleDataChange(String path, Object data)可以得到变更的节点和变更的内容   

举例  
```java
public class ZkClientWatcher2 {

	/** zookeeper地址 */
	static final String CONNECT_ADDR = "192.168.1.31:2181,192.168.1.32:2181,192.168.1.33:2181";
	/** session超时时间 */
	static final int SESSION_OUTTIME = 10000;//ms
	
	
	public static void main(String[] args) throws Exception {
		ZkClient zkc = new ZkClient(new ZkConnection(CONNECT_ADDR), SESSION_OUTTIME);
		
		zkc.createPersistent("/super", "1234");
		
		//对父节点添加监听子节点变化。
		zkc.subscribeDataChanges("/super", new IZkDataListener() {
			@Override
			public void handleDataDeleted(String path) throws Exception {
				System.out.println("删除的节点为:" + path);
			}
			
			@Override
			public void handleDataChange(String path, Object data) throws Exception {
				System.out.println("变更的节点为:" + path + ", 变更内容为:" + data);
			}
		});
		
		Thread.sleep(3000);
		zkc.writeData("/super", "456", -1);
		Thread.sleep(1000);

		zkc.delete("/super");
		Thread.sleep(Integer.MAX_VALUE);
	}
}
```

输出  
```
变更的节点为:/super, 变更内容为:456
删除的节点为:/super
```
特别感谢互联网架构师白鹤翔老师，本文大多出自他的视频讲解。 
笔者主要是记录笔记，以便之后翻阅，正所谓好记性不如烂笔头，烂笔头不如云笔记





































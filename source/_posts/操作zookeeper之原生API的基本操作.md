---
title: 操作zookeeper之原生API的基本操作
date: 2017-12-10 21:25:21
tags: [zookeeper]
categories: [架构]
author: kaishun
id: 123
permalink: zookeeper4
---


**关键字：** java原生API，创建连接，创建节点同步方式，获取节点信息，获取子节点信息，修改节点的值
，判断节点是否存在，删除节点，Zookeeper创建删除等节点的异步方式
java惭怍zooleeper，一种是原生API，一种是zkclient方式，一种是curator框架操作    
github： https://github.com/zhaikaishun/zookeeper_tutorial

<!-- more -->



## java原生API  
引入对应版本的jar包，例如我是3.4.5的zookeeper，那么引入zookeeper-3.4.5.jar  
简要介绍一下原生API的简单使用  
连接zookeeper  
```java
ZooKeeper zk = new ZooKeeper(String connectString, int sessionTimeout, Watcher watcher)
```  
- connectString 连接服务器裂变，用逗号分开，例如192.168.1.31:2181,192.168.1.32:2181,192.168.1.33:2181，当然写一个就可以了    
- sessionTimeout 心跳检测时间周期（毫秒）    
- Watcher watcher，事件处理通知器    
- 这个其实有很多个构造方法，具体的，敲一下代码看一下就一清二楚了  
比如下面这个例子吧
## 创建连接  
```java
public class ZookeeperBase {

	/** zookeeper地址 */
	static final String CONNECT_ADDR = "192.168.1.31:2181,192.168.1.32:2181,192.168.1.33:2181";
	/** session超时时间 */
	static final int SESSION_OUTTIME = 2000;//ms 
	/** 信号量，阻塞程序执行，用于等待zookeeper连接成功，发送成功信号 */
	static final CountDownLatch connectedSemaphore = new CountDownLatch(1);
	
	public static void main(String[] args) throws Exception{
		
		ZooKeeper zk = new ZooKeeper(CONNECT_ADDR, SESSION_OUTTIME, new Watcher(){
			@Override
			public void process(WatchedEvent event) {
				//获取事件的状态
				KeeperState keeperState = event.getState();
				EventType eventType = event.getType();
				//如果是建立连接
				if(KeeperState.SyncConnected == keeperState){
					if(EventType.None == eventType){
						//如果建立连接成功，则发送信号量，让后续阻塞程序向下执行
						System.out.println("zk 建立连接");
						connectedSemaphore.countDown();
					}
				}
			}
		});

		//进行阻塞
		connectedSemaphore.await();

		System.out.println("执行了");
	}
}

```  
1. new 了一个Zookeeper的实例，注意内部有一个new Watcher()方法，并且重写了Watcher的process方法   
2. WatchedEvent.getState()，返回一个事件的状态KeeperState，keeperState状态有很多种：例如 SyncConnected（异步连接了）AuthFailed（认证失败） ConnectedReadOnly（只读连接）Disconnected（断开连接）等  
3. EventType.getType返回一个节点事件，有 None(没有任何操作) ，NodeCreated(节点创建) NodeDeleted(节点删除),NodeDataChanged(节点数据改变),NodeChildrenChanged(子节点改变) 等等的事件  
4. 注意这里用了一个CountDownLatch来进行线程之间通线，由于上面的**客户端和服务器端回话的建立是一个异步过程**， 所以程序会往下执行，但是到了connectedSemaphore.await() 进行阻塞，直到connectedSemaphore.countDown();之后，阻塞被唤醒，才往下执行。  
当zookeeper集群启动后，运行代码  
```
zk 建立连接
执行了
```  
## 创建节点同步方式    
同步方式： 
- 参数1，节点路径（名称）：、nodeName （不允许递归创建节点，也就是说父节点不存在的情况下，不允许创建子节点）  
- 参数2，节点内容：要求类型是字节数组（也就是说，不支持序列化方式，如果要实现序列化，可以用java相关序列化框架。如Kryo框架等）  
- 参数3，节点权限，一般使用Ids.OPEN_ACL_UNSAFE权限即可。（这个参数一般在权限没有太高要求的场景下使用）  
- 参数4，节点类型：创建节点的类型：CreateMode.*。提供四种节点类型  
PERSISTENT（持久节点）, PERSISTENT_SEQUENTIAL（持久顺序节点）, EPHEMERAL（临时节点）, EPHEMERAL_SEQUENTIAL（临时顺序节点）  这几种节点很重要，具体作用和案例，可以参考网上的。例如临时节点一般只在本次session中有效，经常用来做分布式锁  
  
**创建父节点**  
```java
		String result = zk.create("/testRoot", "testRoot".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
		System.out.println(result);
	--------输出--------
	/testRoot
```  
如果上面的目录存在，再去创建就会报一下错误  
```
KeeperException$NodeExistsException: KeeperErrorCode = NodeExists for /testRoot
```  
**创建子节点**：  
```
		//创建子节点
		zk.create("/testRoot/children", "children data".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
```  
如果子节点的父节点不存在，那么会报  KeeperErrorCode = NoNode for /testRoot1/children 错误  

 

## 获取节点信息  

```java
		byte[] data = zk.getData("/testRoot", false, null);
		System.out.println(new String(data));
```
## 获取子节点信息  
获取相对路径  
```
System.out.println(zk.getChildren("/testRoot", false));
```  
通过相对路径加上前面的路径，然后可以一次获取子节点的信息

## 修改节点的值  
```java
		zk.setData("/testRoot", "modify data root".getBytes(), -1);
		byte[] data = zk.getData("/testRoot", false, null);
		System.out.println(new String(data));
```


## 判断节点是否存在  
```java
zk.exists("/testRoot/children", false)
```
## 删除节点  
```java
zk.delete("/testRoot/children", -1);
```

## Zookeeper创建删除等节点的异步方式  
在同步参数基础上增加两个参数  
- 参数5，注册一个异步回调函数，要实现AsynCallBack.StringCallBack接口，重写processResult(int rc,String path,Object ctx,String name)方法，当节点创建完毕后执行此方法。  
**rc**: 为服务端响应代码0表示调用成功、-4表示端口连接、-110表示指定节点存在、-112表示回话已经过期。  
**path**：接口调用时传入API的数据节点的路径参数。  
**ctx**：为调用接口传入API的ctx值。    
**name**：实际在服务器端创建节点的名称。  
- 参数6，传递给回调函数的参数，一般为上下文（Context）信息  
举例：  
```
		zk.delete("/node01", -1, new AsyncCallback.VoidCallback() {
			@Override
			public void processResult(int rc, String path, Object ctx) {
				try {
					Thread.sleep(1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println("rc: "+rc);
				System.out.println("path: "+path);
				System.out.println("ctx: "+ctx);
			}
		},"a");
		System.out.println("继续执行");
		Thread.sleep(5000);

```
输出  
```
继续执行
rc: 0
path: /node01
ctx: a
```


特别感谢互联网架构师白鹤翔老师，本文大多出自他的视频讲解。 
笔者主要是记录笔记，以便之后翻阅，正所谓好记性不如烂笔头，烂笔头不如云笔记











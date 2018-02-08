---
title: zookeeper的watch（原生API）
date: 2017-12-20 21:25:21
tags: [zookeeper]
categories: [架构]
author: kaishun
id: 124
permalink: zookeeper5
---


github: https://github.com/zhaikaishun/zookeeper_tutorial


----------



## Zookeeper的watcher事件  
zookeeper有watch事件，是**一次性触发**的，当watch监视的数据发生变化时，通知设置了该watch的client，即watcher。  
同样，其watcher是监听数据发生了某些变化，那就一定会有对应的事件类型，和状态类型。   
<!-- more -->
**事件类型**（znode节点相关的）  
- EventType.NodeCreated  
- EventType.NodeDataChanged  
- EventType.NodeChildrenChanged  
- EventType.NodeDeleted  
**状态类型：**（是跟客户端实例相关的,简单的说就是客户端和服务器端连接状态相关的）    
- KeeperState.Disconnected  
- KeeperState.SyncConnected
- KeeperState.AuthFailed  认证失败
- KeeperState.Expired   过期  

## watcher和watch
简单的说，一个节点上的某个程序监控某个节点，那么这个节点上的这个程序就是一个watcher，而监听的这个事件（动作），就是一个watch。watch事件，是**一次性触发**的，只能监听一次，第二次对此节点的修改就监听不到了，如果想一直监听，大概有两种方案，一种是在出发事件后执行方法的时候有个watch的参数再设置为true，一种是这个时候再创建一个watch，这种是有点麻烦。   
看一个例子吧：    
最好是去github上下载下来自己运行一下。  

- 设置watcher, 我这里每次create的时候，都设置了一下watcher，例如下面代码中 this.zk.exists(path, ifsetTrue);  
- 需要实现implements Watcher接口以及重写实现方法process，我这里监听比较简单。也就不多说了。  
- 反正是要注意，如果只设置一次监听，那么监听完之后，第二次就监听不到了。若需要多次监听，那么最好是再监听一次
```
/**
 * Zookeeper Wathcher 
 * 本类就是一个Watcher类（实现了org.apache.zookeeper.Watcher类）
 * @author（alienware）
 * @since 2015-6-14
 */
public class ZooKeeperWatcher implements Watcher {

	/** 定义原子变量 */
	AtomicInteger seq = new AtomicInteger();
	/** 定义session失效时间 */
	private static final int SESSION_TIMEOUT = 10000;
	/** zookeeper服务器地址 */
	private static final String CONNECTION_ADDR = "192.168.1.31:2181";
	/** zk父路径设置 */
	private static final String PARENT_PATH = "/testWatch";
	/** zk子路径设置 */
	private static final String CHILDREN_PATH = "/testWatch/children";
	/** 进入标识 */
	private static final String LOG_PREFIX_OF_MAIN = "【Main】";
	/** zk变量 */
	private ZooKeeper zk = null;
	/** 信号量设置，用于等待zookeeper连接建立之后 通知阻塞程序继续向下执行 */
	private CountDownLatch connectedSemaphore = new CountDownLatch(1);

	/**
	 * 创建ZK连接
	 * @param connectAddr ZK服务器地址列表
	 * @param sessionTimeout Session超时时间
	 */
	public void createConnection(String connectAddr, int sessionTimeout) {
		this.releaseConnection();
		try {
			zk = new ZooKeeper(connectAddr, sessionTimeout, this);
			System.out.println(LOG_PREFIX_OF_MAIN + "开始连接ZK服务器");
			connectedSemaphore.await();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * 关闭ZK连接
	 */
	public void releaseConnection() {
		if (this.zk != null) {
			try {
				this.zk.close();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

	/**
	 * 创建节点
	 * @param path 节点路径
	 * @param data 数据内容
	 * @return 
	 */
	public boolean createPath(String path, String data,boolean ifsetTrue) {
		try {
			//设置监控(由于zookeeper的监控都是一次性的所以 每次必须设置监控)
			this.zk.exists(path, ifsetTrue);
			System.out.println(LOG_PREFIX_OF_MAIN + "节点创建成功, Path: " + 
							   this.zk.create(	/**路径*/ 
									   			path, 
									   			/**数据*/
									   			data.getBytes(), 
									   			/**所有可见*/
								   				Ids.OPEN_ACL_UNSAFE, 
								   				/**永久存储*/
								   				CreateMode.PERSISTENT ) + 	
							   ", content: " + data);
		} catch (Exception e) {
			e.printStackTrace();
			return false;
		}
		return true;
	}

	/**
	 * 读取指定节点数据内容
	 * @param path 节点路径
	 * @return
	 */
	public String readData(String path, boolean needWatch) {
		try {
			return new String(this.zk.getData(path, needWatch, null));
		} catch (Exception e) {
			e.printStackTrace();
			return "";
		}
	}

	/**
	 * 更新指定节点数据内容
	 * @param path 节点路径
	 * @param data 数据内容
	 * @return
	 */
	public boolean writeData(String path, String data) {
		try {
			System.out.println(LOG_PREFIX_OF_MAIN + "更新数据成功，path：" + path + ", stat: " +
								this.zk.setData(path, data.getBytes(), -1));
		} catch (Exception e) {
			e.printStackTrace();
		}
		return false;
	}

	/**
	 * 删除指定节点
	 * 
	 * @param path
	 *            节点path
	 */
	public void deleteNode(String path) {
		try {
			this.zk.delete(path, -1);
			System.out.println(LOG_PREFIX_OF_MAIN + "删除节点成功，path：" + path);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * 判断指定节点是否存在
	 * @param path 节点路径
	 */
	public Stat exists(String path, boolean needWatch) {
		try {
			return this.zk.exists(path, needWatch);
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}

	/**
	 * 获取子节点
	 * @param path 节点路径
	 */
	private List<String> getChildren(String path, boolean needWatch) {
		try {
			return this.zk.getChildren(path, needWatch);
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}

	/**
	 * 删除所有节点
	 */
	public void deleteAllTestPath() {
		if(this.exists(CHILDREN_PATH, false) != null){
			this.deleteNode(CHILDREN_PATH);
		}
		if(this.exists(PARENT_PATH, false) != null){
			this.deleteNode(PARENT_PATH);
		}		
	}
	
	/**
	 * 收到来自Server的Watcher通知后的处理。
	 */
	@Override
	public void process(WatchedEvent event) {
		
		System.out.println("进入 process 。。。。。event = " + event);
		
		try {
			Thread.sleep(200);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		if (event == null) {
			return;
		}
		
		// 连接状态
		KeeperState keeperState = event.getState();
		// 事件类型
		EventType eventType = event.getType();
		// 受影响的path
		String path = event.getPath();
		
		String logPrefix = "【Watcher-" + this.seq.incrementAndGet() + "】";

		System.out.println(logPrefix + "收到Watcher通知");
		System.out.println(logPrefix + "连接状态:\t" + keeperState.toString());
		System.out.println(logPrefix + "事件类型:\t" + eventType.toString());

		if (KeeperState.SyncConnected == keeperState) {
			// 成功连接上ZK服务器
			if (EventType.None == eventType) {
				System.out.println(logPrefix + "成功连接上ZK服务器");
				connectedSemaphore.countDown();
			} 
			//创建节点
			else if (EventType.NodeCreated == eventType) {
				System.out.println(logPrefix + "节点创建");
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				this.exists(path, true);
			} 
			//更新节点
			else if (EventType.NodeDataChanged == eventType) {
				System.out.println(logPrefix + "节点数据更新");
				System.out.println("我看看走不走这里........");
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println(logPrefix + "数据内容: " + this.readData(PARENT_PATH, true));
			} 
			//更新子节点
			else if (EventType.NodeChildrenChanged == eventType) {
				System.out.println(logPrefix + "子节点变更");
				try {
					Thread.sleep(3000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println(logPrefix + "子节点列表：" + this.getChildren(PARENT_PATH, true));
			} 
			//删除节点
			else if (EventType.NodeDeleted == eventType) {
				System.out.println(logPrefix + "节点 " + path + " 被删除");
			}
			else ;
		} 
		else if (KeeperState.Disconnected == keeperState) {
			System.out.println(logPrefix + "与ZK服务器断开连接");
		} 
		else if (KeeperState.AuthFailed == keeperState) {
			System.out.println(logPrefix + "权限检查失败");
		} 
		else if (KeeperState.Expired == keeperState) {
			System.out.println(logPrefix + "会话失效");
		}
		else ;

		System.out.println("--------------------------------------------");

	}

	/**
	 * <B>方法名称：</B>测试zookeeper监控<BR>
	 * <B>概要说明：</B>主要测试watch功能<BR>
	 * @param args
	 * @throws Exception
	 */
	public static void main(String[] args) throws Exception {

		//建立watcher
		ZooKeeperWatcher zkWatch = new ZooKeeperWatcher();
		//创建连接
		zkWatch.createConnection(CONNECTION_ADDR, SESSION_TIMEOUT);
		//System.out.println(zkWatch.zk.toString());
		
		Thread.sleep(1000);
		
		// 清理节点
		zkWatch.deleteAllTestPath();
		
		if (zkWatch.createPath(PARENT_PATH, System.currentTimeMillis() + "",true)) {
			
			Thread.sleep(1000);
			
			
			// 读取数据
			System.out.println("---------------------- read parent ----------------------------");
			//zkWatch.readData(PARENT_PATH, true);
			
			// 读取子节点
			System.out.println("---------------------- read children path ----------------------------");
			zkWatch.getChildren(PARENT_PATH, true);

			// 更新数据
			zkWatch.writeData(PARENT_PATH, System.currentTimeMillis() + "");
			
			Thread.sleep(1000);
			
			// 创建子节点
			zkWatch.createPath(CHILDREN_PATH, System.currentTimeMillis() + "",true);
			
			Thread.sleep(1000);
			
			zkWatch.writeData(CHILDREN_PATH, System.currentTimeMillis() + "");
		}
		
		Thread.sleep(50000);
		// 清理节点
		zkWatch.deleteAllTestPath();
		Thread.sleep(1000);
		zkWatch.releaseConnection();
	}

}

```

输出就在这里了，想具体了解的话，自己敲一下，然后覆盖一下代码。一个一个功能的执行，查看功能即可    
```
【Main】开始连接ZK服务器
进入 process 。。。。。event = WatchedEvent state:SyncConnected type:None path:null
【Watcher-1】收到Watcher通知
【Watcher-1】连接状态:	SyncConnected
【Watcher-1】事件类型:	None
【Watcher-1】成功连接上ZK服务器
--------------------------------------------
进入 process 。。。。。event = WatchedEvent state:SyncConnected type:NodeCreated path:/testWatch
【Main】节点创建成功, Path: /testWatch, content: 1508077399368
【Watcher-2】收到Watcher通知
【Watcher-2】连接状态:	SyncConnected
【Watcher-2】事件类型:	NodeCreated
【Watcher-2】节点创建
--------------------------------------------
---------------------- read parent ----------------------------
---------------------- read children path ----------------------------
进入 process 。。。。。event = WatchedEvent state:SyncConnected type:NodeDataChanged path:/testWatch
【Main】更新数据成功，path：/testWatch, stat: 30064771078,30064771079,1508071193606,1508071194762,1,0,0,0,13,0,30064771078

【Watcher-3】收到Watcher通知
【Watcher-3】连接状态:	SyncConnected
【Watcher-3】事件类型:	NodeDataChanged
【Watcher-3】节点数据更新
我看看走不走这里........
【Watcher-3】数据内容: 1508077400538
--------------------------------------------
进入 process 。。。。。event = WatchedEvent state:SyncConnected type:NodeCreated path:/testWatch/children
【Main】节点创建成功, Path: /testWatch/children, content: 1508077401581
【Watcher-4】收到Watcher通知
【Watcher-4】连接状态:	SyncConnected
【Watcher-4】事件类型:	NodeCreated
【Watcher-4】节点创建
--------------------------------------------
进入 process 。。。。。event = WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/testWatch
【Watcher-5】收到Watcher通知
【Watcher-5】连接状态:	SyncConnected
【Watcher-5】事件类型:	NodeChildrenChanged
【Watcher-5】子节点变更
【Main】更新数据成功，path：/testWatch/children, stat: 30064771080,30064771081,1508071195777,1508071196784,1,0,0,0,13,0,30064771080

【Watcher-5】子节点列表：[children]
--------------------------------------------
进入 process 。。。。。event = WatchedEvent state:SyncConnected type:NodeDataChanged path:/testWatch/children
【Watcher-6】收到Watcher通知
【Watcher-6】连接状态:	SyncConnected
【Watcher-6】事件类型:	NodeDataChanged
【Watcher-6】节点数据更新
我看看走不走这里........
【Watcher-6】数据内容: 1508077400538
--------------------------------------------
进入 process 。。。。。event = WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/testWatch
【Main】删除节点成功，path：/testWatch/children
【Main】删除节点成功，path：/testWatch
【Watcher-7】收到Watcher通知
【Watcher-7】连接状态:	SyncConnected
【Watcher-7】事件类型:	NodeChildrenChanged
【Watcher-7】子节点变更

```  

## 实际应用一个场景  
我们希望zookeeper对分布式系统的配置文件进行管理，也就是说多个服务器进行watcher,zookeeper节点发送变化，则我们实时更新配置文件。我们要完成多个应用服务器注册watcher，然后实时观察数据的变化，然后反馈给媒体服务器变更的数据信息，观察zookeeper节点

下面是一个例子： 代码在bjsxt.zookeeper.cluster中  
本例子模拟多台服务器同时监控一个节点。然后另外一个程序进行管理，所监控的这几台机器得到节点变更的通知。  
本例中，Client1和Client2相当于两台服务器，共同watch一个节点。Test相当于管理者，用来管理这两个客户端的配置。  
代码如下,具体的需要自己下载下来进行调试  
Client1
```java
public class Client1 {

	public static void main(String[] args) throws Exception{
		
		ZKWatcher myWatcher = new ZKWatcher();
		Thread.sleep(100000000);
	}
}
```  
Client2
```
public class Client2 {

	public static void main(String[] args) throws Exception{
		
		ZKWatcher myWatcher = new ZKWatcher();
		Thread.sleep(100000000);
	}
}
```
ZKWatcher
```java
public class ZKWatcher implements Watcher {

	/** zk变量 */
	private ZooKeeper zk = null;
	
	/** 父节点path */
	static final String PARENT_PATH = "/super";
	
	/** 信号量设置，用于等待zookeeper连接建立之后 通知阻塞程序继续向下执行 */
	private CountDownLatch connectedSemaphore = new CountDownLatch(1);
	
	private List<String> cowaList = new CopyOnWriteArrayList<String>();
	
	
	/** zookeeper服务器地址 */
	public static final String CONNECTION_ADDR = "192.168.1.31:2181,192.168.1.32:2181,192.168.1.33:2181";
	/** 定义session失效时间 */
	public static final int SESSION_TIMEOUT = 30000;
	
	public ZKWatcher() throws Exception{
		zk = new ZooKeeper(CONNECTION_ADDR, SESSION_TIMEOUT, this);
		System.out.println("开始连接ZK服务器");
		connectedSemaphore.await();
	}


	@Override
	public void process(WatchedEvent event) {
		// 连接状态
		KeeperState keeperState = event.getState();
		// 事件类型
		EventType eventType = event.getType();
		// 受影响的path
		String path = event.getPath();
		System.out.println("受影响的path : " + path);
		
		
		if (KeeperState.SyncConnected == keeperState) {
			// 成功连接上ZK服务器
			if (EventType.None == eventType) {
				System.out.println("成功连接上ZK服务器");
				connectedSemaphore.countDown();
				try {
					if(this.zk.exists(PARENT_PATH, false) == null){
						this.zk.create(PARENT_PATH, "root".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);		 
					}
					List<String> paths = this.zk.getChildren(PARENT_PATH, true);
					for (String p : paths) {
						System.out.println(p);
						this.zk.exists(PARENT_PATH + "/" + p, true);
					}
				} catch (KeeperException | InterruptedException e) {
					e.printStackTrace();
				}		
			} 
			//创建节点
			else if (EventType.NodeCreated == eventType) {
				System.out.println("节点创建");
				try {
					this.zk.exists(path, true);
				} catch (KeeperException | InterruptedException e) {
					e.printStackTrace();
				}
			} 
			//更新节点
			else if (EventType.NodeDataChanged == eventType) {
				System.out.println("节点数据更新");
				try {
					//update nodes  call function
					this.zk.exists(path, true);
				} catch (KeeperException | InterruptedException e) {
					e.printStackTrace();
				}
			} 
			//更新子节点
			else if (EventType.NodeChildrenChanged == eventType) {
				System.out.println("子节点 ... 变更");
				try {
					List<String> paths = this.zk.getChildren(path, true);
					if(paths.size() >= cowaList.size()){
						paths.removeAll(cowaList);
						for(String p : paths){
							this.zk.exists(path + "/" + p, true);
							//this.zk.getChildren(path + "/" + p, true);
							System.out.println("这个是新增的子节点 : " + path + "/" + p);
							//add new nodes  call function
						}
						cowaList.addAll(paths);
					} else {
						cowaList = paths;
					}
					System.out.println("cowaList: " + cowaList.toString());
					System.out.println("paths: " + paths.toString());
					
				} catch (KeeperException | InterruptedException e) {
					e.printStackTrace();
				}
			} 
			//删除节点
			else if (EventType.NodeDeleted == eventType) {
				System.out.println("节点 " + path + " 被删除");
				try {
					//delete nodes  call function
					this.zk.exists(path, true);
				} catch (KeeperException | InterruptedException e) {
					e.printStackTrace();
				}
			}
			else ;
		} 
		else if (KeeperState.Disconnected == keeperState) {
			System.out.println("与ZK服务器断开连接");
		} 
		else if (KeeperState.AuthFailed == keeperState) {
			System.out.println("权限检查失败");
		} 
		else if (KeeperState.Expired == keeperState) {
			System.out.println("会话失效");
		}
		else ;

		System.out.println("--------------------------------------------");
	}
	
	

}
```
Test
```
public class Test {


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
						connectedSemaphore.countDown();
						System.out.println("zk 建立连接");
					}
				}
			}
		});

		//进行阻塞
		connectedSemaphore.await();
		
//		//创建子节点
//		zk.create("/super/c1", "c1".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
		//创建子节点
//		zk.create("/super/c2", "c2".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
		//创建子节点
		String result = zk.create("/super/c3", "c3".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
		System.out.println(result);
		//创建子节点
//		zk.create("/super/c4", "c4".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
		
//		zk.create("/super/c4/c44", "c44".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
		
		//获取节点信息
//		byte[] data = zk.getData("/testRoot", false, null);
//		System.out.println(new String(data));
//		System.out.println(zk.getChildren("/testRoot", false));
		
		//修改节点的值
//		zk.setData("/super/c1", "modify c1".getBytes(), -1);
//		zk.setData("/super/c2", "modify c2".getBytes(), -1);
//		byte[] data = zk.getData("/super/c2", false, null);
//		System.out.println(new String(data));		
		
//		//判断节点是否存在
//		System.out.println(zk.exists("/super/c3", false));
//		//删除节点
//		zk.delete("/super/c3", -1);

		zk.close();
	
	}
}

```

先启动Client1,client1打印  
```java
开始连接ZK服务器
受影响的path : null
成功连接上ZK服务器
```

再启动Client2,
```java
开始连接ZK服务器
受影响的path : null
成功连接上ZK服务器
--------------------------------------------
```

再启动Test类，此时Client1和Client2打印的内容是  
```java
开始连接ZK服务器
受影响的path : null
成功连接上ZK服务器
--------------------------------------------  
受影响的path : /super
子节点 ... 变更
这个是新增的子节点 : /super/c3
cowaList: [c3]
paths: [c3]

```
Test打印的内容是  
```java
zk 建立连接
/super/c3
```
更多测试，需要自己来运行并且思考结果。这里只起抛砖引玉的作用  


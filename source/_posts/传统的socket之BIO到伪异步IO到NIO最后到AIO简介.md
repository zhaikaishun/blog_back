---
title: 传统的socket之BIO到伪异步IO到NIO最后到AIO简介
date: 2017-07-30 21:25:21
tags: [Netty]
categories: [架构]
author: kaishun
id: 80
permalink: BIO-NIO-AIO
---
关键字：NIO， IO,BIO,AIO的简介以及演变原因
 
本人对nio确实也了解的不深，此文只是简介    
代码在 https://github.com/zhaikaishun/NettyTutorial  下的socket01
<!-- more -->



## 传统的BIO通信：同步阻塞模式  
看图，侵删  

![bio通信模型图](http://blog.anxpp.com/usr/uploads/2016/05/549520916.png)  

传统的模式是BIO模式，是同步阻塞的，直接看下面这个例子吧。
**Server端**
```java
        ServerSocket server = null; //bio, 使用一个ServerSocket类进行socket传输
        int PROT = 9999;
		try {
			server = new ServerSocket(PROT);
			System.out.println(" server start .. ");
			//进行阻塞，
			while (true){
				Socket socket = server.accept();//!!直到client建立socket连接的时候，才会走下面的操作
				System.out.println("client建立scoket连接后才走这里");
				//新建一个线程执行客户端的任务
				new Thread(new ServerHandler(socket)).start();
			}

		//下面的不用看了
		} catch (Exception e)
....

```
**ServerHandler处理类**,主要用来输出，这里停留5000毫秒
```
in = new BufferedReader(new InputStreamReader(this.socket.getInputStream()));
out = new PrintWriter(this.socket.getOutputStream(), true);
String body = null;
while((body = in.readLine())!=null){
	System.out.println("Server :" + body);
	Thread.sleep(5000);
	//给客户端响应
	out.println("这是服务器端回送响的应数据.");
}
```

**Client端**
```
String ADDRESS = "127.0.0.1";
int PORT = 9999;

socket = new Socket(ADDRESS, PORT);
in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
out = new PrintWriter(socket.getOutputStream(), true);

//向服务器端发送数据
out.println("接收到客户端的请求数据...");
System.out.println("执行完这一句后，下面的in.readLine是阻塞的");
//in.readLine();是阻塞的，必须等待服务器端完成之后才能读的到
String response = in.readLine();
System.out.println("Client: " + response);

```

分析NIO的模式， 先运行Server，Server输出
```
 server start .. 
```
再运行client，client输出
```
执行完这一句后，下面的in.readLine是阻塞的  
```
Server端输出
```
client建立scoket连接后才走这里
Server :接收到客户端的请求数据...
```
5秒后再运行client输出: 
```
Client: 这是服务器端回送响的应数据.
```
总结： Server端的 Socket socket = server.accept(); 会造成阻塞，直到client建立连接的时候才会执行后面的操作
Client端的： String response = in.readLine(); 也会造成阻塞，直到Server.accept响应后才能执行后面的操作
而且，每个Client都会在Server端建立一个线程，如果client并发较多的时候，Server服务器会承受不住从而导致瘫痪

### 伪异步IO的模式
看图：侵删  
![伪异步IO](http://blog.anxpp.com/usr/uploads/2016/05/614169023.png)  
在没有实现NIO之前的一种模式
直接看例子,和上面那个差不错，只不过Server端使用了线程池而已,将客户端的socket封装成一个task任务，这样client并发多的时候，就会通过等待来执行，不会让线程一下子起的太多,下面就只见到记录一下Server端的代码，其他代码和上面的例子一致
**Server端**
```
int PORT = 9999;
ServerSocket server = null;
BufferedReader in = null;
PrintWriter out = null;
try {
	server = new ServerSocket(PORT);
	System.out.println("server start");
	Socket socket = null; 
	HandlerExecutorPool executorPool = new HandlerExecutorPool(50, 1000);  //其实整体都是一样，只不过这里再加了一个线程池而已
	while(true){
		socket = server.accept();
		executorPool.execute(new ServerHandler(socket));
	}
	
} catch (Exception e) 

```
**线程池封装类HandlerExecutorPool**
```
public class HandlerExecutorPool {

	private ExecutorService executor;
	public HandlerExecutorPool(int maxPoolSize, int queueSize){
		this.executor = new ThreadPoolExecutor(
				Runtime.getRuntime().availableProcessors(),
				maxPoolSize, 
				120L, 
				TimeUnit.SECONDS,
				new ArrayBlockingQueue<Runnable>(queueSize));
	}
	
	public void execute(Runnable task){
		this.executor.execute(task);
	}
}
```



## NIO和IO的关系与区别： 
阻塞概念：应用程序在获取网络数据的时候，如果网络传输很慢，那么程序就一直等着，直到传输完毕为止
非阻塞：应用程序直接可以获取已经准备好的数据，无需等待

NIO： 同步非阻塞

同步：应用程序直接参与io读写，程序会直接阻塞到某个方法上
异步： 所有的io都交给操作系统，当操作系统完成了io的操作的时候，会给我们应用程序发通知

同步异步说的是Server服务端的执行方式
阻塞说的是技术，接受数据的方式，状态（IO,NIO）

## NIO  
几大概念
buffer
channel
selecter 就是一个插座
![nio基本图](http://or49tneld.bkt.clouddn.com/17-9-10/30347089.jpg)


### buffer
使用一下flip方法复位，位置才回到0，但是后面空的会清零（比较好），不建议使用posistion的方法进行处理
具体的buffer操作案例，这里也不太想记了（nio真的直接用起来非常少）, 代码都在kaishun.nio.test.TestBuffer 类下  


## AIO  
异步非阻塞： 我这里也没有细看，暂时也不是很明白，TODO zhaikaishun 2017-06-10[有时间吧]  




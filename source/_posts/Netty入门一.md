---
title: Netty入门一
date: 2017-08-01 21:25:21
tags: [Netty]
categories: [架构]
author: kaishun
id: 81
permalink: netty1
---

关键字： Netty简介，Netty实现通信的步骤，绑定多个端口，TCP粘包、拆包问题，DellmiterBasedFrameDecoder(自定义分隔符)， FixedLengthFrameDecoder(定长)

代码在 https://github.com/zhaikaishun/NettyTutorial  下的socketIO02
<!-- more -->
---
## Netty简介  
Netty是一个高性能、异步事件驱动的NIO框架，它提供了对TCP、UDP和文件传输的支持，作为一个异步NIO框架，Netty的所有IO操作都是异步非阻塞的，通过Future-Listener机制，用户可以方便的主动获取或者通过通知机制获得IO操作结果。
作为当前最流行的NIO框架，Netty在互联网领域、大数据分布式计算领域、游戏行业、通信行业等获得了广泛的应用，一些业界著名的开源组件也基于Netty的NIO框架构建。  

**Netty架构组成**
http://incdn1.b0.upaiyun.com/2015/04/28c8edde3d61a0411511d3b1866f0636.png
代码在SocketIO_02
## **Netty实现通信的步骤**    
**服务器端**  
1. 创建两个的NIO线程组， 一个专门用于网络事件处理（接受客户端的连接），另一个则进行网络通信读写  

2. 创建一个ServerBootstrap对象，配置Netty的一系列参数，例如接受传出数据的缓存大小等等。  

3. 创建一个实际处理数据的类Channellnitializer，进行初始化的准备工作，比如设置传出数据的字符集、格式、已经处理数据的接口  
4. 绑定端口，执行同步阻塞方法等待服务器端启动即可。  




**第一个Netty的例子**  
**Server端**  
```java
public class Server {

	public static void main(String[] args) throws Exception {
		//1 创建线两个程组 
		//一个是用于处理服务器端接收客户端连接的
		//一个是进行网络通信的（网络读写的）
		EventLoopGroup pGroup = new NioEventLoopGroup();
		EventLoopGroup cGroup = new NioEventLoopGroup();
		
		//2 创建辅助工具类，用于服务器通道的一系列配置
		ServerBootstrap b = new ServerBootstrap();
		b.group(pGroup, cGroup)		//绑定俩个线程组
		.channel(NioServerSocketChannel.class)		//指定NIO的模式
		.option(ChannelOption.SO_BACKLOG, 1024)		//设置tcp缓冲区
		.option(ChannelOption.SO_SNDBUF, 32*1024)	//设置发送缓冲大小
		.option(ChannelOption.SO_RCVBUF, 32*1024)	//这是接收缓冲大小
		.option(ChannelOption.SO_KEEPALIVE, true)	//保持连接
		.childHandler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				//3 在这里配置具体数据接收方法的处理
				sc.pipeline().addLast(new ServerHandler());
			}
		});
		
		//4 进行绑定 
		ChannelFuture cf1 = b.bind(8765).sync();
		//ChannelFuture cf2 = b.bind(8764).sync();
		//5 等待关闭
		cf1.channel().closeFuture().sync();
		//cf2.channel().closeFuture().sync();
		pGroup.shutdownGracefully();
		cGroup.shutdownGracefully();
	}
}
```
**ServerHandler**  
```java
public class ServerHandler extends ChannelHandlerAdapter {

	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		System.out.println("server channel active... ");
	}

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg)
			throws Exception {
			ByteBuf buf = (ByteBuf) msg;
			byte[] req = new byte[buf.readableBytes()];
			buf.readBytes(req);
			String body = new String(req, "utf-8");
			System.out.println("Server :" + body );
			String response = "进行返回给客户端的响应：" + body ;
			ctx.writeAndFlush(Unpooled.copiedBuffer(response.getBytes()));
			//.addListener(ChannelFutureListener.CLOSE);
	}

	@Override
	public void channelReadComplete(ChannelHandlerContext ctx)
			throws Exception {
		System.out.println("读完了");
		ctx.flush();
	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable t)
			throws Exception {
		ctx.close();
	}

}
```

**Client端**  
```java
public class Client {

	public static void main(String[] args) throws Exception{
		
		EventLoopGroup group = new NioEventLoopGroup();
		Bootstrap b = new Bootstrap();
		b.group(group)
		.channel(NioSocketChannel.class)
		.handler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				sc.pipeline().addLast(new ClientHandler());
			}
		});
		
		ChannelFuture cf1 = b.connect("127.0.0.1", 8765).sync();
		//ChannelFuture cf2 = b.connect("127.0.0.1", 8764).sync();
		//发送消息
		Thread.sleep(1000);
		cf1.channel().writeAndFlush(Unpooled.copiedBuffer("777".getBytes()));
		cf1.channel().writeAndFlush(Unpooled.copiedBuffer("666".getBytes()));
		//cf2.channel().writeAndFlush(Unpooled.copiedBuffer("888".getBytes()));
		Thread.sleep(2000);
		cf1.channel().writeAndFlush(Unpooled.copiedBuffer("888".getBytes()));
		//cf2.channel().writeAndFlush(Unpooled.copiedBuffer("666".getBytes()));
		
		cf1.channel().closeFuture().sync();
		//cf2.channel().closeFuture().sync();
		group.shutdownGracefully();
	}
}
```  
**ClientHandle**  
```java
public class ClientHandler extends ChannelHandlerAdapter{


	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
	}

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		try {
			ByteBuf buf = (ByteBuf) msg;
			
			byte[] req = new byte[buf.readableBytes()];
			buf.readBytes(req);
			
			String body = new String(req, "utf-8");
			System.out.println("Client :" + body );
			String response = "收到服务器端的返回信息：" + body;
		} finally {
			ReferenceCountUtil.release(msg);
		}
	}

	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {

	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause)
			throws Exception {
		ctx.close();
	}
}
```
先运行Server, 再运行Client    
Server端输出  
```java
server channel active... 
Server :777666
读完了
Server :888
读完了
```  
Client端输出  
```
Client :进行返回给客户端的响应：777666
Client :进行返回给客户端的响应：888
```  

例子解读：  
Server端是不停的监听Client端的消息, Client和Server端的连接是不会停止的，如果想要在Client和Server之间通线后停止Client和Server端的连接，在ServerHandler的channelRead中加上addListener(ChannelFutureListener.CLOSE)，然后在Client端添加cf1.channel().closeFuture().sync(); group.shutdownGracefully();  就是Client也在异步的监听，如果消息完成，就close掉连接    
例如上面那个例子  
ServerHandle处
```
ctx.writeAndFlush(Unpooled.copiedBuffer(response.getBytes())).addListener(ChannelFutureListener.CLOSE)  
```  
Client处添加  
```
cf1.channel().closeFuture().sync();
group.shutdownGracefully();
```

这样的话，在Client端通信一次之后就断开了，也就是Server只能收到777666的信息，888的信息是接收不到的。  
**注意**： 这个异步关闭，一定要在Server端进行关闭，不要在Client端关闭。   
如果不想关闭，想保持长连接，那就不加上面那一些代码即可  


## 绑定多个端口  
例如上面那个例子，希望再加一个端口  
Server端加上  
```
ChannelFuture cf2 = b.bind(8764).sync();  
cf2.channel().closeFuture().sync();
```
Client端  
```
ChannelFuture cf2 = b.connect("127.0.0.1", 8764).sync();  
cf2.channel().closeFuture().sync();  
```


## TCP粘包、拆包问题  
熟悉tcp编程的可能知道，无论是服务端还是客户端，当我们读取或者发送数据的时候，都需要考虑TCP底层的粘包个拆包机制。  
tcp是一个“流”协议，所谓流就是没有界限的传输数据，在业务上，一个完整的包可能会被TCP分成多个包进行发送，也可能把多个小包封装成一个大的数据包发送出去，这就是所谓的tcp粘包、拆包问题。  
分析TCP粘包拆包问题的产生原因：  
1.  应用程序write写入的字节大于套接口缓冲区的大小  
2.  进行mss大小的TCP分段  
3.  以太网的payload大于MTU进行IP分片
粘包拆包问题，根据业界主流协议，有三种主要方案： 
1. 消息定长，例如每个报文的大小固定200个字节，如果不够，空位补空格  
2. 在包尾部增加特殊字符进行分割，例如加回车等  
3. 将消息分为消息头和消息体，在消息头中包含表示消息总长度的字段，然后进行业务逻辑的处理。  


## TCP粘包拆包之分隔符类 DellmiterBasedFrameDecoder(自定义分隔符)  
代码在bhz.netty.ende1下  
**Server端**
```java
public class Server {

	public static void main(String[] args) throws Exception{
		//1 创建2个线程，一个是负责接收客户端的连接。一个是负责进行数据传输的
		EventLoopGroup pGroup = new NioEventLoopGroup();
		EventLoopGroup cGroup = new NioEventLoopGroup();
		
		//2 创建服务器辅助类
		ServerBootstrap b = new ServerBootstrap();
		b.group(pGroup, cGroup)
		 .channel(NioServerSocketChannel.class)
		 .option(ChannelOption.SO_BACKLOG, 1024)
		 .option(ChannelOption.SO_SNDBUF, 32*1024)
		 .option(ChannelOption.SO_RCVBUF, 32*1024)
		 .childHandler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				//设置特殊分隔符
				ByteBuf buf = Unpooled.copiedBuffer("$_".getBytes());
				sc.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, buf));
				//设置字符串形式的解码
				sc.pipeline().addLast(new StringDecoder());
				sc.pipeline().addLast(new ServerHandler());
			}
		});
		
		//4 绑定连接
		ChannelFuture cf = b.bind(8765).sync();
		
		//等待服务器监听端口关闭
		cf.channel().closeFuture().sync();
		pGroup.shutdownGracefully();
		cGroup.shutdownGracefully();
		
	}
	
}
```  
**ServerHandle端**  
```java
public class ServerHandler extends ChannelHandlerAdapter {


	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		System.out.println(" server channel active... ");
	}

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		String request = (String)msg;
		System.out.println("Server :" + msg);
		String response = "服务器响应：" + msg + "$_";
		ctx.writeAndFlush(Unpooled.copiedBuffer(response.getBytes()));
	}

	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		
	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable t) throws Exception {
		ctx.close();
	}
}
```
**Client端**  
```java
public class Client {

	public static void main(String[] args) throws Exception {
		
		EventLoopGroup group = new NioEventLoopGroup();
		
		Bootstrap b = new Bootstrap();
		b.group(group)
		 .channel(NioSocketChannel.class)
		 .handler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				//
				ByteBuf buf = Unpooled.copiedBuffer("$_".getBytes());
				sc.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, buf));
				sc.pipeline().addLast(new StringDecoder());
				sc.pipeline().addLast(new ClientHandler());
			}
		});
		
		ChannelFuture cf = b.connect("127.0.0.1", 8765).sync();
		
		cf.channel().writeAndFlush(Unpooled.wrappedBuffer("bbbb$_".getBytes()));
		cf.channel().writeAndFlush(Unpooled.wrappedBuffer("cccc$_".getBytes()));
		
		
		//等待客户端端口关闭
		cf.channel().closeFuture().sync();
		group.shutdownGracefully();
		
	}
}
```
ClientHandle端  
```java
public class ClientHandler extends ChannelHandlerAdapter{

	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		System.out.println("client channel active... ");
	}

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		try {
			String response = (String)msg;
			System.out.println("Client: " + response);
		} finally {
			ReferenceCountUtil.release(msg);
		}
	}

	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
		ctx.close();
	}

}

```
先启动Server端，再启动Client端  
Client端输出
```
client channel active... 
Client: 服务器响应：bbbb
Client: 服务器响应：cccc
```
Server端输出  
```
server channel active... 
Server :bbbb
Server :cccc
```





## TCP粘包拆包之FixedLengthFrameDecoder(定长)  

和上面的代码类似 
Server端
```
public class Server {

	public static void main(String[] args) throws Exception{
		//1 创建2个线程，一个是负责接收客户端的连接。一个是负责进行数据传输的
		EventLoopGroup pGroup = new NioEventLoopGroup();
		EventLoopGroup cGroup = new NioEventLoopGroup();
		
		//2 创建服务器辅助类
		ServerBootstrap b = new ServerBootstrap();
		b.group(pGroup, cGroup)
		 .channel(NioServerSocketChannel.class)
		 .option(ChannelOption.SO_BACKLOG, 1024)
		 .option(ChannelOption.SO_SNDBUF, 32*1024)
		 .option(ChannelOption.SO_RCVBUF, 32*1024)
		 .childHandler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				//设置定长字符串接收
				sc.pipeline().addLast(new FixedLengthFrameDecoder(5));
				//设置字符串形式的解码
				sc.pipeline().addLast(new StringDecoder());
				sc.pipeline().addLast(new ServerHandler());
			}
		});
		
		//4 绑定连接
		ChannelFuture cf = b.bind(8765).sync();
		
		//等待服务器监听端口关闭
		cf.channel().closeFuture().sync();
		pGroup.shutdownGracefully();
		cGroup.shutdownGracefully();
	}
}

```  
**ServerHandler**  
```
public class ServerHandler extends ChannelHandlerAdapter {


	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		System.out.println(" server channel active... ");
	}

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		String request = (String)msg;
		System.out.println("Server :" + msg);
		String response =  request ;
		ctx.writeAndFlush(Unpooled.copiedBuffer(response.getBytes()));
	}

	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		
	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable t) throws Exception {

	}
	
}

```  

**Client**  
```java
public class Client {

	public static void main(String[] args) throws Exception {
		
		EventLoopGroup group = new NioEventLoopGroup();
		
		Bootstrap b = new Bootstrap();
		b.group(group)
		 .channel(NioSocketChannel.class)
		 .handler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				sc.pipeline().addLast(new FixedLengthFrameDecoder(5));
				sc.pipeline().addLast(new StringDecoder());
				sc.pipeline().addLast(new ClientHandler());
			}
		});
		
		ChannelFuture cf = b.connect("127.0.0.1", 8765).sync();
		
		cf.channel().writeAndFlush(Unpooled.wrappedBuffer("aaaaabbbbb".getBytes()));
		cf.channel().writeAndFlush(Unpooled.copiedBuffer("ccccccc".getBytes()));
		
		//等待客户端端口关闭
		cf.channel().closeFuture().sync();
		group.shutdownGracefully();
		
	}
}
```  

**ClientHandler**  
```java
public class ClientHandler extends ChannelHandlerAdapter{

	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		System.out.println("client channel active... ");
	}

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		String response = (String)msg;
		System.out.println("Client: " + response);
	}

	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
	}

}

```

先运行Server端，再运行Client端    
Client端输出  
```
client channel active... 
Client: aaaaa
Client: bbbbb
Client: ccccc
```
Server端输出
```
 server channel active... 
Server :aaaaa
Server :bbbbb
Server :ccccc
```  
可以看到，每次定长都是5个字符， 有两个cc，由于不够五个，导致丢掉了  


## 自定义协议粘包拆包  
使用消息头和消息体， 暂未记录笔记。  TODO zhaikaishun  




特别感谢互联网架构师白鹤翔老师，本文大多出自他的视频讲解。
笔者主要是记录笔记，以便之后翻阅，正所谓好记性不如烂笔头，烂笔头不如云笔记











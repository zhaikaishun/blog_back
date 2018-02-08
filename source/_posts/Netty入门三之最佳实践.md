---
title: Netty入门三之最佳实践
date: 2017-08-07 21:25:21
tags: [java]
categories: [架构]
author: kaishun
id: 82
permalink: netty3
---


关键字 最佳实践： 数据通信，心跳检测

代码在 https://github.com/zhaikaishun/NettyTutorial 代码在SocketIO_03下  
<!-- more -->


# Netty最佳实践  

## 实际场景一：数据通信  
我们需要考虑两台或者多台机器使用Netty如何进行通信，作者个人大体上把他分为3种  

1.  第一种，使用长连接通道不断开的形式进行通信，也就是服务器和客户端的通道一直处于开启状态，如果服务器性能足够好，并且我们的客户端数量比较少的情况下，还是比较推荐这种方式的  

2.  第二种，一次性批量提交数据，采用短连接方式。也就是说数据先保存到临时表中，当到达临界值时进行一次性批量提交，这种情况是做不到实时传输，在对实时性不高的应用程序可以推荐使用。  

3. 第三种，我们可以使用一种特殊的长连接，在制定某一时间之内，服务器与某台客户端没有任何通信，则断开连接，下次连接则是客户端向服务器发送请求的时候，再次建立连接，但是这种模式我们需要考虑2个因素：  
第一点： 如何在超时后关闭通道？关闭通道后我们又如何再次建立连接？  
第二点： 客户端宕机时，我们无需考虑，下次客户端重启之后我们就可以与服务器建立连接，但是当服务器宕机时，我们的客户端如何与服务器进行连接呢？   

第一点解决办法：netty有自带的工具类在超时后关闭通道。关闭后，client再次发送连接请求即可再次建立连接  
第二点解决办法： 用一个脚本定时的监听Netty是否有宕机，若有宕机，则运行脚本再次启动Server端即可  

举个例子： 代码在SocketIO_03 bhz.netty.runtime, 具体的去github上看吧  
Server端，没什么改变，就是加了一个sc.pipeline().addLast(new ReadTimeoutHandler(5));  说明如果5秒钟某个Client没有和我通信，就断开和这个Client的连接。  
Server端  
```java
sc.pipeline().addLast(new ReadTimeoutHandler(5)); 

```  
同理Client端也是需要加这个  
```
sc.pipeline().addLast(new ReadTimeoutHandler(5)); 
```
如何让Client在断开后重连呢？  
主要是New一个线程，这个线程在断开后开启，然后通过判断是否连接关闭，如果连接关闭，那么就发起一个连接。  
```
	public ChannelFuture getChannelFuture(){
		
		if(this.cf == null){
			this.connect();
		}
		if(!this.cf.channel().isActive()){
			this.connect();
		}
		
		return this.cf;
	}
```


## 实际场景二：心跳检测

我们使用Socket通信一般经常会处理多个服务器之间的心跳检测，我们去维护服务器集群，肯定要有一台或多台服务器主机（Master），然后还应该有N台（Slave），那么我们的主机肯定要时时刻刻知道自己下面的从服务器的各方面情况，然后进行实时监控的功能，这个在分布式架构里面叫做心跳检测或者说心跳监控，最佳处理方案我还是觉得使用一些通信框架进行实现，我们的Netty就可以去做这样一件事。  
这里先介绍一个工具jar包sigar，这个可以用来查看一些服务器的信息  
使用方法： 网上下载sigar-bin-lib包，我github上有， 打开hyperic-sigar-1.6.4\sigar-bin\lib  将对应的配置文件复制到服务器的jdk的bin目录下， 例如我这里是64位的jdk程序，那么我复制sigar-amd64-winnt.dll 到我的java的bin目录下  
linux的我一般复制libsigar-amd64-linux.so到jdk bin目录下， 具体的百度搜索有很多。这里简要介绍如何使用  
API例子也在 SocketIO_03的bhz.utils包中


心跳例子： 代码在SocketIO_03 bhz.netty.heartBeat  
**步骤一**：  
1. Server端需要检测Client端的认证是否通过（为了确保安全）  
2. 如果通过，Client端定时发送自己所在电脑的信息，Server端接收信息  

**RequestInfo类： 用来存放机器的信息**  
```java
public class RequestInfo implements Serializable {

	private String ip ;
	private HashMap<String, Object> cpuPercMap ;
	private HashMap<String, Object> memoryMap;
	//.. other field
	
	public String getIp() {
		return ip;
	}
	public void setIp(String ip) {
		this.ip = ip;
	}
	public HashMap<String, Object> getCpuPercMap() {
		return cpuPercMap;
	}
	public void setCpuPercMap(HashMap<String, Object> cpuPercMap) {
		this.cpuPercMap = cpuPercMap;
	}
	public HashMap<String, Object> getMemoryMap() {
		return memoryMap;
	}
	public void setMemoryMap(HashMap<String, Object> memoryMap) {
		this.memoryMap = memoryMap;
	}
	
	
}
```


**Server端代码**  
```java
public class Server {

	public static void main(String[] args) throws Exception{
		
		EventLoopGroup pGroup = new NioEventLoopGroup();
		EventLoopGroup cGroup = new NioEventLoopGroup();
		
		ServerBootstrap b = new ServerBootstrap();
		b.group(pGroup, cGroup)
		 .channel(NioServerSocketChannel.class)
		 .option(ChannelOption.SO_BACKLOG, 1024)
		 //设置日志
		 .handler(new LoggingHandler(LogLevel.INFO))
		 .childHandler(new ChannelInitializer<SocketChannel>() {
			protected void initChannel(SocketChannel sc) throws Exception {
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingDecoder());
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingEncoder());
				sc.pipeline().addLast(new ServerHeartBeatHandler());
			}
		});
		
		ChannelFuture cf = b.bind(8765).sync();
		
		cf.channel().closeFuture().sync();
		pGroup.shutdownGracefully();
		cGroup.shutdownGracefully();
		
	}
}
```

**ServerHeartBeatHandler处理类**  
```java
public class ServerHeartBeatHandler extends ChannelHandlerAdapter {
    
	/** key:ip value:auth */
	private static HashMap<String, String> AUTH_IP_MAP = new HashMap<String, String>();
	private static final String SUCCESS_KEY = "auth_success_key";
	
	static {
		AUTH_IP_MAP.put("192.168.1.101", "1234");
	}
	
	private boolean auth(ChannelHandlerContext ctx, Object msg){
			//System.out.println(msg);
			String [] ret = ((String) msg).split(",");
			String auth = AUTH_IP_MAP.get(ret[0]);
			if(auth != null && auth.equals(ret[1])){
				ctx.writeAndFlush(SUCCESS_KEY);
				return true;
			} else {
				ctx.writeAndFlush("auth failure !").addListener(ChannelFutureListener.CLOSE);
				return false;
			}
	}
	
	@Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		if(msg instanceof String){
			auth(ctx, msg);
		} else if (msg instanceof RequestInfo) {
			
			RequestInfo info = (RequestInfo) msg;
			System.out.println("--------------------------------------------");
			System.out.println("当前主机ip为: " + info.getIp());
			System.out.println("当前主机cpu情况: ");
			HashMap<String, Object> cpu = info.getCpuPercMap();
			System.out.println("总使用率: " + cpu.get("combined"));
			System.out.println("用户使用率: " + cpu.get("user"));
			System.out.println("系统使用率: " + cpu.get("sys"));
			System.out.println("等待率: " + cpu.get("wait"));
			System.out.println("空闲率: " + cpu.get("idle"));
			
			System.out.println("当前主机memory情况: ");
			HashMap<String, Object> memory = info.getMemoryMap();
			System.out.println("内存总量: " + memory.get("total"));
			System.out.println("当前内存使用量: " + memory.get("used"));
			System.out.println("当前内存剩余量: " + memory.get("free"));
			System.out.println("--------------------------------------------");
			
			ctx.writeAndFlush("info received!");
		} else {
			ctx.writeAndFlush("connect failure!").addListener(ChannelFutureListener.CLOSE);
		}
    }


}

```  

**Client端代码**  
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
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingDecoder());
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingEncoder());
				sc.pipeline().addLast(new ClienHeartBeattHandler());
			}
		});
		
		ChannelFuture cf = b.connect("127.0.0.1", 8765).sync();

		cf.channel().closeFuture().sync();
		group.shutdownGracefully();
	}
}

```
**ClienHeartBeattHandler处理类**
```java
public class ClienHeartBeattHandler extends ChannelHandlerAdapter {

    private ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
    
    private ScheduledFuture<?> heartBeat;
	//主动向服务器发送认证信息
    private InetAddress addr ;
    
    private static final String SUCCESS_KEY = "auth_success_key";

	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		addr = InetAddress.getLocalHost();
        String ip = addr.getHostAddress();
		String key = "1234";
		//证书
		String auth = ip + "," + key;
		ctx.writeAndFlush(auth);
	}
	
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    	try {
        	if(msg instanceof String){
        		String ret = (String)msg;
        		if(SUCCESS_KEY.equals(ret)){
        	    	// 握手成功，主动发送心跳消息
        	    	this.heartBeat = this.scheduler.scheduleWithFixedDelay(new HeartBeatTask(ctx), 0, 2, TimeUnit.SECONDS);
        		    System.out.println(msg);    			
        		}
        		else {
        			System.out.println(msg);
        		}
        	}
		} finally {
			ReferenceCountUtil.release(msg);
		}
    }

    private class HeartBeatTask implements Runnable {
    	private final ChannelHandlerContext ctx;

		public HeartBeatTask(final ChannelHandlerContext ctx) {
		    this.ctx = ctx;
		}
	
		@Override
		public void run() {
			try {
				System.out.println("ff");
				RequestInfo info = new RequestInfo();
			    //ip
			    info.setIp(addr.getHostAddress());
		        Sigar sigar = new Sigar();
		        //cpu prec
		        CpuPerc cpuPerc = sigar.getCpuPerc();
		        HashMap<String, Object> cpuPercMap = new HashMap<String, Object>();
		        cpuPercMap.put("combined", cpuPerc.getCombined());
		        cpuPercMap.put("user", cpuPerc.getUser());
		        cpuPercMap.put("sys", cpuPerc.getSys());
		        cpuPercMap.put("wait", cpuPerc.getWait());
		        cpuPercMap.put("idle", cpuPerc.getIdle());
		        // memory
		        Mem mem = sigar.getMem();
				HashMap<String, Object> memoryMap = new HashMap<String, Object>();
				memoryMap.put("total", mem.getTotal() / 1024L);
				memoryMap.put("used", mem.getUsed() / 1024L);
				memoryMap.put("free", mem.getFree() / 1024L);
				info.setCpuPercMap(cpuPercMap);
			    info.setMemoryMap(memoryMap);
			    ctx.writeAndFlush(info);
				System.out.println("gg");
			} catch (Exception e) {
				e.printStackTrace();
			}
		}

	    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
	    	cause.printStackTrace();
			if (heartBeat != null) {
			    heartBeat.cancel(true);
			    heartBeat = null;
			}
			ctx.fireExceptionCaught(cause);
	    }
	    
	}
}
```

**MarshallingCodeCFactory工具类**  
用来解编码的，不需要看懂，能用就行了  
```java
/**
 * Marshalling工厂
 * @author（alienware）
 * @since 2014-12-16
 */
public final class MarshallingCodeCFactory {

    /**
     * 创建Jboss Marshalling解码器MarshallingDecoder
     * @return MarshallingDecoder
     */
    public static MarshallingDecoder buildMarshallingDecoder() {
    	//首先通过Marshalling工具类的精通方法获取Marshalling实例对象 参数serial标识创建的是java序列化工厂对象。
		final MarshallerFactory marshallerFactory = Marshalling.getProvidedMarshallerFactory("serial");
		//创建了MarshallingConfiguration对象，配置了版本号为5 
		final MarshallingConfiguration configuration = new MarshallingConfiguration();
		configuration.setVersion(5);
		//根据marshallerFactory和configuration创建provider
		UnmarshallerProvider provider = new DefaultUnmarshallerProvider(marshallerFactory, configuration);
		//构建Netty的MarshallingDecoder对象，俩个参数分别为provider和单个消息序列化后的最大长度
		MarshallingDecoder decoder = new MarshallingDecoder(provider, 1024 * 1024 * 1);
		return decoder;
    }

    /**
     * 创建Jboss Marshalling编码器MarshallingEncoder
     * @return MarshallingEncoder
     */
    public static MarshallingEncoder buildMarshallingEncoder() {
		final MarshallerFactory marshallerFactory = Marshalling.getProvidedMarshallerFactory("serial");
		final MarshallingConfiguration configuration = new MarshallingConfiguration();
		configuration.setVersion(5);
		MarshallerProvider provider = new DefaultMarshallerProvider(marshallerFactory, configuration);
		//构建Netty的MarshallingEncoder对象，MarshallingEncoder用于实现序列化接口的POJO对象序列化为二进制数组
		MarshallingEncoder encoder = new MarshallingEncoder(provider);
		return encoder;
    }
}

```

注意Sigar的配置文件放在jdk bin目录下，当运行Server端，再启动Client端的时候，如果认证失败，则服务器主动断开与client的连接，如果认证通过，那么Client就每2秒钟发送一次心跳检测给Server端.  



特别感谢互联网架构师白鹤翔老师，本文大多出自他的视频讲解。
笔者主要是记录笔记，以便之后翻阅，正所谓好记性不如烂笔头，烂笔头不如云笔记  

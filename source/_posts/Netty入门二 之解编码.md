---
title: Netty入门二 之解编码
date: 2017-08-03 21:25:21
tags: [netty]
categories: [架构]
author: kaishun
id: 83
permalink: netty2
---

关键字：Netty解编码，JBoss Marshalling，  

代码在 https://github.com/zhaikaishun/NettyTutorial 在SocketIO_02  kaishun.netty.serial下 


<!-- more -->

## Netty解编码技术  
解编码技术，说白了就是java序列化技术，序列化的目的就两个，第一进行网络传输，第二对象持久化  
虽然我们可以使用java进行对象序列化，netty去传输，但是java序列化的硬伤太多，比如java序列化没法跨语言、序列化够码流太大、序列化性能太低等等...   
主流的编码解码框架：  
JBoss 的 Marshalling包    
google的Protobuf    
给予Protobuf的Kyro    
MessagePack框架    

## JBoss Marshalling  
Jboss Marshalling是一个java对象序列化包，对jdk默认的序列化框架进行了优化，但又保持跟java.io.Serializable接口的兼容，同时增加了一些可调的参数和附加特性，  
类库 jboss-marshalling-1.3.0、jboss-marshalling-serial-1.3.0   
Jboss Marshalling与Netty结合后进行序列化对象的代码编写非常简单，我们一起来看一下   

首先有个MarshallingCodeCFactory类  
这个类不需要看懂，就要有一个这个类就好了  
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

Server端其他基本不变，就这里  
```java
		 .childHandler(new ChannelInitializer<SocketChannel>() {
			protected void initChannel(SocketChannel sc) throws Exception {
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingDecoder());
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingEncoder());
				sc.pipeline().addLast(new ServerHandler());
			}
		});
```
Client端也是这里有变化  
```java
		 .handler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingDecoder());
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingEncoder());
				sc.pipeline().addLast(new ClientHandler());
			}
		});

		for(int i = 0; i < 5; i++ ){
			Req req = new Req();
			req.setId("" + i);
			req.setName("pro" + i);
			req.setRequestMessage("数据信息" + i);	
//			String path = System.getProperty("user.dir") + File.separatorChar + "sources" +  File.separatorChar + "001.jpg";
//			File file = new File(path);
//	        FileInputStream in = new FileInputStream(file);
//	        byte[] data = new byte[in.available()];
//	        in.read(data);
//	        in.close();
//			req.setAttachment(GzipUtils.gzip(data));
			cf.channel().writeAndFlush(req);
		}
```

Req类  
```
public class Req implements Serializable{

	private static final long  SerialVersionUID = 1L;
	
	private String id ;
	private String name ;
	private String requestMessage ;
	private byte[] attachment;
	
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getRequestMessage() {
		return requestMessage;
	}
	public void setRequestMessage(String requestMessage) {
		this.requestMessage = requestMessage;
	}
	public byte[] getAttachment() {
		return attachment;
	}
	public void setAttachment(byte[] attachment) {
		this.attachment = attachment;
	}
}
```
Resp类  
```java
public class Resp implements Serializable{
	
	private static final long serialVersionUID = 1L;
	
	private String id;
	private String name;
	private String responseMessage;
	
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getResponseMessage() {
		return responseMessage;
	}
	public void setResponseMessage(String responseMessage) {
		this.responseMessage = responseMessage;
	}
	

}
```

ClientHandler  
```java
	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		try {
			Resp resp = (Resp)msg;
			System.out.println("Client : " + resp.getId() + ", " + resp.getName() + ", " + resp.getResponseMessage());			
		} finally {
			ReferenceCountUtil.release(msg);
		}
	}
```
ServerHandler  
```java
	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		Req req = (Req)msg;
		System.out.println("Server : " + req.getId() + ", " + req.getName() + ", " + req.getRequestMessage());
//		byte[] attachment = GzipUtils.ungzip(req.getAttachment());
//
//		String path = System.getProperty("user.dir") + File.separatorChar + "receive" +  File.separatorChar + "001.jpg";
//        FileOutputStream fos = new FileOutputStream(path);
//        fos.write(attachment);
//        fos.close();
		
		Resp resp = new Resp();
		resp.setId(req.getId());
		resp.setName("resp" + req.getId());
		resp.setResponseMessage("响应内容" + req.getId());
		ctx.writeAndFlush(resp);//.addListener(ChannelFutureListener.CLOSE);
	}

```

总结上面的代码：  
其实只需要在Server和Client端加上这两个，其他方法还是一样的。  
```
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingDecoder());
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingEncoder());
```  

补充： 如果还想进行大文件（例如图片视频等传输），将上面的注释的代码打开即可，这块代码还使用了Gzip进行压缩，压缩之后再进行传输，这是需要注意的。




特别感谢互联网架构师白鹤翔老师，本文大多出自他的视频讲解。
笔者主要是记录笔记，以便之后翻阅，正所谓好记性不如烂笔头，烂笔头不如云笔记






































---
title: jar读取资源配置文件，jar包内包外，以及包内读取目录的方法
date: 2017-07-06 21:25:21
tags: [java]
categories: [programme]
author: kaishun
id: 77
permalink: java-read-jarfile
---

java程序打成jar包后，经常碰到一些资源文件找不到等问题，最近总结了一下之前用到的几种获取路径、资源文件的方法

<!-- more -->
## 测试代码
代码如下，并且打成jar包
```java
package cn.zks.pathtest;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;

public class TestPath {

	public static void main(String[] args) {
		TestPath testPath = new TestPath();
		testPath.testPath();

	}
	
	
	public  void testPath(){
		try {
			
			String userPath = System.getProperty("user.dir");
			String path = TestPath.class.getProtectionDomain().getCodeSource().getLocation().getPath();
			String pathGetClass =this.getClass().getProtectionDomain().getCodeSource().getLocation().getPath();
			String classLoaderPath = getClass().getClassLoader().toString();	
			
			System.out.println("userPath: "+userPath);
			System.out.println("path: "+path);
			System.out.println("pathGetClass: "+pathGetClass);
			System.out.println("classLoaderPath: "+classLoaderPath);
			
			//看看getClassLoader是读取jar包里面还是读取jar包外面
			InputStream inputStream = getClass().getClassLoader().getResourceAsStream("text.txt");
			BufferedReader br = new BufferedReader(new InputStreamReader(inputStream)); 		
			String line="";
			while((line=br.readLine())!=null){
				System.out.println("getClassLoader: "+line);
			}
			
			//看看pathGetClass读取的是jar包里面，还是jar包外面
			File file = new File(pathGetClass+"text.txt");
			BufferedReader reader = new BufferedReader(new FileReader(file));
			String tempString ="";
			while ((tempString = reader.readLine()) != null){
				System.out.println("pathGetClass: "+tempString);
			}
			
			
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
	
}

```

## 测试1

第一次，jar包里面不放text.txt, 只在jar包外面放text.txt,外面的text的内容是:it is jar out ，在jar包当前目录运行jar，运行后
```
G:\testpath>java -jar testpath.jar
userPath: G:\testpath
path: /G:/testpath/testpath.jar
pathGetClass: /G:/testpath/testpath.jar
classLoaderPath: java.net.URLClassLoader@14ae5a5
getClassLoader: it is jar out
pathGetClass: it is jar out

```
userPath是当前执行命令的目录，

## 测试2
第二次：jar包里面放text.txt,里面的text.txt内容是:it is in jar ，外面也放了text.txt, 内容是it is jar out：, 在jar包当前目录运行jar，运行后
```
G:\testpath>java -jar testpath.jar
userPath: G:\testpath
path: /G:/testpath/testpath.jar
pathGetClass: /G:/testpath/testpath.jar
classLoaderPath: java.net.URLClassLoader@14ae5a5
getClassLoader: it is in jar
pathGetClass: it is jar out
```
## 测试3
第三次：在Administrator目录运行指定目录的jar包,Administrator目录下没有text.txt 
```
C:\Users\Administrator>java -jar G:\testpath\testpath.jar
userPath: C:\Users\Administrator
path: /G:/testpath/testpath.jar
pathGetClass: /G:/testpath/testpath.jar
classLoaderPath: java.net.URLClassLoader@14ae5a5
getClassLoader: it is in jar
java.io.FileNotFoundException: .\text.txt (系统找不到指定的文件。)
        at java.io.FileInputStream.open0(Native Method)
        at java.io.FileInputStream.open(Unknown Source)
        at java.io.FileInputStream.<init>(Unknown Source)
        at java.io.FileReader.<init>(Unknown Source)
        at cn.zks.pathtest.TestPath.testPath(TestPath.java:44)
        at cn.zks.pathtest.TestPath.main(TestPath.java:17)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
        at java.lang.reflect.Method.invoke(Unknown Source)
        at org.eclipse.jdt.internal.jarinjarloader.JarRsrcLoader.main(JarRsrcLoader.java:58)
``` 

## 总结： 
1. getClass().getClassLoader().getResourceAsStream(path), 先是读取jar包内部有没有这个path，如果内部没有时，再读取jar包当前目录下的path, 值得注意的是这个需要有一个实例化类，才能getClass，否则是编译不过的
2. this.getClass().getProtectionDomain().getCodeSource().getLocation().getPath(); 得到的是程序运行所在路径
3. System.getProperty("user.dir"); 和 this.getClass().getProtectionDomain().getCodeSource().getLocation().getPath() 是不一样的， System.getProperty("user.dir")得到的是用户执行的时候的那个路径，不一定是jar包所在路径  
  
总之：我还是尽量选择第一个,getClass().getClassLoader().getResourceAsStream(path)，但是要小心第一种的配置，是优先读取jar包内部的路径，jar包内部读取不到时才读取当前jar包位置的路径

对于System.getProperty()更多用法，参考：http://blog.csdn.net/kongqz/article/details/3987198  


## 读取jar包内某个目录所有文件的方法
读取jar包内某个目录的所有文件，还是比较麻烦的，getClassLoader().getResourceAsStream也只能得到某一个文件，并不能得到文件夹，网上的大多数方法都不太可取，最后，自己想了一个取巧的办法来获取，方法如下  
    
1. 得到jar包所在路径  
2. JarFile打开这这个jar包
3. JarFile.entries();
4. 就是过滤到自己的目录
5. 使用getClassLoader().getResourceAsStream来分别获取每一个文件

代码示例:    
jar包内有个conf文件夹，里面有很多个文件，我想要输出里面的内容  
```
        	String path = TestPath.class.getProtectionDomain().getCodeSource().getLocation().getPath();
        	System.out.println("path: "+path); 
        	JarFile localJarFile = new JarFile(new File(path));
	
        	Enumeration<JarEntry> entries = localJarFile.entries();
	        while (entries.hasMoreElements()) {
	            JarEntry jarEntry = entries.nextElement();
	            System.out.println(jarEntry.getName());
	            String innerPath = jarEntry.getName();
	            if(innerPath.startsWith("conf")){
	            	InputStream inputStream = TestPath.class.getClassLoader().getResourceAsStream(innerPath);
		            BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));         
		            String line="";
		            while((line=br.readLine())!=null){
		                  System.out.println(innerPath+"内容为:"+line);
		            }
	            }
	        }
```


**17年9月6号额外补充**  
## System.getProperty方法
```
public class TestSystemProperty {
	
	public static void main(String[] args) {
	
		System.out.println("java版本号：" + System.getProperty("java.version")); // java版本号
		System.out.println("Java提供商名称：" + System.getProperty("java.vendor")); // Java提供商名称
		System.out.println("Java提供商网站：" + System.getProperty("java.vendor.url")); // Java提供商网站
		System.out.println("jre目录：" + System.getProperty("java.home")); // Java，哦，应该是jre目录
		System.out.println("Java虚拟机规范版本号：" + System.getProperty("java.vm.specification.version")); // Java虚拟机规范版本号
		System.out.println("Java虚拟机规范提供商：" + System.getProperty("java.vm.specification.vendor")); // Java虚拟机规范提供商
		System.out.println("Java虚拟机规范名称：" + System.getProperty("java.vm.specification.name")); // Java虚拟机规范名称
		System.out.println("Java虚拟机版本号：" + System.getProperty("java.vm.version")); // Java虚拟机版本号
		System.out.println("Java虚拟机提供商：" + System.getProperty("java.vm.vendor")); // Java虚拟机提供商
		System.out.println("Java虚拟机名称：" + System.getProperty("java.vm.name")); // Java虚拟机名称
		System.out.println("Java规范版本号：" + System.getProperty("java.specification.version")); // Java规范版本号
		System.out.println("Java规范提供商：" + System.getProperty("java.specification.vendor")); // Java规范提供商
		System.out.println("Java规范名称：" + System.getProperty("java.specification.name")); // Java规范名称
		System.out.println("Java类版本号：" + System.getProperty("java.class.version")); // Java类版本号
		System.out.println("Java类路径：" + System.getProperty("java.class.path")); // Java类路径
		System.out.println("Java lib路径：" + System.getProperty("java.library.path")); // Java lib路径
		System.out.println("Java输入输出临时路径：" + System.getProperty("java.io.tmpdir")); // Java输入输出临时路径
		System.out.println("Java编译器：" + System.getProperty("java.compiler")); // Java编译器
		System.out.println("Java执行路径：" + System.getProperty("java.ext.dirs")); // Java执行路径
		System.out.println("操作系统名称：" + System.getProperty("os.name")); // 操作系统名称
		System.out.println("操作系统的架构：" + System.getProperty("os.arch")); // 操作系统的架构
		System.out.println("操作系统版本号：" + System.getProperty("os.version")); // 操作系统版本号
		System.out.println("文件分隔符：" + System.getProperty("file.separator")); // 文件分隔符
		System.out.println("路径分隔符：" + System.getProperty("path.separator")); // 路径分隔符
		System.out.println("直线分隔符：" + System.getProperty("line.separator")); // 直线分隔符
		System.out.println("操作系统用户名：" + System.getProperty("user.name")); // 用户名
		System.out.println("操作系统用户的主目录：" + System.getProperty("user.home")); // 用户的主目录
		System.out.println("当前程序所在目录：" + System.getProperty("user.dir")); // 当前程序所在目录
	}
}
```

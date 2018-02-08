---
title: 【多线程高并发】ThreadLocal，高并发下的单例模式
date: 2017-08-27 21:25:21
tags: [java]
categories: [架构]
author: kaishun
id: 82
permalink: thread7
---

## 2.3 ThreadLocal  
ThreadLocal概念: 线程局部变量，是一种多线程间并发访问变量的解决方案。与其synchronized等加锁方式不同，THreadLocal完全不提供锁，而使用空间换时间的手段，为每个线程提供变量的独立副本，以保障线程安全。  
在高并发量或者竞争激烈的场景，使用ThreadLocal可以在一定程度上减少锁竞争。  
当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。  
案例： 
每个线程一个独立的变量副本，所以t2线程执行为null  
<!-- more -->
```java
public class ConnThreadLocal {

	public static ThreadLocal<String> th = new ThreadLocal<String>();
	
	public void setTh(String value){
		th.set(value);
	}
	public void getTh(){
		System.out.println(Thread.currentThread().getName() + ":" + this.th.get());
	}
	
	public static void main(String[] args) throws InterruptedException {
		
		final ConnThreadLocal ct = new ConnThreadLocal();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				ct.setTh("张三");
				ct.getTh();
			}
		}, "t1");
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					Thread.sleep(1000);
					ct.getTh();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}, "t2");
		
		t1.start();
		t2.start();
	}
	
}
----------输出------------
t1:张三
t2:null
```  

## 2.4 高并发下的单例模式  

**懒汉模式**  需要两层的if判断  
```java
public class DubbleSingleton {

	private static DubbleSingleton ds;
	
	public  static DubbleSingleton getDs(){
		if(ds == null){
			try {
				//模拟初始化对象的准备时间...
				Thread.sleep(3000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			synchronized (DubbleSingleton.class) {
				if(ds == null){
					ds = new DubbleSingleton();
				}
			}
		}
		return ds;
	}
	
	public static void main(String[] args) {
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println(DubbleSingleton.getDs().hashCode());
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println(DubbleSingleton.getDs().hashCode());
			}
		},"t2");
		Thread t3 = new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println(DubbleSingleton.getDs().hashCode());
			}
		},"t3");
		
		t1.start();
		t2.start();
		t3.start();
	}
}
------输出的hashCode码是一致的----------
21936835
21936835
21936835
```  

**饿汉模式**  
```
public class Singletion {
	
	private static class InnerSingletion {
		private static Singletion single = new Singletion();
	}
	
	public static Singletion getInstance(){
		return InnerSingletion.single;
	}
	
}
```
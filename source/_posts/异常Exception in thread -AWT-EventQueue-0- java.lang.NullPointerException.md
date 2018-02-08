---
title: 异常Exception in thread -AWT-EventQueue-0- java.lang.NullPointerException
date: 2017-05-04 21:25:21
tags: [java,Exception]
categories: [programme]
author: kaishun
id: 55
permalink: java-awt-eventqueue-exception
blogexcerpt: 更新UI等操作时异常，类似于Android中的更新ui一样，ui线程相当于主线程，再对此进行直接更新，两个线程事物会冲突报这个异常，在Android中使用的是handle的方法，同理,类似于，在java 界面编程上，我们把其他的线程放在一个队列当中，我们称这个队列为消息队列，可以使用如下方法，把更新UI的操作，放在下面的run方法中去。 这样，就实现了消息队列的方式。最后由主线程有序的处理这些消息事件 
---

Exception in thread "AWT-EventQueue-0" java.lang.NullPointerException  
更新UI等操作时异常  
**个人理解原因**：  
类似于Android中的更新ui一样，ui线程相当于主线程，再对此进行直接更新，两个线程事物会冲突报这个异常，在Android中使用的是handle的方法，同理,类似于，在java 界面编程上，我们把其他的线程放在一个队列当中，我们称这个队列为消息队列，可以使用如下方法，把更新UI的操作，放在下面的run方法中去。 这样，就实现了消息队列的方式。 最后由主线程有序的处理这些消息事件
```
SwingUtilities.invokeLater(new   Runnable(){ 
			public   void   run() { 
			//放入方法
			} 
 });
```

例子如下
```
SwingUtilities.invokeLater(new   Runnable(){ 
		public   void   run() { 
		tree.updateUI(); 
		} 
 });
```
```
SwingUtilities.invokeLater(new   Runnable() { 
		public   void   run() { 
			SwingUtilities.updateComponentTreeUI(jf); 
		} 
			});
```
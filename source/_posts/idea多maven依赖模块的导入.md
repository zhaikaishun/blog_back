---
title: idea多maven依赖模块的导入
date: 2018-02-21 21:25:21
tags: [工具]
categories: [工具]
author: kaishun
id: 144
permalink: idea-multi-import
---
---
idea多maven依赖模块的导入

先任意导入一个项目  
File -> New ->module from exitited source ->选择maven  

![idea_mult1](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/idea_multimodel/idea1.png)    
下一步  
勾选Search for projects recurisivery  

![idea_mult2](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/idea_multimodel/idea2.png)   

**一定要把不要的去掉, 其实如果项目是干净的，没过多导入项目，不会有问题**  

![idea_mult3](https://raw.githubusercontent.com/zhaikaishun/blog_img/master/blog/idea_multimodel/idea1.png)    

然后一路next就行了。  

打包：  
右侧MavenProject, 调不出就百度  
选中module，在Lifecyle中直接顺序 clean，install即可



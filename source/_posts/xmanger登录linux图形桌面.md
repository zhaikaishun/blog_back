---
title: SecureCRT8.0安装与破解
date: 2017-09-25 21:25:21
tags: [linux]
categories: [linux]
author: kaishun
id: 111
permalink: xmange-remote-desktop
---

最近经常在本地调试的东西，放在服务器上就运行不了，有时候本地的机器远远没有服务器上的机器强大，这时候想的就是在服务器上装上各种环境，例如eclipse，idea或者其他的环境。这时候，就需要自己能直接使用桌面来进行调试了。 想起之前都是使用ubuntu进行开发，centos自带的桌面系统也是可以支持这些idea的呀，可是我们不可能跑到机房去使用远程桌面系统，这时候，Xmanger就派上用场了
<!-- more -->




## 本地下载xmanger
xmanger 下载地址: 链接：http://pan.baidu.com/s/1dFxrXKl 密码：6ir5

## 安装  
按照提示即可

## 远程连接  
我本地windows的IP是192.168.1.103 我需要连接服务器centos的桌面，IP地址是192.168.1.31  
 
进入到安装目录运行Xstart.exe     
  
  
![找到xstart](http://or49tneld.bkt.clouddn.com/17-11-15/70586506.jpg)

按照如图进行填写，其中注意填写   
  
  
/usr/bin/gnome-session --display $DISPLAY  
![linux2](http://or49tneld.bkt.clouddn.com/17-11-15/91801141.jpg)

点击运行,然后输入密码    
![linux3](http://or49tneld.bkt.clouddn.com/17-11-15/50608273.jpg)

出现x11已经打开，完成
![linux4](http://or49tneld.bkt.clouddn.com/17-11-15/89695878.jpg)  
![linux5](http://or49tneld.bkt.clouddn.com/17-11-15/66575772.jpg)

如果想关闭图形界面，右击右下角X图标-->关闭 

如果这样不行，可能linux是纯净版的，纯净版的个人并没有试过，可以参考网上的内容,
我这linux6.5自带的是可以这样使用的。  


## 如果使用root登录可能碰到错误  
```
You are currently trying to run as the root super user.  The super user is a specialized account that is not designed to run a normal user session.  Various programs will not function properly, and actions performed under this account can cause unrecoverable damage to the operating system.
```
个人不建议用root用户登录，可以自己创建用户登录，创建办法如下


**添加用户**  
```
useradd  kaishun
passwd kaishun  

```

**给kaishun用户赋予root权限**  
切换到root用户 赋予etc/sudoers777权限，然后打开
```
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
kaishun    ALL=(ALL)       ALL

## Allows people in group wheel to run all commands
 %wheel ALL=(ALL)       ALL
```
把sudoers的权限改回来成440
```
chmod 440 /etc/sudoers
```

后面就可以登陆了
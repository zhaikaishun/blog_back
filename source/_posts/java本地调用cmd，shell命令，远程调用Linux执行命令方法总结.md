---
title: java本地调用cmd，shell命令，远程调用Linux执行命令方法总结
date: 2017-07-20 21:25:21
tags: [java]
categories: [programme]
author: kaishun
id: 80
permalink: java-rmi
blogexcerpt: 有时候经常会碰到需要远程调用Linux或者本地调用Linux或者本地调用cmd的一些命令，最近小结了一下这几种用法,本地调用cmd命令,远程调用Linux执行命令,jar包下载 http://central.maven.org/maven2/com/jcraft/jsch/0.1.54/jsch-0.1.54.jar 参考代码如下
---

有时候经常会碰到需要远程调用Linux或者本地调用Linux或者本地调用cmd的一些命令，最近小结了一下这几种用法
<!-- more -->
## 本地调用cmd命令
```java
    @Test
    public void testCmd()throws Exception{
        String cmd="cmd /c date"; //命令的前面必须要有cmd /c
        execCmd(cmd);
    }

    public static void execCmd(String cmd){
        try{
            Runtime rt = Runtime.getRuntime();
            //执行命令, 最后一个参数，可以使用new File("path")指定运行的命令的位置
            Process proc = rt.exec(cmd,null,null);  
            InputStream stderr =  proc.getInputStream();
            InputStreamReader isr = new InputStreamReader(stderr,"GBK");
            BufferedReader br = new BufferedReader(isr);
            String line="";
            while ((line = br.readLine()) != null) { // 打印出命令执行的结果
                System.out.println(line);
            }
        }catch (Exception e){
            e.printStackTrace();
        }

    }
```

## 本地调用Linux命令
```
    @Test
    public void testCmd()throws Exception{
        String cmd="/bin/sh -c date"; //命令的前面必须要有/bin/sh -c
        execCmd(cmd);
        
    }

    public static void execCmd(String cmd){
        try{
            Runtime rt = Runtime.getRuntime();
            //执行命令, 最后一个参数，可以使用new File("path")指定运行的命令的位置
            Process proc = rt.exec(cmd,null,null);  
            InputStream stderr =  proc.getInputStream();
            InputStreamReader isr = new InputStreamReader(stderr,"GBK");
            BufferedReader br = new BufferedReader(isr);
            String line="";
            while ((line = br.readLine()) != null) { // 打印出命令执行的结果
                System.out.println(line);
            }
        }catch (Exception e){
            e.printStackTrace();
        }

    }
```

## 远程调用Linux执行命令
jar包下载
```
http://central.maven.org/maven2/com/jcraft/jsch/0.1.54/jsch-0.1.54.jar
```
或者maven引用
```
<dependency>
    <groupId>com.jcraft</groupId>
    <artifactId>jsch</artifactId>
    <version>0.1.54</version>
</dependency>
```

参考代码如下
```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import com.jcraft.jsch.ChannelExec;
import com.jcraft.jsch.JSch;
import com.jcraft.jsch.JSchException;
import com.jcraft.jsch.Session;

public class SSHHelper {
    /**
     * 远程 执行命令并返回结果调用过程 是同步的（执行完才会返回）
     * @param host	主机名
     * @param user	用户名
     * @param psw	密码
     * @param port	端口
     * @param command	命令
     * @return
     */
    public static String exec(String host,String user,String psw,int port,String command){
        StringBuffer sb= new StringBuffer();
        Session session =null;
        ChannelExec openChannel =null;
        try {
            JSch jsch=new JSch();
            session = jsch.getSession(user, host, port);
            java.util.Properties config = new java.util.Properties();
            config.put("StrictHostKeyChecking", "no");//跳过公钥的询问
            session.setConfig(config);
            session.setPassword(psw);
            session.connect(5000);//设置连接的超时时间
            openChannel = (ChannelExec) session.openChannel("exec");
            openChannel.setCommand(command); //执行命令
            int exitStatus = openChannel.getExitStatus(); //退出状态为-1，直到通道关闭
            System.out.println(exitStatus);

            // 下面是得到输出的内容
            openChannel.connect();
            InputStream in = openChannel.getInputStream();
            BufferedReader reader = new BufferedReader(new InputStreamReader(in));
            String buf = null;
            while ((buf = reader.readLine()) != null) {
                sb.append(buf+"\n");
            }
        } catch (JSchException | IOException e) {
            sb.append(e.getMessage()+"\n");
        }finally{
            if(openChannel!=null&&!openChannel.isClosed()){
                openChannel.disconnect();
            }
            if(session!=null&&session.isConnected()){
                session.disconnect();
            }
        }
        return sb.toString();
    }


    public static void main(String args[]){
        String exec = exec("192.168.1.xx", "user", "name", 22, "ls");
        System.out.println(exec);
    }
}

```

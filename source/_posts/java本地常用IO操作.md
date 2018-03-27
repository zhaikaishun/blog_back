---
title: java本地常用IO操作
date: 2017-07-10 21:25:21
tags: [java,io]
categories: [programme]
author: kaishun
id: 79
permalink: xml-io-util
blogexcerpt: BufferWriter,有乱码TODO
---

## BufferWriter 
<!-- more -->
```
package cn.mastercom.filterSpark;

import java.io.*;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * Created by Administrator on 2017/3/18.
 */
public class GenerateConfXml {
    public static void main(String[] args)  {
        FileInputStream fis = null;
        InputStreamReader isr = null;
        BufferedReader br = null ;//ÓÃÓÚ°ü×°InputStreamReader,Ìá¸ß´¦ÀíÐÔÄÜ¡£ÒòÎªBufferedReaderÓÐ»º³åµÄ£¬¶øInputStreamReaderÃ»ÓÐ¡£
        String str ="";
        String str1="";

        try {
            fis = new FileInputStream("conf_test/TB_MODEL_SIGNAL_SAMPLE.sql");
            isr = new InputStreamReader(fis);
            br = new BufferedReader(isr);// ´Ó×Ö·ûÊäÈëÁ÷ÖÐ¶ÁÈ¡ÎÄ¼þÖÐµÄÄÚÈÝ,·â×°ÁËÒ»¸önew InputStreamReaderµÄ¶ÔÏó
            while ((str = br.readLine())!=null){
                str1+=str.trim()+"qqqq"+"\n";
            }
            System.out.println(str1);

        } catch (Exception e) {
            e.printStackTrace();
        }

        try {
            br.close();
            isr.close();
            fis.close();
            // ¹Ø±ÕµÄÊ±ºò×îºÃ°´ÕÕÏÈºóË³Ðò¹Ø±Õ×îºó¿ªµÄÏÈ¹Ø±ÕËùÒÔÏÈ¹Øs,ÔÙ¹Øn,×îºó¹Øm
        } catch (IOException e) {
            e.printStackTrace();
        }



        File file = new File("conf_test/TB_MODEL_SIGNAL_SAMPLE.xml");
        if(!file.exists()){
            try {
                file.createNewFile();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        FileWriter fw = null;
        try {
        //ÕâÀïÈç¹ûÊÇ   FileWriter fw = new FileWriter(file.getAbsoluteFile(),true);, ÄÇÃ´ÊÇÔÚºóÃæÌí¼Ó
            fw = new FileWriter(file.getAbsoluteFile());
            BufferedWriter bw = new BufferedWriter(fw);
            bw.write("<?xml version=\"1.0\" encoding=\"UTF-8\"?>  \n");
            bw.write("<table>  \n");
            bw.write(xml);
            bw.write("</table>");
            bw.close();
        }catch (IOException e) {
            e.printStackTrace();
        }


    }

    private static String attrTransform(String fieldOrAttr) {
        if("[varchar]".equals(fieldOrAttr)){
            fieldOrAttr = "string";
        }else if("[bigint]".equals(fieldOrAttr)){
            fieldOrAttr = "long";
        }else if("[smallint]".equals(fieldOrAttr)||"[tinyint]".equals(fieldOrAttr)){
            fieldOrAttr = "int";
        }
        return fieldOrAttr;
    }
}
```

,FileOutStrem+StringBuffer
```java
public class TestFileIo {
    public static void main(String[] args) {
        File file=new File("G:\\sqldata2\\result\\out1358\\test.txt");
        if(!file.exists())
            try {
                file.createNewFile();
            } catch (IOException e) {
                e.printStackTrace();
            }
        FileOutputStream out= null;
        try {
            out = new FileOutputStream(file,true);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        StringBuffer sb=new StringBuffer();
        sb.append("ÕâÊÇµÚ"+1+"ÐÐ:Ç°Ãæ½éÉÜµÄ¸÷ÖÖ·½·¨¶¼²»¹ØÓÃ,ÎªÊ²Ã´×ÜÊÇÆæ¹ÖµÄÎÊÌâ "+"\n");
        try {
            out.write(sb.toString().getBytes("utf-8"));
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            out.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    }
```


```
/**
     * 
     */
    public static void readFileByLines(String fileName) {
        File file = new File(fileName);
        BufferedReader reader = null;
        try {
            System.out.println("ÒÔÐÐÎªµ¥Î»¶ÁÈ¡ÎÄ¼þÄÚÈÝ£¬Ò»´Î¶ÁÒ»ÕûÐÐ£º");
            reader = new BufferedReader(new FileReader(file));
            String tempString = null;
            int line = 1;
            // Ò»´Î¶ÁÈëÒ»ÐÐ£¬Ö±µ½¶ÁÈënullÎªÎÄ¼þ½áÊø
            while ((tempString = reader.readLine()) != null) {
                // ÏÔÊ¾ÐÐºÅ
                System.out.println("line " + line + ": " + tempString);
                line++;
            }
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e1) {
                }
            }
        }
    }
```
```


/**
     * 
     */
    public static void readFileByBytes(String fileName) {
        File file = new File(fileName);
        InputStream in = null;
        try {
            System.out.println("ÒÔ×Ö½ÚÎªµ¥Î»¶ÁÈ¡ÎÄ¼þÄÚÈÝ£¬Ò»´Î¶ÁÒ»¸ö×Ö½Ú£º");
            // Ò»´Î¶ÁÒ»¸ö×Ö½Ú
            in = new FileInputStream(file);
            int tempbyte;
            while ((tempbyte = in.read()) != -1) {
                System.out.write(tempbyte);
            }
            in.close();
        } catch (IOException e) {
            e.printStackTrace();
            return;
        }
        try {
            System.out.println("ÒÔ×Ö½ÚÎªµ¥Î»¶ÁÈ¡ÎÄ¼þÄÚÈÝ£¬Ò»´Î¶Á¶à¸ö×Ö½Ú£º");
            // Ò»´Î¶Á¶à¸ö×Ö½Ú
            byte[] tempbytes = new byte[100];
            int byteread = 0;
            in = new FileInputStream(fileName);
            ReadFromFile.showAvailableBytes(in);
            // ¶ÁÈë¶à¸ö×Ö½Úµ½×Ö½ÚÊý×éÖÐ£¬bytereadÎªÒ»´Î¶ÁÈëµÄ×Ö½ÚÊý
            while ((byteread = in.read(tempbytes)) != -1) {
                System.out.write(tempbytes, 0, byteread);
            }
        } catch (Exception e1) {
            e1.printStackTrace();
        } finally {
            if (in != null) {
                try {
                    in.close();
                } catch (IOException e1) {
                }
            }
        }
    }
```

°´×Ö·û¶ÁÈ¡ÎÄ¼þÄÚÈÝ

/**
     * ÒÔ×Ö·ûÎªµ¥Î»¶ÁÈ¡ÎÄ¼þ£¬³£ÓÃÓÚ¶ÁÎÄ±¾£¬Êý×ÖµÈÀàÐÍµÄÎÄ¼þ
     */
    public static void readFileByChars(String fileName) {
        File file = new File(fileName);
        Reader reader = null;
        try {
            System.out.println("ÒÔ×Ö·ûÎªµ¥Î»¶ÁÈ¡ÎÄ¼þÄÚÈÝ£¬Ò»´Î¶ÁÒ»¸ö×Ö½Ú£º");
            // Ò»´Î¶ÁÒ»¸ö×Ö·û
            reader = new InputStreamReader(new FileInputStream(file));
            int tempchar;
            while ((tempchar = reader.read()) != -1) {
                // ¶ÔÓÚwindowsÏÂ£¬\r\nÕâÁ½¸ö×Ö·ûÔÚÒ»ÆðÊ±£¬±íÊ¾Ò»¸ö»»ÐÐ¡£
                // µ«Èç¹ûÕâÁ½¸ö×Ö·û·Ö¿ªÏÔÊ¾Ê±£¬»á»»Á½´ÎÐÐ¡£
                // Òò´Ë£¬ÆÁ±Îµô\r£¬»òÕßÆÁ±Î\n¡£·ñÔò£¬½«»á¶à³öºÜ¶à¿ÕÐÐ¡£
                if (((char) tempchar) != '\r') {
                    System.out.print((char) tempchar);
                }
            }
            reader.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        try {
            System.out.println("ÒÔ×Ö·ûÎªµ¥Î»¶ÁÈ¡ÎÄ¼þÄÚÈÝ£¬Ò»´Î¶Á¶à¸ö×Ö½Ú£º");
            // Ò»´Î¶Á¶à¸ö×Ö·û
            char[] tempchars = new char[30];
            int charread = 0;
            reader = new InputStreamReader(new FileInputStream(fileName));
            // ¶ÁÈë¶à¸ö×Ö·ûµ½×Ö·ûÊý×éÖÐ£¬charreadÎªÒ»´Î¶ÁÈ¡×Ö·ûÊý
            while ((charread = reader.read(tempchars)) != -1) {
                // Í¬ÑùÆÁ±Îµô\r²»ÏÔÊ¾
                if ((charread == tempchars.length)
                        && (tempchars[tempchars.length - 1] != '\r')) {
                    System.out.print(tempchars);
                } else {
                    for (int i = 0; i < charread; i++) {
                        if (tempchars[i] == '\r') {
                            continue;
                        } else {
                            System.out.print(tempchars[i]);
                        }
                    }
                }
            }

        } catch (Exception e1) {
            e1.printStackTrace();
        } finally {
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e1) {
                }
            }
        }
    }
```
java¶ÁÐ´Ñ¹ËõÎÄ¼þ

.gzÎÄ¼þµÄ¶ÁÐ´

¶ÁÈ¡ 
ÆäÊµ¾ÍÊÇ¶àÁËÒ»²½£¬¾ÍÊÇ°ÑÓÃInputStreamReader ¶Ô FileInputStream½øÐÐÁËÒ»²ã°ü×°
```
public static void main(String[] args) throws Exception {
    FileInputStream fis =new FileInputStream("path");
    GZIPInputStream gzipInputStream = new GZIPInputStream(fis);
    InputStreamReader in = new InputStreamReader(gzipInputStream);
    BufferedReader br = new BufferedReader(in);
    String str="";
    while ((str=br.readLine())!=null){
        String[] split = str.split("\t");
        System.out.println(split[0]);
    }
    //¹Ø±ÕÁ÷£¬²Î¿¼Ç°ÃæµÄ£¬ÕâÀï¾Í²»Ð´ÁË 
}
```
Ð´ 
Ïà±ÈÓÚÃ»ÓÐÑ¹ËõµÄ£¬¾Í¶àÁËÒ»²½ GZIPOutputStream ¶Ô FileOutputStreamµÄ°ü×°
```
    public static void saveToGZFiles(String text, File file) {
        if(!file.exists()){
            try {
                file.createNewFile();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        try {
            FileOutputStream out= null;
            out = new FileOutputStream(file,true);
            GZIPOutputStream gzipOut = new GZIPOutputStream(out);
            StringBuffer sb=new StringBuffer();
            sb.append(text);
            gzipOut.write(sb.toString().getBytes("utf-8"));
           // out.close();
            gzipOut.close();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
```
¶ÁÈ¡ÎÄ¼þ¼ÐÏÂµÄËùÓÐÎÄ¼þ
```
String path="E:\\datatest\\input\\TB_CQTSIGNAL_SAMPLE_01_170221";
File files = new File(path);
if(files.isFile()){
  //ËµÃ÷ÊÇÎÄ¼þ
    System.out.println("ÎÄ¼þ");
    System.out.println("path=" + files.getPath());
    System.out.println("absolutepath=" + files.getAbsolutePath());
    System.out.println("name=" + files.getName());
}
if(files.isDirectory()){
    //ËµÃ÷ÊÇÎÄ¼þ¼Ð
    String[] filelist = files.list();
    for (int j = 0; j < filelist.length; j++){
        //µ÷ÓÃÉÏÃæµÄ¶ÁÈ¡·½·¨
    fis = new FileInputStream(path+"\\"+filelist[i]);
    ....
    }
}
```
¸´ÖÆÎÄ¼þ

ÒÔÎÄ¼þÁ÷µÄ·½Ê½¸´ÖÆÎÄ¼þ
```
 public void copyFile(String src,String dest) throws IOException...{
         FileInputStream in=new FileInputStream(src);
         File file=new File(dest);
         if(!file.exists())
             file.createNewFile();
         FileOutputStream out=new FileOutputStream(file);
         int c;
         byte buffer[]=new byte[1024];
         while((c=in.read(buffer))!=-1)...{
             for(int i=0;i<c;i++)
                 out.write(buffer[i]);        
         }
         in.close();
         out.close();
     }
```
´´½¨ÎÄ¼þ¼Ð
```
public void createDir(String path)...{
         File dir=new File(path);
         if(!dir.exists())
             dir.mkdir();
}
```
´´½¨ÎÄ¼þ
```
public void createFile(String path,String filename) throws IOException...{
         File file=new File(path+"/"+filename);
         if(!file.exists())
             file.createNewFile();
}
```
É¾³ýÎÄ¼þ
```
public void delFile(String path,String filename)...{
         File file=new File(path+"/"+filename);
         if(file.exists()&&file.isFile())
             file.delete();
}
```
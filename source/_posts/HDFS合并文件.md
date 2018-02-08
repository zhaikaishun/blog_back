---
title: HDFS合并文件
date: 2016-05-15 23:13:21
tags: [hdfs]
categories: [大数据,hdfs]
author: kaishun
id: 1
permalink: hdfs-merge-file
blogexcerpt: hdfs提供了一种FileUtil.copyMerge（）的方法， 注意下面的 false。 这个，如果改为true，就会删除这个目录
toc: true
---
![hdfs-commands](http://or49tneld.bkt.clouddn.com/18-1-23/63075597.jpg)  

# **HDFS到HDFS的合并**
hdfs提供了一种FileUtil.copyMerge（）的方法， 注意下面的 false 这个，如果改为true，就会删除这个目录，public void copyMerge(String folder, String file) {Path src = new Path(folder).......;
```java
	public void copyMerge(String folder, String file) {

		Path src = new Path(folder);
		Path dst = new Path(file);

		try {
			FileUtil.copyMerge(src.getFileSystem(conf), src,
					dst.getFileSystem(conf), dst, false, conf, null);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}


```

# **上传文件到HDFS并且合并**
```java

	/**
	 * 读取本地文件到HDFS系统<br>
	 * 请保证文件格式一直是UTF-8，从本地->HDFS
	 * 
	 */
	/**
	 * 
	 * @param localDirname 源文件所在位置
	 * @param hdfsPath 要放在服务器的位置
	 * @param destFileName 要合并成的文件名称
	 * @param filter
	 * @return
	 */
	public boolean putMerge(String localDirname, String hdfsPath,String destFileName, String filter )
	{
		try
		{
			File dir = new File(localDirname);
			if(!dir.isDirectory())
			{
				System.out.println(localDirname+"不是目录 ");
				return false;
			}

			File[] files = dir.listFiles();
			if(files.length ==0)
				return false;
			
			System.out.println("Begin move " + localDirname + " to " + hdfsPath);
			
			while(checkFileExist(hdfsPath+"/"+destFileName))
			{
				if(destFileName.contains(".x"))
					destFileName += "x";
				else
					destFileName += ".x";
			}
			
			mkdir(hdfsPath);
			
			Path f = new Path(hdfsPath+"/"+destFileName);
			FSDataOutputStream os = fs.create(f, true);
			byte[] buffer = new byte[10240000];
 			
			for(int i=0; i<files.length; i++)
			{
				if(MainSvr.bExitFlag == true)
					break;	
				 
				File file = files[i];
				if(!file.getName().toLowerCase().contains(filter.toLowerCase()))
					continue;
				FileInputStream is = new FileInputStream(file);					
				GZIPInputStream gis = null;
				if(file.getName().toLowerCase().endsWith("gz"))
					gis=new GZIPInputStream(is);
				
				while(MainSvr.bExitFlag != true)
				{
		            int bytesRead =0;
		            if(gis == null)
		            	bytesRead= is.read(buffer);
		            else
		            	bytesRead= gis.read(buffer,0,buffer.length);
		            if (bytesRead >= 0) 
		            {
		                os.write(buffer, 0, bytesRead);
		            }
		            else
		            {
		            	break;
		            }
				}
	            if(gis != null)
	            	gis.close();
				is.close();
			}
			os.close();
			if(MainSvr.bExitFlag)
				return false;
			System.out.println("Success move " + localDirname + " to " + hdfsPath);
			return true;
		}
		catch (Exception e)
		{
			e.printStackTrace();
			log.info("putMerge error:" + e.getMessage());
		}
		return false;
	}
	
	
	public boolean mkdir(String dirName)
	{	
		try
		{
			if (checkFileExist(dirName))
			return true;			
			Path f = new Path(dirName);
			System.out.println("Create and Write :" + dirName + " to hdfs");
			return hdfs.mkdirs(f);
		}
		catch (Exception e)
		{
			System.out.println("mkdir Fail:" + e.getMessage());
			e.printStackTrace();		
		}

		return false;
	}
```
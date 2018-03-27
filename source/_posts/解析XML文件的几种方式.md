---
title: 解析XML文件的几种方式
date: 2017-06-26 21:25:21
tags: [java,xml解析]
categories: [programme]
author: kaishun
id: 75
permalink: xml-parse1  
blogexcerpt: 前言,虽然更喜欢用json,但是xml更灵活，在配置文件的时候更方便 要用SAXRead需要的jar包,dom4j-1.6.1.jar, jaxen-1.1-beta-10.jar,  XML文件格式如下.读取所有的表名称, 根据表名称，获取所有的字段以及属性, 另外介绍一下XMLConfiguration, 将所有的字段以及属性放入到map中, 使用XMLConfiguration 需要引入的jar包commons-configuration-1.0.jar
---

## 前言
虽然更喜欢用json,但是xml更灵活，在配置文件的时候更方便

## 要用SAXRead需要的jar包
dom4j-1.6.1.jar, jaxen-1.1-beta-10.jar


## XML文件格式如下
```xml
<?xml version="1.0" encoding="UTF-8"?>  
  <!-- 
  tablename保存数据表的名称
  fieldname保存字段名称， type对应的是字段的属性，都必须得有
    -->
<tables>  
	<table tablename="people">
		<fieldname fieldname="name" type="string"/>
		<fieldname fieldname="age" type="int"/>
		<splitsign rule="," />
	</table>
	
	<table tablename="book">
		<fieldname fieldname="name" type="string"/>
		<fieldname fieldname="price" type="int"/>
		<splitsign rule="." />
	</table>
	
	<table tablename="cat">
		<fieldname fieldname="color" type="string"/>
		<fieldname fieldname="category" type="string"/>
		<splitsign rule=" " />
	</table>
	
	<table tablename="people1">
		<fieldname fieldname="name" type="string"/>
		<fieldname fieldname="age" type="int"/>
		<fieldname fieldname="sex" type="string"/>
		<splitsign rule="," />
	</table>
	
</tables>  
```

## 读取所有的表名称
```java
private String configPath="conf/config_table.xml";
		/**
		 * 
		 * @return an arrayList tableList
		 * @throws DocumentException
		 */
	    public ArrayList getTableName() throws DocumentException{
	    	ArrayList<String> tableNameArray = new ArrayList<String>();
	        
	        SAXReader reader = new SAXReader();  
	          
	        Document document = reader.read(new File(configPath));  
	        
	        String xpathtable = "//tables//table";
	        List<String> list = document.selectNodes(xpathtable);
	        Iterator iter = list.iterator();
	        while(iter.hasNext()){
	        	Element element = (Element) iter.next();
	        	tableNameArray.add(element.attribute(0).getValue());
	        	      	
	        }
	        return tableNameArray;
	    }
	    
``` 

##  根据表名称，获取所有的字段以及属性
```java
 /**
	     * @param table name
	     * @return	an String[] String[0] is field, String[1] is sttribute
	     * @throws DocumentException
	     */
	    public String[] getFieldAndAttr(String tablename) throws DocumentException{
	        // Initialize table field
	    	String fieldString = "";
	    	// Initialize table attribute
	        String attrString="";
	        // Initialize the split sign
	        String splitsignString = "";
	        
	        String[] tableFieldAndAttr=null;
	    	
	        SAXReader reader = new SAXReader();  
	        Document document = reader.read(new File(configPath));  
	        
	        String xpath = "//tables//table[@tablename='"+tablename+"']//fieldname";
	        List<String> list = document.selectNodes(xpath);
	        Iterator iter = list.iterator();
	        while(iter.hasNext()){
	        	Element element = (Element) iter.next();
	        	// connect the field
	        	fieldString = fieldString+element.attribute(0).getValue()+ " ";
	        	// connect the attribute
	        	attrString = attrString+element.attribute(1).getValue()+ " ";
	        }
	        // remove the last character
	        fieldString = fieldString.substring(0,fieldString.length()-1);
	        attrString = attrString.substring(0,attrString.length()-1);
	        
	        String xpathSplit = "//tables//table[@tablename='"+tablename+"']//splitsign";
	        List<String> signlist = document.selectNodes(xpathSplit);
	        Iterator splititer = signlist.iterator();
	        while(splititer.hasNext()){
	        	Element splitElement = (Element) splititer.next();
	        	splitsignString = splitsignString+splitElement.attribute(0).getValue();
	        }
	 
	        tableFieldAndAttr = new String[]{fieldString,attrString,splitsignString};
	        return tableFieldAndAttr;
	    
	    }  
```  

## 另外介绍一下XMLConfiguration, 将所有的字段以及属性放入到map中, 使用XMLConfiguration
需要引入的jar包commons-configuration-1.0.jar,
```
package teststring;

import org.apache.commons.configuration.ConfigurationException;
import org.apache.commons.configuration.XMLConfiguration;

import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

/**
 * Created by Administrator on 2017/3/21.
 */
public class TestConf {

    public static void main(String[] args) {
        Map<String,Object> dataMap=new HashMap<String,Object>();
        try {
            XMLConfiguration config = new XMLConfiguration("conf/datawatch_config.xml");
            config.setEncoding("UTF-8");
            config.setDelimiterParsingDisabled(true);
            Iterator<String> iter = config.getKeys();
            while (iter.hasNext())
            {
                String key = iter.next().toString();
                dataMap.put(key, config.getProperty(key));
            }

        } catch (ConfigurationException e) {
            e.printStackTrace();
        }
    }
}

```

## XMLconfig的XPATH的使用, 使用XPATH解析， xpath是我比较喜欢的一种解析方式
```xml
<comm>
  <data>
    <name>zks</name>
   <age>24</age>
  </data>
  <data>
    <name>skz</name>
    <age>23</age>
  </data>
</comm>

```
需要引入的jar包 commons-jxpath-1.3.jar, 
commons-configuration-1.0.jar  
其中 [commos-jaxpath-1.3下载地址](http://commons.apache.org/proper/commons-jxpath/download_jxpath.cgi)
```
 XMLConfiguration config = new XMLConfiguration("conf/datawatch_config.xml");
 //注意下面的两句话，必须这样才能使用xpath去解析
XPathExpressionEngine xPathExpressionEngine = new XPathExpressionEngine();
config.setExpressionEngine(xPathExpressionEngine);
System.out.println(config.getString("data[1]/name"));
System.out.println(config.getString("data[2]/name"));
System.out.println(config.getString("data[name='skz']/age"));
```
将会输出
zks
skz
23  

更多XPATH解析方法参考 菜鸟教程  


## 补充 
xml如下
```xml
<comm>
  <data>
    <name>zks</name>
   <age>24</age>
   <sex>man</sex>
  </data>
  <data>
    <name>skz</name>
    <age>23</age>
  </data>
</comm>
```  
如果我想得到第一个data下的所有的， 然后按照上面的，存入map中，怎么做呢， 我们可以得到data下的这个config，可以使用SubnodeConfiguration subnodeConfiguration = config.configurationAt(key),代码如下
```java
 SubnodeConfiguration subnodeConfiguration = config.configurationAt("data[1]");
// 下面就可以用了
 Iterator itea1 = subnodeConfiguration.getKeys();
                while (itea1.hasNext()){
                    System.out.println("iter.next().toString(): "+itea1.next().toString());
                     System.out.println("conf.getProperty"+subnodeConfiguration.getProperty(key));
                }
```  
参考自[stackoverflow](http://stackoverflow.com/questions/14388418/apache-commons-xmlconfiguration-how-to-fetch-a-list-of-objects-at-a-given-node) 和 http://commons.apache.org/proper/commons-configuration/userguide/howto_xml.html  
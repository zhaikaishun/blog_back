---
title: Spark Sql 编程式结构DataType转换 代码类小结
date: 2017-05-21 21:25:21
tags: [spark]
categories: [大数据,spark]
author: kaishun
id: 63
permalink: spark-sql-dataType-transform
---

### 基本类型数据转DataType
```java
public DataType attrToDataType(String attrstr){
		DataType returnDataType = DataTypes.StringType;
		
		switch(attrstr.toLowerCase()){
		case "short" :
			returnDataType = DataTypes.StringType;
			break;
		case "char" :
			returnDataType = DataTypes.StringType;
			break;
		case "int" :
			returnDataType = DataTypes.IntegerType;
			break;
		case "integer" :
			returnDataType = DataTypes.IntegerType;
			break;
		case "float" :
			returnDataType = DataTypes.FloatType;
			break;
		case "double" :
			returnDataType = DataTypes.DoubleType;
			break;
		case "string" :
			returnDataType = DataTypes.StringType;
			break;
		case "varchar":
			returnDataType = DataTypes.StringType;
			break;
		case "long":
			returnDataType = DataTypes.LongType;
			break;
		case "boolean":
			returnDataType = DataTypes.BooleanType;
			break;
		case "date":
			returnDataType = DataTypes.DateType;
			break;
		}
		return returnDataType;
	}
```
<!-- more -->
### 基本类型转封装类
```java
	/**
	 * @param fields field name
	 * @param attrstr field attribute
	 * @return an object -- String,Integer,Long and etc
	 */
	public Object arrtToObject(String fields,String attrstr){
		Object fieldObject = fields;
		switch(attrstr.toLowerCase()){
		case "short" :
			fieldObject = Short.parseShort(fields);
			break;		
		case "char" :
			fieldObject = fields;
			break;
		case "int" :
			fieldObject = Integer.parseInt(fields);
			break;
		case "integer" :
			fieldObject = Integer.parseInt(fields);
			break;
		case "float" :
			fieldObject = Float.parseFloat(fields);
			break;
		case "double" :
			fieldObject = Double.parseDouble(fields);
			break;
		case "long":
			fieldObject = Long.parseLong(fields);
			break;
		case "boolean":
			fieldObject = Boolean.parseBoolean(fields);
			break;
		case "date":
			fieldObject = Date.parse(fields);
			break;
		}
		return fieldObject;
	}

```

### 当字段列表和属性列表都生成后，用for循环去createStructField，字段和属性要对应上
```java
   for(int i=0;i<schemaList.length;i++){
	    	fields.add(DataTypes.createStructField(schemaList[i], dataTypesList[i], true));
	    }
		StructType schema = DataTypes.createStructType(fields);
```

### 生成rowRDD的时候，字段和属性也要对应上
```java
JavaRDD<Row> rowRDD = people.map(
		  new Function<String, Row>() {
		    public Row call(String record) throws Exception {
		      String[] fields = record.split(FieldSplitSign);
		      //TODO maybe cause the memory leak
		      Object[] objeck = new Object[fields.length];
//				System.out.println("field length:" + fields.length);
				for(int i=0;i<fields.length;i++){
//					System.out.println("fields[i].trim()"+fields[i].trim());
					objeck[i] = dataTypUtils.arrtToObject(fields[i].trim(),fieldAttyList[i].trim());
		      }
		      return RowFactory.create(objeck);
		    }
		  });
```


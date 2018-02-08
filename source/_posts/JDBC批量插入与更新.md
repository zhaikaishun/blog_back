---
title: JDBC批量插入与更新工具类
date: 2017-05-04 21:25:21
tags: [java,JDBC,工具类]
categories: [programme]
author: kaishun
id: 55
permalink: jdbc-util  
blogexcerpt: 关键字： 批量添加，JDBC批量更新，public void UpdateFileData(String sSQL, ArrayList<String[]> objParams)，JDBCTest
---

## 批量添加
```
	public void UpdateFileData(String sSQL, ArrayList<String[]> objParams)
	{
		GetConn();
		int iResult = 0;

		try
		{
//			Statement ps = _CONN.createStatement();
			PreparedStatement preparedStatement = _CONN.prepareStatement(sSQL);
			for (int i = 0; i <objParams.size() ; i++) {
				preparedStatement.setString(1,objParams.get(i)[0]);
				preparedStatement.setString(2,objParams.get(i)[1]);
				preparedStatement.setString(3,objParams.get(i)[2]);
				preparedStatement.setLong(4,Integer.parseInt(objParams.get(i)[3]));
				preparedStatement.setString(5,objParams.get(i)[4]);
				preparedStatement.setString(6,objParams.get(i)[5]);
				preparedStatement.addBatch();
			}

			preparedStatement.executeBatch();
		} catch (Exception ex)
		{
			System.out.println(ex.getMessage());

		} finally
		{
			CloseConn();
		}

	}
``` 
## 参考网上的例子
```
import java.sql.Connection;
import java.sql.Date;
import java.sql.PreparedStatement;
import java.sql.Statement;
 
import org.junit.Test;
 
import xuezaipiao1.JDBC_Tools;
/**
 * 向Oracle 的 temp 数据表中添加  10万 条记录
 * 测试如何插入，用时最短
 */
 
public class JDBCTest {
     
    /**
     * 
     * 1.使用 Statement .
     * 测试用时:35535
     */
    @Test
    public void testBbatchStatement() {
        Connection conn = null;
        Statement statement = null;
        String sql = null;
        try {
            conn = JDBC_Tools.getConnection();
            JDBC_Tools.beginTx(conn);
             
            long beginTime = System.currentTimeMillis();
            statement = conn.createStatement();
            for(int i = 0;i<100000;i++){
                sql = "INSERT INTO temp values("+(i+1)
                        +",'name_"+(i+1)+"','13-6月 -15')";
                statement.executeUpdate(sql);
            }
            long endTime = System.currentTimeMillis();
            System.out.println("Time : "+(endTime - beginTime));
            JDBC_Tools.commit(conn);
        } catch (Exception e) {
            e.printStackTrace();
            JDBC_Tools.rollback(conn);
        }finally{
            JDBC_Tools.relaseSource(conn, statement);
        }
    }
     
    /**
     * 使用PreparedStatement 
     * 测试用时:9717
     */
    @Test
    public void testBbatchPreparedStatement() {
        Connection conn = null;
        PreparedStatement ps = null;
        String sql = null;
        try {
            conn = JDBC_Tools.getConnection();
            JDBC_Tools.beginTx(conn);
             
            long beginTime = System.currentTimeMillis();
            sql = "INSERT INTO temp values(?,?,?)";
            ps = conn.prepareStatement(sql);
            Date date = new Date(new java.util.Date().getTime());
            for(int i = 0;i<100000;i++){
                ps.setInt(1, i+1);
                ps.setString(2, "name_"+i);
                ps.setDate(3, date);
                ps.executeUpdate();//9717
            }
            long endTime = System.currentTimeMillis();
            System.out.println("Time : "+(endTime - beginTime));
            JDBC_Tools.commit(conn);
        } catch (Exception e) {
             
            e.printStackTrace();
            JDBC_Tools.rollback(conn);
        }finally{
            JDBC_Tools.relaseSource(conn, ps);
        }
    }
         
    /**
     * 测试用时 : 658
     */
    @Test
    public void testBbatch() {
        Connection conn = null;
        PreparedStatement ps = null;
        String sql = null;
        try {
            conn = JDBC_Tools.getConnection();
            JDBC_Tools.beginTx(conn);
             
            long beginTime = System.currentTimeMillis();
            sql = "INSERT INTO temp values(?,?,?)";
            ps = conn.prepareStatement(sql);
            Date date = new Date(new java.util.Date().getTime());
            for(int i = 0;i<100000;i++){
                ps.setInt(1, i+1);
                ps.setString(2, "name_"+i);
                ps.setDate(3, date);
                 
                //积攒SQL
                ps.addBatch();
                 
                //当积攒到一定程度,就执行一次,并且清空记录
                if((i+1) % 300==0){
                    ps.executeBatch();
                    ps.clearBatch();
                }
            }
            //总条数不是批量值整数倍,则还需要在执行一次
            if(100000 % 300 != 0){
                ps.executeBatch();
                ps.clearBatch();
            }
            long endTime = System.currentTimeMillis();
            System.out.println("Time : "+(endTime - beginTime));
            JDBC_Tools.commit(conn);
        } catch (Exception e) {
             
            e.printStackTrace();
            JDBC_Tools.rollback(conn);
        }finally{
            JDBC_Tools.relaseSource(conn, ps);
        }
    }
}
```


## JDBC批量更新
```
public void insertInTransaction(String[] sqls) throws SQLException{
          try ( Connection conn = DriverManager.getConnection(url, user, pass)) {
              //关闭自动提交，即开启事务
              conn.setAutoCommit(false);
              try(Statement stmt = conn.createStatement()) {
                  for (String sql : sqls) {
                      stmt.addBatch(sql);
                  }
                  stmt.executeBatch();
                 //提交事务
                 conn.commit();
             } catch (SQLException e) {
                 conn.rollback();
                 System.out.println("Exception:");
                 System.out.println(e.getMessage());
             } finally {
                 conn.setAutoCommit(true);
             }
         } 
     }
```
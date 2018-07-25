---
title: JDBC连接数据库笔记

date: 2016-06-10

tags: Java

---

![](http://pccmxww5q.bkt.clouddn.com/oracle_java.jpg?imageView2/0/w/560/h/380/q/100 "")


### 准备工作

1. 新建一个Java工程
2. 新建一个数据库warehouse
    * 新建表mylist
    * 向表中添加数据

+-----+-------------+
| id  | name        |
+-----+-------------+
| 111 | liyongsheng |
| 222 | zhangsan    |
| 333 | xiaoming    |
+-----+-------------+

### 1.加载JDBC驱动程序

>通过 java.lang.Class类的静态方法forName(String className)实现

```java
try {
    //加载mysql的驱动类
    Class.forName("com.mysql.jdbc.Driver");
} catch(ClassNotFoundException e) {
    System.Out.Println("驱动程序加载失败");
    e.printStackTrace();
}
```

> 成功加载mysql的驱动类后会将Driver类的实例注册到DriverManager类中

#### 2.提供JDBC连接的URL

* 连接URL定义了连接数据库时的协议、子协议、数据库标识

> 协议：在JDBC中总是以jdbc开头
> 子协议：是桥接的程序或者数据库管理系统的名称
> 数据库标识：标记找到数据库来源的地址和连接端口

```java
    String url = "jdbc:mysql://localhost:3306/warehouse?useUnicode=true&characterEncoding=gbk";
```

### 3.创建数据库的连接

* 要连接数据库，需要向java.lang.DriverManager请求并获得一个Connection对象，该对象就代表一个数据库连接
* 使用方法如下

```java
DriverManager.getConnect(String url, String username, String password);
```

### 4.创建一个Statement对象

> 要执行SQL语句，必须获得java.sql.Statement的实例
Statement的实例分为三种类型：

```java
//执行静态SQL语句
Statement stmt=connection.createStatement();
//执行动态SQL语句
PreparedStatement pstmt = con.prepareStatement(sql);
//执行数据库存储过程
CallableStatement cstmt = con.prepareCall("{CALL demoSp(? , ?)}");
```

### 5.执行SQL语句

Statement提供三种方法：

```java
//1.执行查询数据库的SQL语句，返回一个结果集（ResultSet）对象。
ResultSet rs = stmt.executeQuery("SELECT * FROM ...");
//2.用于执行INSERT、UPDATE或 DELETE语句以及SQL DDL语句
int rows = stmt.executeUpdate("INSERT INTO ...");
//3.用于执行返回多个结果集、多个更新计数或二者组合的语句
boolean flag = stmt.execute(String sql);
```

### 6.处理结果

两种情况:
1. 执行更新返回的是本次操作影响到的记录数。   
2. 执行查询返回的结果是一个ResultSet对象。

```java
while(rs.next()){
    String name = rs.getString("name") ;
    String pass = rs.getString(1) ; // 此方法比较高效
}
```

### 7.关闭JDBC对象

> 关闭顺序和声明顺序相反

1. 关闭resultSet结果集
2. 关闭statement声明
3. 关闭connection连接

```java
if (resultSet != null) {
    try {
        resultSet.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
if (statement != null) {
    try {
        statement.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
if (connection != null) {
    try {
        connection.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
```

> 附源程序

```java
package com.successli.jdbcTest;

import java.sql.*;

/**
 * Created by liyon on 2016.6.1.
 */
public class mysqlConnection {

    public static void main(String[] args) {
        Connection connection;
        Statement statement;
        ResultSet resultSet;

        //1.加载jdbc驱动程序
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            System.out.println("找不到驱动程序，加载驱动失败");
            e.printStackTrace();
        }

        //2.提供JDBC连接的URL
        //String url = "jdbc:mysql://localhost:3306/warehouse";
        String url = "jdbc:mysql://localhost:3306/warehouse?useUnicode=true&characterEncoding=gbk";
        String username = "root";
        String password = "1234";
        try {
            //3.建立连接
            connection = DriverManager.getConnection(url,username,password);
            //4.创建一个statement声明
            statement = connection.createStatement();
            //5.执行SQL语句，返回结果集resultSet
            resultSet = statement.executeQuery("select * from mylist;");
            //6.处理返回结果集resultSet的结果
            while (resultSet.next()) {
                int id = resultSet.getInt(1);
                String name = resultSet.getString(2);
                System.out.println(id);
                System.out.println(name);
            }
            //7.关闭连接
            if (resultSet != null) {
                try {
                    resultSet.close();
                    System.out.println("resultSet已关闭");
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (statement != null) {
                try {
                    statement.close();
                    System.out.println("statement已关闭");
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (connection != null) {
                try {
                    connection.close();
                    System.out.println("connection已关闭");
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }

        } catch (SQLException se) {
            System.out.println("数据库连接失败");
            se.printStackTrace();
        }

    }
}
```

> 执行结果：

```java
111
liyongsheng
222
zhangsan
333
xiaoming
resultSet已关闭
statement已关闭
connection已关闭
```
# JDBC

JDBC（Java Database Connectivity），是 SUN的规范，为Java应用提供与各种数据库连接的一个抽象接口或协议。JDBC API 加上 JDBC Driver，可以访问关系型数据库。应用发送用户请求给特定数据库的行为，被定义在JDBC中的类或接口中。

## JDBC的组成

1. JDBC API ：```java.sql.*```
2. 提供了连接客户端应用的标准
3. JDBC Driver manager：用于实现对特定于某个数据库的驱动的调用以完成用户的请求。
4. JDBC Test Unit：用于测试CURD操作。
5. JDBC-ODBC Bridge Drivers: 将JDBC 方法转化为 ODBC 调用函数，利用了```sun.jdbc.odbc```

### **Interfaces of JDBC API**

A list of popular *interfaces* of JDBC API is given below:

```
- Driver interface
- Connection interface
- Statement interface
- PreparedStatement interface
- CallableStatement interface
- ResultSet interface
- ResultSetMetaData interface
- DatabaseMetaData interface
- RowSet interface
```

### Classes of JDBC API

```
- DriverManager class
- Blob class
- Clob class
- Types class
```

## JDBC 使用

```java
package com.vinayak.jdbc;

import java.sql.*;

public class JDBCDemo {
	
	public static void main(String args[])
		throws SQLException, ClassNotFoundException
	{
		String driverClassName
			= "sun.jdbc.odbc.JdbcOdbcDriver";
		String url = "jdbc:odbc:XE";
		String username = "scott";
		String password = "tiger";
		String query
			= "insert into students values(109, 'bhatt')";

		// 注册驱动 等同于 java.sql.DriverManager.registerDriver(new Driver());
		Class.forName(driverClassName);

		// Obtain a connection
		Connection con = DriverManager.getConnection(
			url, username, password);

		// Obtain a statement
		Statement st = con.createStatement();

		// Execute the query
		int count = st.executeUpdate(query);
		System.out.println(
			"number of rows affected by this query= "
			+ count);

		// Closing the connection as per the
		// requirement with connection is completed
		con.close();
	}
} // class

```

## JDBC 驱动

The JDBC classes are contained in the Java Package `**java.sql**` and `**javax.sql**`.
JDBC helps you to write Java applications that manage these three programming activities:

1. Connect to a data source, like a database.
2. Send queries and update statements to the database
3. Retrieve and process the results received from the database in answer to your query

![image-20220718103413664](D:\project\笔记\数据库\pic\Structure of JDBC .png)

JDBC Driver是客户端一侧的适配器，将 java 程序转化为 DBMS 可以理解的请求。有四种JDBC Drivers:

1. Type-1 driver or JDBC-ODBC bridge driver
2. Type-2 driver or Native-API driver
3. Type-3 driver or Network Protocol driver
4. Type-4 driver or Thin driver

![image-20220718141845455](D:\project\笔记\数据库\pic\JDBC driver.png)

# 项目

![A](D:\project\笔记\数据库\pic\内网穿透项目结构.png)

![image-20220720160931031](D:\project\笔记\数据库\pic\image-20220720160931031.png)

## JDBC 驱动实现

### 需要实现的类

```
java.sql.Driver
java.sql.Connection
java.sql.Statement
java.sql.ResultSet    
```

### Driver

```java
public class MyDriver implements Driver {
    private final org.slf4j.Logger logger = LoggerFactory.getLogger(this.getClass());

    SocketIOClient client = null;
    private final int DRIVER_VERSION_MAJOR = 1;
    private final int DRIVER_VERSION_MINOR = 1;

    //依靠静态函数块注册驱动
    static{
        try {
            DriverManager.registerDriver(new MyDriver());
        } catch (Exception e) {
            throw new RuntimeException("Can't register driver");
        }
    }

    @SneakyThrows
    @Override
    public Connection connect(String url, Properties info) throws SQLException {
        this.client = BeanUtil.getBean(SocketIOClient.class);
        MyConnection conn = new MyConnection(client);
        return (Connection) conn;
    }

    // URL的正确性交给Agent验证
    // 后续需要完善，通过Agent返回的错误消息进行更新
    @Override
    public boolean acceptsURL(String url) throws SQLException {
        return true;
    }

    @Override
    public DriverPropertyInfo[] getPropertyInfo(String url, Properties info) throws SQLException {
        return new DriverPropertyInfo[0];
    }

    @Override
    public int getMajorVersion() {
        return DRIVER_VERSION_MAJOR;
    }

    @Override
    public int getMinorVersion() {
        return DRIVER_VERSION_MINOR;
    }

    @Override
    public boolean jdbcCompliant() {
        return false;
    }
}
```

### Connection

```java
public class MyConnection implements Connection {
    private final org.slf4j.Logger logger = LoggerFactory.getLogger(this.getClass());

    private SocketIOClient client;

	public MyConnection(SocketIOClient client) throws JsonProcessingException {
        this.client = client;
        Gson gson = new Gson();
        SimpleMessage message = new SimpleMessage("hello, Server");
        client.sendEvent("ClientReceive", gson.toJson(message));
        logger.info("向客户端发送消息：ClientReceive, 请求进行数据源注册");
    }
    
    @Override
    public Statement createStatement() throws SQLException {
        return (Statement) new MyStatement(client);
    }
    
    // 返回元数据信息，通过自己实现的 MyDatabaseMetaData
    @Override
    public DatabaseMetaData getMetaData() throws SQLException {
        return new DataBaseMetaData();
    }
}
```

### Statement

```java
public class MyStatement implements Statement {
    private final org.slf4j.Logger logger = LoggerFactory.getLogger(this.getClass());

    private SocketIOClient client;
    private Connection connection;
    private String sql;

    public MyStatement(Connection conn) {
        this.connection = conn;
    }

    private MyStatement(String sql, MyConnection conn){
        this.sql = sql;
        this.connection = conn;
    }

    public MyStatement(SocketIOClient client) {
        this.client = client;
    }


    @Override
    public ResultSet executeQuery(String sql) throws SQLException {
        if(isClosed()) throw new SQLException("This Statement is closed.");
        try{
            resultSet =
        }
        return null;
    }
```


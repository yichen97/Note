# JDBC

## JDBC 简介

 JDBC(Java Database Connectivity)是一个**独立于特定数据库管理系统、通用的SQL数据库存取和操作的公共接口（一组API）**，定义了用来访问数据库的标准Java类库，使用这个类库可以以一种标准的方法、方便地访问数据库资源。

- JDBC为访问不同的数据库提供了一种 **统一的途径**，为开发者屏蔽了一些细节问题。
- JDBC的目标是使Java程序员 **使用JDBC可以连接任何提供了JDBC驱动程序的数据库系统**，这样就使得程序员无需对特定的数据库系统的特点有过多的了解，从而大大简化和加快了开发过程。

![JDBC提供了连接不同数据库的实现](D:\project\笔记\数据库\pic\jdbc.jpg)

- **JDBC接口（API）**包括两个层次：
  - 面向应用的`API：Java API`，抽象接口，供应用程序开发人员使用（连接数据库，执行SQL语句，获得结果）。
  - 面向数据库的``API：Java Driver API`，供开发商开发数据库驱动程序用。

## JDBC应用

### 1、Driver接口

- `Java.sql.Driver` 接口是所有 JDBC 驱动程序需要实现的接口。这个接口是提供给数据库厂商使用的，不同数据库厂商提供不同的实现
- 在程序中不需要直接去访问实现了 Driver 接口的类，而是由驱动程序管理器类(`java.sql.DriverManager`)去调用这些Driver实现。
- **加载和注册JDBC驱动**
  - 加载 JDBC 驱动需调用 Class 类的静态方法 `forName()`，向其传递要加载的 JDBC 驱动的类名
  - `DriverManager` 类是驱动程序管理器类，负责管理驱动程序
  - 通常不用显式调用 `DriverManager` 类的 `registerDriver()` 方法来注册驱动程序类的实例，因为 Driver 接口的驱动程序类都包含了静态代码块，在这个静态代码块中，会调用 `DriverManager.registerDriver()` 方法来注册自身的一个实例。
- **建立连接**
  - 可以调用 `DriverManager` 类的 `getConnection()` 方法建立到数据库的连接
    JDBC URL 用于标识一个被注册的驱动程序，驱动程序管理器通过这个 URL 选择正确的驱动程序，从而建立到数据库的连接。
  - JDBC URL的标准由三部分组成，各部分间用冒号分隔。
    - `jdbc`:<子协议>:<子名称>
    - 协议：JDBC URL中的协议总是`jdbc`
    - 子协议：子协议用于标识一个数据库驱动程序
    - 子名称：一种标识数据库的方法。子名称可以依不同的子协议而变化，用子名称的目的是为了定位数据库提供足够的信息
- 几种常用的JDBC URL

> 对于 Oracle 数据库连接，采用如下形式：
> `jdbc:oracle:thin:@localhost:1521:sid`
> 对于 `SQLServer` 数据库连接，采用如下形式：
> `jdbc:microsoft:sqlserver//localhost:1433; DatabaseName=sid`
> 对于 MYSQL 数据库连接，采用如下形式：
> `jdbc:mysql://localhost:3306/sid`

- 访问数据库

> 数据库连接被用于向数据库服务器发送命令和 SQL 语句，在连接建立后，需要对数据库进行访问，执行 `sql`语句
> 在 `java.sql` 包中有 3 个接口分别定义了对数据库的调用的不同方式：
> `Statement`
> `PrepatedStatement`
> `CallableStatement`

#### 1、Statement

- 通过调用 `Connection` 对象 的 **`createStatement` 方法创建该对象**
  该对象用于执行静态的 SQL 语句，并且返回执行结果
- Statement 接口中定义了下列方法用于执行 SQL 语句：

```java
ResultSet excuteQuery(String sql)
int excuteUpdate(String sql)
```

#### 2、`ResultSet`

- 通过调用 Statement 对象的 `excuteQuery()` 方法创建该对象
- `ResultSet` 对象以逻辑表格的形式封装了执行数据库操作的结果集，`ResultSet` 接口由数据库厂商实现
- `ResultSet` 对象维护了一个指向当前数据行的游标，初始的时候，游标在第一行之前，可以通过 `ResultSet` 对象的 next() 方法移动到下一行
- `ResultSet` 接口的常用方法：

```java
boolean next()
getString()
```


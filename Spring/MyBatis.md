# [MyBatis](https://mybatis.org/mybatis-3/zh/index.html)

# 流程

- 加载核心配置文件流，生成sqlSessionFactory
- 执行应用时从Factory中获取sqlSession
- 从sqlSession中获取Mapper，执行抽象方法
- 提交事务（增删改），关闭sqlSession将资源归还

# 起步

```java
//从 XML 中构建 SqlSessionFactory并获取SqlSession
private static  SqlSessionFactory sqlSessionFactory;

    static {
        try{
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        }catch (IOException e){
            e.printStackTrace();
        }
    }

    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
```

## 编写持久层

- 实体类

```java
public class User {
    private int id;
    private String name;
    private String pw;

    public User() {
    }

    public User(int id, String name, String pw) {
        this.id = id;
        this.name = name;
        this.pw = pw;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getPw() {
        return pw;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setPw(String pw) {
        this.pw = pw;
    }
}
```

- Dao（Data Access Object）接口

```java
public interface UserMapper {
    User getUserById(String id);
}

```

- 接口实现类由UserDaoImpl转变为Mapper配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.dao.UserMapper">
    <select id="getUserById" resultType="com.example.pojo.User">
        select * from dycTest.user where id = #{id}
    </select>
</mapper>

```

## 注册Mapper

```xml
 <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
```

## Tips

Maven 配置资源过滤，防止target中没有配置文件的问题。

```xml
<build>
        <resources>
            <resource>
                <directory>src/main/resource</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>

        </resources>
    </build>
```

## 生命周期（Scope）和生命周期

理解不同作用域和生命周期类别是至关重要的，因为错误的使用会导致非常严重的并发问题。

#### SqlSessionFactoryBuilder

- 一旦创建了 SqlSessionFactory，就不再需要
- 最佳作用域是方法作用域

#### SqlSessionFactory

- 可以看作数据库连接池
- 应用运行期间不要重复创建
- 作用域是应用作用域（最简单的就是单例模式或者静态单例模式）

#### SqlSession

- 连接到连接池的一个请求
- 使用后应及时关闭
- 线程不安全，不能被共享，最佳作用域是请求或方法作用域

# XML配置

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。 

![image-20220223095608041](D:\project\笔记\Spring\图片\image-20220223095608041.png)

## environments

**MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中**

**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

注意一些关键点:

- 默认使用的环境 ID（比如：default="development"）。
- 每个 environment 元素定义的环境 ID（比如：id="development"）。
- **事务管理器的配置（比如：type="JDBC"）。**
- **数据源的配置（比如：type="POOLED"）。**

## properties

可以在外部文件中配置，也可以在properties的子元素中配置，当命名冲突是优先使用外部文件中的键值对。

![image-20220223102659902](D:\project\笔记\Spring\图片\image-20220223102659902.png)

db.properties

```prop
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://211.83.111.224:3308/dycTest?useSSL=true;useUnicode=true;characterEncoding=UTF-8"
username=root
password=102@uestc
```

## 设置（settings）

cacheEnabled：全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。

lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。

mapUnderscoreToCamelCase：是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。

logImpl：指定 MyBatis 所用日志的具体实现，未指定时将自动查找。

## 类型别名（typeAliases）

类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写。例如：

```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>
```

也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如：

```xml
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

每一个在包 `domain.blog` 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 比如 `domain.blog.Author` 的别名为 `author`；若有注解，则别名为其注解值。见下面的例子：

```java
@Alias("author")
public class Author {
    ...
}
```

常见的 Java 类型内建的类型别名: 原始类型前加下划线，类名首字母小写。

```java
int -> _int
List -> list
```

## 映射器

1. 推荐使用

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
```

2.

接口和配置文件必须同名

接口和他的mapper配置文件必须在同一个包下

```xml
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
```

```xml
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

## 其他

typHandlers

objectFactory

plugins

mybatis-plus 使用插件方式实现二次简化编码。

mybatis-gengerator-core 根据数据库内容自动生成CURD代码。

# XML映射文件（Mapper）

将Pojo(Plain Ordinary Java Object)，利用xml的方式实现Mapper接口中的CURD方法。

元素细节属性见官网。

## select

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

在JDBC中，这样的语句将是：

```java
String selectPerson = "SELECT * FROM PERSON WHERE ID=?";
PreparedStatement ps = conn.prepareStatement(selectPerson);
ps.setInt(1,id);
```

## 使用注解

依然需要在核心配置文件注册mapper

```xml
    <mappers>
        <mapper class="com.example.dao.UserMapper"/>
    </mappers>
```

  

```java
package org.mybatis.example;
public interface BlogMapper {
  @Select("SELECT * FROM blog WHERE id = #{id}")
  //当存在多个参数的时候，必须为参数加上@param注解
  Blog selectBlog(@param("id") int id);
}
```

@param

```
基本类型的参数或者String,需要加上
引用类型不需要加
如果只有一个基本类型，可以不加
在sql中应引用@Param中设定的属性名
```



使用注解来映射简单语句会使代码显得更加简洁，但对于稍微复杂一点的语句，Java 注解不仅力不从心，还会让你本就复杂的 SQL 语句更加混乱不堪。 因此，如果你需要做一些很复杂的操作，最好用 XML 来映射语句。

## insert, update 和 delete

```xml
    <insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
        insert into user (name, pw)
        values (#{name}, #{pw})
    </insert>

    <update id = "updateUser">
        update user set
            name = #{name},
            pw = #{pw}
        where id = #{id}
    </update>

    <delete id = "deleteUser" parameterType="int">
        delete from user where id = #{id}
    </delete>
```

```java
//!!! 增删改需要提交事务， 否则数据库内容不会改变
public void insert(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.insertUser(new User(4, "admin4", "123456"));
        sqlSession.commit();
        sqlSession.close();
    }

    public void update(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.updateUser(new User(6, "admin4", "123"));
        sqlSession.commit();
        sqlSession.close();
    }

    public void delete(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.deleteUser(1);
        sqlSession.commit();
        sqlSession.close();
    }
```

使用map代替pojo对象进行操作

```java
    /**
     * user Map to set random property to want value,
     * It's not standard way, but much more flexible and customizable.
     * @param map
     * @return
     */
    int insertUser2(Map<String, Object> map);

    /**
     * user Map to set random property to want value,
     * especially when the properties need to modify are just small part of all of them.
     * @param map
     */
    void updateUser(Map<String, Object> map);
```

```xml
    <insert id="insertUser2" parameterType="java.util.Map">
        insert into user (id, name, pw)
        values (#{num}, #{userName}, #{passWord})
    </insert>

    <update id = "updateUser" parameterType="java.util.Map">
        update user set
            pw = #{password}
        where id = #{id}
    </update>
```

## 模糊查询

```java
 /**
     * fuzzy query
     * @param value
     * @return List<User>
     */
    List<User> getUserLike(String value);
```

```xml
    <select id="getUserLike" resultType="com.example.pojo.User">
        select * from user where name like "%"#{value}"%"
    </select>
```

**防止sql注入**：在sql拼接中使用通配符, 如`"%"#{value}"%"`



## 结果映射

```xml
#{property,javaType=int,jdbcType=NUMERIC}
```

 JDBC 要求，如果一个列允许使用 null 值，并且会使用值为 null 的参数，就必须要指定 JDBC 类型（jdbcType）。



解决pojo属性和数据库字段名不一致的问题

MyBatis 会在幕后自动创建一个 `ResultMap`，再根据属性名来映射列到 JavaBean 的属性上。如果列名和属性名不能匹配上，可以在 SELECT 语句中设置列别名（这是一个基本的 SQL 特性）来完成匹配。比如：

```xml
    <select id="getUserById" resultType="User">
        select id,name,pw as password from user where id = #{id}
    </select>
```

显式结果集映射

```xml
<mapper namespace="com.example.dao.UserMapper">
    <resultMap id="UserMap" type="User">
        <result column="pw" property="password"/>
    </resultMap>
    <select id="getUserById" resultMap="UserMap">
        select id,name,pw from user where id = #{id}
    </select>
</mapper>
```

## 高级结果映射

个人认为：如果所要映射的pojo中不止含有原始类型和String，包含对象的话，则需要通过依赖注入的方式，用查询到的列信息生成对象赋值给pojo。这才是要使用`association`和`collection`标签的原因。

流程：

​	因为逻辑关系的组合，Student 中包含了一个 Teacher  的指针作为属性，而连表查询的结果，显然无法直接映射到一个包含类指针为成员对象的类，此时要用到含有 `association` 标签的 `resultMap`来完成这一操作。

​	而对于`association`标签来说，它的任务就是将查出的列映射为一个类，所以类似于将查询结果映射为一个`pojo`的写法。

```xml
    <select id="getStudent" resultMap="map1">
        select S.s_id id, S.name s_name, T.name t_name
        from
        STUDENT S, TEACHER T
        where S.t_id = T.t_id;
    </select>

    <resultMap id="map1" type="Student">
        <id property="id" column="id"/>
        <result property="name" column="s_name"/>
        <association property="teacher"  javaType="Teacher">
            <result property="name" column="t_name"/>
        </association>
    </resultMap>

```

以下方法也可以达到相同下过，从Students表中获取外键然后查询Teacher属性映射到结果中。

```xml
    <resultMap id="map1" type="Student">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <association property="teacher" column="t_id" javaType="Teacher" select="getTeacher">
            <id property="id" column="t_id"/>
            <result property="name" column="name"></result>
        </association>
    </resultMap>

    <select id="getStudent" resultMap="map1">
        select s_id id, name, t_id from STUDENT
    </select>

    <select id="getTeacher" resultType="Teacher">
        select * from TEACHER T where T.t_id = #{id};
    </select>
```

当成员变量包含集合时，使用`collection`标签进行映射

```xml
<select id="getStudentList" resultMap="map1">
    SELECT  T.t_id, T.name tName, S.s_id, S.name sName
    FROM STUDENT S, TEACHER T
    WHERE S.t_id = T.t_id AND T.t_id = #{id}
</select>

<resultMap id="map1" type="Teacher">
    <id property="id" column="t_id"/>
    <result property="name" column="tName"/>
    <collection property="students" javaType="List" ofType="Student">
        <id property="id" column="s_id"/>
        <result property="name" column="sName"/>
        <result property="t_id" column="t_id"/>
    </collection>
</resultMap>
```

使用子查询方法，为了防止子查询出现错误，子查询应使用resultMap映射

```xml
    <select id="getStudentList" resultMap="teacherStudent">
        select t_id id, name tName from TEACHER where t_id = #{id}
    </select>

    <resultMap id="teacherStudent" type="Teacher">
        <id property="id" column="id"/>
        <result property="name" column="tName"/>
        <collection property="students"  javaType="list" ofType="Student" select="getStudent" column="id">
        </collection>
    </resultMap>

    <resultMap id="map1" type="Student">
        <id property="id" column="s_id"/>
        <result property="name" column="s_name"/>
        <result property="t_id" column="t_id"/>
    </resultMap>

    <select id="getStudent" resultMap="map1">
        select name s_name, s_id, t_id from STUDENT where t_id = #{id}
    </select>
```

# 动态sql

## if

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```

## choose、when、otherwise

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

## trim、where、set

为了防止下例出现的拼接问题

`trim`的逻辑，如果`trim`内不为空，则保证前缀是`prefix`指定的内容，其次，如果`trim`中的内容前缀是`prefixOverrides`中的内容，则将他们删除。`suffixOverrides`同理。

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

*where* 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。

如果 *where* 元素与你期望的不太一样，你也可以通过自定义 trim 元素来定制 *where* 元素的功能。比如，和 *where* 元素等价的自定义 trim 元素为：

```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ...
</trim>
```

```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

```xml
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```

## foreach

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  <where>
    <foreach item="item" index="index" collection="list"
        open="ID in (" separator="," close=")" nullable="true">
          #{item}
    </foreach>
  </where>
</select>
```

## script

```xml
    @Update({"<script>",
      "update Author",
      "  <set>",
      "    <if test='username != null'>username=#{username},</if>",
      "    <if test='password != null'>password=#{password},</if>",
      "    <if test='email != null'>email=#{email},</if>",
      "    <if test='bio != null'>bio=#{bio}</if>",
      "  </set>",
      "where id=#{id}",
      "</script>"})
    void updateAuthorValues(Author author);
```

## bind

```xml
<select id="selectBlogsLike" resultType="Blog">
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
  SELECT * FROM BLOG
  WHERE title LIKE #{pattern}
</select>
```

## sql

`#{}`是占位符，对应变量会被加上单引号，能防止sql注入。 

`${}`是拼接符，直接拼接不做类型转换，一般不建议，但是需要拼接表名的时候，只能用这个。

```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
```

```xml
<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>
```

# 缓存

## 一级缓存

在应用运行过程中，我们有可能在一次数据库会话中，执行多次查询条件完全相同的SQL，MyBatis提供了一级缓存的方案优化这部分场景，如果是相同的SQL语句，会优先命中一级缓存，避免直接对数据库进行查询，提高性能。

![image-20220226105920702](D:\project\笔记\Spring\图片\image-20220226105920702.png)

每个SqlSession中持有了Executor，每个Executor中有一个LocalCache。当用户发起查询时，MyBatis根据当前执行的语句生成`MappedStatement`，在Local Cache进行查询，如果缓存命中的话，直接返回结果给用户，如果缓存没有命中的话，查询数据库，结果写入`Local Cache`，最后返回结果给用户。具体实现类的类关系图如下图所示。

1. MyBatis一级缓存的生命周期和SqlSession一致。
2. MyBatis一级缓存内部设计简单，只是一个没有容量限定的HashMap，在缓存的功能性上有所欠缺。
3. MyBatis的一级缓存最大范围是SqlSession内部，有多个SqlSession或者分布式的环境下，数据库写操作会引起脏数据，建议设定缓存级别为Statement。

## 二级缓存

![image-20220226105435136](D:\project\笔记\Spring\图片\image-20220226105435136.png)

默认情况下，只启用了本地的会话缓存，它仅仅对一个会话中的数据进行缓存。 要启用全局的二级缓存，只需要在你的 SQL 映射文件中添加一行：

```xml
<cache/>
```

sqlSession提交事务之后，会存入二级缓存。

readOnly（只读）属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓存对象的相同实例。 因此这些对象不能被修改。这就提供了可观的性能提升。而可读写的缓存会（通过序列化，所以如果对象没有实现序列化接口会失败）返回缓存对象的拷贝。 速度上会慢一些，但是更安全，因此默认值是 false。

可以设置：

```xml
<cache readOnly="true"/>
或者返回的类实现序列化接口
```

如果多个SqlSession之间需要共享缓存，则需要使用到二级缓存。开启二级缓存后，会使用CachingExecutor装饰Executor，进入一级缓存的查询流程前，先在CachingExecutor进行二级缓存的查询，具体的工作流程如下所示。

![image-20220226111115936](D:\project\笔记\Spring\图片\image-20220226111115936.png)

二级缓存开启后，同一个namespace下的所有操作语句，都影响着同一个Cache，即二级缓存被多个SqlSession共享，是一个全局的变量。

当开启缓存后，数据的查询执行的流程就是 二级缓存 -> 一级缓存 -> 数据库。

1. MyBatis的二级缓存相对于一级缓存来说，实现了`SqlSession`之间缓存数据的共享，同时粒度更加的细，能够到`namespace`级别，通过Cache接口实现类不同的组合，对Cache的可控性也更强。
2. MyBatis在多表查询时，极大可能会出现脏数据，有设计上的缺陷，安全使用二级缓存的条件比较苛刻。
3. 在分布式环境下，由于默认的MyBatis Cache实现都是基于本地的，分布式环境下必然会出现读取到脏数据，需要使用集中式缓存将MyBatis的Cache接口实现，有一定的开发成本，直接使用Redis、Memcached等分布式缓存可能成本更低，安全性也更高。

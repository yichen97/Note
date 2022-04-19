# `InnoDB`

# MySQL

MySQL实例在系统上表现为一个进程。

![img](D:\project\笔记\数据库\pic\403167-20190116145915277-683033214.jpg)

由图，可以看出MySQL最上层是连接组件。下面服务器是由**连接池**、**管理工具和服务**、**SQL接口**、**解析器**、**优化器**、**缓存**、**存储引擎**、**文件系统**组成。

**连接池**：由于每次连接需要消耗很多时间，连接池的作用就是将这些连接缓存下来，下次可以直接用已经建立好的连接，提升服务器性能。
**管理工具和服务**：系统管理和控制工具，例如备份恢复、MySQL复制、集群等
**SQL接口**：接受用户的SQL命令，并且返回用户需要查询的结果。比如select from就是调用SQL Interface
**解析器**: SQL命令传递到解析器的时候会被解析器验证和解析。解析器是由Lex和YACC实现的，是一个很长的脚本， 

主要功能：
	a . 将SQL语句分解成数据结构，并将这个结构传递到后续步骤，以后SQL语句的传递和处理就是基于这个结构的
	b. 如果在分解构成中遇到错误，那么就说明这个SQL语句是不合理的
**优化器**：查询优化器，SQL语句在查询之前会使用查询优化器对查询进行优化。他使用的是“选取-投影-联接”策略进行查询。
用一个例子就可以理解：

```sql
select uid,name from user where gender = 1;
这个select 查询先根据where 语句进行选取，而不是先将表全部查询出来以后再进行gender过滤
这个select查询先根据uid和name进行属性投影，而不是将属性全部取出以后再进行过滤
将这两个查询条件联接起来生成最终查询结果
```

**缓存器**： 查询缓存，如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。
这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等

## 体系架构

### 后台线程

#### Master Thread

- 异步刷新缓冲池中的数据到磁盘

- 保证数据一致性

#### IO Thread

负责异步IO的回调处理，分为 write, read, insert buffer, log IO Thread。

#### Purge Thread

回收undo页

#### Page Thread

刷新脏页

### 内存

基于磁盘的数据库系统（Disk-base Database）

#### 1. 缓冲池

通过消耗内存空间，弥补磁盘访问速度慢的问题。数据库读页时，会将读到的页置于缓冲池中，若在缓冲池中被命中，直接读取该页。

对于数据库的修改操作，应先修改缓冲池中的页，然后再以一定频率刷新到磁盘上。

![img](D:\project\笔记\数据库\pic\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NhMjFodWFuZw==,size_16,color_FFFFFF,t_70)

#### 2. 管理策略

LRU(Latest Recent Used) 

midpoint insertion Strategy，新读取的页插入到中间点位置，加入中间点是为了防止索引或者扫描操作，使缓冲池中的页被刷出。

`innodb_old_blocks_time` 表示页从被读取到midpoint到可以被加入LRU列表的热端所需的时间。

需要从缓存池中分页时：

- 查询Free是否有空闲页，将空闲页从Free列表删除加入LRU列表

- 否则从LRU列表淘汰页，将该页分配给新页。

**脏页**：缓冲池中被修改而与磁盘上的数据不一样的页，存在于LRU列表中，也存在于FLUSH列表中。

#### 3. 重做日志

记录那些已经提交但尚未持久化的事务。物理日志。

redo log buffer, 不需要很大，因为一般情况下每一秒都会将重做日志刷新到日志文件。

以下情况会将重做日志缓冲中的内容刷新到外部磁盘的重做日志文件中：

1. Master Thread 每一秒发起刷新。
2. 事务提交。
3. 重做日志缓冲池剩余大小小于一半。

#### 4. 回滚日志

记录那些尚未提交，仍在执行中的事务。逻辑日志。

undo log buffer



## CHECKPOINT

**DDL（Data Definition Languages）语句：**数据定义语言，这些语句定义了不同的数据段、数据库、表、列、索引等数据库对象的定义。常用的语句关键字主要包括 create、drop、alter等。

**DML（Data Manipulation Language）语句：**数据操纵语句，用于添加、删除、更新和查询数据库记录，并检查数据完整性，常用的语句关键字主要包括 insert、delete、udpate 和select 等。(增添改查）

**Write Ahead Log** 策略，即当事务提交时，先写重做日志，再修改页。为了满足ACID中 Durability 持久性的要求。

检查点技术的目的：

1. 缩短数据库的回复时间
2. 缓冲池不够用时，将脏页刷新到磁盘
3. 重做日志不可用时，刷新脏页。

**分类**：

- `sharp CheckPoint`: 发生在数据库关闭的时候，将所有的脏页都耍新回磁盘。
- `Fuzzy CheckPoint`:  
  - `Master Tread CheckPoint` : 以一定的时间间隔异步刷新脏页。
  - `FLUSH_LRU_LIST CheckPoint`: 保证有足够的空闲页，移除LRU列表末尾的页，若其中有脏页，那么进行`CheckPoint`。
  - `Async/Sync Flush CheckPoing` : 日志文件不可用，脏页过多，需要从Flush列表中刷新足够的脏页。
  - `Dirty Page Too Much CheckPoint` : 缓冲池中脏页过多。

## `Master Thread`工作方式

## 关键特性

### 插入缓冲
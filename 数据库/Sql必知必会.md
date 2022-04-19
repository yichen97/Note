# Sql必知必会

不提倡将业务逻辑主键作为数据表主键，否则会导致数据库表与业务逻辑耦合，随着业务的变化可能导致需要修改底层代码。

## DISTINCT

```
DISTINCT 关键字会作用于查找的所有列。
```

## ORDER BY

```
ORDER BY 应处于句子的末尾，如 WHERE 的后面
```

### 多列排序

```
ORDER BY 未必只能对所选择的列进行排序，未选择的列也可以。
对于多列进行排序时，如：
ORDER BY column1, column2
会先按照 column1 进行排序，只有在column1顺序相同的时候，才会按照 column2 进行排序。
```

### DESC

```
DESC 仅对前面紧跟着的那一个列名代表的列起效，如：
ORDER BY column1 DESC, column2, column3 DESC
表示，在多列排序的基础上，在针对 column1 和 column3 进行排序的时候采用倒叙排列。
```

## WHERE

```
BETWEEN a AND b 包含边界
IS (NOT) NULL
```

### IN

```
WHERE 字句中用来指定要匹配值的关键字，功能与 OR 相当
WHERE column IN ('A', 'B')
也可以包含其他的 select 语句
```

## 通配符（wildcard）

仅用于文本字段,  将通配符置于开头是耗时的。

```
% 除匹配多个字符外，还能匹配 1 个或 0 个字符。
_ 匹配单个字符。
当某列的长度未被填满时，匹配时应注意尾部可能有填充的空格。
```

## 计算字段

从数据库列数据中映射而来的新字段，配合 alias 使用

如拼接两个列的数据返回。

```java
SELECT CONCAT(RTRIM(column1), '(' , RTRIM(column1), ')') AS A FROM...
```

## 聚集函数（aggregate function）

```
MIN, MAX, AVG, COUNT, SUM
```

GROUP BY & HAVING

SLEECT 中每一列，除聚集函数外 都必须在 GROUP BY 字句中给出。

HAVING 用法类似于 WHERE 用于过滤分组。

## 利用子查询建立计算字段

如果想要查询每位顾客的订单总数，可顾客的id与订单号被记录在另一张order表中：

```sql
SELECT 
CUST_NAME, 
CUST_STATE, 
(SELECT COUNT(*) FROM ORDERS WHERE ORDERS.CUST_ID = COUSTOMERS.CUST_ID) AS ORDERS
FROM CUSTOMERS
ORDER BY CUST_NAME;
```

## 组合查询

- 一个查询从不同的表中返回结构数据

- 对一张表进行多次查询

多次查询的结果会被拼接成一次查询结果，这也意味着，单个查询返回结果的模式应该是相同的，列数据的类型不必完全相同，但至少可以兼容（可以隐式转换）

- union 默认消除了重复行，否则应使用 union all
- 可以在末尾附上 order by进行排序，列名以及别名以第一个查询语句的列名为准

## 插入

插入应给出列名，这保证了语句的正确性，并且可以进行部分插入，但是只有运行为 null 或有默认值的列可以省略。

INSERT  SELECT 语句可以将查询的到列按照顺序插入表中，与SELECT返回的列名无关。

类似的用法还有 CREATE SELECT

## 删除

外键是由 DBMS 提供的保证引用完整新的方案。所以当某行数据连接到另一个表时，删除操作会失败。

## Group by

[group by](https://blog.csdn.net/u014717572/article/details/80687042)

显然使用group by进行查询的时候，只能返回group by所使用的列，返回其他的列则需要使用聚合函数。

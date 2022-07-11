# Redis 核心技术与实战

![70a5bc1ddc9e3579a2fcb8a5d44118b4](D:\project\笔记\数据库\pic\70a5bc1ddc9e3579a2fcb8a5d44118b4.jpeg)

## 基础篇

### 1. 简单的键值数据库内容

基本操作：put、get、delete、scan（根据一段 key 的范围返回相应的 value 值）

构成：访问框架、索引模块、操作模块和存储模块四部分

![img](D:\project\笔记\数据库\pic\30e0e0eb0b475e6082dd14e63c13ed44.jpg)
# Spring面经整理



## 不使用`AutoWired`，如何注入依赖

```
依赖注入的方式主要有构造注入和setter注入，可以使用构造器注入或者@Resource注入
```

[为什么IDEA不推荐你使用`@Autowired`?](https://baijiahao.baidu.com/s?id=1715596620406357296&wfr=spider&for=pc)

## `@Autowired`和`@Resource`的区别是什么？

```java
、、@Autowired 默认byType装配
//使用@Qualifier("user1")改按名称装配，或用@Primary解决类型重复的问题
@Service
public class UserService {

    @Autowired
    @Qualifier("user1")
    private IUser user;
}

//@Resource 默尔维尼byName装配
```

## Spring是如何解决循环依赖

三级缓存解决，单例setter的注入循环依赖问题。

二级缓存足以解决循环依赖，多出的一级是为了配合AOP的需求。

[Spring 是如何解决循环依赖的?](https://bbs.huaweicloud.com/blogs/268055?utm_source=zhihu&utm_medium=bbs-ex&utm_campaign=other&utm_content=content)

## Spring中项目中的实体类

下面以一个时序图建立简单模型来描述上述对象在三层架构应用中的位置：

（1）用户发出请求（可能是填写表单），表单的数据在展示层被匹配为VO。

（2）展示层把VO转换为服务层对应方法所要求的DTO，传送给服务层。

（3）服务层首先根据DTO的数据构造（或重建）一个DO，调用DO的业务方法完成具体业务。

（4）服务层把DO转换为持久层对应的PO（可以使用ORM工具，也可以不用），调用持久层的持久化方法，把PO传递给它，完成持久化操作。

（5）对于一个逆向操作，如读取数据，也是用类似的方式转换和传递。

BO（Business Object）一种被组合出来的DO。
# RabbitMQ

## 消息队列的功能

### 流量削峰

### 降低系统耦合

**应用场景**：用户下单后，订单系统需要调用库存系统的接口。

![image-20220518214632053](D:\project\笔记\消息队列\RabbitMQ\pic\image-20220518214632053.png)

 当库存系统出了问题无法访问时, 订单也会失败。订单系统与库存系统耦合。

![image-20220518214718138](D:\project\笔记\消息队列\RabbitMQ\pic\image-20220518214718138.png)

订单系统：用户下单后，订单系统完成持久化处理，将消息写入消息队列，返回用户订单下单成功。

库存系统：订阅下单的消息,采用pull或push的方式,获取下单信息,库存系统根 据下单信息,进行库存操作.

如果库存系统出问题，不会再影响到订单系统。库存系统正常后再向消息队列订阅消息即可。这就是将原来耦合的两个系统实现了应用解耦。

### 异步处理

**应用场景:** 网站用户注册, 将注册信息写入数据库成功后, 发送注册邮件和注册短信.

![image-20220518214839162](D:\project\笔记\消息队列\RabbitMQ\pic\image-20220518214839162.png)

上图中, 假设注册信息写入数据库需要50ms，不考虑其它开销，发送注册邮件与注册短信是并行的，也需要50ms。那么对于用户来说，响应时间就是共100ms.

![image-20220518214950783](D:\project\笔记\消息队列\RabbitMQ\pic\image-20220518214950783.png)

## RabbitMQ中的概念

- **producer**： producer 是一个发送消息的应用
- **exchange**：producer 并不会直接将消息发送到 queue 上，而是将消息发送给 exchange，由 exchange 按照一定规则转发给指定queue
- **queue**： queue 用来存储 producer 发送的消息
- **consumer**： consumer是接收并处理消息的应用

[图解RabbitMQ](https://zhuanlan.zhihu.com/p/48779080)

## 持久化

队列持久化

消息持久化

## 分发

轮询分发

不公平分发

预取值

## 发布确认

单一确认

批量确认

异步确认

## 交换机

- 生产者指定Message的routing key，并指定Message发送到哪个Exchange
- Queue会通过binding key绑定到指定的Exchange
- Exchange根据对比Message的routing key和Queue的binding key，然后按一定的分发路由规则，决定Message发送到哪个Queue

**每一类Exchange都有自己的分发路由规则：**

**Fanout Exchange**：忽略key对比，发送Message到Exchange下游绑定的所有Queue

**Direct Exchange**：比较Message的routing key和Queue的binding key，完全匹配时，Message才会发送到该Queue

**Topic Exchange**：比较Message的routing key和Queue的binding key，按规则匹配成功时，Message才会发送到该Queue

**default** **Exchange**


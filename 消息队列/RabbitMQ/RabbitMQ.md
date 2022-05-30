# RabbitMQ

## 消息队列的定义

RabbitMQ是实现了高级消息队列协议（AMQP）的开源消息代理软件（亦称为面向消息的中间件）。

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
- **Broker：**可以看做RabbitMQ的服务节点。一般请下一个Broker可以看做一个RabbitMQ服务器。

消息一般包括消息体和标签，标签是用来区分某个消息队列（MQ）Topic下的消息分类，消费者根据标签（tag）的内容来过滤消息。

### 交换机

- 生产者指定Message的routing key，并指定Message发送到哪个Exchange
- Queue会通过binding key绑定到指定的Exchange
- Exchange根据对比Message的routing key和Queue的binding key，然后按一定的分发路由规则，决定Message发送到哪个Queue

**每一类Exchange都有自己的路由规则：**

**Fanout Exchange**：忽略key对比，发送Message到Exchange下游绑定的所有Queue

**Direct Exchange**：比较Message的routing key和Queue的binding key，完全匹配时，Message才会发送到该Queue

**Topic Exchange**：比较Message的routing key和Queue的binding key，按规则（通配符）匹配成功时，Message才会发送到该Queue

**default** **Exchange**

### 分发到消费者

轮询分发

不公平分发

预取值 ： 可以通过设置预取值提高消费者消费能力，缓解消息堆积。

[图解RabbitMQ](https://zhuanlan.zhihu.com/p/48779080)

### 工作模式

**简单模式**

队列到消费者，一对一

**工作模式**

队列到消费者，一对多

**发布订阅模式**

交换机到多个队列，到多个消费者

**路由模式**

交换机路由到多个队列，到多个消费者

**Topic模式**

通配符模式绑定队列，到多个消费者

## 保证可靠性

![image-20220530213020045](D:\project\笔记\消息队列\RabbitMQ\pic\rabbitmq消息流动.png)

### 可能产生丢失的环节

1.  生产者 => 队列  ，RabbitMQ服务器宕机或停止服务
2.  队列中 ， 消息未持久化， RabbitMQ宕机重启会丢失数据
3.  队列 => 消费者， 消息没有被正确消费



### 生产者 => 队列

#### 事务机制

**使用事务机制的话会降低RabbitMQ的性能**

RabbitMQ中与事务机制有关的方法有三个：

txSelect用于将当前`channel`设置成`transaction`模式 (与`confirm`模式互斥)

txCommit用于提交事务

txRollback用于回滚事务

在通过txSelect开启事务之后，我们便可以发布消息给broker代理服务器了，如果txCommit提交成功了，则消息一定到达了broker了，如果在txCommit执行之前broker异常崩溃或者由于其他原因抛出异常，这个时候我们便可以捕获异常通过txRollback回滚事务了。

#### 发布确认

生产者将`channel`设置为confirm模式，所有在该信道上面发布的消息都会被指定一个唯一ID，消息被投递到所有匹配队列后，broker会向生产者发送确认。

![image-20220530213832405](D:\project\笔记\消息队列\RabbitMQ\pic\confirm模式.png)

```yml
spring:
  rabbitmq:
    publisher-confirms: true
#    publisher-returns: true
    template:
      mandatory: true
# publisher-confirms：设置为true时。当消息投递到Exchange后，会回调confirm()方法进行通知生产者
# publisher-returns：设置为true时。当消息匹配到Queue并且失败时，会通过回调returnedMessage()方法返回消息
# spring.rabbitmq.template.mandatory: 设置为true时。指定消息在没有被队列接收时会通过回调returnedMessage()方法退回。

```

单一确认：同步阻塞，一次发一条

批量确认：同步阻塞，批量发布提高吞吐量，需要保存整个批，保证错误时可以重新发布。

异步确认：异步

### MQ防止丢失

#### 持久化

#### 集群镜像备份

![image-20220530222501367](D:\project\笔记\消息队列\RabbitMQ\pic\镜像集群.png)

设置镜像也有一些策略：

- 同步至所有的，一般不这么做，性能会受到极大影响
- 同步最多N个机器
- 只同步至符合指定名称的nodes

### 消费者应答

![image-20220530214533203](D:\project\笔记\消息队列\RabbitMQ\pic\消费者手动确认.png)

#### 自动应答但是处理失败的情况

属于代码异常，可以采取手动确认的方式或者SpringBoot 重试机制

> 很多场景并不是一定要启用消费者应答模式，SpringBoot 给我们提供了一种重试机制，当消费者执行的业务方法报错时会重试执行消费者业务方法。如果消费者中抛出异常，SpringBoot 就会重试。

## 保证顺序性














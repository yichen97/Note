# Netty

## java 多线程

java的main方法是一个主线程

开辟新的线程时，不同的线程在不同的栈空间中执行，CPU按一定策略同时处理多个线程。

### 线程安全

竞态条件（race condition）：当多线程访问和操作同一对象的时候，最终结果和执行时序有关，正确性是不能够人为控制的，可能正确也可能不正确。

内存可见性：内存是一个硬件，执行速度比CPU慢几百倍，所以在计算机中，CPU在执行运算的时候，不会每次运算都和内存进行数据交互，而是先把一些数据写入CPU中的缓存区（寄存器和各级缓存），在结束之后写入内存。这个过程是及其快的，单线程下并没有任何问题。

但是在多线程下就出现了问题，一个线程对内存中的一个数据做出了修改，但是并没有及时写入内存（暂时存放在缓存中）；这时候另一个线程对同样的数据进行修改的时候拿到的就是内存中还没有被修改的数据，也就是说一个线程对一个共享变量的修改，另一个线程不能马上看到，甚至永远看不到。

### 线程创建

1. 继承Thread，重写run方法

```java
public class Main {
    public static void main(String[] args) {
        new MyThread().start();
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "\t" + Thread.currentThread().getId());
    }
}
```

2. 实现Runnable接口，重写run方法，然后使用Thread类来包装

   ```java
   public class Main {
       public static void main(String[] args) {
       	 // 将Runnable实现类作为Thread的构造参数传递到Thread类中，然后启动Thread类
           MyRunnable runnable = new MyRunnable();
           new Thread(runnable).start();
       }
   }
   
   class MyRunnable implements Runnable {
       @Override
       public void run() {
           System.out.println(Thread.currentThread().getName() + "\t" + Thread.currentThread().getId());
       }
   }
   ```

   变体写法：

   ```java
   public class Main {
       public static void main(String[] args) {
           // 匿名内部类
           new Thread(new Runnable() {
               @Override
               public void run() {
                   System.out.println(Thread.currentThread().getName() + "\t" + Thread.currentThread().getId());
               }
           }).start();
   
           // 尾部代码块, 是对匿名内部类形式的语法糖
           new Thread() {
               @Override
               public void run() {
                   System.out.println(Thread.currentThread().getName() + "\t" + Thread.currentThread().getId());
               }
           }.start();
   
           // Runnable是函数式接口，所以可以使用Lamda表达式形式
           Runnable runnable = () -> {System.out.println(Thread.currentThread().getName() + "\t" + Thread.currentThread().getId());};
           new Thread(runnable).start();
       }
   }
   ```

   3. Callable(有返回值需要的时候再来看)

### 线程的状态

![线程状态](图片/线程状态.png)



无限等待状态 WATTING（ex:线程通信）

```java
object method wait() notify() notifyAll()
```

### 线程池

一个线程的循环队列，防止创建大量执行时间很短的线程。

### java锁

为了是不同情形下线程安全，需要加锁。

[java锁-掘金](https://juejin.cn/post/6844903715099377671)

![img](https://user-gold-cdn.xitu.io/2018/11/16/1671b05fcf28435d?imageslim)

#### 一种悲观锁 synchronized

 1. 代码块同步

 2. 方法同步

    普通方法同步：锁对象是该类的实例对象。

    静态方法同步：锁对象是该类的Class文件对象。

#### Lock锁

## java网络编程

### BIO（Blocking I/O）

![img](D:\project\笔记\Netty\图片\BIO通信模型.jpg)

​	采用 **BIO 通信模型** 的服务端，通常由一个独立的 Acceptor 线程负责监听客户端的连接。我们一般通过在 `while(true)` 循环中服务端会调用 `accept()` 方法等待接收客户端的连接的方式监听请求，请求一旦接收到一个连接请求，就可以建立通信套接字在这个通信套接字上进行读写操作，此时不能再接收其他客户端连接请求，只能等待同当前连接的客户端的操作执行完成， 不过可以通过多线程来支持多个客户端的连接，如上图所示。

​	在 Java 虚拟机中，线程是宝贵的资源，线程的创建和销毁成本很高，除此之外，线程的切换成本也是很高的。尤其在 Linux 这样的操作系统中，线程本质上就是一个进程，创建和销毁线程都是重量级的系统函数。如果并发访问量增加会导致线程数急剧膨胀可能会导致线程堆栈溢出、创建新线程失败等问题，最终导致进程宕机或者僵死，不能对外提供服务。

### NIO 

[Java NIO 概览](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247484951&idx=1&sn=0cef67df4b883b198da467c927533316&source=41#wechat_redirect)

NIO的特性

| Nio（New io/ Non-blocking io） | IO     |
| ------------------------------ | ------ |
| 面向缓冲区                     | 面向流 |
| 非阻塞                         | 阻塞   |
| 选择器使的线程消耗更少         |        |



核心组件：Channels 、Buffers 、Selectors



### 黏包半包

黏包：发送端的发送策略。 半包：接收端的接受策略。
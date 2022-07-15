

# Java并发

## 并发编程的挑战

并发可能遇到的问题：

- 线程切换
- 死锁
- 资源限制

## Java并发机制的底层实现原理

### volatile

是轻量级的synchronized，保证共享变量的可见性。

volatile 修饰的变量在执行写操作时, 附加一条包含Lock的指令：

1. 将当前处理器缓存行的数据写回到系统内存。

2. 这个写回内存的操作会使得其他CPU里缓存了该地址的数据无效。

   

   volatile与缓存一致性协议MESI并没有直接关系。

M：被修改的。处于这一状态的数据，只在本CPU中有缓存数据，而其他CPU中没有。同时其状态相对于内存中的值来说，是已经被修改的，且没有更新到内存中。
E：独占的。处于这一状态的数据，只有在本CPU中有缓存，且其数据没有修改，即与内存中一致。
S：共享的。处于这一状态的数据在多个CPU中都有缓存，且与内存一致。
 I：无效的。本CPU中的这份缓存已经无效。

    一个处于M状态的缓存行，必须时刻监听所有试图读取该缓存行对应的主存地址的操作，如果监听到，则必须在此操作执行前把其缓存行中的数据写回内存。
    一个处于S状态的缓存行，必须时刻监听使该缓存行无效或者独享该缓存行的请求，如果监听到，则必须把其缓存行状态设置为I。
    一个处于E状态的缓存行，必须时刻监听其他试图读取该缓存行对应的主存地址的操作，如果监听到，则必须把其缓存行状态设置为S。
所以，在写操作频繁的时候，可能使得CPU频繁放弃缓存行而降低效率，此时**可以填充缓冲行**来解决。

### synchronized

- 对于普通同步方法，锁是当前实例对象
- 对于静态同步方法，锁是当前Class对象
- 对于同步方法块，锁是Synchronized括号里配置的对象

当一个线程试图访问同步代码块时，它首先必须要得到锁，推出或抛出异常时必须要释放锁



#### 锁升级与对比：

![img](D:\project\笔记\操作系统\pic\锁优缺点对比)

![在这里插入图片描述](D:\project\笔记\操作系统\pic\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NDM0NzQy,size_16,color_FFFFFF,t_70#pic_center)

无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态

锁可以升级，但不能降级。

![preview](D:\project\笔记\并发\pic\1.jpg)

##### 偏向锁 

大多数时候，锁不存在线程竞争，而且总是由同一线程多次获得。为了让线程获得锁的代价更低，一如了偏向锁。获取锁时，会在对象头和栈帧中的锁记录里记录偏向的线程ID。

- Mark word中是否存在指向当前线程的偏向锁，测试成功表示已经获得锁。
- 测试失败，查看偏向锁的标识。
- 标识为0，CAS竞争锁。
- 表示为1，CAS将偏向锁指向当前线程。

偏向锁只有竞争出现才会撤销，只有在全局安全点（没有字节码执行时），暂停拥有锁的线程，检查原来持有该对象锁的线程是否依然存活，如果挂了，则可以将对象变为无锁状态，然后重新偏向新的线程，如果原来的线程依然存活，则马上执行那个线程的操作栈，检查该对象的使用情况，如果仍然需要持有偏向锁，则偏向锁升级为轻量级锁，（ **偏向锁就是这个时候升级为轻量级锁的**）。

##### 轻量级锁

轻量级锁认为竞争存在，但是竞争的程度很轻，一般两个线程对于同一个锁的操作都会错开，或者说稍微等待一下（自旋），另一个线程就会释放锁。 但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁膨胀为重量级锁，重量级锁使除了拥有锁的线程以外的线程都阻塞，防止CPU空转。

- JVM会在栈帧中创建用于存储锁记录的空间，并将Mark Word复制到锁记录中。
- 然后线程尝试使用CAS将对象头中的Mark Word更新为指向栈帧中记录的指针。
- 若成功，当前线程获得锁。
- 若失败，其他线程竞争锁，自旋获取锁。

### 原子操作的实现

操作系统实现原子操作：内存操作的原子性、锁总线、缓存一致性协议。

#### Java实现原子操作

1. 循环CAS

​		使用循环CAS实现原子操作，利用了处理器提供的CMPXCHG指令。

​		实现CAS的基础是，比较和替换操作的原子性。

​		CAS存在的问题：

​		ABA问题，通过在变量前追加版本号解决。

​		循环开销过大

​		只能保证一个共享变量的原子性

2. 锁操作

​		除了偏向锁外（与偏向锁的应用范围有关），Java通过CAS获取锁，保证原子性。

## Java内存模型

### 内存模型基础

#### 并发两个关键问题

- 线程间如何通信

- 线程间如何同步

#### Java内存抽象模型

Java为了屏蔽硬件和操作系统访问内存的各种差异，提出了「Java内存模型」的规范，保证了Java程序在各种平台下对内存的访问都能得到一致效果

![img](D:\project\笔记\并发\pic\主存操作)

线程之间的共享变量存储在主内存中，每个线程都有一个私有的本地内存。

### happens-before原则

- 程序顺序原则：一个线程中每个操作，happens-before 于该线程中的任意后续操作。
- 监视器锁原则：对一个锁的解锁，happens-before于随后对这个所得加锁。
- volatile 变量规则：对一个volatile域的写， happens-before 于任意后续对这个volatile域的读。
- 传递性原则
- 如果线程A执行操作`ThreadB.start()`,那么A的`ThreadB.start()`操作先于B线程中所有操作
- 如果A执行操作`ThreadB.join()`，那么B中的所有操作先于A从B中返回。

一个happens-before 规则依靠一个或多个编译器和处理器重排序规则。

### 重排序

数据具有依赖性，happens-before可以保证线程内的正确结果。

### 顺序一致性模型

- 一个线程中所有操作必须按照程序的顺序执行
- （不管线程是否同步）所有线程都只能看到一个单一的操作顺序，每个操作都必须原子执行且立刻对所有线程可见。

### volatile 的内存语义

#### volatile 的特性：

可见性：对于一个volatile 变量的读，总是能看到（任意线程）对这个volatile 变量最后的写入

原子性：对任意单个volatile变量的读/写具有原子性。

从内存语义的角度来说，volatile 写和锁的释放有相同的内存语义，volatile 的读和锁的获取有相同的内存语义。

#### volatile内存语义的实现

就是在volatile「前后」加上「内存屏障」，使得编译器和CPU无法进行重排序，致使有序，并且写volatile变量对其他线程可见。

通过Lock前缀指令来实现的，lock指令能保证：禁止CPU和编译器的重排序（保证了有序性）、保证CPU写核心的指令可以立即生效且其他核心的缓存数据失效（保证了可见性）。

### 锁的内存语义

#### 锁释放和获取的内存语义

当线程释放锁时，JMM 会把线程对应的本地内存中的变量刷新到主存。

当线程获取锁时，JMM会把该线程对应的本地内存置为无效，从而使锝被监视器保护的临界区代码必须从主存中读取共享变量。

#### 锁内存语义的实现

### final 域的内存语义

#### final 域的重排序规则

- 在构造函数内对一个final 域的写入，域随后把这个构造对象的引用复制给一个引用变量，这两个操作之间不能重排序。
- 初次读一个包含final 域的对象的引用，与随后初次都这个final域这两个操作之间不能重排序。

防止final 引用不会从构造函数中溢出。

### 双重检查锁定与延迟初始化

双重检查锁定

对象初始化过程：

1. 为对象分派内存空间

2. 初始化对象

3. 设置指向该对象的指针

方法一 ： 可以使用volatile 禁止2，3的重排序保证双重检查锁定的正确性。

```java
public class SafeDoubleCheckedLocking{
    private volatile static Instance instance;
    
    public static Instance getInstance(){
        if(instance == null){
            synchronized (SafeDoubleCheckedLocking.class){
                if(instance == null){
                    Instance instance = new Instance();
                }
            }
        }
    } 
}
```

方法二：基于类初始化方案，略。

### Java并发编程基础

#### 什么是线程

为什么使用多线程：

- 更多的处理器核心

- 更快的响应时间

- 更好的并发模型

#### 线程的优先级

优先级高的线程相对能获得更多的处理器资源，但优先级不能被作为程序正确运行的依赖。

#### 线程的状态

NEW、RUNNABLE、BLOCKED、WAITING、TIME_WAITING、TERMINATED

![线程状态图](D:\project\笔记\并发\pic\线程状态变迁.jpg)

#### Daemon线程

守护线程，重要被用作程序中后台调度，以及支持性工作。当一个Java虚拟机中不存在非Daemon线程的时候，Java虚拟机将会退出。

### 启动和中止线程

#### 构造线程

运行线程之前首先要构造一个线程对象，属性包括 所属组、线程优先级、是否为Daemon线程等。

#### 启动线程

调用start()方法：同步告知虚拟机，只要线程规划器空闲，应立即调用start()方法的线程。

#### 理解中断

`Thread.interrupt()`方法是一个实例方法，**依靠设置终端标志，通知目标线程终端。如果该线程没有做相关处理，仍会继续运行。**

`Thread.isInterrupted`是一个实例方法，判断当前线程是否被中断。

`Thread.interrupted`静态方法，也可以用来判断当前线程的中断状态，但会同时清楚当前线程的终端标志位状态。

#### 废弃的操作

`stop()`方法由于粗暴的立刻停止线程，不能保证资源的回收而被弃用。

`suspend()`挂起，在被阻塞的同时不释放资源，容易引发死锁。

### 线程间通信

除了中断外，还可以使用一个`volatile boolean`变量来控制是否需要中止该线程。

synchronized本质是对一个对象的监视器的排他获取。任何对象都有对应的监视器。

![img](D:\project\笔记\并发\pic\对象、同步队列、监视器和执行线程之间的关系图.jpg)

#### 等待通知机制

一个线程A在调用了对象O的wait()方法进入等待状态，而另一个线程B调用了对象O的`notify()`或者`notifyAll()`方法，线程A收到通知后从O的`wait()`方法返回，继而执行后续操作。

- 使用wait、`notify()`、`notifyAll()`操作需要先对调用对象加锁。
- `notify()`、`notifyAll()`方法调用后，等待线程依旧不会从wait 返回, 需要调用的`notify()`、`notifyAll()`释放锁之后，等待线程才有机会从wait 返回。这是为了，线程从wait 方法返回时能够感知到通知线程对变量做的修改。

![查看源图像](D:\project\笔记\并发\pic\waitNotify运行过程.jpg)

#### 经典等待通知范式

消费者

1. 获取锁
2. 查询条件不满足则调用wait()，被通知后仍要检查
3. 条件满足执行相应逻辑

```java
synchronized(对象) {
    while(条件不满足) {
        对象.wait()
    }
}
```

生产者

1. 获取锁
2. 改变条件
3. 通知所有等待在对象上的线程

```java
synchronized(对象) {
    改变条件
        对象.notifyAll();
}
```

#### `Threan.join `操作

除 join() 外，还提供具有超时特性的重载。

join的含义是，当前线程中止之后，才从`thread.join`返回。

#### `ThreadLocal` 的使用

一个以Thread 对象为键，任意对象为值得存储结构，为当前线程保存一个值。

### 线程应用实例

#### 等待超时模型

```java
public synchronized Object get(long mills) throws InterruptedException {
    long future = System.currentTimeMills() + mills;
    long remaining = mills;
    while( (result == null) && remaining > 0){
        wait(remaning);
        remaning = future - System.currentTimeMills();
    }
    return result;
} 
```

#### 线程池技术及其示例

减少线程创建和消亡的开销，避免过多线程导致的上下文切换，面对过多的任务可以平缓的劣化。

生产者-消费者问题

```java
class Worker implement Runnable{
    private volatile boolean running = true;
    
    public void run(){
        while(running){
        synchronized(jobs){
            while(jobs.isEmpty()){
                try{
                    worker.wait();
                }catch(InterruptedException e){
                    Thread.currentThread().interrupt();
                    return;
                }
            }
            job = jobs.removeFirst();
        }
        if(job != null){
            try{
                job.run();
            }catch(Exception e){
                ...
            }
        }
       
    }
}
```

#### Web服务器

![查看源图像](D:\project\笔记\并发\pic\simpleHttpSever.jpg)

## Java中的锁

### Synchronized 和 Lock

synchronized, 经过优化，与Lock 的效率差别已经不是讨论的重点。

synchronized关键字隐式地获取锁， 将锁的获取和释放固化了，对于连续保持获取和保持释放多个锁就很难实现了。 Lock显示的获取和释放锁，拥有了锁获取与释放的可操作性,可中断的获取锁以及超时获取锁等多中 synchronized 不具备的功能。

尝试非阻塞的获取锁：尝试获取锁，如果锁没有其他线程持有， 成功获取并持有，获取失败不会被立刻阻塞。

能被中断的获取锁：在**获取锁的过程中**能够响应中断（ lockInterruptibly() ）

超时获取锁：在指定截止时间之前获取锁，超时返回。



Lock 主要API

```java
void lock(): 获取锁，调用该方法当前线程将会获取锁，当锁获得后，从该方法返回。

void lockInterruptibly() throws InterruptedException: 可中断的获取锁，可在获取中响应中断。

boolean tryLock(): 尝试非阻塞地获取锁，调用方法后立即返回，如果能够获取则返回true，否则返回false

boolean tryLock(long time, TimeUnit unit) throws InterruptedException : 超时获取锁，当前线程在以下三种情况会返回
    1. 当前线程在超时时间内获取到了所
    2. 当前线程在超时时间内被中断
    3. 超时时间结束，返回false

void unLock(): 释放锁，总是应该释放锁，
    
Condition newCondition(): 获取等待通知组件，该组件和当前所绑定，当前线程只有获得了锁，才能调用wait() 方法，调用后释放锁。
    调用。
```

### 队列同步器

AbstractQueuedSynchronizier（队列同步器），是用来构建锁和其他同步组建的基本框架，使用一个int型变量表示同步状态，通过内置FIFO队列来完成资源获取线程的排队工作。

同步器的设计基于模板方法，主要实现方法是继承、子类通过继承同步器并实现它的抽象方法来管理同步状态。

修改同步状态的方法：

```java
getState(): 获取当前状态
setState(int newState): 设置同步状态
compareAndSetState(int except, int update):使用CSA设置当前状态
```

同步器**可重写**方法：

```java
boolean tryAcquire(int arg): 独占式获取同步状态
boolean tryRelease(int arg): 独占式释放同步状态
int tryAcquireShared(int arg): 共享式获取同步状态，返回值大于等于0则成功
boolean tryRelease(int arg): 共享式释放同步状态
boolean isHeldExclusively() 当前同步器是否在独占模式下被线程占用
```

同步器**提供的模板方法**：

```java
void acquire(int arg):独占式获取同步状态，成功返回，不成功则进入等待队列。该方法会调用重写的 tryAcquire()
void acquireInterruptibly(int arg): 与acquire(int arg)相同，但是通过首先检测中断标志的方法响应线程的中断
boolean tryAcquireNanos(int arg, long nanos): 在上面方法的基础上增加了超时限制
void acquireShared(int arg): 与独占式的主要区别是，同一时间可以有多个线程获得同步状态
void acquireSharedInterruptibly(int arg): 类上
boolean tryAcquireSharedNanos(int arg, long nanos): 类上
boolean release(int arg):独占式释放同步状态，该方法会在释放同步状态后，将同步队列中的第一个节点包含的线程唤醒
boolean releaseShared(int arg): 
Collection<Thread> getQueuedThread():获取同步队列中等待的集合。
```

### 同步队列实现分析

#### 同步队列

以来一个FIFO双向队列来完成同步状态管理。当前线程获取同步状态失败时，同步器会将当前线程以及等待状态信息构造成一个节点（Node）将其加入同步队列中，同时会阻塞当前线程。当同步状态释放时，会把首节点中的线程唤醒，使其自此尝试获取同步状态。

底层有LockSupport支持

节点属性

```java
waitState:
 0 INITIAL 初始状态
 1 CANCELLED 由于同步队列中等待的线程等待超时或被中断，需要从同步队列中取消等待，节点进入该状态，状态将不会再变化。
-1 SIGNAL 后继节点的线程处于等待状态，如果当前节点释放了同步状态和或者被取消，将会通知后继节点。
-2 CONDITION 节点在等待队列中，调用 signal()方法后会被加入等待队列。
-3 PROPAGATE 表示下一次共享同步状态获取将会被无条件传播下去

 Node prev
 Node next
 Node nextWaiter
 Thread thread
```

首节点是获取同步状态成功的节点，首节点在释放同步状态的时候会唤醒后继节点，后继节点会在获取同步状态成功的时候，将自己设置为首节点。

#### 独占式同步状态获取与释放

acquire 方法

```java
public final void acquire(int arg){
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
        selfInterrupt();
}
```

逻辑：尝试获取同步状态失败，则构造同步节点，加入同步队列。开始进入一个获取同步状态否则就等待的死循环中，等待状态下可以被前驱节点的出队或者被中断来实现。

#### acquireQueued

![img](D:\project\笔记\并发\pic\acquireQueued流程图)

 shouldParkAfterFailedAcquire，根据前驱节点的状态维护队列。

    1、如果当前线程节点的前驱节点为SINGAL状态，则表明当前线程处于等待状态，返回true，当前线程阻塞
    2、如果当前线程节点的前驱节点状态为CANCELLED（值为1），则表明前驱节点线程已经等待超时或者被中断，此时需要将该节点从同步队列中移除掉。最后返回false
    3、如果当前节点节点前驱节点非SINGAL，CANCELLED状态，则通过CAS将其前驱节点的等待状态设置为SINGAL，返回false。

如果shouldParkAfterFailedAcquire方法返回true，则需要调用parkAndCheckInterrupt方法

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    //前驱节点的等待状态
    int ws = pred.waitStatus;
    //当前线程处于等待状态  返回true
    if (ws == Node.SIGNAL)
        return true;
    //说明前驱节点线程等待超时或者被中断，需要将前驱节点从同步队列中移除
    if (ws > 0) {
        do {
            //从同步队列中移除前驱节点
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    //状态为Condition或者propageate
    } else {
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}

```

 当线程释放同步状态后，将唤醒其后继节点：

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            //唤醒后继节点
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

唤醒线程使用unparkSuccessor方法

```java
private void unparkSuccessor(Node node) {
    //头结点的等待状态
    int ws = node.waitStatus;
    //如果头结点的状态小于0（初始状态）
    if (ws < 0)
        //使用CAS操作将头结点的状态设置为0
        compareAndSetWaitStatus(node, ws, 0);
    //头结点的后继节点
    Node s = node.next;
    //当后继节点为空或者等待超时或被中断
    if (s == null || s.waitStatus > 0) {
        s = null;
        //从同步队列尾部开始，找到一个可以唤醒的节点
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    //唤醒后继节点
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

#### 共享锁的获取

申请流程

```java
public final void acquireShared(int arg) {...} // 获取共享锁的入口
# protected int tryAcquireShared(int arg); // 尝试获取共享锁
private void doAcquireShared(int arg) {...} // AQS中获取共享锁流程整合
private Node addWaiter(Node mode){...} // 将node加入到同步队列的尾部
# protected int tryAcquireShared(int arg); // 尝试获取共享锁
private void setHeadAndPropagate(Node node, int propagate) {...} // 设置 同步队列的head节点，以及触发"传播"操作
# private void doReleaseShared() {...} // 遍历同步队列，调整节点状态，唤醒待申请节点
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {...} // 如果获取锁失败，则整理队中节点状态，并判断是否要将线程挂起
private final boolean parkAndCheckInterrupt() {...} // 将线程挂起，并在挂起被唤醒后检查是否要中断线程（返回是否中断）
private void cancelAcquire(Node node) {...} // 取消当前节点获取排它锁，将其从同步队列中移除
```

**doAcquireShared**

```java
private void doAcquireShared(int arg) {
	final Node node = addWaiter(Node.SHARED); // addWaiter()前文有介绍，这里不展开，注意这里是SHARED类型的节点
	boolean failed = true; // 同样一个状态标识，goto t1
	try {
		boolean interrupted = false;
		for (;;) {
			final Node p = node.predecessor(); // 获取前置节点
			if (p == head) {  // 如果为head节点
				int r = tryAcquireShared(arg); // 尝试申请共享锁
				if (r >= 0) { // 申请成功
					setHeadAndPropagate(node, r); // goto t2
					p.next = null; // help GC
					if (interrupted)
						selfInterrupt();
					failed = false;
					return;
				}
			}
			if (shouldParkAfterFailedAcquire(p, node) &&
				parkAndCheckInterrupt())
				interrupted = true;
		}
	} finally {
		if (failed) // t1，是否操作取消节点
			cancelAcquire(node);
	}
}

// t2，设置节点nodde为head
private void setHeadAndPropagate(Node node, int propagate) {
	Node h = head; // Record old head for check below
	setHead(node);
	// 这部分找时间详细分析
	if (propagate > 0 || h == null || h.waitStatus < 0 ||
		(h = head) == null || h.waitStatus < 0) {
		Node s = node.next;
		if (s == null || s.isShared())
			doReleaseShared();
	}
}
```

#### 共享锁的释放

释放流程

```java
public final boolean releaseShared(int arg) {...} // 释放共享锁的入口
# protected boolean tryReleaseShared(int arg); // 尝试释放共享锁
private void doReleaseShared() {...} // 遍历同步队列，调整节点状态，唤醒待申请节点
```

```java
//当前节点竞争到共享变量后，将heaad设置为当前节点，并且在当前节点
//设置为head之时因为当前以shared形式竞争到共享变量,则表示有非共享的进行了释放
//所以可以唤醒后续的共享的等待节点
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    setHead(node);
     //propagate>0表示当前还有信号量可以给后续节点使用
     //waitStatus<0表示其处于propogate或者signal的状态,
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        //这里s为null表示其不知道后续节点是什么，所以也尝试releaseShared
        //不过next节点为空的情况是当新加入节点时时先将新加的节点通过cas设置为tail,然后再修改
        //其前面的节点的next为新加入节点的,所以在这中间,当前节点tail节点时其也可能为空的
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) { //这里表示队列中至少有两个节点
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {//当前head是signal,表示其后继需要unpark,则进行unpark
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            //当前head状态为0,表示其后继节点还没有进行park的,但是需要将当前节点切换为propagate,表示
            //其需要传播released
            else if (ws == 0 && 
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   //h不等于head则表示在当前操作时中间发生了head的变化
            break;
    }
}
```

![](D:\project\笔记\并发\pic\setHeadAndPropagate.jpg)

#### 传播状态(PROPAGATE)

PROPAGATE的产生是源于jdk1.6的bug导致，队列中存在两个以上线程，此时当拥有同步状态的两个线程几乎同时 release 时，由于头节点更新较慢，导致会有一个线程持续挂起。

```java
当第一次 release 发生， 头节点 waitState 变为 0， 并向后唤醒一个节点。 后继节点会尝试将自己设为头节点， 当头节点设置完成前。如果第二次 release 发生，此时头节点 waitState 状态为0， 不会向后唤醒节点。而已经传递的Propagate 为1，也不会继续唤醒，导致一个本该唤醒的节点持续挂起。
```

```java
新版代码会在设置新的头结点时，查看旧头节点的值是否 < 0。判断是否在设置头结点的过程中，有release 命令。如果有也会再次调用doReleaseShared。
```



1.6 bug 代码：

![img](D:\project\笔记\并发\pic\propagate1.6bug.jpg)

```java
现在的版本：
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
		(h = head) == null || h.waitStatus < 0) {
		Node s = node.next;
		if (s == null || s.isShared())
			doReleaseShared();
	}
```



![image-20220411165550511](D:\project\笔记\并发\pic\1.6bug.jpg)

## `ThreadLocal`

ThreadLocal一半作为操作`ThreadLocalMap`中`Entry`的键，是操作线程私有数据的入口。一般设置为弱引用，即在下一次GC时删除。这是为了方式`ThreadLocal`的生命周期与线程一致，对于线程池之类生命周期很长的线程来说尤为致命。但是`Entry`的值依然是强引用，虽然`ThreadLocalMap`会自动清理那些已经被回收的`ThreadLocal`对应的值，但是这种清理依赖于接下来的调用，并不可靠。所以有发生内存泄漏的风险。使用`ThreadLocal`的最佳实践是使用完之后就立即`remove`。


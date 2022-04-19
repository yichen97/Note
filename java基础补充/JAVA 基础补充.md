## JAVA 基础补充

## Lamda、Stream

[掘金]（https://juejin.cn/post/6844904184953700360）

![img](D:\project\笔记\java基础补充\图片\Stream和lamda.jpg)

利用Java.util.funciton 和function.utilcompare对应的接口可以承接来自其他类中的方法引用。

### lambda表达式的格式

```java
//在 Java 中，Lambda 表达式的格式是像下面这样
// 无参数，无返回值
() -> log.info("Lambda")

 // 有参数，有返回值
(int a, int b) -> { a+b }

//二者等价于：
log.info("Lambda");

private int plus(int a, int b){
   return a+b;
}

```



#### 定义并使用一个函数接口

常见的函数接口 `Comparator`、`Runnable`

四个基本的函数式接口：

1. `Function<T, R>` 输入T类型参数，一个返回R类型结果

2. `Consumer<T>` 接受一个T类型输入参数，没有返回参数

3. `Supplier<T>`没有输入参数，返回T类型结果

4. `Predicate<T>`接受一个输入参数，返回true 或者 false

1. 定义了名称为 KiteFunction 的函数式接口，使用 `@FunctionalInterface`注解，然后声明了具有两个参数的方法 `run`，都是泛型类型，返回结果也是泛型。

   还有一点很重要，函数式接口中**只能声明一个可被实现的方法**，你不能声明了一个 `run`方法，又声明一个 `start`方法，到时候编译器就不知道用哪个接收了。而用**`default` 关键字修饰**的方法则没有影响。

```java
@FunctionalInterface
public interface KiteFunction<T, R, S> {

    /**
     * 定义一个双参数的方法
     * @param t
     * @param s
     * @return
     */
    R run(T t,S s);
}
```

2. 定义一个与`KiteFunction`中`run`方法对应的方法

   在 FunctionTest 类中定义了方法 `DateFormat`，一个将 `LocalDateTime`类型格式化为字符串类型的方法。

   ```java
   public class FunctionTest {
       public static String DateFormat(LocalDateTime dateTime, String partten) {
           DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(partten);
           return dateTime.format(dateTimeFormatter);
       }
   }
   ```

3. 用方法引用的方式调用

   正常情况下我们直接使用 `FunctionTest.DateFormat()`就可以了。

   而用函数式方式，是这样的。

   ```java
   KiteFunction<LocalDateTime,String,String> functionDateFormat = FunctionTest::DateFormat;
   String dateString = functionDateFormat.run(LocalDateTime.now(),"yyyy-MM-dd HH:mm:ss");
   ```

   ```java
   public static void main(String[] args) throws Exception {
   
           KiteFunction<LocalDateTime, String, String> functionDateFormat = (LocalDateTime dateTime, String partten) -> {
               DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(partten);
               return dateTime.format(dateTimeFormatter);
           };
           String dateString = functionDateFormat.run(LocalDateTime.now(), "yyyy-MM-dd HH:mm:ss");
           System.out.println(dateString);
   }
   
   ```

   他们之间的关系大概就是：

   lamda => 用于表示方法的语法糖

   function => 用于包装使用方法的接口，输入输出必须一致，并将输入输出以泛型形式标识清楚

### Stream的API

`Collection`接口提供了 `stream()`方法，让我们可以在一个集合方便的使用 Stream API 来进行各种操作。值得注意的是，我们执行的任何操作都不会对源集合造成影响，你可以同时在一个集合上提取出多个 stream 进行操作。

![img](D:\project\笔记\java基础补充\图片\StreamApi.jpg)

#### collect

在进行了一系列操作之后，我们最终的结果大多数时候并不是为了获取 Stream 类型的数据，而是要把结果变为 List、Map 这样的常用数据结构，而 `collection`就是为了实现这个目的。

```java
private static void collect(){
    Stream<Integer> integerStream = Stream.of(1,2,5,7,8,12,33);
    List<Integer> list = integerStream.filter(s -> s.intValue()>7).collect(Collectors.toList());
}
```

`collect`的另一个重载方法，你可以理解为它的参数是按顺序执行的，这样就清楚了，这就是个 ArrayList 从创建到调用 `addAll`方法的一个过程。

```java
private static void collect(){
    Stream<Integer> integerStream = Stream.of(1,2,5,7,8,12,33);
    List<Integer> list = integerStream.filter(s -> s.intValue()>7).collect(ArrayList::new, ArrayList::add,
            ArrayList::addAll);
}
```

正如文档所写的：

The mutable reduction operation is called [`collect()`](../../../java/util/stream/Stream.html#collect-java.util.stream.Collector-), as it collects together the desired results into a result container such as a `Collection`. A `collect` operation requires three functions: a supplier function to construct new instances of the result container, an accumulator function to incorporate an input element into a result container, and a combining function to merge the contents of one result container into another. The form of this is very similar to the general form of ordinary reduction:

```
 <R> R collect(Supplier<R> supplier,
               BiConsumer<R, ? super T> accumulator,
               BiConsumer<R, R> combiner);
```

#### toArray

`collection`是返回列表、map 等，`toArray`是返回数组，有两个重载，一个空参数，返回的是 `Object[]`。另一个接收一个 `IntFunction<R>`类型参数。

比如像下面这样使用，参数是 `User[]::new`也就是new 一个 User 数组，长度为最后的 Stream 长度。

```java
private static void toArray() {
    List<User> users = getUserData();
    Stream<User> stream = users.stream();
    //数组对象可以是类对象（以前居然一直不知道）
    User[] userArray = stream.filter(user -> user.getGender().equals(0) && user.getAge() > 50).toArray(User[]::new);
}

```

#### reduce

它的作用是每次计算的时候都用到上一次的计算结果，比如求和操作，前两个数的和加上第三个数的和，再加上第四个数，一直加到最后一个数位置，最后返回结果，就是 `reduce`的工作过程。

```java
private static void reduce(){
    Stream<Integer> integerStream = Stream.of(1,2,5,7,8,12,33);
    Integer sum = integerStream.reduce(0,(x,y)->x+y);
    System.out.println(sum);
}
```

#### 并行 Stream

Stream 本质上来说就是用来做数据处理的，为了加快处理速度，Stream API 提供了并行处理 Stream 的方式。通过 `users.parallelStream()`或者`users.stream().parallel()` 的方式来创建并行 Stream 对象，支持的 API 和普通 Stream 几乎是一致的。

**什么情况下使用或不应使用并行流操作呢？**

1. 必须在多核 CPU 下才使用并行 Stream，听上去好像是废话。
2. 在数据量不大的情况下使用普通串行 Stream 就可以了，使用并行 Stream 对性能影响不大。
3. CPU 密集型计算适合使用并行 Stream，而 IO 密集型使用并行 Stream 反而会更慢。
4. 虽然计算是并行的可能很快，但最后大多数时候还是要使用 `collect`合并的，如果合并代价很大，也不适合用并行 Stream。
5. 有些操作，比如 limit、 findFirst、forEachOrdered 等依赖于元素顺序的操作，都不适合用并行 Stream。

## `File`类 和Io流

​	File类是`java.io`包下代表`与平台无关`的`文件和目录`。File可以新建、删除、重命名文件和目录，但是不能访问文件内容本身，如果需要访问内容的话，需要通过`输入/输出流`进行访问。

​	File类可以使用文件路径字符串创建File实例，路径既可以是绝对路径，也可以是相对路径。一般相对路径的话是由系统属性`user.dir`指定，即为Java VM所在路径。常用方法分为获取，判断，增删。

[File详解及实践](https://juejin.cn/post/6844903984163979271)

[看完这个，Java IO从此不在难](https://juejin.cn/post/6844903678227267597)

![按字节和字符划分.png](D:\project\笔记\java基础补充\图片\1.jpg)

## 访问权限

| 修饰符     | 类内部 | 同包 | 子类 | 任何地方 |
| ---------- | ------ | ---- | ---- | -------- |
| public     | Yes    | Yes  | Yes  | Yes      |
| protected  | Yes    | Yes  | Yes  |          |
| 包访问权限 | Yes    | Yes  |      |          |
| private    | Yes    |      |      |          |

## 匿名内部类

匿名内部类如何访问在其外面定义的变量：匿名内部类不能访问外部类方法中的局部变量，`除非该变量被声明为final类型` 

 

> 这里所说的“匿名内部类”主要是指在其外部类的成员方法内定义的同时完成实例化的类，若其访问该成员方法中的局部变量，**局部变量必须要被final修饰。原因是编译器实现上的困难：内部类对象的生命周期很有可能会超过局部变量的生命周期。**
>
> 局部变量的生命周期：当该方法被调用时，该方法中的局部变量在栈中被创建，当方法调用结束时，退栈，这些局部变量全部死亡。而内部类对象生命周期与其它类对象一样：自创建一个匿名内部类对象，系统为该对象分配内存，直到没有引用变量指向分配给该对象的内存，它才有可能会死亡（被JVM垃圾回收）。所以完全可能出现的一种情况是：成员方法已调用结束，局部变量已死亡，但匿名内部类的对象仍然活着。
>
> 如果匿名内部类的对象访问了同一个方法中的局部变量，就要求只要匿名内部类对象还活着，那么栈中的那些它要所访问的局部变量就不能“死亡”。
>
> 解决方法：匿名内部类对象可以访问同一个方法中被定义为final类型的局部变量。定义为final后，编译器会把匿名内部类对象要访问的所有final类型局部变量，都拷贝一份作为该对象的成员变量。这样，即使栈中局部变量已经死亡，匿名内部类对象照样可以拿到该局部变量的值，因为它自己拷贝了一份，且与原局部变量的值始终保持一致（final类型不可变）。

## StringBuilder

​	jdk5.0 后加入，此后的字符串拼接编译器自动调用`StringBuilder`, 使用`append `方法拼接，`toString` 方法输出。

​	当你为一个类编写`toString()`方法时，如果字符串操作比较简单，那就可以信赖编译器。但是，如果你要在`toString()`方法中使用循，编译器可能会在循环中多次创建StringBuilder对象。

[你真的了解String和StringBuilder吗?](https://juejin.cn/post/6844903876844322824)

## HashMap

[HashMap为何从头插入改为尾插入](https://juejin.cn/post/6844903682664824845)

## 值传递，引用传递

java的内存模型大体分为 **堆** 和 **栈** （细分还有方法区，和程序计数器等）。

> 1.基本类型的变量放在栈里；
>
> 2.封装类型中，对象放在堆里，对象的引用放在栈里。

对于非对象类型，java 参数传递都是值传递， 比如int.
java 会直接复制一份值到方法参数里面去使用。

而对于对象类型，其实也是值传递，java 参数传递值的是对象的引用，相当于对象在堆里面的内存地址。

## 反射

[好怕怕的类加载器](https://zhuanlan.zhihu.com/p/54693308)

## 泛型

错误转型会出现ClassCastException。

编译器在编译期对相关操作的变量类型进行约束，编译后泛型被擦除。

代码编译时，编译器会自动进行类型转换无需手动强转。

底层还是Object，但是由于被编译器教研过，自动转换不会出错。

利用反射绕过编译器对泛型的检查：

```java
public class GenericClassDemo {

    public static void main(String[] args) throws Exception {

        List<String> list = new ArrayList<>();
        list.add("aaa");
        list.add("bbb");
        // 编译器会阻止
        // list.add(333);

        // 但泛型约束只存在于编译期，底层仍是Object，所以运行期可以往List存入任何类型的元素
        Method addMethod = list.getClass().getDeclaredMethod("add", Object.class);
        addMethod.invoke(list, 333);

        // 打印输出观察是否成功存入Integer（注意用Object接收）
        for (Object obj : list) {
            System.out.println(obj);
        }
    }
}
```

为了使泛型操作更加灵活，提供两个通配符，extends 和 super。

**为保证取出安全，用于接受取出结果的必须为该泛型通配符所表示的范围的上界及以上。**

**对于Extends来说就是目标类，而super来说则是java对象的共同父类Object**

**为保证存入安全，add的类应是该泛型通配符所能表示的下界及以下。**

**在Extends中，没有下界，所以不能add元素，在super中，下界为目标类，所以所有目标类及目标类的子类都可以存入其中。**



而对于extends和super的取舍，《Effective Java》提出了所谓的：**PECS(Producer Extends Consumer Super)**

- 频繁往外读取内容的（向外提供内容，所以是Producer），适合用<? extends T>：extends**返回值稍微精确些，**对调用者友好
- 经常往里插入的（消耗数据，所以是Consumer），适合用<? super T>：super**允许存入**子类型元素

## final关键字

### 修饰引用

- 修饰基本数据类型，则该引用为常量，值无法修改
- 修饰引用数据类型，如对象、数组，则对象、数组本身可修改，但引用值不可修改
- 修饰成员变量，必须立刻赋值，否则无法通过编译

### 修饰方法

可以被继承，不能被重写，可以被重载

### 修饰类

修饰类时，类无法被继承，如String。

## Volatile 关键字

[Java并发编程：volatile关键字解析](https://www.cnblogs.com/dolphin0520/p/3920373.html)

[volatile使用规则](https://www.cnblogs.com/flydoging/p/11157247.html)

通常来说，使用volatile必须具备以下2个条件：

　　**1）对变量的写操作不依赖于当前值**，或者是只能由一个线程修改变量。

　　**2）该变量没有包含在具有其他变量的不变式中**

只有在状态真正独立于程序内其他内容时才能使用 volatile。

使用例子，[Volatile + Double Check](https://blog.csdn.net/faker7/article/details/105622466)

double check保证单例，volatile保证类的完整性。

解决缓存不一致问题：

一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：

　　**1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。**

　　**2）禁止进行指令重排序。**

volatile关键字禁止指令重排序有两层意思：

　　1）当程序执行到volatile变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行；

　　2）在进行指令优化时，不能将在对volatile变量访问的语句放在其后面执行，也不能把volatile变量后面的语句放到其前面执行。

在并发编程中，我们通常会遇到以下三个问题：原子性问题，可见性问题，有序性问题。

要想并发程序正确地执行，必须要保证原子性、可见性以及有序性。只要有一个没有被保证，就有可能会导致程序运行不正确。所以**volatile 并不能保证并发程序运行结果正确，单单解决可见性的问题。而解决可见性，依靠的是类似内存屏障的方式禁止指令重排。**

- Java内存模型只保证了基本读取和赋值是原子性操作，如果要实现更大范围操作的原子性，可以通过synchronized和Lock来实现。

- 当一个共享变量被volatile修饰时，其共享方式依然是通过主存传递。只不过修改的值被**立即写回**（指令意义上的立即, 意味着读取的操作会被排在写入之后）主存，其他缓存中的值失效，必须再从主存读取。

  ```
  “观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令”
  
  　　lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），内存屏障会提供3个功能：
  
  　　1）它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
  
  　　2）它会强制将对缓存的修改操作立即写入主存；
  
  　　3）如果是写操作，它会导致其他CPU中对应的缓存行无效。
  ```

  

- 在Java里面，可以通过volatile关键字来保证一定的“有序性”（具体原理在下一节讲述）。另外可以通过synchronized和Lock来保证有序性，很显然，synchronized和Lock保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。

  　　另外，Java内存模型具备一些先天的“有序性”，即不需要通过任何手段就能够得到保证的有序性，这个通常也称为 happens-before 原则。如果两个操作的执行次序无法从happens-before原则推导出来，那么它们就不能保证它们的有序性，虚拟机可以随意地对它们进行重排序。

    　　下面就来具体介绍下happens-before原则（先行发生原则）：

  - **程序次序规则：一个线程内，按照代码控制流顺序，书写在前面的操作先行发生于书写在后面的操作**
  - **锁定规则：一个unLock操作先行发生于后面对同一个锁lock操作**
  - **volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作**
  - **传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C**
  - 线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作
  - 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
  - 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
  - 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始



## 抽象方法与接口

### 抽象类

在抽象类中，将这一类对象的共性抽取作为抽象类中的属性或者普通方法，个性则由其子类自行重写。

是为了实设计与实现相分离的产物。

- **有抽象方法的类必然是抽象类**
-  **抽象类不可以被实例化，不能被new来实例化抽象类**
-  **抽象类可以包含属性，方法，构造方法，但是构造方法不能用来new实例，只能被子类调用**
-  **抽象类只能用来继承**
-  **抽象类的抽象方法必须被子类继承**

### 接口 

接口中的变量会被隐式地指定为public static final变量，而方法会被隐式地指定为public abstract方法

### 区别

1.语法层面上的区别

　　1）抽象类可以提供成员方法的实现细节，而接口中只能存在public abstract 方法；

　　2）抽象类中的成员变量可以是各种类型的，而接口中的成员变量只能是public static final类型的；

　　3）接口中不能含有静态代码块以及静态方法，而抽象类可以有静态代码块和静态方法；

　　4）一个类只能继承一个抽象类，而一个类却可以实现多个接口。

2.设计层面上的区别

​	抽象类是对一种事物的抽象，即对类抽象，而接口是对行为的抽象。

​	抽象类作为很多子类的父类，它是一种模板式设计。而接口是一种行为规范，它是一种辐射式设计。

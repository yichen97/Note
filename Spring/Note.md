## 常用依赖
```xml
<dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>

    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.7.0</version>
        <scope>test</scope>
    </dependency>
    </dependencies>
```

## 注解说明
是注解生效需要配置扫描和生效
- @Autowired : 首先依据类型查找，如果不行依据名字查找，附加注解@Qulifier(value="xxx")
- @Nullable : 字段可为空
- @Resourece : 与Autowired相反
- @Component : “组件”，注释在类上，表示该类被托管到容器中
  - 衍生注解 @Repostory : 注释Dao层的类，功能相同
  - 衍生注解 @Service : 注释Service层的类，功能相同
  - 衍生注解 @Controller : 注释Controller层的类，功能相同
- @Value : 注释在字段或者set方法上可以注入
- @Scope : 设置作用域类型
- @Configuration : 配置类，相当于beans.xml
  - @ComponentScan : 配置包扫描
  - @Bean : 配置类中注册Bean，写在方法中，方法名相当于id，可以通过arg-beanName来指定bean名，返回类型相当于class属性，返回对象就是要注册的bean
  - @Import : 导入其他的配置类

# 	笔记

## 工厂方法

IOC的底层实现就是对象工厂，依靠反射创建对象，实现进一步解耦。

```java
class UserFactory{
    public static UserDao getDao(){
        String classValue = "class属性值"; //通过对xml的解析得到
        Class clazz = Class.forName(classValue); //通过反射创建对象
        return (UserDao)clazz.newInstance(); //创建并返回所需实例
    }
}
```

在spring中：

`BeanFactory: 一般不用，IOC容器的基本实现，是spring内部使用的接口。（懒加载）`

`ApplicationContext: BeanFactory的子接口，提供更丰富的功能。`

## 工厂Bean

创建工厂类

- 实现接口`factoryBean`
- 重写`getObject`方法。
- 从容器中获得的实例类型将是“产品”实例。

## 动态代理

动态代理：在减少耦合的前提下，扩展功能，**代理类利用反射根据代理的类动态生成**，减少编码。

创造代理对象的途径：

有接口的时候，使用JDK动态代理，创建接口实现类的代理对象。

没有接口的时候，使用CGLIB动态代理，创建子类的代理对象。

### JDK方法

动态代理，动态生成了一个代理类，这个代理类继承了Proxy类，并实现了传入的接口数组。所以可以被向上转型为各个接口使用。

InvocationHandler的存在是为了接受要代理的对象，并增强方法。

最后的代理类，在调用方法时，会调用父类的成员对象InvocationHandler的方法，实现方法的增强。通过反编译可以看到代理类为invoke方法传递参数的细节。



所以创建动态代理的条件应包括，用于生成代理Class对象的 classloader、和代理应持有的接口方法。

调用代理类中以InvocationHandler为参数类型的构造函数，生成动态代理类并向上转型为所需接口实例。

[动态代理](https://www.yuque.com/books/share/2b434c74-ed3a-470e-b148-b4c94ba14535/psmait)

相对于静态代理，动态创建代理类，实现代理。



![img](D:\project\笔记\Spring\图片\1618551293097-75bd5ff8-4e7b-44d6-9c52-54b2d7b5021e.png)

Class对象是在加载中创建的,作为方法区数据的访问入口。

```java
Class Class<T>
	T - the type of the class modeled by this Class object. For example, the type of String.class is Class<String>. Use Class<?> if the class being modeled is unknown.
    Instances of the class Class represent classes and interfaces in a running Java application. An enum is a kind of class and an annotation is a kind of interface. Every array also belongs to a class that is reflected as a Class object that is shared by all arrays with the same element type and number of dimensions. The primitive Java types (boolean, byte, char, short, int, long, float, and double), and the keyword void are also represented as Class objects.
Class has no public constructor. Instead Class objects are constructed automatically by the Java Virtual Machine as classes are loaded and by calls to the defineClass method in the class loader.
```

在对象实例化时，对象实例会被存放在堆空间中，方法区的方法则由所有对象共用。

获得Class对象的方法：

```
1. Class.forName("全限定类名");
2. 类名.getClass();
3. 对象实例.getClass();
```

由上可知，想要获取Class对象，必须有Class对象对应的对象的Class文件，或者实例，也就是说至少得先有代理类才能通过反射创建代理类。



```java
public interface UserDao {

    int add(int a, int b);

    String update(String s);
}
```

```java
public class JdkProxy {
    public static void main(String[] args) {
        UserDao userDao = (UserDao) Proxy.newProxyInstance(ProxyHandler.class.getClassLoader(), new Class<?>[]{UserDao.class}, new ProxyHandler(new UserDaoImpl()));
        userDao.update("123");
    }
}

class ProxyHandler implements InvocationHandler {
        private Object object;
        public ProxyHandler(Object obj){
            this.object = obj;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            Object res = method.invoke(object, args);
            return res;
        }
}

class UserDaoImpl implements UserDao{
    @Override
    public int add(int a, int b) {
        return a + b;
    }

    @Override
    public String update(String s) {
        System.out.println(s);
        return s;
    }
}
```

```java
创建动态代理对象的步骤包括：
1. 创建一个invocationHandler的实现类，该类持有一个UserDaoImpl实例。
    ProxyHandler看起来就是一个静态代理类，用于增强方法。
	InvocationHandler ProxyHandler = new ProxyHandler(new UserDaoImpl());
2. 使用Proxy类中的getProxyClass()方法生成一个动态代理的Class对象，所以需要传入classloader
	Class<Proxy> proxyClass = Proxy.getProxyClass(ProxyHandler.class.getClassLoader(), new Class<?>[] {UserDao.class});
3. 获得代理类的带有InvocationHandler参数的构造器constructor，是继承自Proxy类的构造器。
   因为重载机制的存在，想要获得指定的构造函数，必须传入参数类型，获取方法的写法也类似
    Constructor<?> constructor = proxyClass.getConstructor(InvocationHandler.class);
4. 通过构造器constructor来创建一个代理类对象实例,传入创建的Handler
   因为代理类实现了接口，所以可以直接向上转型并调用方法
    InvocationHandler 以组合的方式存在于Proxy类中,所以下面的步骤相当于，使用一个被包装的被代理类，proxyHandler来构造一个代理类。最终生成的类是一个继承了Proxy的实现所需传入接口的动态代理类。
    UserDao userDao = (UserDao) constructor.newInstance(proxyHandler);
效果等同于：
    UserDao userDao = (UserDao) Proxy.newProxyInstance(ProxyHandler.class.getClassLoader(), new Class<?>[]{UserDao.class}, new ProxyHandler(new UserDaoImpl()));
```

![image-20220407160337126](D:\project\笔记\Spring\图片\image-20220407160337126.png)

![image-20220407160350438](D:\project\笔记\Spring\图片\image-20220407160350438.png)

![3创建类.png](D:\project\笔记\Spring\图片\1.jpg)



![6类图.png](D:\project\笔记\Spring\图片\2.jpg)

[反编译看JDK的动态代理原理](https://blog.csdn.net/weixin_46421629/article/details/106086027)

还有很关键的一点是，从反编译的最终的Proxy类，可以看到，所有方法的执行，都通过接口的invoke方法间接调用调用。而实现handler的类的invoke方法中，调用了被代理类实例的方法，并增加了额外的代码。








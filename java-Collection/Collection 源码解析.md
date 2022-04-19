# 集合

## Java包含的集合类型

​	Java 集合框架主要包括两种类型的容器，一种是集合（Collection），存储一个元素集合，另一种是图（Map），存储键/值对映射。Collection 接口又有 3 种子类型，List、Set 和 Queue，再下面是一些抽象类，最后是具体实现类，常用的有 ArrayList、LinkedList、HashSet、LinkedHashSet、HashMap、LinkedHashMap 等等。

![New集合继承关系图](D:\project\笔记\java基础补充\图片\fb.webapp)

## Collection

```java
package asdf;
/**
 * Created by ping.miao on 2015/7/29.
 *  Collection接口源码分析
 */
import java.util.Iterator;
 
public interface Collection<E> extends Iterable<E> {
    int size();//返回当前集合中元素的个数
    boolean isEmpty();//判断集合是否为空，为空的时候返回true
    boolean contains(Object o);//判断集合中手否包含一个元素与o相等 (o==null ? e==null : o.equals(e))
    Iterator<E> iterator();//产生一个包含该集合所有元素的迭代器
    Object[] toArray();/*
    返回一个包含集合所有元素的数组，这个数组是新产生的一个数组，集合并不对其引用进行维护，换句话说数组的修改
    并不反应在集合上，相当于把集合的元素copy到数组里
    */
    <T> T[] toArray(T[] a);/**同样是返回集合所有元素组成的数组，不同是该方法返回的数组类型是参数指定
     *的数组类型，所以，该方法可能会抛出ArrayStoreException - 指定类型不是此Collection每个元素的运行的超类型
     * NullPointerException - 如果指定的数组为 null
     */
    boolean add(E e);/**插入元素
     * UnsupportedOperationException - 如果此 collection 不支持 add 操作
     * ClassCastException - 如果指定元素的类不允许它添加到此 collection 中
     * NullPointerException - 如果指定的元素为 null，并且此 collection 不允许 null 元素
     * IllegalArgumentException - 如果元素的某属性不允许它添加到此 collection 中
     * IllegalStateException - 如果由于插入限制，元素不能在此时间添加
    */
    boolean remove(Object o);/**
     * 删掉元素o
     * 假如集合中有满足 (o==null ? e==null : o.equals(o)) 的元素，返回true
     * 可能会抛出异常：ClassCastException - 如果指定元素的类型与此 collection 不兼容（可选）
     * NullPointerException - 如果指定的元素为 null，并且此 collection 不允许 null 元素（可选）
     * UnsupportedOperationException - 如果此 collection 不支持 remove 操作
     */
    boolean containsAll(java.util.Collection<?> c);
    boolean addAll(java.util.Collection<? extends E> c);
    boolean removeAll(java.util.Collection<?> c);
    boolean retainAll(java.util.Collection<?> c);
    void clear();
    boolean equals(Object o);
    int hashCode();//产生hashCode
}
```

## 标记接口

[RandomAccess 这个空架子有何用?](https://juejin.cn/post/6844903519066193927)

## ArrayList

ArrayList的底层是一个Object[]数组用于存储元素，当数组内元素已满之后，要再添加元素的时候就会触发ArrayList的扩容机制。

线程不安全的,可以插入null

### 属性

elementData

size

### 构造

**ArrayList 有两个构造函数，分别是无参构造，和int 对象的参数构造。ArrayList的初始长度就是参数携带值的长度**，如果是无参的话，就默认为**10**。

**扩容机制：**当ArrayList内需要添加新元素，但是List满了的话，就会触发这个扩容机制。每次扩容为ArrayList当前数组的1.5倍长度。

- 使用无参的构造函数，会赋予一个默认的值，但是Object[]的长度还是0，要直到放入第一个元素的时候才会将Object[]数组扩容到10
- 使用有参的构造函数，就会直接把Object[]数组创建为对应的数值。

```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
      	//创建一个新的Object[]数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

/**
 * Constructs an empty list with an initial capacity of ten.
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

```

### 扩容

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

//若是第一次加入元素，应扩容到10。
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;//定义在父类AbstractList上，定义了modCount属性，用于记录数组修改次数

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //扩容仍不够，将所需容量直接赋值
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

为什么扩容因子是1.5？

[C++ STL 中 vector 内存用尽后, 为什么每次是 2 倍的增长, 而不是 3 倍或其他值?](https://www.zhihu.com/question/36538542/answer/67994276)

使用 `k=2` 增长因子的问题在于，每次扩展的新尺寸必然刚好大于之前分配的总和，也就是说，之前分配的内存空间不可能被使用。这样对于缓存并不友好。最好把增长因子设为`1 < k < 2`。

## LinkedList

双向链表

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### 添加

```java
//常用的添加元素方法
public boolean add(E e) {
    //使用尾插法
    linkLast(e);
    return true;
}

//在链表尾部添加元素
public void addLast(E e) {
        linkLast(e);
    }

//在链表尾端添加元素
void linkLast(E e) {
    	//尾节点
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
    	//判断是否是第一个添加的元素
        //如果是将新节点赋值给last
        //如果不是把原首节点的prev设置为新节点
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        //将集合修改次数加1
        modCount++;
    }
```

在链表首尾添加元素很高效，在中间添加元素比较低效，首先要找到插入位置的节点，在修改前后节点的指针。

### 删除

```java
 public boolean remove(Object o) {
     //因为LinkedList允许存在null，所以需要进行null判断        
     if (o == null) {
         //从首节点开始遍历
         for (Node<E> x = first; x != null; x = x.next) {
             if (x.item == null) {
                 //调用unlink方法删除指定节点
                 unlink(x);
                 return true;
             }
         }
     } else {
         for (Node<E> x = first; x != null; x = x.next) {
             if (o.equals(x.item)) {
                 unlink(x);
                 return true;
             }
         }
     }
    return false;
 } 

//删除指定位置的节点，其实和上面的方法差不多
	//通过node方法获得指定位置的节点，再通过unlink方法删除
    public E remove(int index) {
        checkElementIndex(index);
       
        return unlink(node(index));
    }

 //删除指定节点
    E unlink(Node<E> x) {
        //获取x节点的元素，以及它上一个节点，和下一个节点
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;
		//如果x的上一个节点为null，说明是首节点，将x的下一个节点设置为新的首节点
        //否则将x的上一节点设置为next，将x的上一节点设为null
        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }
		//如果x的下一节点为null，说明是尾节点，将x的上一节点设置新的尾节点
        //否则将x的上一节点设置x的上一节点，将x的下一节点设为null
        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }
		//将x节点的元素值设为null，等待垃圾收集器收集
        x.item = null;
        //链表节点个数减1
        size--;
        //将集合修改次数加1
        modCount++;
        //返回删除节点的元素值
        return element;
    }
```



## AbstractCollection

```
HashSet调用了AbstractCollection的toArray()方法,这个过程使用了迭代器，这将解释使用List.addAll(Collection<? extends E>)方法加入集合的时候，元素为何顺序的问题。至于迭代器，所有的继承Collection的类都应有这个方法，而由于各实现类索引元素的方式不同。具体实现则有其中的内部类来完成。
```



```java
public <T> T[] toArray(T[] a) {
        // Estimate size of array; be prepared to see more or fewer elements
        int size = size();
        T[] r = a.length >= size ? a :
                  (T[])java.lang.reflect.Array
                  .newInstance(a.getClass().getComponentType(), size);
        Iterator<E> it = iterator();

        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) { // fewer elements than expected
                if (a == r) {
                    r[i] = null; // null-terminate
                } else if (a.length < i) {
                    return Arrays.copyOf(r, i);
                } else {
                    System.arraycopy(r, 0, a, 0, i);
                    if (a.length > i) {
                        a[i] = null;
                    }
                }
                return a;
            }
            r[i] = (T)it.next();
        }
        // more elements than expected
        return it.hasNext() ? finishToArray(r, it) : r;
    }
```

## HashSet

`HashSet`实现了`Set`接口，它的底层是由`HashMap`来支持的。`HashSet`的元素实际上是存储在底层`HashMap`的`key`上的。由于`HashMap`的无序不重复特性，`HashSet`存储的元素也是无序的，并且元素也不能重复，同时也只允许存储一个`null`元素。

主要属性：

```java
// HashSet底层map
private transient HashMap<E,Object> map;
// 虚拟对象
private static final Object PRESENT = new Object();
```

`HashSet`是通过`HashMap`来保存元素，由于只需要在`key`中保存，所以采用虚拟对象`PRESENT`对应`map`中插入`key-value`的`value`值的引用。每次向`map`中添加元素时，键值对对应的`value`都是`PRESENT`。

增删改查直接通过操作`map`完成。

## HashMap

实现 Map 接口， 键值对映射集合，键不允许重复。 

HashMap = 哈希 + 数组 + 链表/红黑树

### 哈希表什么时候使用红黑树，为什么使用

jdk8 相对于 jdk7， 在链表长度大于8之且数据总量超过 64 时，链表会转为红黑树。

之所以设置为8，是由于哈希冲突的概率符合泊松分布，哈希冲突超过8的概率足够小。

使用红黑树替代了链表，红黑树搜索时间复杂度是 O(logn)，而链表是糟糕的 O(n)。

使用红黑树的原因是不平衡的二叉查找树的性能会降低，红黑树是一种可以维持树的平衡性的数据结构，当然维护红黑树需要一定的开销，所以没有直接在哈希冲突的时候使用红黑树。

### 为什么默认负载因子为0.75

默认负载因子（0.75）在时间和空间成本上提供了很好的折衷。较高的值会降低空间开销，但提高查找成本（体现在大多数的HashMap类的操作，包括get和put）。设置初始大小时，应该考虑预计的entry数在map及其负载系数，并且尽量减少rehash操作的次数。如果初始容量大于最大条目数除以负载因子，rehash操作将不会发生。

### 索引计算方式

```java
static final int hash(Object key) {   
     int h;
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    /* 
     h = key.hashCode() 为第一步：取hashCode值
     h ^ (h >>> 16)  为第二步：高位参与运算
    */
}
```

> JDK1.8 为什么要 hashcode 异或其右移十六位的值？

JDK1.8 优化了高位运算的算法，通过hashCode()的高16位异或低16位实现：(h = k.hashCode()) ^ (h >>> 16)。这么做可以在数组 table 的 length 比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销。

> 为什么 hash 值要与length-1相与？

- 把 hash 值对数组长度取模运算，模运算的消耗很大，没有位运算快。
- 当 length 总是 2 的n次方时，h& (length-1) 运算等价于对length取模，也就是 h%length，但是 & 比 % 具有更高的效率。

> HashMap数组的长度为什么是 2 的幂次方？

这样做效果上等同于取模，在速度、效率上比直接取模要快得多。除此之外，2 的 N 次幂有助于减少碰撞的几率。如果 length 为2的幂次方，则 length-1 转化为二进制必定是11111……的形式，在与h的二进制与操作效率会非常的快，而且空间不浪费。

### 计算数组容量

如果传入的数不是二的次幂, n-1 是为了不使二的次幂被进位，如1000 => 1111 => 10000

```java
 public static int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= (n >>> 1);
        n |= (n >>> 2);
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return n + 1;
    }
```

### put方法流程

1. 首先根据 key 的值计算 hash 值，找到该元素在数组中存储的下标；
2. 如果数组是空的，则调用 resize 进行初始化；
3. 如果没有哈希冲突直接放在对应的数组下标里；
4. 如果冲突了，且 key 已经存在，就覆盖掉 value；
5. 如果冲突后，发现该节点是红黑树，就将这个节点挂在树上；
6. 如果冲突后是链表，判断该链表是否大于 8 ，如果大于 8 并且数组容量小于 64，就进行扩容；如果链表节点大于 8 并且数组的容量大于 64，则将这个结构转换为红黑树；否则，链表插入键值对，若 key 存在，就覆盖掉 value。

> key 可以为 Null 吗?

可以，key 为 Null 的时候，hash算法最后的值以0来计算，也就是放在数组的第一个位置。

一般用Integer、String 这种不可变类当 HashMap 当 key，而且 String 最为常用。

- 因为字符串是不可变的，所以在它创建的时候 hashcode 就被缓存了，不需要重新计算。这就是 HashMap 中的键往往都使用字符串的原因。
- 因为获取对象的时候要用到 equals() 和 hashCode() 方法，那么键对象正确的重写这两个方法是非常重要的,这些类已经很规范的重写了 hashCode() 以及 equals() 方法。

> 用可变类当 HashMap 的 key 有什么问题?

hashcode 可能发生改变，导致 put 进去的值，无法 get 出。

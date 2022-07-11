# Redis

![img](D:\project\笔记\数据库\pic\70a5bc1ddc9e3579a2fcb8a5d44118b4.jpeg)

## 键值对数据库

![image-20220414194541998](D:\project\笔记\数据库\pic\redis数据库结构.png)

redis数据库的组织结构是一个字典，形成了键值对数据库的存储结构。值得数据类型可以是String、list、set、Zset、Hash等等，这些数据类型的实现又和不同的底层数据结构有关。

数据类型：存储数据的逻辑数据结构

数据结构：底层实现数据结构

![image-20220415104212848](D:\project\笔记\数据库\pic\数据类型与数据结构.png)

# 数据结构与对象

## 简单动态字符串

（simple dynamic string，SDS）

可由于保存字符串，或二进制信息。

### 结构

```c
struct sdsdr{
	int len;
	int free;
    char buff[];
}
```

### 与c字符串的关系

### 区别

- **常数时间获取字符串长度**
- **杜绝缓冲区溢出**：空间不足自动扩展
- **减少修改字符串时的内存重分配次数**：从c串N次修改至少N次降低至最多N次
  1. 预分配 小于1M 多分配等大空间，大于1M多分配1M空间。
  2. 惰性空间释放：避免缩短字符串立刻释放空间的消耗。
- **二进制安全**：c串除末尾外不能出现空串，SDS不以空字符为结尾
- **兼容部分c函数**：接上条，虽然不以空字符结尾，但会在末尾补上空字符，使得可以兼容部分c函数。

## 链表

List的实现之一

### 特点

双向、无环、带头尾指针、带长度技术器

提供了自定义函数dup、free、match函数，使得该链表可以存储任意数据类型。

### 结构

略

### 缺点

- 相对于压缩列表，内存不连续不能很好地利用CPU缓存

- 保存一个链表节点的值，都需要额外分配给节点结构头。

## 压缩列表

List 和 哈希表 的底层实现之一

适用于元素少，且元素小的场合。

### 特点

是一种内存**紧凑**的**顺序型**数据结构，占用连续的内存空间，不仅可以利用CPU缓存，而且会针对不同长度的数据进行相应的编码，可以有效节省内存开支 。

### 结构

![image-20220415112319139](D:\project\笔记\数据库\pic\压缩列表结构.png)

```c
zlbytes : 占用内存总字节数
zltail : 表尾指针偏移的字节数，指向末尾节点entryN
zllen : 节点数
entry : 节点数据结构
zlend : 末尾标记
```

```c
previous_entry_length: 记录前一个节点的长度，也就是说可以由后向前查找
encoding: 记录了当前节点的数据类型和长度
data/content: 数据（字节数组或者整数）
```

`prevlen`，根据前一个entry的长度是否大于254字节，分为1字节和5字节。也就是说，当修该元素导致长度越过临界值的时候，可能会导致下一个元素的长度调整。而这种调整有可能越过临界值而导致下一个元素的调整，从而引发连锁反应。

在实际使用中，连锁更新发生该类很小，并且压缩列表的元素较少较小，即使发生连锁更新影响也不大。

### 缺点

- 为了保证查询效率，只能保存少了数据

- 新增或修改元素时，压缩列表占用的内存空间需要重新分配，甚至可能引发连锁更新。

## 哈希表

Redis采用链式哈希的方式解决哈希冲突。

### 结构

一个字典带两个哈希表，一个存东西，一个rehash

![image-20220415113754876](D:\project\笔记\数据库\pic\dict.png)

```c
typedef struct dictht{
    dictEntry **table;
    unsigned long size;	
    unsigned long sizemask; //哈希表大小掩码
    unsigned long used;
}dictht;

typedef struct dictEntry{
    void *key;
    
    union{
        void *val;
        unit64_t u64;
        int64_t s64;
    } v; //保存数据的联合体（整数或者指针）
    
    struct dictEntry *next;
} dictEntry;
```

### rehash

1. 扩容 ：`ht[1]`是第一个大于等于`ht[0].used * 2` 的 2 的 n 次幂。
2. 压缩： `ht[1]`是第一个大于等于`ht[0].used` 的 2 的 n 次幂。

扩容时机：未指向持久化时，负载因子超过1，持久化时，附在因子超过5

扩容方式：渐进式

1. 为`ht[1]`分配空间，让字典同时持有`ht[0]`和`ht[1]`
2. 在字典中维持一个索引变量`rehashindex`，并将它的值设置为0，表示rehash工作正式开始
3. 在`rehash`进行期间，每次对字典进行CURD时，程序除执行指定操作外，还会顺带将`ht[0]`哈希表在`rehashindex`索引上的所有键值对rehash到`ht[1]`, 当rehash工作完成后，程序rehashindex属性值增一。
4. `ht[0]`为空后，将`rehashindex`的值设置为-1，表示操作完成。

rehash过程中：

CURD等操作会在两个表上一起完成，此外新增加的节点一律加到`ht[1]`中，保证`ht[0]`中的结点不会增加。

## 跳表

**是有序集合的一种实现方式，支持范围查找。增删改查复杂度平均均为 `logn`，效率不逊于AVL树和红黑树。**

**在插入节点时， 由随机数决定是否向上插入索引节点，平均层高为`1/(1-p)`。**

https://juejin.cn/post/6910476641990868999#heading-5

**逻辑上，可以仍可以看作是一个一个链表，只不过每个链表节点包含着一个指向后续节点的索引集合。**

所以，实现上可以为每个节点维护一个索引集合，也可以以索引节点的方式实现。

下面给出一种简单实现，为了简化， 以`int`作为键，以`Integer.MAX_VALUE` 和 `Integer.MIN_VALUE`来填充`head`和`tail`作为哨兵节点。

```java
public class SkipList{
    private final int MAX_HEIGHT = 32;
    int size = 0;
    int height = 1;
    Random random;
    SkipListNode head;
    SkipListNode tail;

    public static void main(String[] args) {
        SkipList skipList = new SkipList();
        skipList.add(new SkipListNode(1, "1"));
        skipList.add(new SkipListNode(2, "2"));
        skipList.add(new SkipListNode(7, "7"));
        skipList.add(new SkipListNode(8, "8"));
        skipList.add(new SkipListNode(24, "24"));
        skipList.add(new SkipListNode(5, "5"));
        skipList.add(new SkipListNode(10, "10"));
        skipList.add(new SkipListNode(4, "4"));
        skipList.print();
        System.out.println(skipList.get(10).value);
        skipList.add(new SkipListNode(10, "11"));
        System.out.println(skipList.get(10).value);


    }
	//初始化跳表
    SkipList(){
        //设置哨兵节点，保证遍历的时候不会出现空指针
        this.head = new SkipListNode(Integer.MIN_VALUE, "null");
        this.tail = new SkipListNode(Integer.MAX_VALUE, "null");
        head.right = tail;
        this.random = new Random();
    }

    public SkipListNode get(int key){
        SkipListNode p = this.head;
        //之所以还要判断p!=null 是因为向下搜索是没有哨兵节点的
        while(p != null && p.key!= tail.key){
            //搜索到key, 返回节点，之所以直接返回节点是为了add相同key的时候修改方便。
            if(p.key == key){
                return p;
            }else{
                //指针移动只有两个方向，因为有哨兵的关系，可以直接判断右侧的key
                if(p.right.key > key){
                    p = p.down;
                }else{
                    p = p.right;
                }
            }
        }
        return null;
    }

    public void delete(int key){
       SkipListNode p = head;
        while(p != null && p.key!= tail.key) {
            //由于head节点的存在，保证目标节点只会出现在右侧，出现则删除，并开始向下遍历，下指针为空则遍历完成。
            if (p.right.key == key) {
                    p.right = p.right.right;
                    p = p.down;
            } else if (p.right.key > key) {
                    p = p.down;
            } else {
                    p = p.right;
            }
        }
    }

    public void add(SkipListNode node){
        //能否找到key相同的节点，找到就更改，否则集合大小+1；
        SkipListNode findValue = get(node.key);
        if(find == null){
            size++;
        }else{
            //节点已存在可以直接将值修改并返回
            find.value = node.value;
            return;
        }
        
        SkipListNode p = head;
        int key = node.key;
        //使用栈存储应该被修改的节点
        Deque<SkipListNode> stack = new ArrayDeque<>();
        
        while(p != null && p.key != tail.key){
            if(p.key == key){
                break;
            }
            if(p.right.key > key){
                //从最高层的前导节点开始
                stack.push(p);
                p = p.down;
            }else{
                p = p.right;
            }
        }
        
        //当前修改所在层
        int level = 1;
        //存储加入过的节点作为下节点
        SkipListNode downNode = null;
        
        while(!stack.isEmpty()){
            p = stack.pop();
            //每个插入的节点要重新创建
            SkipListNode newNode = new SkipListNode(node.key, node.value);
            //插入
            newNode.right = p.right;
            p.right = newNode;
            newNode.down = downNode;
            downNode = newNode;
            //level 超过限制
            if(level > MAX_HEIGHT){
                break;
            }else{
                //判断是否再建立高一层的索引
                double num = random.nextDouble();
                if(num > 0.5){
                    break;
                }else{
                    level++;
                }
            }
            //当前层高超过原有的最高高度，应新建一层
            if(level > height){
                //创建新的头尾节点，并将下指针指向旧的头尾节点
                SkipListNode newHead = new SkipListNode(head.key, null);
                SkipListNode newTail = new SkipListNode(tail.key, null);
                newHead.down = head;
                newHead.right = newTail;
                newTail.down = tail;
                head = newHead;
                tail = newTail;
                //新建一层，此时栈必定为空，将head入栈
                stack.push(head);
            }
        }
        //保存高度
        height = Math.max(level, height);
    }

    public void print(){
        SkipListNode p = head;
        while(p != null){
            SkipListNode level = p;
            while(level != null){
                System.out.print(level.key + " ->");
                level = level.right;
            }
            System.out.println();
            p = p.down;
        }
    }

    static class SkipListNode{
        int key;
        Object value;
        SkipListNode right;
        SkipListNode down;

        SkipListNode(int key, Object value){
            this.key = key;
            this.value = value;
            this.right = null;
            this.down = null;
        }
    }
}

```

## 整数集合

整数集合是集合键的底层实现之一，当一个集合只包含整数值元素，并且这个集合元素不多时，使用整数集合作为底层实现。

底层是数组实现（有序、无重复）

```c
typedef struct intset{
    //编码方式 数组中的数据将按照该方式读取
    unit32_t encoding;
    //集合包含的元素数量
    unit32_t length;
    //保存元素的数组 
    int8_t contents[];
} intset;
```

### 升级

步骤：

1. 根据新元素的类型，扩展底层数组空间大小，为新元素分配空间

2. 将底层数组现有的所有元素都转换成与新元素相同的类型，将元素置于对应位置
3. 将新元素添加到底层数组里面

## 对象

结构

```c
typedef struct redisObject{
    // 类型
    unsigned type:4;
    // 编码
    unsigned encoding:4;
    // 指向底层数据结构的指针
    void *ptr;
} robj;
```

**类型**：字符串、列表、哈希、集合、有序集合

**底层数据结构**：SDS、压缩列表、双向链表、哈希表、整数集合、跳表

# 数据库

## 键删除

### 定时删除

优点：内存友好，能及时释放空间

缺点：对CPU时间不友好，过期键较多时会占用大量CPU时间，影响吞吐量和响应时间。时间时间的实现是无序链表，在大量时间事件中查找删除效率低。

### 惰性删除

优点：CPU时间友好

缺点：内存不友好，有可能造成一种几乎与内存泄露的内存占用，即很多过期键长时间未被访问。

### 定期删除

前两者的折衷。

每个一段时间，执行一次删除过期键操作，并通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响。

实际使用中结合使用惰性删除和定期删除。


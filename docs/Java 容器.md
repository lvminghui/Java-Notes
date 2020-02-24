# 一 概览
容器的思维导图：
<div align="center"> <img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/imgs/1.png"/> </div><br>


## Collection

### List

Java 的 List 是非常常用的数据类型。List 是有序的 Collection。Java List 一共三个实现类：
分别是 ArrayList、Vector 和 LinkedList。

* ArrayList：ArrayList 是最常用的 List 实现类，内部是通过数组实现的，它允许对元素进行快速随机访问。数
  组的缺点是每个元素之间不能有间隔，当数组大小不满足时需要增加存储能力，就要将已经有数
  组的数据复制到新的存储空间中。当从 ArrayList 的中间位置插入或者删除元素时，需要对数组进
  行复制、移动、代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。
* Vector：Vector 与 ArrayList 一样，也是通过数组实现的，不同的是它支持线程的同步，即某一时刻只有一
  个线程能够写 Vector，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此，
  访问它比访问 ArrayList 慢。
* LinkList：LinkedList 是用链表结构存储数据的，很适合数据的动态插入和删除，随机访问和遍历速度比较慢。另外，他还提供了 List 接口中没有定义的方法，专门用于操作表头和表尾元素，可以当作**堆**
  **栈、队列和双向队列**使用。

### Set 

Set 注重独一无二的性质,该体系集合用于存储无序(存入和取出的顺序不一定相同)元素，**值不能重**
**复**。对象的相等性本质是对象 hashCode 值（java 是依据对象的内存地址计算出的此序号）判断
的，如果想要让两个不同的对象视为相等的，就必须覆盖 Object 的 hashCode 方法和 equals 方
法。

* HashSet：基于哈希表实现，存入数据是按照哈希值，所以并不是按照存入的顺序排序，为保证存入的唯一性，存入元素哈希值相同时，会使用 equals 方法比较，如果比较出不同就放入同一个哈希桶里。
* TreeSet：基于红黑树实现，支持有序性操作，每增加一个对象都会进行排序，将对象插入的二叉树指定的位置。
* LinkHashSet（ HashSet+LinkedHashMap ）：继承于 HashSet、又是基于 LinkedHashMap 来实现的， 具有 HashSet 的查找效率 。

### Queue

## Map

1. HashMap： 根据键的 hashCode 值存储数据，大多数情况下可以直接定位到它的值，因而具有很快
   的访问速度，但遍历顺序却是不确定的。 HashMap 非线程安全，底层实现为数组+链表+红黑树。
2. ConcurrentHashMap：**支持并发操作的HashMap**，使用Segment实现线程安全。
3. HashTable：Hashtable 是遗留类，不建议使用，很多映射的常用功能与 HashMap 类似，线程安全的。
4. TreeMap： 红黑树实现，可排序。
5. LinkHashMap： 是 HashMap 的一个子类，保存了记录的插入顺序，在用 Iterator 遍历 LinkedHashMap 时，先得到的记录肯定是先插入的，也可以在构造时带参数，按照访问次序排序。



# 二 源码分析

## ArrayList

### 1.定义

``` java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

 ArrayList 是基于数组实现的，所以支持快速随机访问。 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。 

**RandomAccess** 接口标识着该类支持快速随机访问（只是一个定义了类型的接口，无作用）。  **Cloneable 接口**，即覆盖了函数 clone()，**能被克隆**。  **java.io.Serializable 接口**，这意味着ArrayList**支持序列化**，**能通过序列化去传输**。 

 数组的默认大小为 10。 

### 2.扩容机制

**核心方法：**

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

 整个流程就是对旧数组位移运算得到新数组，然后把旧数组整个复制到新数组上，操作代价很高，新容量的大小为 `oldCapacity + (oldCapacity >> 1)`，也就是旧容量的 1.5 倍。

补充：

**移位运算符简介**：移位运算符就是在二进制的基础上对数字进行平移。按照平移的方向和填充数字的规则分为三种:<<(左移)、>>(带符号右移)和>>>(无符号右移)。**作用**：**对于大数据的2进制运算,位移运算符比那些普通运算符的运算要快很多,因为程序仅仅移动一下而已,不去计算,这样提高了效率,节省了资源**  。

### 3.添加和删除

**在末尾添加元素：**

```java
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

这里看到ArrayList在末尾添加元素的实质就相当于为数组赋值。

**在指定位置添加元素：**

```java
public void add(int index, E element) {    
    rangeCheckForAdd(index);   	
    ensureCapacityInternal(size + 1);  // Increments modCount!! 
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);    
    elementData[index] = element;    size++;}
```

 让数组自己复制自己实现让index开始之后的所有成员后移一个位置。

**删除指定元素：**

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
```

流程是把需要删除的元素右边的元素向左移动一位，覆盖了需要删除的元素。调用了arraycopy，所以操作代价也很高。
## CopyOnWriteArrayList

concurrent 并发包下的类，是ArrayList的线程安全解决方案， 通过ReentrantLock获取对象锁的方式来实现线程安全。  

**读写分离的特点**

读：

```java
@SuppressWarnings("unchecked")
private E get(Object[] a, int index) {
    return (E) a[index];
}
```

写：

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

final void setArray(Object[] a) {
    array = a;
}
```

写的操作需要加锁，防止并发写入导致数据丢失，不直接操作原数组，先copy一个数组进行操作，写完后setArray方法把新的复制数组赋值给旧数组。

 CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。 

缺点：

* 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右； 

* 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。 

所以 CopyOnWriteArrayList 不适合内存敏感以及对实时性要求很高的场景。 

## LinkedList

#### 1. 概览

内部私有类Node：

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
```

定义：

```java
transient Node<E> first;
transient Node<E> last;
```

 补充：**transient**关键字标记的成员变量不参与序列化过程。 

综上可看出，LinkedList是由双向列表实现，使用Node存储节点信息，每个节点都有前节点（next），本节点（item），后节点（prev）。

#### 2.与 ArrayList 的比较

* ArrayList 基于动态数组实现，LinkedList 基于双向链表实现。 
* ArrayList插入删除元素时分两种情况：①把元素添加到末尾所需的时间复杂度是O(1)，②在指定位置添加删除元素时就需要移动整个数组，操作代价就比较大了。LinkedList 对于添加删除方法只需要断链然后更改指向，所需的消耗就很小了。
* ArrayList 支持高效的随机元素访问 而LinkedList 不支持。
* ArrayList的空 间浪费主要体现在在list列表的结尾会预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗比ArrayList更多的空间（因为要存放直接后继和直接前驱以及数据）。 

## HashMap

#### 1.整体原理分析 

 HashMap类中有一个非常重要的字段，就是  Node[] table，即哈希桶数组 

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
```



 Node是HashMap的一个内部类，实现了Map.Entry接口，本质是就是一个映射(键值对)。 

hash:hashcode经过扰动函数得到的值， 然后通过 `(n - 1) & hash` 判断当前元素存放的位置，如果当前

位置存在hash和key值不相同的元素就使用拉链法解决冲突。

**“拉链法”** 就是：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。 

Node<K,V> next：next就是用于链表的指向。

所谓**扰动函数**指的就是 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法 换句话说使用扰动函数之后可以减少碰撞。 

底层存储结构：
<div align="center"> <img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/imgs/2.png"/> </div><br>


#### 2.put方法分析


map.put("a","b")的整个流程：

 	1. 先判断散列表是否没有初始化或者为空，如果是就扩容
 	2. 根据键值 key 计算 hash 值，得到要插入的数组索引 
 	3. 判断要插入的那个数组是否为空：
      	1. 如果为空直接插入。
      	2. 如果不为空，判断 key 的值是否是重复（用 equals 方法）：
           	1. 如果是就直接覆盖
           	2. 如果不重复就再判断此节点是否已经是红黑树节点：
                	1. 如果是红黑树节点就把新增节点放入树中
                	2. 如果不是，就开始遍历链表：
                     	1. 循环判断直到链表最底部，到达底部就插入节点，然后判断是否大于链表长度是否大于8：
                          1. 如果大于8就转换为红黑树
                          2. 如果不大于8就继续下一步
                    	2. 到底部之前发现有重复的值，就覆盖。
	4. 判断是否需要扩容，如果需要就扩容。

下面上源码：

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    //tab: 引用当前hashMap的散列表
	//p: 表示当前散列表的元素
	//n: 表示散列表数组的长度
	//i: 表示路由寻址结果
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table未初始化或者长度为0，进行扩容（采用了延时初始化，第一次put才会初始化散列表。）
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 寻址找到的桶位为null
    //(n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 寻址找到的桶位已经存在元素
    else {
        Node<K,V> e; K k;
        // 比较桶中第一个元素(数组中的结点)的hash值相等，key相等，也就是判断是否是重复的值。
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                // 完全一致
            	//e:找到的一个与当前要插入的元素一直的元素
                e = p;
        // hash值不相等，即key不相等；且为红黑树结点
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 为链表结点
        else {
            // 在链表最末插入结点
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部
                if ((e = p.next) == null) {
                    // 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    // 结点数量达到阈值，转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 表示在桶中找到key值、hash值与插入元素相等的结点
        if (e != null) { 
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //替换
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
} 
```

综上put的方法流程图为：
<div align="center"> <img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/imgs/put方法流程图.png"/> </div><br>


tip： JDK1.7之前的put方法和现在流程不同的地方就是采用头插法插入元素。

#### 3.扩容（resize方法）

扩容会伴随着一次重新hash分配，并且会遍历hash表中所有的元素，是非常耗时的。在编写程序中，要尽量避免resize。 源码在下面，仔细阅读即可理解此方法。

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    //散列表已经被初始化了，是一次正常扩容
    if (oldCap > 0) {
        // 超过最大值就不再扩充了
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold：翻倍
    }
     //散列表没有被初始化
    else if (oldThr > 0) // 初始化的容量是已经指定了的
        newCap = oldThr;
    else { 
        // 默认初始化容量
        newCap = DEFAULT_INITIAL_CAPACITY;//16
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);//12
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            //当前桶位有数据，但不清楚是什么数据。
            if ((e = oldTab[j]) != null) {
                //方便 GC
                oldTab[j] = null;
                //如果是单个元素
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                //如果是红黑树
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { //桶位已经形成链表
                    Node<K,V> loHead = null, loTail = null;//低位链表
                    Node<K,V> hiHead = null, hiTail = null;//高位链表
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引，给低位链表赋值
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap，给高位链表赋值
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到桶里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到桶里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

在扩容时看原hash值新增的那个bit位是1还是0就好了，是0的话索引没有变，是1的话索引变成“原索引+oldCap（旧数组大小）”，下图位resize（）方法示意图： 
<div align="center"> <img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/imgs/3.png"/> </div><br>


## ConcurrentHashMap

**JDK1.7：**

 ConcurrentHashMap 和 HashMap 实现上类似，最主要的差别是 ConcurrentHashMap 采用了分段锁（Segment），每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数）。  **Segment 继承自 ReentrantLock ，默认的并发级别为 16 。** 

简单理解就是，ConcurrentHashMap 是一个 Segment 数组，Segment 通过继承
ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每
个 Segment 是线程安全的，也就实现了全局的线程安全。

结构如图：
<div align="center"> <img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/imgs/concurrentHashMap7.png"/> </div><br>

**JDK1.8：**

使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用内置锁 synchronized。  synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升N倍。 


## HashSet 

实现原理： HashSet底层由HashMap实现 ，值存放于HashMap的key上 ，HashMap的value统一为PRESENT 。

检查重复： 先对插入的元素的hashcode值和现有的元素的hashcode作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现，直接插入。但是如果发现有相同hashcode值的对象，这时会调用`equals（）`方法来检查hashcode相等的对象是否真的相同。  如果两者相同，HashSet就不会让加入操作成功 。

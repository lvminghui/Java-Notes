# 一 概览
容器的思维导图：
<div align="center"> <img src="https://github.com/lvminghui/Java-Notes/blob/master/docs/imgs/1.png"/> </div><br>


## Collection

#### List

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

#### Set 

Set 注重独一无二的性质,该体系集合用于存储无序(存入和取出的顺序不一定相同)元素，**值不能重**
**复**。对象的相等性本质是对象 hashCode 值（java 是依据对象的内存地址计算出的此序号）判断
的，如果想要让两个不同的对象视为相等的，就必须覆盖 Object 的 hashCode 方法和 equals 方
法。

* HashSet：基于哈希表实现，存入数据是按照哈希值，所以并不是按照存入的顺序排序，为保证存入的唯一性，存入元素哈希值相同时，会使用 equals 方法比较，如果比较出不同就放入同一个哈希桶里。
* TreeSet：基于红黑树实现，支持有序性操作，每增加一个对象都会进行排序，将对象插入的二叉树指定的位置。
* LinkHashSet（ HashSet+LinkedHashMap ）：继承于 HashSet、又是基于 LinkedHashMap 来实现的， 具有 HashSet 的查找效率 。

#### Queue

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


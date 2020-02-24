# Java基础面试题

## Java 语言有哪些特点/什么是Java？

1. 简单易学；
2. 面向对象（封装，继承，多态）；
3. 平台无关性（ Java 虚拟机实现平台无关性）；
4. GC实现垃圾回收；
5. 异常处理机制；
6. 支持多线程；
7. 支持网络编程并且很方便；
8. 编译与解释并存；

## 面向对象和面向过程的区别

- **面向过程** ：**面向过程性能比面向对象高。** 因为类调用时需要实例化，开销比较大，比较消耗资源，所以当性能是最重要的考量因素的时候，比如单片机、嵌入式开发、Linux/Unix等一般采用面向过程开发。但是，**面向过程没有面向对象易维护、易复用、易扩展。**
- **面向对象** ：**面向对象易维护、易复用、易扩展。** 因为面向对象有封装、继承、多态性的特性，所以可以设计出低耦合的系统，使系统更加灵活、更加易于维护。但是，**面向对象性能比面向过程低**。

## Java和C++的区别?

- 都是面向对象的语言，都支持封装、继承和多态
- Java 不提供指针来直接访问内存，程序内存更加安全
- Java 的类是单继承的，C++ 支持多重继承；虽然 Java 的类不可以多继承，但是接口可以多继承。
- Java 有自动内存管理机制，不需要程序员手动释放无用内存
- **在 C 语言中，字符串或字符数组最后都会有一个额外的字符‘\0’来表示结束。但是，Java 语言中没有结束符这一概念。**

## Java有几种基本数据类型

八种：**byte**,short,**long**,**int**,char,**float,double,boolean**

## 基本类型和引用类型？他们的区别

除了8中基本数据类型都是引用类型， 为了面向对象操作的一致性，每种数据类型都有对应的包装类。

**不同点**：

* 赋值方法不同，基本类型直接赋值，引用类型通过 new 创建对象，然后再把对象赋予相应的变量。
* 比较方面的不同，== 号的比较：引用类型比较的是引用地址，基本类型比较的是值
* 在数据做为参数传递的时候，基本数据类型是值传递，而引用数据类型是引用传递（地址传递）。 

* **分别放在 JVM 的哪里？**

基本数据类型在被创建时，在栈上给其划分一块内存，将数值直接存储在栈上（也不完全一定）。

而引用数据类型在被创建时，首先要在栈上给其引用（句柄）分配一块内存，而对象的具体信息都存储在堆内存上，然后由栈上面的引用指向堆中对象的地址。

**引用类型的创建过程：**

现在为其创建一个对象MyDate d1 = new MyDate(8,8,2008);

在内存中的具体创建过程是：

1）首先在栈内存中位其d1分配一块空间；

2）然后在堆内存中为MyDate对象分配一块空间，并为其三个属性设初值0，0，0；

3）根据类MyDate中对属性的定义，为该对象的三个属性进行赋值操作；

4）调用构造方法，为三个属性赋值为8，8，2008；（注意这个时候d1与MyDate对象之间还没有建立联系）

5）将MyDate对象在堆内存中的地址，赋值给栈中的d1;通过句柄d1可以找到堆中对象的具体信息。 

## 重载和重写的区别

#### 重载

发生在同一个类中，方法名必须相同，参数类型不同、个数不同、顺序不同，方法返回值和访问修饰符可以不同。

#### 重写

重写是子类对父类的允许访问的方法的实现过程进行重新编写,发生在子类中，方法名、参数列表必须相同，返回值范围小于等于父类，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类。另外，如果父类方法访问修饰符为 private 则子类就不能重写该方法。**也就是说方法提供的行为改变，而方法的外貌并没有改变。**

## Java 面向对象编程三大特性: 封装 继承 多态

### 封装

把描述一个对象的属性和行为的代码封装在一个类中，属性用变量定义，行为用方法进行定义，方法可以直接访问同一个对象中的属性。 

### 继承

子类继承父类的特征和行为。子类可以有父类非私有的方法，属性。子类也可以对父类进行扩展，也可以重写父类的方法。缺点就是提高代码之间的耦合性。 

### 多态

多态是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定(比如：向上转型，只有运行才能确定其对象属性)。方法覆盖和重载体现了多态性。 

## final 在 java 中有什么作用？

- final 修饰的类叫最终类，该类不能被继承。
- final 修饰的方法不能被重写。
- final 修饰的变量不可更改，**其不可更改指的是其引用不可修改，对于引用类型值还是可能改变的，举个列子：String 内部对于 value 的定义；而对于基本类型来说就叫做常量了。** 

**final、finally、finalize 有什么区别？**

- final可以修饰类、变量、方法，修饰类表示该类不能被继承、修饰方法表示该方法不能被重写、修饰变量表示该变量是一个常量不能被重新赋值。
- finally一般作用在try-catch代码块中，在处理异常的时候，通常我们将一定要执行的代码方法finally代码块中，表示不管是否出现异常，该代码块都会执行，一般用来存放一些关闭资源的代码。
- finalize是一个方法，属于Object类的一个方法，而Object类是所有类的父类，该方法一般由垃圾回收器来调用，当我们调用System的gc()方法的时候，由垃圾回收器调用finalize(),回收垃圾。 

## String StringBuffer 和 StringBuilder 的区别是什么?

**线程安全**

 `StringBuilder`是线程不安全的，效率较高；而`StringBuffer`是线程安全的，效率较低。

**性能**

每次对 String 类型进行改变的时候，都会生成一个新的 String 对象，然后将指针指向新的 String 对象。StringBuffer 每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 StringBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

**数据可变和不可变**

1. `String`的值不可变的。
2. `StringBuffer`和`StringBuilder`的值是可变的，底层使用的是可变字符数组：`char[] value;`

**使用场景**

* 如果需要操作少量的数据用 String
* 单线程操作字符串缓冲区的情况下操作大量数据使用 StringBuilder  
* 多线程操作字符串缓冲区 下操作大量数据使用 StringBuffer 

##  String 是如何实现不可变的？

首先要明确：**String 不可变的是字符串的值不变。**

让我们看 String 源码：

```java
private final char value[];
```

从源码来看, String 类内部是用 char 数组来保存字符串的值, 并且 char[] 是 final 的, 这里的 final 意味着什么呢? 

- value 必须在构造时为其赋值
- 赋值后 value 的引用不能再变

 当我们实例化一个 String 对象并得到其引用后, 构造已经结束了, 即 value 的引用已经不能再变了 。 那么 value 的值呢, 理论上是可以改变的, 只要我们拿到 value 的引用, 可以直接通过下标改变他的值 。

然而，因为 String 并没有提供接口来改变 value 的值，所以value 的值我们从 String 外部获取不到，也改变不了。这才是String 才是不可变的真正原因，并不仅仅是使用 final 修饰了 value 数据。

补充：然而，并不是真正的完全不能获取，利用反射可以直接获取类内部属性。

## String 为什么设置为不可变？

* 为了实现字符串常量池(只有当字符是不可变的，字符串池才有可能实现) 
*  为了线程安全(字符串自己便是线程安全的) 
* 为了保证同一个对象调用 hashCode() 都产生相同的值，String 设置为不可变可以对这个条件有很好的支持，这也是 Map 类的 key 使用 String 的原因。





### Exception、Error、运行时异常与一般异常有何异同 /java 异常体系

**所有的异常都是从Throwable继承而来的** 

**Error**是错误，对于所有的编译时期的错误以及系统错误都是通过Error抛出的。 

 **Exception** 它规定的异常是程序本身可以处理的异常。 

**checked exception**可检查的异常，这是编码时非常常用的，所有checked exception都是需要在代码中处理的。它们的发生是可以预测的，正常的一种情况，可以合理的处理。比如IOException，或者一些自定义的异常。除了RuntimeException及其子类以外，都是checked exception。

**Unchecked Exception**

RuntimeException及其子类都是unchecked exception。比如NPE空指针异常，除数为0的算数异常ArithmeticException等等，这种异常是运行时发生，无法预先捕捉处理的。比如 NullPointerException ， SQLException， NumberFormatException ， FileNotFoundException， NoSuchMethodException。







## 接口和抽象类的区别

1. 接口的方法默认是 public，所有方法在接口中不能有实现(Java 8 开始接口方法可以有默认实现），而抽象类可以有非抽象的方法。
2. 接口中除了static、final变量，不能有其他变量，而抽象类中则不一定。
3. 一个类可以实现多个接口，但只能实现一个抽象类。接口自己本身可以通过extends关键字扩展多个接口。
4. 接口方法默认修饰符是public，抽象方法可以有public、protected和default这些修饰符
5. 从设计层面来说，抽象是对类的抽象，是一种模板设计，而接口是对行为的抽象，是一种行为的规范。

## Object类有哪些常用的方法？

equals 方法，hashCode 方法，toString 方法，wait 和 notify 系列的几个， getclass

### == 和 equals 的区别

 == 对于基本类型来说是值比较，对于引用类型来说是比较的是引用；而 equals 默认情况下是 `==`比较，只是很多类重写了 equals 方法，比如 String、Integer 等把它变成了值比较，所以一般情况下 equals 比较的是值是否相等。 

## hashCode 与 equals (重要)

### hashCode（）介绍

hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个 int 整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。

散列表存储的是键值对(key-value)，它的特点是：能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！（可以快速找到所需要的对象）

### 为什么要有 hashCode

**我们先以“HashSet 如何检查重复”为例子来说明为什么要有 hashCode：** 当你把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与其他已经加入的对象的 hashcode 值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 `equals()`方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。这样我们就大大减少了 equals 的次数，相应就大大提高了执行速度。

通过我们可以看出：`hashCode()` 的作用就是**获取哈希码**，也称为散列码；它实际上是返回一个int整数。这个**哈希码的作用**是快速确定该对象在哈希表中的索引位置。

**`hashCode() `在哈希表中才有用，在其它情况下没用**。

### hashCode（）与equals（）的相关规定

1. 如果两个对象相等，则hashcode一定也是相同的
2. 两个对象相等,对两个对象分别调用equals方法都返回true
3. 两个对象有相同的hashcode值，它们也不一定是相等的
4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

**哪些场景下，子类需要重写 equals 方法和 hashCode 方法？**  

需要判断两个对象状态的相等性的时候。

### 为什么要重写 hashcode( ) 还要重写 equals( ) ？反之亦可问。

重写equals方法是为了按我们自己的想法来比较两个对象是否相等。如果不重写hashCode方法，可能出现**具有相同含义的不同对象**（他们的hashCode不同）的情况。而如果只重写 hashCode 不重写 equals 方法，因为 equals其实就是 == ，只是判断两个对象是否是同一个对象，所以不能得到我们想要的结果。所以需要同时重写equals和hashCode方法，目的是为了准确定位到我们期望的key。

**在 hashmap 中考虑：**

**通过阅读源码得知，在 hashMap 的 put 方法中，寻址找到的桶位如果上面已经有元素了，就判断 hash 值是否相同的同时也要通过 equals 判断（equals 是判断 map 的key 值），都为 true 才覆盖原来的值。如果只重写其中任意一个就会造成值的重复。 **

通俗点的解释

hashcode就类似 门牌号，小区非常大但定位你住哪里告诉门牌号就可以，非常快速定位到（非常像组数下标）

equals就是找到门牌号后需要比较里面具体的房间，少一个都不可以。

## Java序列化中如果有些字段不想进行序列化，怎么办？

对于不想进行序列化的变量，使用transient关键字修饰。

transient关键字的作用是：阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被transient修饰的变量值不会被持久化和恢复。transient只能修饰变量，不能修饰类和方法。

ArrayList 中存储数据的数组 elementData 是用 transient 修饰的，因为这个数组是动态扩展的，并不是所有的空间都被使用，因此就不需要所有的内容都被序列化。通过重写序列化和反序列化方法，使得可以只序列化数组中有内容的那部分数据。 

### 幂等性

幂等（idempotent、idempotence）是一个数学与计算机学概念，常见于抽象代数中，即f(f(x)) = f(x).简单的来说就是一个操作多次执行产生的结果与一次执行产生的结果一致。有些系统操作天生就具有幂等性例如数据库的select语句，但更多时候是需要程序员来做保证的，尤其是在分布式系统环境中，接口能不能做到保证幂等性对系统的影响可能是非常大的，例如很常见的支付下单等场景，由于分布式环境中网络的复杂性，用户误操作，网络抖动，消息重复，服务超时导致业务自动重试等等各种情况都可能会使线上数据产生了不一致，造成生产事故。 



## 怎么防止前端重复提交？

1. 提交按钮后屏蔽提交按钮(前端js控制)
2. 前端生产唯一id， 后端在数据库设计时业务字段加唯一约束，防止数据库插入重复数据。 
3. 利用Session防止表单重复提交

**ToDoList：**

描述下 HashMap get 方法的主要执行逻辑和流程；

Java 异常，什么是 checked Exception 和 unchecked Exception，举几个具体的例子；是否研究过 Spring Boot 中的异常；

ConcurrentHashMap 的特性和实现原理；

什么是分库分表，以及分库分表的具体方法和使用场景；

数据库事务的 ACID ；

什么是分布式锁以及其实现原理和使用场景；


# == 和 equals 的区别
## == 解读
对于基本类型和引用类型 == 的作用效果是不同的，如下所示：
基本类型：比较的是值是否相同；
引用类型：比较的是引用是否相同；
## equals
Object类的equals方法：
```java
 public boolean equals(Object obj) {
        return (this == obj);
    }
```
可以看出其实就是==
而String类中重写了父类Object的equals方法：
```java
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```
可以看出此方法就是先使用==比较，如果不同再把对象转换为字符串逐一字符比较。
**总结** ：== 对于基本类型来说是值比较，对于引用类型来说是比较的是引用；而 equals 默认情况下是 `==`比较，只是很多类重写了 equals 方法，比如 String、Integer 等把它变成了值比较，所以一般情况下 equals 比较的是值是否相等。
# 缓存池和字符串常量池
## 缓存池
基本类型的valueOf() 方法会调用缓存池比较值的大小：
```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```
判断值是否在缓存池中，如果在的话就直接返回缓存池的内容，不在就新建一个。
Integer 缓存池的大小默认为 -128~127。
编译器会在自动装箱过程调用 valueOf() 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象。
```java
Integer m = 123;
Integer n = 123;
System.out.println(m == n); // true
```
基本类型对应的缓冲池如下：
boolean values true and false
all byte values
short values between -128 and 127
int values between -128 and 127
char in the range \u0000 to \u007F
在 jdk 1.8 所有的数值类缓冲池中，Integer 的缓冲池 IntegerCache 很特殊，这个缓冲池的下界是 - 128，上界默认是 127，但是这个上界是可调的，在启动 jvm 的时候，通过 -XX:AutoBoxCacheMax=<size> 来指定这个缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界。
## 字符串常量池
**字符串常量池 String Pool**
保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中。
当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。
如果是采用 "bbb" 这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。

### String s = new String(“abc”) 会创建几个对象
使用这种方式一共会创建两个字符串对象（前提是 String Pool 中还没有 “abc” 字符串对象）。
“abc” 属于字符串字面量，因此编译时期会在 String Pool 中创建一个字符串对象，指向这个 “abc” 字符串字面量；
而使用 new 的方式会在堆中创建一个字符串对象。

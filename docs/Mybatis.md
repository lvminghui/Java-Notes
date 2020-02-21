



# MyBatis 整体架构

MyBatis 最上面是接口层，接口层就是开发人员在 Mapper 或者是 Dao 接口中的接口定义，是查询、新增、更新还是删除操作；中间层是数据处理层，主要是配置 Mapper -> XML 层级之间的参数映射，SQL 解析，SQL 执行，结果映射的过程。上述两种流程都由基础支持层来提供功能支撑，基础支持层包括连接管理，事务管理，配置加载，缓存处理等。

![image-20200220202650971](C:\Users\吕明辉\AppData\Roaming\Typora\typora-user-images\image-20200220202650971.png)

## 接口层

在不与 Spring 集成的情况下，使用 MyBatis 执行数据库的操作主要如下：

```java
InputStream is = Resources.getResourceAsStream("myBatis-config.xml");
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(is);
sqlSession = factory.openSession();
```

其中的`SqlSessionFactory`,`SqlSession`是 MyBatis 接口的核心类，尤其是 SqlSession，这个接口是 MyBatis 中最重要的接口，这个接口能够让你执行命令，获取映射，管理事务。

#### **数据处理层**

- **配置解析**

在 Mybatis 初始化过程中，会加载 `mybatis-config.xml` 配置文件、映射配置文件以及 Mapper 接口中的注解信息，解析后的配置信息会形成相应的对象并保存到 `Configration` 对象中。之后，根据该对象创建 SqlSessionFactory 对象。待 Mybatis 初始化完成后，可以通过 SqlSessionFactory 创建 SqlSession 对象并开始数据库操作。

- **SQL 解析与 scripting 模块**

Mybatis 实现的动态 SQL 语句，几乎可以编写出所有满足需要的 SQL。

Mybatis 中 scripting 模块会根据用户传入的参数，解析映射文件中定义的动态 SQL 节点，形成数据库能执行的 SQL 语句。

- **SQL 执行**

SQL 语句的执行涉及多个组件，包括 MyBatis 的四大核心，它们是: `Executor`、`StatementHandler`、`ParameterHandler`、`ResultSetHandler`。SQL 的执行过程可以用下面这幅图来表示

![image-20200220202531623](C:\Users\吕明辉\AppData\Roaming\Typora\typora-user-images\image-20200220202531623.png)



#### **基础支持层**

- 反射模块

Mybatis 中的反射模块，对 Java 反射进行了很好的封装，提供了简易的 API，方便上层调用，并且对反射操作进行了一系列的优化，比如，缓存了类的 `元数据（MetaClass）`和对象的`元数据（MetaObject）`，提高了反射操作的性能。

- 类型转换模块

Mybatis 的别名机制，能够简化配置文件，该机制是类型转换模块的主要功能之一。类型转换模块的另一个功能**是实现 JDBC 类型与 Java 类型的转换**。在 SQL 语句绑定参数时，会将数据由 Java 类型转换成 JDBC 类型；在映射结果集时，会将数据由 JDBC 类型转换成 Java 类型。

- 日志模块

在 Java 中，有很多优秀的日志框架，如 Log4j、Log4j2、slf4j 等。Mybatis 除了提供了详细的日志输出信息，还能够集成多种日志框架，其日志模块的主要功能就是集成第三方日志框架。

- 资源加载模块

该模块主要封装了类加载器，确定了类加载器的使用顺序，并提供了加载类文件和其它资源文件的功能。

- 解析器模块

该模块有两个主要功能：一个是封装了 `XPath`，为 Mybatis 初始化时解析 `mybatis-config.xml`配置文件以及映射配置文件提供支持；另一个为处理动态 SQL 语句中的占位符提供支持。

- 数据源模块

Mybatis 自身提供了相应的数据源实现，也提供了与第三方数据源集成的接口。数据源是开发中的常用组件之一，很多开源的数据源都提供了丰富的功能，如连接池、检测连接状态等，选择性能优秀的数据源组件，对于提供 ORM 框架以及整个应用的性能都是非常重要的。

- 事务管理模块

一般地，Mybatis 与 Spring 框架集成，由 Spring 框架管理事务。但 Mybatis 自身对数据库事务进行了抽象，提供了相应的事务接口和简单实现。

- 缓存模块

Mybatis 中有`一级缓存`和`二级缓存`，这两级缓存都依赖于缓存模块中的实现。但是需要注意，这两级缓存与 Mybatis 以及整个应用是运行在同一个 JVM 中的，共享同一块内存，如果这两级缓存中的数据量较大，则可能影响系统中其它功能，所以需要缓存大量数据时，优先考虑使用 Redis、Memcache 等缓存产品。

- Binding 模块

在调用 `SqlSession` 相应方法执行数据库操作时，需要制定映射文件中定义的 SQL 节点，如果 SQL 中出现了拼写错误，那就只能在运行时才能发现。为了能尽早发现这种错误，Mybatis 通过 Binding 模块将用户自定义的 Mapper 接口与映射文件关联起来，系统可以通过调用自定义 Mapper 接口中的方法执行相应的 SQL 语句完成数据库操作，从而避免上述问题。注意，在开发中，我们只是创建了 Mapper 接口，而并没有编写实现类，这是因为 Mybatis 自动为 Mapper 接口创建了动态代理对象。

# MyBatis 核心组件

## SqlSessionFactory

SqlSessionFactory 是 MyBatis 框架中的一个接口，它主要负责的是

- MyBatis 框架初始化操作
- 为开发人员提供`SqlSession` 对象

### SqlSessionFactory 的执行流程

下面来对 SqlSessionFactory 的执行流程来做一个分析

首先第一步是 SqlSessionFactory 的创建

```
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

从这行代码入手，首先创建了一个 `SqlSessionFactoryBuilder` 工厂，这是一个建造者模式的设计思想，由 builder 建造者来创建 SqlSessionFactory 工厂

然后调用 SqlSessionFactoryBuilder 中的 `build` 方法传递一个`InputStream` 输入流，Inputstream 输入流中就是你传过来的配置文件 mybatis-config.xml，SqlSessionFactoryBuilder 根据传入的 InputStream 输入流和`environment`、`properties`属性创建一个`XMLConfigBuilder`对象。SqlSessionFactoryBuilder 对象调用 XMLConfigBuilder 的`parse()`方法。

XMLConfigBuilder 会解析`/configuration`标签，configuration 是 MyBatis 中最重要的一个标签。

#### configuration 的配置

- `properties`，外部属性，这些属性都是可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递。

```xml
<properties>
    <property name="driver" value="com.mysql.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://localhost:3306/test" />
    <property name="username" value="root" />
    <property name="password" value="root" />
</properties>
```

一般用来给 `environment` 标签中的 `dataSource` 赋值

```xml
<environment id="development">
  <transactionManager type="JDBC" />
  <dataSource type="POOLED">
    <property name="driver" value="${driver}" />
    <property name="url" value="${url}" />
    <property name="username" value="${username}" />
    <property name="password" value="${password}" />
  </dataSource>
</environment>
```

* `settings` ，MyBatis 中极其重要的配置，它们会改变 MyBatis 的运行时行为。 

可以在此标签内设置 缓存，懒加载，自动驼峰命名规则映射等。

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
</settings>
```

## SqlSession

在 MyBatis 初始化流程结束，也就是 SqlSessionFactoryBuilder -> SqlSessionFactory 的获取流程后，我们就可以通过 SqlSessionFactory 对象得到 `SqlSession` 然后执行 SQL 语句了。 

 SqlSession 对象是 MyBatis 中最重要的一个对象，这个接口能够让你执行命令，获取映射，管理事务。SqlSession 中定义了一系列模版方法，让你能够执行简单的 `CRUD` 操作，也可以通过 `getMapper` 获取 Mapper 层，执行自定义 SQL 语句，因为 SqlSession 在执行 SQL 语句之前是需要先开启一个会话，涉及到事务操作，所以还会有 `commit`、 `rollback`、`close` 等方法。这也是模版设计模式的一种应用。 

## Executor

#### **Executor 的继承结构**

每一个 SqlSession 都会拥有一个 Executor 对象，这个对象负责增删改查的具体操作，我们可以简单的将它理解为 JDBC 中 Statement 的封装版。也可以理解为 SQL 的执行引擎，要干活总得有一个发起人吧，可以把 Executor 理解为发起人的角色。

Executor 执行器，它有两个实现类，分别是`BaseExecutor`和 `CachingExecutor`。

`BaseExecutor` 是一个抽象类，这种通过抽象的实现接口的方式是`适配器设计模式之接口适配` 的体现，是 Executor 的默认实现，实现了大部分 Executor 接口定义的功能，降低了接口实现的难度。BaseExecutor 的子类有三个，分别是 SimpleExecutor、ReuseExecutor 和 BatchExecutor。

`SimpleExecutor` : 简单执行器，是 MyBatis 中**默认使用**的执行器，每执行一次 update 或 select，就开启一个 Statement 对象，用完就直接关闭 Statement 对象 (可以是 Statement 或者是 PreparedStatment 对象)

`ReuseExecutor` : 可重用执行器，这里的重用指的是重复使用 Statement，它会在内部使用一个 Map 把创建的 Statement 都缓存起来，每次执行 SQL 命令的时候，都会去判断是否存在基于该 SQL 的 Statement 对象，如果存在 Statement 对象并且对应的 connection 还没有关闭的情况下就继续使用之前的 Statement 对象，并将其缓存起来。因为每一个 SqlSession 都有一个新的 Executor 对象，所以我们缓存在 ReuseExecutor 上的 Statement 作用域是同一个 SqlSession。

`BatchExecutor` : 批处理执行器，用于将多个 SQL 一次性输出到数据库

`CachingExecutor`: 缓存执行器，先从缓存中查询结果，如果存在就返回之前的结果；如果不存在，再委托给 Executor delegate 去数据库中取，delegate 可以是上面任何一个执行器。

### Executor 的具体执行过程

* 当有一个查询请求访问的时候，首先会经过 Executor 的实现类 `CachingExecutor` ，先从缓存中查询 SQL 是否是第一次执行，如果是第一次执行的话，那么就直接执行 SQL 语句，并创建缓存，如果第二次访问相同的 SQL 语句的话，那么就会直接从缓存中提取。 
* 如果没有的话，就再重新创建 `Executor` 执行器执行 SQL 语句，  创建我们上面提到的三种执行器
* 到这里，执行器所做的工作就完事了，Executor 会把后续的工作交给 `StatementHandler` 继续执行。 

### StatementHandler

 `StatementHandler` 是四大组件中最重要的一个对象，负责操作 Statement 对象与数据库进行交互，在工作时还会使用 `ParameterHandler` 和 `ResultSetHandler`对参数进行映射，对结果进行实体类的绑定

**StatementHandler 的继承结构**和 `Executor`  比较相似

主要有三个实现类

- **SimpleStatementHandler**: 管理 Statement 对象并向数据库中推送不需要预编译的 SQL 语句。
- **PreparedStatementHandler**: 管理 Statement 对象并向数据中推送需要预编译的 SQL 语句。
- **CallableStatementHandler**：管理 Statement 对象并调用数据库中的存储过程。

#### StatementHandler 的创建

 MyBatis 会根据 SQL 语句的类型进行对应 StatementHandler 的创建 。









#### #{}和${}的区别是什么？

#{}是预编译处理，${}是字符串替换。mybatis在处理#{}时，会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值；mybatis在处理${}时，就是把${}替换成变量的值。使用#{}可以有效的防止SQL注入，提高系统安全性。 

#### Xml 映射文件中，除了常见的 select|insert|updae|delete 标签之外，还有哪些标签？

还有很多其他的标签， resultMap ， parameterMap 加上动态 sql 的 9 个标签，`trim|where|set|foreach|if|choose|when|otherwise|bind`等，其中为 sql 片段标签，通过``标签引入 sql 片段，``为不支持自增的主键生成策略标签。



### mybatis 有几种分页方式？

数组分页

sql分页

拦截器分页

RowBounds分页

###  mybatis 逻辑分页和物理分页的区别是什么？

- 物理分页速度上并不一定快于逻辑分页，逻辑分页速度上也并不一定快于物理分页。
- 物理分页总是优于逻辑分页：没有必要将属于数据库端的压力加诸到应用端来，就算速度上存在优势,然而其它性能上的优点足以弥补这个缺点。


## 谈谈你对 Spring的理解

让 java 开发模块化，并且全面。Spring 通过控制反转降低耦合性，一个对象的依赖通过被动注入的方式而非主动 new，还通过代理模式实现了面向切面编程。

## IOC 是什么，什么是 Spring IOC 容器？

IOC 是一种设计思想。 **IOC 容器是 Spring 用来实现 IOC 的载体， IOC 容器在某种程度上就是个Map（key，value）,Map 中存放的是各种对象。** Spring 框架的核心就是是 IOC 容器。容器创建 Bean 对象， 将对象之间的相互依赖关系交给 IOC 容器来管理，将它们装配在一起，配置它们并管理它们的完整生命周期， 很大程度上简化应用的开发，降低了耦合度。

IOC 容器使用依赖注入来管理组成应用程序的组件。容器通过读取提供的配置,比如 XML，注解或 Java 代码来接收对象信息进行实例化，配置和组装。

### IoC 的实现机制

实现原理就是工厂模式加反射机制。  

* Spring 容器在启动的时候，先会保存所有注册进来的 Bean 的定义信息， 注册到 BeanFactory 中。 注册也只是将这些信息都保存到了注册中心(说到底核心是一个 beanName-> beanDefinition 的 map) 
* 设置 BeanFactory 的类加载器，添加几个 BeanPostProcessor，手动注册几个特殊的 bean 
* 具体的子类可以在这步的时候添加一些特殊的 BeanFactoryPostProcessor 的实现类或做点什么事 
* 注册 BeanPostProcessor 的实现类，可以在 Bean 初始化前后执行的方法，注意和上一步的区别 
* 初始化国际化，事件广播器的模块，注册事件监听器
* 然后 **Spring容器会合适的时机创建这些 Bean**
  1. 用到这个 Bean 的时候；利用 getBean 创建 Bean,创建好以后保存在容器中；
  2. 统一创建剩下所有的 Bean 的时候；调用 finishBeanFactoryInitialization() 初始化所有剩下的单例 Bean。
* 最后广播事件，ApplicationContext 初始化/刷新完成 

具体源码实现分析请看我的另一篇文章 。

### Spring Bean

#### 什么是 Spring Bean？

它们是是基于用户提供的配置创建的，由 Spring IoC 容器实例化，配置，装配和管理，构成了用户应用程序主干的对象 。

#### Bean 的 作用域（scope）

- **Singleton** - 每个 Spring IoC 容器仅有一个单实例。
- **Prototype** - 每次请求都会产生一个新的实例。
- **Request** - 每一次 HTTP 请求都会产生一个新的实例，并且该 bean 仅在当前 HTTP 请求内有效。
- **Session** - 每一次 HTTP 请求都会产生一个新的 bean，同时该 bean 仅在当前 HTTP session 内有效。
- **Global-session** - 类似于标准的 HTTP Session 作用域，不过它仅仅在基于 portlet 的 web 应用中才有意义。如果你在 web 中使用 global session 作用域来标识 bean，那么 web 会自动当成 session 类型来使用。

仅当用户使用支持 Web 的 ApplicationContext 时，最后三个才可用。

#### Bean 的生命周期

防止篇幅过长和内容重复，请看 SpringIOC 源码分析

#### Spring 中的单例 bean 的线程安全问题

Spring容器中的Bean是否线程安全，容器本身并没有提供 Bean 的线程安全策略，因此可以说 Spring 容器中的Bean本身不具备线程安全的特性，但是具体还是要结合具体 scope 的 Bean 去研究。 

常见的有两种解决办法：

1. 在Bean对象中尽量避免定义可变的成员变量（不太现实）。
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。

### spring 自动装配 bean 有哪些方式？

Spring容器负责创建应用程序中的bean同时通过ID来协调这些对象之间的关系。作为开发人员，我们需要告诉Spring要创建哪些bean并且如何将其装配到一起。

spring中bean装配有两种方式：

- 隐式的bean发现机制和自动装配
- 在java代码或者XML中进行显示配置

### ApplicationContext和BeanFactory的区别

可以从ApplicationContext继承的这几个接口入手，除去BeanFactory相关的两个接口就是ApplicationContext独有的功能 

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
    MessageSource, ApplicationEventPublisher, ResourcePatternResolver {} 
```



BeanFactory 粗暴简单，可以理解为就是个 HashMap，Key 是 BeanName，Value 是 Bean 实例。通常只提供注册（put），获取（get）这两个功能。我们可以称之为 “低级容器”。

ApplicationContext 可以称之为 “高级容器”。因为他比 BeanFactory 多了更多的功能。他继承了多个接口。因此具备了更多的功能。 例如资源的获取，支持多种消息（例如 JSP tag 的支持），对 BeanFactory 多了工具级别的支持等待。所以你看他的名字，已经不是 BeanFactory 之类的工厂了，而是 “应用上下文”， 代表着整个大容器的所有功能。 

### 循环依赖

bean A依赖于另一个bean B时，bean B依赖于bean A： 

当Spring上下文加载所有bean时，它会尝试按照它们完全工作所需的顺序创建bean。例如，如果我们没有循环依赖，如下例所示：

豆A→豆B→豆C.

Spring将创建bean C，然后创建bean B（并将bean注入其中），然后创建bean A（并将bean B注入其中）。

但是，当具有循环依赖时，Spring无法决定应该首先创建哪个bean，因为它们彼此依赖。

以setter方式构成的循环依赖，spring可以帮你解决，以构造器方式构成的循环依赖，spring无法搞定。

setter 注入和构造器注入的区别就在于创建bean的过程中，setter注入可以先用无参数构造方法返回bean实例，再注入依赖的属性，而constructor方式无法先返回bean的实例，必须先实例化它所依赖的属性，这样一来就会造成死循环所以会失败。 

我们使用比较多的在属性上@Autowired的方式，在spring内部的处理中，与setter方法不太一样，但用此种方式循环依赖也可以解决，原理同上，只要能先实例化出来，提前暴露出来，就可以解决循环依赖的问题。 

## AOP

### 解释一下什么是 AOP？

AOP（Aspect-Oriented Programming，面向切面编程），可以说是 OOP 的补充和完善。OOP 定义了从上到下的关系，但并不适合定义从左到右的关系。例如权限认证、日志、事务处理。日志代码往往水平地散布在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。而AOP技术将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。

## AspectJ 和 Spring AOP 的对比：

**Spring AOP：**

- 它基于动态代理来实现。默认地，如果使用接口的，用 JDK 提供的动态代理实现，如果没有接口，使用 CGLIB 实现。大家一定要明白背后的意思，包括什么时候会不用 JDK 提供的动态代理，而用 CGLIB 实现。
- Spring 的 IOC 容器和 AOP 都很重要，Spring AOP 需要依赖于 IOC 容器来管理。
- Spring AOP 只能作用于 Spring 容器中的 Bean，它是使用纯粹的 Java 代码实现的，只能作用于 bean 的方法。
- Spring 提供了 AspectJ 的支持，一般来说我们用**纯的** Spring AOP 就够了。
- 很多人会对比 Spring AOP 和 AspectJ 的性能，Spring AOP 是基于代理实现的，在容器启动的时候需要生成代理实例，在方法调用上也会增加栈的深度，使得 Spring AOP 的性能不如 AspectJ 那么好。

**AspectJ：**

- 属于静态织入，它是通过修改代码来实现的，它的织入时机可以是：
  - Compile-time weaving：编译期织入，如类 A 使用 AspectJ 添加了一个属性，类 B 引用了它，这个场景就需要编译期的时候就进行织入，否则没法编译类 B。
  - Post-compile weaving：也就是已经生成了 .class 文件，或已经打成 jar 包了，这种情况我们需要增强处理的话，就要用到编译后织入。
  - **Load-time weaving**：指的是在加载类的时候进行织入，要实现这个时期的织入，有几种常见的方法。1、自定义类加载器来干这个，这个应该是最容易想到的办法，在被织入类加载到 JVM 前去对它进行加载，这样就可以在加载的时候定义行为了。2、在 JVM 启动的时候指定 AspectJ 提供的 agent：`-javaagent:xxx/xxx/aspectjweaver.jar`。

- AspectJ 能干很多 Spring AOP 干不了的事情，它是 **AOP 编程的完全解决方案**。Spring AOP 致力于解决的是企业级开发中最普遍的 AOP 需求（方法织入），而不是力求成为一个像 AspectJ 一样的 AOP 编程完全解决方案。
- 因为 AspectJ 在实际代码运行前完成了织入，所以大家会说它生成的类是没有额外运行时开销的。

### 区别

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。

### **什么是切点（JoinPoint）**

程序运行中的一些时间点, 例如一个方法的执行, 或者是一个异常的处理.

在 Spring AOP 中, join point 总是方法的执行点。

### **什么是通知（Advice）？**

特定 JoinPoint 处的 Aspect 所采取的动作称为 Advice。Spring AOP 使用一个 Advice 作为拦截器，在 JoinPoint “周围”维护一系列的拦截器。

### **有哪些类型的通知（Advice）？**

- **Before** - 这些类型的 Advice 在 joinpoint 方法之前执行，并使用 @Before 注解标记进行配置。
- **After Returning** - 这些类型的 Advice 在连接点方法正常执行后执行，并使用@AfterReturning 注解标记进行配置。
- **After Throwing** - 这些类型的 Advice 仅在 joinpoint 方法通过抛出异常退出并使用 @AfterThrowing 注解标记配置时执行。
- **After (finally)** - 这些类型的 Advice 在连接点方法之后执行，无论方法退出是正常还是异常返回，并使用 @After 注解标记进行配置。
- **Around** - 这些类型的 Advice 在连接点之前和之后执行，并使用 @Around 注解标记进行配置。



## SpringMVC

## Springmvc 请求处理的流程

1. 客户端发送url请求，前端控制器（**DispatcherServlet**）接收到这个请求然后转发给处理器映射器（**HandlerMapping**）。

2. 处理器映射器会对url请求进行分析，找到对应的后端控制器（Handler），并且生成处理器对象及处理器拦截器（形成一条执行链）返回给前端控制器。

3. 根据处理器映射器返回的后端控制器(Handler)的名称/索引， 前端控制器 会找合适的处理器适配器( **HandlerAdapter** )

4. 处理器适配器会去执行后端控制器(Handler在开发的时候会被叫成**Controller**）。补充：执行之前会有转换器、数据绑定、校验器等等操作。完成上面这些才会去执行后端控制器。

5. 后端控制器Handler执行完成之后返回一个 **ModelAndView** 对象， Model 是返回的数据对象，View 是个逻辑上的 View。 

6. 处理器适配器会将这个 ModelAndView 返回前端控制器。前端控制器会将 ModelAndView 对象交给合适的视图解析器 **ViewResolver** 。

7. 视图解析器（ViewResolver）解析 ModelAndView 对象,返回 **视图对象（view）**。

8. 前端控制器请求视图对视图对象（View）进行渲染(数据填充)之后返回并响应给浏览器/客户端。

### Springmvc 有哪些组件

**1、**前端控制器DispatcherServlet（不需要工程师开发）,由框架提供（重要）**

作用：**Spring MVC 的入口函数。接收请求，响应结果，相当于转发器，中央处理器。有了 DispatcherServlet 减少了其它组件之间的耦合度。用户请求到达前端控制器，它就相当于mvc模式中的c，DispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，DispatcherServlet的存在降低了组件之间的耦合性。**

**2、处理器映射器HandlerMapping(不需要工程师开发),由框架提供**

作用：根据请求的url查找Handler。HandlerMapping负责根据用户请求找到Handler即处理器（Controller），SpringMVC提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

**3、处理器适配器HandlerAdapter**

作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler 通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

**4、处理器Handler(需要工程师开发)**

注意：编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。 由于Handler涉及到具体的用户业务请求，所以一般情况需要工程师根据业务需求开发Handler。

**5、视图解析器View resolver(不需要工程师开发),由框架提供**

作用：进行视图解析，根据逻辑视图名解析成真正的视图（view） View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。 springmvc框架提供了很多的View视图类型，包括：jstlView、freemarkerView、pdfView等。 一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求开发具体的页面。

**6、视图View(需要工程师开发)**

View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf...）

## 常用注解

* @Controller 负责注册一个bean 到spring 上下文中. 

* @ResponseBody 将java对象转换成json格式的字符串，返回给浏览器

  该注解用于将 Controller 的方法返回的对象，通过适当的 **HttpMessageConverter** 转换为指定格式后，写入到 Response 对象的 body 数据区。

**HttpMessageConverter是处理器适配器创建的，用于数据转换**。

* @RequestBody 接收JSON数据

  该注解用于读取 Request 请求的 body 部分数据，使用系统默认配置的 HttpMessageConverter 进行解析，然后把相应的数据绑定到要返回的对象上 

* @RequestController  **@RestController=@ResponseBody+@Controller** 

* @PathVariable   URL 中的 {xxx} 占位符可以通过@PathVariable(“xxx“) 绑定到操作方法的入参中。

*  @ControllerAdvice 使一个Contoller成为全局的异常处理类,类中用@ExceptionHandler方法注解的方法可以处理所有Controller发生的异常。

*  RequestMapping 是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。 

* @ModelAttribute最主要的作用是将数据添加到模型对象中，用于视图页面展示时使用。 等价于 model.addAttribute("attributeName", abc)。

  

##  Spring 框架中用到了哪些设计模式？

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。
- **策略模式：**

详细待补充。。。

## Spring 事务

### Spring 管理事务的方式有几种？

1. 编程式事务，在代码中硬编码。(不推荐使用)
2. 声明式事务，在配置文件中配置（推荐使用）

**声明式事务又分为两种：**

1. 基于XML的声明式事务
2. 基于注解的声明式事务

### @Transactional(rollbackFor = Exception.class)注解

我们知道：Exception分为运行时异常RuntimeException和非运行时异常。事务管理对于企业应用来说是至关重要的，即使出现异常情况，它也可以保证数据的一致性。

当`@Transactional`注解作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。如果类或者方法加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。

在`@Transactional`注解中如果不配置`rollbackFor`属性,那么事物只会在遇到`RuntimeException`的时候才会回滚,加上`rollbackFor=Exception.class`,可以让事物在遇到非运行时异常时也回滚。

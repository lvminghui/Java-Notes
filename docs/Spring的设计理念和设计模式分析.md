# Spring 的架构

Spring 总共有十几个组件，不过真正核心的组件只有几个：**Core，Context，Beans**。

## Spring 的设计理念

然而最最核心的是 Beans 组件，为什么这么说，其实在  Spring 就是面向 Bean 的编程（BOP,Bean Oriented Programming），Bean 在 Spring 中才是真正的主角。 

Bean 在 Spring 中作用就像 Object 对 OOP 的意义一样，没有对象的概念就像没有面向对象编程，Spring 中没有 Bean 也就没有 Spring 存在的意义。就像一次演出舞台都准备好了但是却没有演员一样。为什么要 Bean 这种角色 Bean 或者为何在 Spring 如此重要，这由 Spring 框架的设计目标决定，Spring 为何如此流行，我们用 Spring 的原因是什么，想想你会发现原来 Spring 解决了一个非常关键的问题他可以让你把对象之间的依赖关系转而用配置文件来管理，也就是他的依赖注入机制。而这个注入关系在一个叫 Ioc 容器中管理，那 Ioc 容器就是被 Bean 包裹的对象。Spring 正是通过把对象包装在 Bean 中而达到对这些对象的管理以及一些列额外操作的目的。 

### 核心组件如何协同工作

前面说 Bean 是 Spring 中关键因素，那 Context 和 Core 又有何作用呢？前面把 Bean 比作一场演出中的演员的话，那 Context 就是这场演出的舞台背景，而 Core 应该就是演出的道具了。只有他们在一起才能具备演出一场好戏的最基本条件。当然有最基本的条件还不能使这场演出脱颖而出，还要他表演的节目足够的精彩，这些节目就是 Spring 能提供的特色功能了。

我们知道 Bean 包装的是 Object，而 Object 必然有数据，如何给这些数据提供生存环境就是 Context 要解决的问题，对 Context 来说他就是要发现每个 Bean 之间的关系，为它们建立这种关系并且要维护好这种关系。所以 Context 就是一个 Bean 关系的集合，这个关系集合又叫 Ioc 容器，一旦建立起这个 Ioc 容器后 Spring 就可以为你工作了。那 Core 组件又有什么用武之地呢？其实 Core 就是发现、建立和维护每个 Bean 之间的关系所需要的一些列的工具，从这个角度看来，Core 这个组件叫 Util 更能让你理解。

## 核心组件详解

下面将详细介绍每个组件内部类的层次关系，以及它们在运行时的时序顺序。我们在使用 Spring 是应该注意的地方。

### Bean 组件

前面已经说明了 Bean 组件对 Spring 的重要性，下面看看 Bean 这个组件式怎么设计的。Bean 组件在 Spring 的 org.springframework.beans 包下。这个包下的所有类主要解决了三件事：Bean 的定义、Bean 的创建以及对 Bean 的解析。对 Spring 的使用者来说唯一需要关心的就是 Bean 的创建，其他两个由 Spring 在内部帮你完成了，对你来说是透明的。

####  Bean 工厂

Spring Bean 的创建是典型的工厂模式，它的顶级接口是 BeanFactory。防止大家产生看晕，对于这个工厂的继承层次我就不说的太详细了，所有的接口都有使用的场合，我们需要知道 BeanFactory 有三个较重要的子类：ListableBeanFactory（可列表的）、HierarchicalBeanFactory（有继承关系的） 和 AutowireCapableBeanFactory（可以自动装配的）。 而我们最常使用的 ApplicationContext 类继承了 ListableBeanFactory、HierarchicalBeanFactory。 

ListableBeanFactory 接口表示这些 Bean 是可列表的，而 HierarchicalBeanFactory 表示的这些 Bean 是有继承关系的，也就是每个 Bean 有可能有父 Bean。AutowireCapableBeanFactory 接口定义 Bean 的自动装配规则。 

综上，ApplicationContext 容器通过继承可以拥有可列表，有继承关系的能力，而它内部也定义了一个得到自动装配的工厂接口，从而实现了自动装配能力。

####  BeanDefinition 

说完了 Bean 工厂，现在来说说 Bean 本身，而 Bean 的定义主要由 BeanDefinition 描述。

其实  BeanDefinition 就是我们所说的 Spring 的 Bean，我们自己定义的各个 Bean （`<bean/>` 或 注解）其实会转换成一个个 BeanDefinition 存在于 Spring 的 BeanFactory 中。 所以，如果有人问你 Bean 是什么的时候，你要知道 Bean 在代码层面上可以简单认为是 BeanDefinition 的实例。 得到 BeanDefinition  了，下面要对他进行解析，这个过程很复杂，因为很多可扩展的地方，xxxReader就是解析类，我暂时不过多介绍。

### Context 组件

Context 在 Spring 的 org.springframework.context 包下，前面已经讲解了 Context 组件在 Spring 中的作用，他实际上就是给 Spring 提供一个运行时的环境，用以保存各个对象的状态。下面看一下这个环境是如何构建的。 

**ApplicationContext** 是 Context 的顶级父类，他除了能标识一个应用环境的基本信息外，他还继承了五个接口：EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource,ApplicationEventPublisher, ResourcePatternResolver，这五个接口主要是扩展了 Context 的功能。 

ApplicationContext 继承了 ResourceLoader 接口，使得 ApplicationContext 可以访问到任何外部资源，这将在 Core 中详细说明。 

ApplicationContext 的常用子类主要包含： 

* ClassPathXmlApplicationContext：可以加载类路径下的配置文件，不在的话加载不了

* FileSystemXmlApplicationContext：可以加载磁盘任意路径下的配置文件（必须有访问权限）

* AnnotationConfigApplicationContext  用于读取注解创建容器的

总体来说 ApplicationContext 必须要完成以下几件事：

- 标识一个应用环境
- 利用 BeanFactory 创建 Bean 对象
- 保存对象关系表
- 能够捕获各种事件

**Context 作为 Spring 的 Ioc 容器，基本上整合了 Spring 的大部分功能，或者说是大部分功能的基础。**

### Core 组件

Core 组件作为 Spring 的核心组件，他其中包含了很多的关键类，其中一个重要组成部分就是定义了资源的访问方式  Resource。这种把所有资源都抽象成一个接口的方式很值得在以后的设计中拿来学习。下面就重要看一下这个部分在 Spring 的作用。 

Resource 接口封装了各种可能的资源类型，也就是对使用者来说屏蔽了文件类型的不同。  对资源的提供者来说，Resource 也可以把资源包装起来交给其他人，所以也屏蔽了资源的提供者。 

那么 Context 和 Resource 的类关系是怎么样的？他们是如何协同工作的？

Context 是把资源的加载、解析和描述工作委托给了 ResourcePatternResolver 类来完成，他相当于一个接头人，他把资源的加载、解析和资源的定义整合在一起便于其他组件使用。Core 组件中还有很多类似的方式。 

## 创建 BeanFactory 工厂

介绍了 Core 组件、Bean 组件和 Context 组件的结构与相互关系，下面这里从使用者角度看一下他们是如何运行的，  以及我们如何让 Spring 完成各种功能，Spring 到底能有那些功能，这些功能是如何得来的。

**Ioc 容器实际上就是 Context 组件结合其他两个组件共同构建了一个 Bean 关系网。** 

下面就是经典的 refresh() 方法：

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
   // 来个锁，不然 refresh() 还没结束，你又来个启动或销毁容器的操作，那不就乱套了嘛
   synchronized (this.startupShutdownMonitor) {

      // 刷新前的预处理,初始化一些属性设置,检验属性的合法等,保存容器中的一些早期的事件；
      prepareRefresh();

      // 获取BeanFactory,这步完成后，配置文件就会解析成一个个 Bean 定义，注册到 BeanFactory 中，
      // 当然，这里说的 Bean 还没有初始化，只是配置信息都提取出来了，
      // 注册也只是将这些信息都保存到了注册中心(说到底核心是一个 beanName-> beanDefinition 的 map)
      //将创建的BeanFactory【DefaultListableBeanFactory】返回；
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      //BeanFactory的预准备工作（对BeanFactory进行一些设置）
      //设置 BeanFactory 的类加载器，添加几个 BeanPostProcessor，比如：
      //ApplicationContextAwareProcessor
      //手动注册几个特殊的 bean
      prepareBeanFactory(beanFactory);

      try {
         // 这里是提供给子类的扩展点，到这里的时候，所有的 Bean 都加载、注册完成了，但是都还没有初始化
         // 具体的子类可以在这步的时候添加一些特殊的 BeanFactoryPostProcessor 的实现类或做点什么事
         postProcessBeanFactory(beanFactory);
         // 调用 BeanFactoryPostProcessor 各个实现类的 postProcessBeanFactory(factory) 方法
         //两大接口：
  		 //BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor
         invokeBeanFactoryPostProcessors(beanFactory);

         // 注册 BeanPostProcessor 的实现类，注意看和 BeanFactoryPostProcessor 的区别
         // 此接口两个方法: postProcessBeforeInitialization 和 	
         //postProcessAfterInitialization
         //注册BeanPostProcessor（Bean的后置处理器）作用：拦截 bean 的注册过程
         // 两个方法分别在 Bean 初始化之前和初始化之后得到执行。注意，到这里 Bean 还没初始化
         registerBeanPostProcessors(beanFactory);

         // 初始化当前 ApplicationContext 的组件（做国际化功能；消息绑定，消息解析）；
         initMessageSource();

         // 初始化当前 ApplicationContext 的事件广播器
         initApplicationEventMulticaster();

         // 具体的子类可以在这里初始化一些特殊的 Bean（在初始化 singleton beans 之前）
         // 留给子容器（子类）,子类重写这个方法，在容器刷新的时候可以自定义逻辑；
         onRefresh();

         // 注册事件监听器，监听器需要实现 ApplicationListener 接口	
         registerListeners();

         // 初始化所有的单实例bean；
         //（lazy-init 的除外）
         finishBeanFactoryInitialization(beanFactory);

         // 最后，广播事件，ApplicationContext 初始化完成
         finishRefresh();
      }

      catch (BeansException ex) {
          
         // 销毁已经初始化的 singleton 的 Beans，以免有些 bean 会一直占用资源
         destroyBeans();

         cancelRefresh(ex);
          
         throw ex;
      }
   }
}
```



在初始化所有的单实例bean的过程中，有一个非常重要的 Bean —— FactoryBean，可以说 Spring 一大半的扩展的功能都与这个 Bean 有关，这是个特殊的 Bean 是一个工厂 Bean，可以产生 Bean 的 Bean，这里的产生 Bean 是指 Bean 的实例，如果一个类继承 FactoryBean 用户只要实现他的 getObject 方法，就可以自己定义产生实例对象的方法。然而在 Spring 内部这个 Bean 的实例对象是 FactoryBean，通过调用这个对象的 getObject 方法就能获取用户自定义产生的对象，从而为 Spring 提供了很好的扩展性。Spring 获取 FactoryBean 本身的对象是在前面加上 & 来完成的。 

##### Bean 实例创建流程图

![bean的实例化](C:\Users\吕明辉\Desktop\github笔记\spring\bean的实例化.gif)



普通的 Bean 就是通过调用 getBean 方法来创建的。

getBean 调用 doGetBean，再调用 createBean，再调用 doCreateBean。

doCreateBean 有三个重要方法，一个是创建 Bean 实例的 createBeanInstance 方法，一个是依赖注入的 populateBean 方法，还有就是回调方法 Bean初始化 initializeBean 。  

创建好的实例对象放入缓存对象保存。

### Ioc 容器的扩展点

对 Spring 的 Ioc 容器来说，主要有这么几个可扩展的点。BeanFactoryPostProcessor， BeanPostProcessor。他们分别是在构建 BeanFactory 和构建 Bean 对象时调用。还有就是 InitializingBean 和 DisposableBean， 他们分别是在 Bean 实例创建和销毁时被调用。用户可以实现这些接口中定义的方法，Spring 就会在适当的时候调用他们。还有一个是 FactoryBean 他是个特殊的 Bean，这个 Bean 可以被用户更多的控制。

这些扩展点通常也是我们使用 Spring 来完成我们特定任务的地方，如何精通 Spring 就看你有没有掌握好 Spring 有哪些扩展点，并且如何使用他们，要知道如何使用他们就必须了解他们内在的机理。可以用下面一个比喻来解释。

我们把 Ioc 容器比作一个箱子，这个箱子里有若干个球的模子，可以用这些模子来造很多种不同的球，还有一个造这些球模的机器，这个机器可以产生球模。那么他们的对应关系就是：BeanFactory 是那个造球模的机器，球模就是 Bean，而球模造出来的球就是 Bean 的实例。那前面所说的几个扩展点又在什么地方呢？ BeanFactoryPostProcessor 对应到当造球模被造出来时，你将有机会可以对其做出适当的修正，也就是他可以帮你修改球模。而 InitializingBean 和 DisposableBean 是在球模造球的开始和结束阶段，你可以完成一些预备和扫尾工作。BeanPostProcessor 就可以让你对球模造出来的球做出适当的修正。最后还有一个 FactoryBean，它可是一个神奇的球模。这个球模不是预先就定型了，而是由你来给他确定它的形状，既然你可以确定这个球模型的形状，当然他造出来的球肯定就是你想要的球了，这样在这个箱子里你可以发现所有你想要的球。

# Spring 中设计模式分析

 Spring 主要使用了工厂模式，单例模式，模版模式，代理模式和策略模式。 


















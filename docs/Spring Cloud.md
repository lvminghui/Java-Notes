## Spring Cloud 的理解？

Spring Cloud就是微服务系统架构的一站式解决方案，在平时我们构建微服务的过程中需要做如 **服务发现注册** 、**配置中心** 、**消息总线** 、**负载均衡** 、**断路器** 、**数据监控** 等操作，而 Spring Cloud 为我们提供了一套简易的编程模型，使我们能在 Spring Boot 的基础上轻松地实现微服务项目的构建。 

### 服务发现框架——Eureka

**服务注册 Register** ：

当 Eureka 客户端（服务提供者）向 Eureka Server 注册时，它提供自身的**元数据** ，比如IP地址、端口，运行状况指示符URL，主页等。

**服务续约 Renew** ： 

**Eureka 客户端会每隔30秒(默认情况下)发送一次心跳来续约** 。 通过续约来告知 Eureka Server  该 Eureka 客户仍然存在，没有出现问题。正常情况下，如果 Eureka Server 在90秒没有收到 Eureka 客户端的续约，它会将实例从其注册表中删除。 

**获取注册列表信息 Fetch Registries** ： 

Eureka 客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息如果与 Eureka  客户端的缓存信息不同,Eureka 客户端自动处理。Eureka  客户端和 Eureka 服务器可以使用JSON / XML格式进行通讯。

**服务下线 Cancel** ： 

Eureka 客户端在程序关闭时向 Eureka 服务器发送取消请求。发送请求后，该客户端实例信息将从服务器的实例注册表中删除。该下线请求不会自动完成，它需要调用以下内容：`DiscoveryManager.getInstance().shutdownComponent();` 

 **服务剔除 Eviction** ： 

在默认的情况下，当 Eureka 客户端连续90秒(3个续约周期)没有向服务器发送服务续约，即心跳，服务器会将该服务实例从服务注册列表删除，即服务剔除。 

 **架构图**： 

![image-20200229172633604](C:\Users\吕明辉\AppData\Roaming\Typora\typora-user-images\image-20200229172633604.png)

 可以充当服务发现的组件有很多：Zookeeper ，Consul ， Eureka 等。 

### 负载均衡之 Ribbon

Ribbon 是一个客户端/进程内负载均衡器，**运行在消费者端** 。 

其工作原理就是 `Consumer` 端获取到了所有的服务列表之后，在其**内部** 使用**负载均衡算法** ，进行对多个系统的调用。 

#### Nginx 和 Ribbon 的对比

nginx 是客户端所有请求统一交给 nginx，由 nginx 进行实现负载均衡请求转发，属于服务器端负载均衡。 

Ribbon 是从 eureka 注册中心服务器端上获取服务注册信息列表，缓存到本地，然后在本地实现轮询负载均衡策略，属于客户端负载均衡。 

#### Ribbon 的几种负载均衡算法

负载均衡，不管 `Nginx` 还是 `Ribbon` 都需要其算法的支持，如果我没记错的话 `Nginx` 使用的是 轮询和加权轮询算法。而在 `Ribbon` 中有更多的负载均衡调度算法，其默认是使用的 `RoundRobinRule` 轮询策略。 

**RoundRobinRule** ：轮询策略。`Ribbon` 默认采用的策略。若经过一轮轮询没有找到可用的 `provider`，其最多轮询 10 轮。若最终还没有找到，则返回 null。 

默认轮询算法，并且可以更换默认的负载均衡算法，只需要在配置文件中做出修改。 

### Open Feign

OpenFeign 也是运行在消费者端的，使用 Ribbon 进行负载均衡，所以 OpenFeign 直接内置了 Ribbon。 主要用于消费者和服务者的调用

### Hystrix--断路器

Hystrix 就是一个能进行 **熔断** 和 **降级** 的库，通过使用它能提高整个系统的弹性。 

所谓 **熔断** 就是服务雪崩的一种有效解决方案。当指定时间窗内的请求失败率达到设定阈值时，系统将通过 **断路器** 直接将此请求链路断开。 

也就是我们上面服务A调用服务B在指定时间窗内，调用的失败率到达了一定的值，那么 Hystrix 则会自动将服务A与B之间的请求都断了，以免导致服务雪崩现象。 

 其实这里所讲的 **熔断** 就是指的 Hystrix中的 **断路器模式** ，你可以使用简单的 `@HystrixCommand` 注解来标注某个方法，这样 Hystrix 就会使用 **断路器** 来“包装”这个方法，每当调用时间超过指定时间时(默认为1000ms)，断路器将会中断对这个方法的调用。比如： 

```java
@HystrixCommand(
    commandProperties = {@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1200")}
)
public List<Xxx> getXxxx() {
    // ...省略代码逻辑
}
```

**降级是为了更好的用户体验，当一个方法调用异常时，通过执行另一种代码逻辑来给用户友好的回复** 。这也就对应着 Hystrix 的 **后备处理** 模式。你可以通过设置 `fallbackMethod` 来给一个方法设置备用的代码逻辑。比如这个时候有一个热点新闻出现了，用户会通过id去查询新闻的详情，但是因为这条新闻太火了，大量用户同时访问可能会导致系统崩溃，那么我们就进行 **服务降级** ，一些请求会做一些降级处理比如当前人数太多请稍后查看等等。 

```java
// 指定了后备方法调用
@HystrixCommand(fallbackMethod = "getHystrixNews")
@GetMapping("/get/news")
public News getNews(@PathVariable("id") int id) {
    // 调用新闻系统的获取新闻api 代码逻辑省略
}
//
public News getHystrixNews(@PathVariable("id") int id) {
    // 做服务降级
    // 返回当前人数太多，请稍后查看
}
```

## 微服务网关——Zuul

ZUUL 是为了实现动态路由、监视、弹性和安全性而构建的。**就是这样的一个对于消费者的统一入口。** 

#### 路由功能

`Zuul` 需要向 Eureka 进行注册， 以此来拿到所有 `Consumer` 的信息，从而直接可以做路由映射。

**基本配置：**

```yml
server:
  port:9000
eureka:
  client:
    service-url:
      # 这里只要注册 Eureka 就行了
      defaultZone:http://localhost:9997/eureka
```

然后在启动类上加入 `@EnableZuulProxy` 注解就行了。没错，就是那么简单。 

**统一前缀**

这个很简单，就是我们可以在前面加一个统一的前缀，比如我们刚刚调用的是 `localhost:9000/consumer1/studentInfo/update`，这个时候我们在 `yaml` 配置文件中添加如下。

```
zuul:  prefix:/zuul
```

这样我们就需要通过 `localhost:9000/zuul/consumer1/studentInfo/update` 来进行访问了。

**路由策略配置**

你会发现前面的访问方式(直接使用服务名)，需要将微服务名称暴露给用户，会存在安全性问题。所以，可以自定义路径来替代微服务名称，即自定义路由策略。

```
zuul:  routes:    consumer1:/FrancisQ1/**    consumer2:/FrancisQ2/**
```

这个时候你就可以使用 `localhost:9000/zuul/FrancisQ1/studentInfo/update` 进行访问了。

**服务名屏蔽**

这个时候你别以为你好了，你可以试试，在你配置完路由策略之后使用微服务名称还是可以访问的，这个时候你需要将服务名屏蔽。

```
zuul:  ignore-services:"*"
```

**路径屏蔽**

`Zuul` 还可以指定屏蔽掉的路径 URI，即只要用户请求中包含指定的 URI 路径，那么该请求将无法访问到指定的服务。通过该方式可以限制用户的权限。

```
zuul:  ignore-patterns:**/auto/**
```

这样关于 auto 的请求我们就可以过滤掉了。

> ** 代表匹配多级任意路径
>
> *代表匹配一级任意路径

**敏感请求头屏蔽**

默认情况下，像 Cookie、Set-Cookie 等敏感请求头信息会被 zuul 屏蔽掉，我们可以将这些默认屏蔽去掉，当然，也可以添加要屏蔽的请求头。

#### 过滤功能

所有请求都经过网关(Zuul)，那么我们可以进行各种过滤，这样我们就能实现 **限流** ，**灰度发布** ，**权限控制** 等等。 

过滤器类型：Pre、Routing、Post。前置Pre就是在请求之前进行过滤，Routing路由过滤器就是我们上面所讲的路由策略，而Post后置过滤器就是在 `Response` 之前进行过滤的过滤器。

## 配置管理——Config

当我们的微服务系统开始慢慢地庞大起来，那么多 `Consumer` 、`Provider` 、`Eureka Server` 、`Zuul` 系统都会持有自己的配置，这个时候我们在项目运行的时候可能需要更改某些应用的配置，如果我们不进行配置的统一管理，我们只能**去每个应用下一个一个寻找配置文件然后修改配置文件再重启应用** 。

首先对于分布式系统而言我们就不应该去每个应用下去分别修改配置文件，再者对于重启应用来说，服务无法访问所以直接抛弃了可用性，这是我们更不愿见到的。

`Spring Cloud Config` 就是能将各个 应用/系统/模块的配置文件存放到 **统一的地方然后进行管理** (Git 或者 SVN)。 

我们的应用只有启动的时候才会进行配置文件的加载，那么我们的 `Spring Cloud Config` 就暴露出一个接口给启动应用来获取它所想要的配置文件，应用获取到配置文件然后再进行它的初始化工作。 

怎么进行动态修改配置文件呢？  一般我们会使用 `Bus` 消息总线 + `Spring Cloud Config` 进行配置的动态刷新。 

### Spring Cloud Bus

你可以简单理解为 `Spring Cloud Bus` 的作用就是**管理和广播分布式系统中的消息** ，也就是消息引擎系统中的广播模式。当然作为 **消息总线** 的 `Spring Cloud Bus` 可以做很多事而不仅仅是客户端的配置刷新功能。 

而拥有了 `Spring Cloud Bus` 之后，我们只需要创建一个简单的请求，并且加上 `@ResfreshScope` 注解就能进行配置的动态修改了 。

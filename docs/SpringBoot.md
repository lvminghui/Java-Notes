### 什么是 Spring Boot？

**如果必须启动一个新的 Spring 项目，我们必须添加构建路径或 maven 依赖项，配置 application server，添加 Spring 配置。因此，启动一个新的 Spring 项目需要大量的工作，因为我们目前必须从头开始做所有事情**。  Spring Boot 是这个问题的解决方案。Spring boot 构建在现有 Spring 框架之上。使用 Spring boot，可以避免以前必须执行的所有样板代码和配置。 

### Spring Boot 的优点是什么?

编码：减少开发、测试的时间和工作量。

配置：使用 JavaConfig 有助于避免使用 XML。没有 web.xml 文件，只需添加带 @ configuration 注释的类。

避免大量 maven 导入和各种版本冲突。

部署：不需要单独的 Web 服务器。这意味着您不再需要启动 Tomcat 或其他任何东西。

## Spring Boot 启动流程⭐

首先 prepareEnvironment 配置环境，然后准备 Context 上下文，ApplicationContext 的后置处理器，初始化 lnitializers，通知处理上下文准备和加载时间，然后开始refresh。

prepareEnvironment 

createApplicationContext
postProcessApplicationContext
applylnitializers
listeners.contextPrepared
listeners.contextLoaded
refreshContext

### Spring Boot提供了两种常用的配置文件：

- properties文件
- yml文件

## Spring Boot 的核心注解是哪个？⭐

启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：

@SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。

@EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能：       @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。

@ComponentScan：Spring组件扫描。

## Spring Boot 自动配置原理是什么⭐

@EnableAutoConfiguration这个注解开启自动配置，它的作用：

* 利用EnableAutoConfigurationImportSelector给容器中导入一些组件
* 这个类父类有一个方法：selectImports()，这个方法返回 **configurations** ：
* List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);获取候选的配置
  * 将 类路径下  META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中
* 加载某个组件的时候根据注解的条件判断每个加入的组件是否生效，如果生效就把类的属性和配置文件绑定起来
* 这时就读取配置文件的值加载组件

### SpringBoot、SpringMVC和Spring区别

spring boot只是一个配置工具,整合工具,辅助工具.

springmvc是框架,项目中实际运行的代码

Spring 框架就像一个家族，有众多衍生产品例如 boot、security、jpa等等。但他们的基础都是Spring 的ioc和 aop，ioc 提供了依赖注入的容器， aop解决了面向横切面的编程，然后在此两者的基础上实现了其他延伸产品的高级功能。

用最简练的语言概括就是：

- Spring 是一个“引擎”；
- Spring MVC 是基于Spring的一个 MVC 框架；
- Spring Boot 是基于Spring的条件注册的一套快速开发整合包

## SpringBoot 拦截器和过滤器

​        1、Filter是依赖于Servlet容器，属于Servlet规范的一部分，而拦截器则是独立存在的，可以在任何情况下使用。

　　2、Filter的执行由Servlet容器回调完成，而拦截器通常通过动态代理的方式来执行。

　　3、Filter的生命周期由Servlet容器管理，而拦截器则可以通过IoC容器来管理，因此可以通过注入等方式来获取其他Bean的实例，因此使用会更方便。



```java
@WebFilter(urlPatterns = "/*", filterName = "logFilter")
public class LogCostFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
 
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        long start = System.currentTimeMillis();
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("LogFilter Execute cost=" + (System.currentTimeMillis() - start));
    }
 
    @Override
    public void destroy() {
 
    }
}
```



这段代码的逻辑比较简单，就是在方法执行前先记录时间戳，然后通过过滤器链完成请求的执行，在返回结果之间计算执行的时间。这里需要主要，这个类必须继承Filter类，这个是Servlet的规范，这个跟以前的Web项目没区别。 这里直接用@WebFilter就可以进行配置，同样，可以设置url匹配模式，过滤器名称等。这里需要注意一点的是@WebFilter这个注解是Servlet3.0的规范，并不是Spring boot提供的。除了这个注解以外，我们还需在配置类中加另外一个注解：@ServletComponetScan，指定扫描的包。 

```java
@SpringBootApplication
@MapperScan("com.pandy.blog.dao")
@ServletComponentScan("com.pandy.blog.filters")
public class Application {
    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }
}
```

上面我们已经介绍了过滤器的配置方法，接下来我们再来看看如何配置一个拦截器。我们使用拦截器来实现上面同样的功能，记录请求的执行时间。首先我们实现拦截器类： 

```java
public class LogCostInterceptor implements HandlerInterceptor {
    long start = System.currentTimeMillis();
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        start = System.currentTimeMillis();
        return true;
    }
 
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("Interceptor cost="+(System.currentTimeMillis()-start));
    }
 
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
    }
}
```

 　这里我们需要实现HandlerInterceptor这个接口，这个接口包括三个方法，preHandle是请求执行前执行的，postHandler是请求结束执行的，但只有preHandle方法返回true的时候才会执行，afterCompletion是视图渲染完成后才执行，同样需要preHandle返回true，该方法通常用于清理资源等工作。除了实现上面的接口外，我们还需对其进行配置： 

```java
@Configuration
public class InterceptorConfig extends WebMvcConfigurerAdapter {
 
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogCostInterceptor()).addPathPatterns("/**");
        super.addInterceptors(registry);
    }
}
```

 这里我们继承了WebMVCConfigurerAdapter，看过前面的文章的朋友应该已经见过这个类了，在进行静态资源目录配置的时候我们用到过这个类。这里我们重写了addInterceptors这个方法，进行拦截器的配置，主要配置项就两个，一个是指定拦截器，第二个是指定拦截的URL。 



## spring boot处理一个http请求的全过程 

* 由前端发起请求
* 根据路径，Springboot会加载相应的Controller进行拦截 
* 拦截处理后，跳转到相应的Service处理层 
* 跳转到ServiceImplement(service实现类) 
* 在执行serviceimplement时会加载Dao层，操作数据库 
* 再跳到Dao层实现类 
* 执行会跳转到mapper层 
* 然后MallMapper会继续找对应的mapper.xml配置文件 
* 之后便会跳转到第4步继续执行，执行完毕后会将结果返回到第1步，然后 
* 便会将数据以JSON的形式返回到页面，同时返回状态码，正常则会返回200，便会回到步骤1中查询判断。 


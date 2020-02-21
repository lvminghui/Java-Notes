### 什么是 Spring Boot？

**如果必须启动一个新的 Spring 项目，我们必须添加构建路径或 maven 依赖项，配置 application server，添加 Spring 配置。因此，启动一个新的 Spring 项目需要大量的工作，因为我们目前必须从头开始做所有事情**。  Spring Boot 是这个问题的解决方案。Spring boot 构建在现有 Spring 框架之上。使用 Spring boot，可以避免以前必须执行的所有样板代码和配置。 

### Spring Boot 的优点是什么?

编码：减少开发、测试的时间和工作量。

配置：使用 JavaConfig 有助于避免使用 XML。没有 web.xml 文件，只需添加带 @ configuration 注释的类。

避免大量 maven 导入和各种版本冲突。

部署：不需要单独的 Web 服务器。这意味着您不再需要启动 Tomcat 或其他任何东西。

### Spring Boot提供了两种常用的配置文件：

- properties文件
- yml文件

### SpringBoot、SpringMVC和Spring区别

spring boot只是一个配置工具,整合工具,辅助工具.

springmvc是框架,项目中实际运行的代码

Spring 框架就像一个家族，有众多衍生产品例如 boot、security、jpa等等。但他们的基础都是Spring 的ioc和 aop，ioc 提供了依赖注入的容器， aop解决了面向横切面的编程，然后在此两者的基础上实现了其他延伸产品的高级功能。

用最简练的语言概括就是：

- Spring 是一个“引擎”；
- Spring MVC 是基于Spring的一个 MVC 框架；
- Spring Boot 是基于Spring的条件注册的一套快速开发整合包。

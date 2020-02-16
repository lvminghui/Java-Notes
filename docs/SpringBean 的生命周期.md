## Bean 的完整生命周期

* Bean容器/BeanFactory 通过对象的构造器或工厂方法先实例化 Bean；

* 再根据实例化好的 BeanDefintion 中的信息再通过设定好的方法（典型的有setter，统称为BeanWrapper）对 Bean 设置属性值，即依赖注入的过程。

* 检查 xxxAware 相关接口，比如 BeanNameAware，BeanClassLoaderAware，ApplicationContextAware（ BeanFactoryAware）等等，如果有就调用相应的 setxxx 方法把所需要的xxx传入到 Bean 中。

  **补充**：关于 Aware ，Aware 就是感知的意思， Aware 的目的是为了让Bean获得Spring容器的服务。 实现了这类接口的 bean 会存在“意识感”，从而让容器调用setxxx方法把所需要的xxx传到 Bean中。

* 此时检查是否存在有于 Bean 关联的任何  BeanPostProcessors， 执行 postProcessBeforeInitialization() 方法（前置处理器）。

* 如果 Bean 实现了InitializingBean接口（正在初始化的 Bean），执行afterPropertiesSet()方法。

* 检查是否配置了自定义的 init-method 方法，如果有就调用。

* 此时检查是否存在有于 Bean 关联的任何  BeanPostProcessors， 执行 postProcessAfterInitialization() 方法（后置处理器）。返回 wrapperBean（包装后的 Bean）。

* 这时就可以开始使用 Bean 了，当容器关闭时，会检查 Bean 是否实现了 DisposableBean 接口，如果有就调用 destory() 方法。

* 如果 Bean 配置文件中的定义包含 destroy-method 属性，执行指定的方法。 

上面整个过程就是 Bean 的整个生命周期了

程序员的主要工作是编写业务逻辑代码，业务逻辑代码一般都是技术无关性的，即 Spring 代码不会侵入业务逻辑代码中。虽然我们使用了很多 Spring 的注解，但注解属于元数据（和XML一样），不属于代码侵入。



有些时候却**不得不让自己的代码和 Spring 框架耦合，通过实现相应的 Aware 接口**，注入其对应的 Bean：

- BeanNameAware：可获得beanName，即Bean的名称
- ResourceLoaderAware：可获得ResourceLoader，即用来加载资源的Bean
- BeanFactoryAware：可获得BeanFactory，即容器的父接口，用于管理Bean的相关操作
- EnvironmentAware：可获得Environment，即当前应用的运行环境
- MessageSourceAware：可获得MessageSource，即用来解析文本信息的Bean
- ApplicationEventPublisherAware：可获得ApplicationEventPublisher，即用来发布系统时间的Bean
- ApplicationContextAware：可自动注入ApplicationContext，即容器本身




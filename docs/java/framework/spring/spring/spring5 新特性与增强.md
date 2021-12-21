Spring是一个支持快速开发Java EE应用程序的框架。它提供了一系列底层容器和基础设施，并可以和大量常用的开源框架无缝集成，可以说是开发Java EE应用程序的必备。



Spring Framework主要包括几个模块：

- 支持IoC和AOP的容器

- 支持JDBC和ORM的数据访问模块

- 支持声明式事务的模块

- 支持基于Servlet的MVC开发

- 支持基于Reactive的Web开发

- 以及集成JMS、JavaMail、JMX、缓存等其他模块

    

从2004年3月到现在，已经经历了1.0、1.1、1.2、2.0、2.5、3.0、3.1、4.0、4.1、4.2、4.3、5.1、5.1几个主要的版本
[https://repo.spring.io/libs-release-local/org/springframework/spring/](https://repo.spring.io/libs-release-local/org/springframework/spring/)

![在这里插入图片描述](https://img-note.langyastudio.com/202111091458251.png?x-oss-process=style/watermark)
https://spring.io/projects/spring-framework#learn

Spring 5 是一个重要的版本，距离 Spring Framework 4 差不多四年。在此期间，大多数增强都是在 SpringBoot 项目中完成的。在本文中，我们将很快了解到 Spring5 发行版中的一些令人兴奋的特性。



#### 基准升级
要构建和运行 Spring 5 应用程序，你至少需要 `J2EE 7` 和 `JDK 8`。以前的 JDK 和 JavaEE 版本不再支持。 

JavaEE 7 包含：
- Servlet 3.1

- JMS 2.0

- JPA 2.1

- JAX-RS 2.0

- Bean Validation 1.1

与 Java 基准类似，许多其他框架的基准也有变化。例如：
- Hibernate 5

- Jackson 2.6

- EhCache 2.10

- JUnit 5

- Tiles 3

另外，请记下各种服务器最低支持版本。
- Tomcat 8.5+

- Jetty 9.4+

- WildFly 10+

- Netty 4.1+

- Undertow 1.4+

    

#### JDK 8+ 和 Java EE 7+ 以上版本
- **整个框架的代码基于 java 8**

- 通过使用泛型等特性提高可读性

- 对 java 8 提高直接的代码支撑
运行时兼容 JDK 9

- Java EE 7 API 需要 Spring 相关的模块支持
运行时兼容 Java EE 8 API

-  取消的包 , 类 和 方法
包 beans.factory.access
包 dbc.support.nativejdbc
从 spring-aspects 模块移除了包 mock.staicmock，不在提 AnnotationDrivenStaticEntityMockingControl 支持

- 许多不建议使用的类和方法在代码库中删除

    

#### JDK8 的增强
- 访问 Resource 时提供 getFile 和 isFile 防御式抽象

- 基于 NIO 的 readableChannel 也提供了这个新特性

- 有效的方法参数访问基于 java 8 反射增强

- 在 Spring 核心接口中增加了声明 default 方法的支持一贯使用 JDK 7 Charset 和 StandardCharsets 的增强
兼容 JDK 9

-  Spring 5 框架自带了通用的日志封装

-  spring-jcl 替代了通用的日志，仍然支持可重写

-  持续实例化 via 构造函数(修改了异常处理)

- 自动检测 log4j 2.x, SLF4J, JUL（java.util.Logging）而不是其他的支持

    

#### 核心容器
- 支持候选组件索引(也可以支持环境变量扫描)
- 支持 @Nullable 注解
- 函数式风格 GenericApplicationContext/AnnotationConfigApplicationContext
- 基本支持 bean API 注册
- 在接口层面使用 CGLIB 动态代理的时候，提供事务，缓存，异步注解检测
- XML 配置作用域流式
- Spring WebMVC
- 全部的 Servlet 3.1 签名支持在 Spring-provied Filter 实现
- 在 Spring MVC Controller 方法里支持 Servlet4.0 PushBuilder 参数
- 多个不可变对象的数据绑定(Kotlin/Lombok/@ConstructorPorties)
- 支持 jackson2.9
- 支持 JSON 绑定 API
- 支持 protobuf3
- 支持 Reactor3.1 Flux 和 Mono



#### SpringWebFlux
- 新的 spring-webflux 模块，一个基于 reactive 的 spring-webmvc ，完全的异步非阻塞，旨在使用 enent-loop 执行模型和传统的线程池模型。

- spring-core 相关的基础设施，比如 Encode 和 Decoder 可以用来编码和解码数据流；DataBuffer 可以使用 java - ByteBuffer 或者 Netty ByteBuf; ReactiveAdapterRegistry 可以对相关的库提供传输层支持。

- 在 spring-web 包里包含 HttpMessageReade 和 HttpMessageWrite 

    

#### Kotlin 支持
Kotlin 是一种静态类型的 JVM 语言，它让代码具有表现力，简洁性和可读性。 Spring 5 对 Kotlin 有很好的支持



#### 测试方面的改进
- 完成了对 JUnit 5’s Juptier 编程和拓展模块在 Spring TestContext 框架

- SpringExtension: 是 JUnit 多个可拓展 API 的一个实现，提供了对现存 Spring TestContext Framework 的支持，使用:
@ExtendWith(SpringExtension.class) 注解引用。
@SpringJunitConfig: 一个复合注解
@ContextConfiguration 来源于 Spring TestContext 框架
@DisabledIf  如果提供的该属性值为 true 的表达或占位符，信号：注解的测试类或测试方法被禁用

- 在 Spring TestContext 框架中支持并行测试

- 具体细节查看 Test 章节 通过 SpringRunner 在 Spring TestContext 框架中支持 TestNG, Junit5, 新的执行之前和之后测试回调。

- 在 testexecutionlistener API 和 testcontextmanager 新 beforetestexecution() 和 aftertestexecution() 回调。MockHttpServletRequest 新增了 getContentAsByteArray() 和 getContentAsString() 方法来访问请求体

- 如果字符编码被设置为 mock 请求，在 print() 和 log() 方法中可以打印 Spring MVC Test 的 redirectedUrl() 和forwardedUrl() 方法支持带变量表达式 URL 模板。

- XMLUnit 升级到了 2.3 版本。

    

#### 移除的特性
随着 Java、JavaEE 和其他一些框架基准版本的增加，Spring 5 取消了对几个框架的支持。例如:
- Portlet

- Velocity

- JasperReports

- XMLBeans

- JDO

- Guava

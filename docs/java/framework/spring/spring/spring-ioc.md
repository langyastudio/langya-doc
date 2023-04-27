> 本文来自廖雪峰，郎涯进行简单排版与补充



## IoC 容器

### IoC 原理

容器是一种为某种特定组件的运行提供必要支持的一个软件环境。

使用容器运行组件，除了提供一个组件运行环境之外，容器还提供了许多底层服务。例如，`Servlet` 容器（Tomcat）底层实现了 TCP 连接，解析 HTTP 协议等非常复杂的服务，如果没有容器来提供这些服务，我们就无法编写像 `Servlet` 这样代码简单，功能强大的组件。



传统的应用程序中，控制权在程序本身，程序的控制流程完全由开发者控制，例如：

`CartServlet `创建了 `BookService`，在创建 `BookService` 的过程中，又创建了 `DataSource` 组件。这种模式的缺点是，一个组件如果要使用另一个组件，必须先知道如何正确地创建它。

如果一个系统有大量的组件，其生命周期和相互之间的依赖关系如果由组件自身来维护，不但大大增加了系统的复杂度，而且会导致组件之间极为紧密的耦合，继而给测试和维护带来了极大的困难。

因此，核心问题是：

1. 谁负责创建组件？
2. 谁负责根据依赖关系组装组件？
3. 销毁时，如何按依赖顺序正确销毁？



解决这一问题的核心方案就是 `IoC`。Spring 的核心就是提供了一个 `IoC` 容器，`IoC` 全称 Inversion of Control，直译为**控制反转**。它可以管理所有轻量级的 `JavaBean` 组件，提供的底层服务包括组件的生命周期管理、配置和组装服务、`AOP`支持，以及建立在 `AOP` 基础上的声明式事务服务等。



**在 `IoC` 模式下，控制权发生了反转，即从应用程序转移到了 `IoC` 容器，所有组件不再由应用程序自己创建和配置，而是由 `IoC` 容器负责，这样，应用程序只需要直接使用已经创建好并且配置好的组件。容器中被管理的对象称为Bean。**



Spring是通过元数据和 POJO 来定义和管理 Bean 的：

- POJO：简单的Java对象

- 元数据：描述如何管理POJO的数据



为了能让组件在`IoC`容器中被“装配”出来，需要某种“注入”机制，例如，`BookService` 自己并不会创建 `DataSource`，而是等待外部通过 `setDataSource()` 方法来注入一个 `DataSource`：

```java
public class BookService {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

不直接 `new` 一个 `DataSource`，而是注入一个 `DataSource`，这个小小的改动虽然简单，却带来了一系列好处：

1. `BookService` 不再关心如何创建 `DataSource`，因此，不必编写读取数据库配置之类的代码

2. `DataSource ` 实例被注入到 `BookService`，同样也可以注入到 `UserService`，因此，共享一个组件非常简单

3. 测试 `BookService` 更容易，因为注入的是 `DataSource`，可以使用内存数据库，而不是真实的 `MySQL` 配置

    

因此，`IoC `又称为**依赖注入**（`DI`：Dependency Injection），它解决了一个最主要的问题：**将组件的创建+配置与组件的使用相分离，并且由 `IoC` 容器负责管理组件的生命周期**



因为`IoC`容器要负责实例化所有的组件，因此，有必要告诉容器如何创建组件，以及各组件的依赖关系。

一种最简单的配置是通过XML文件来实现，例如：

```xml
<beans>
    <bean id="dataSource" class="HikariDataSource" />
    <bean id="bookService" class="BookService">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="userService" class="UserService">
        <property name="dataSource" ref="dataSource" />
    </bean>
</beans>
```

上述 XML 配置文件指示 `IoC` 容器创建 3 个 `JavaBean` 组件，并把 id 为 `dataSource` 的组件通过属性 `dataSource`（即调用`setDataSource()`方法）注入到另外两个组件中。

在 Spring 的 `IoC` 容器中，我们把 **所有组件统称为`JavaBean`**，即配置一个组件就是配置一个Bean。



#### 依赖注入方式

我们从上面的代码可以看到，依赖注入可以通过 `set()` 属性方法实现。

但依赖注入也可以通过构造方法实现，很多 Java 类都具有带参数的构造方法，如果我们把 `BookService` 改造为通过构造方法注入，那么实现代码如下：

```java
public class BookService {
    private DataSource dataSource;

    public BookService(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

Spring的 `IoC` 容器同时**支持属性注入和构造方法注入**，并允许混合使用。



#### 无侵入容器

在设计上，Spring 的 `IoC` 容器是一个高度可扩展的无侵入容器。所谓无侵入，是指应用程序的组件无需实现 Spring 的特定接口，或者说，组件根本不知道自己在 Spring 的容器中运行。

1. 应用程序组件既可以在 Spring 的 `IoC` 容器中运行，也可以自己编写代码自行组装配置
2. 测试的时候并不依赖 Spring 容器，可单独进行测试，大大提高了开发效率



### Bean 装配 

- Spring 的 **IoC 容器接口是 `ApplicationContext`**，并提供了多种实现类

- 通过 XML 配置文件创建 IoC 容器时，使用 `ClassPathXmlApplicationContext`

- 持有 IoC 容器后，通过 `getBean()` 方法获取 Bean 的引用



编写一个特定的`application.xml`配置文件，告诉Spring的IoC容器应该如何创建并组装Bean：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService" class="com.itranswarp.learnjava.service.UserService">
        <property name="mailService" ref="mailService" />
    </bean>

    <bean id="mailService" class="com.itranswarp.learnjava.service.MailService" />
</beans>
```

注意观察上述配置文件，其中与 XML Schema 相关的部分格式是固定的，我们只关注两个 `<bean ...>` 的配置：

- 每个 `<bean ...>` 都有一个 `id` 标识，相当于 Bean 的唯一ID
- 在 `userService` Bean中，通过 `<property name="..." ref="..." />` 注入了另一个Bean
- Bean 的顺序不重要，Spring 根据依赖关系会自动正确初始化

把上述 XML 配置文件用 Java 代码写出来，就像这样：

```java
UserService userService = new UserService();
MailService mailService = new MailService();
userService.setMailService(mailService);
```

只不过 Spring 容器是通过读取 XML 文件后使用反射完成的。

如果注入的不是Bean，而是`boolean`、`int`、`String`这样的数据类型，则通过 `value` 注入，例如，创建一个`HikariDataSource`：

```xml
<bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource">
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test" />
    <property name="username" value="root" />
    <property name="password" value="password" />
    <property name="maximumPoolSize" value="10" />
    <property name="autoCommit" value="true" />
</bean>
```

> `application.xml` 放入 resources 中



#### ApplicationContext

我们从创建 Spring 容器的代码：

```java
ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
```

可以看到，Spring 容器就是 `ApplicationContext`，它是一个接口，有很多实现类，这里我们选择 `ClassPathXmlApplicationContext`，表示它会自动从 classpath 中查找指定的 XML 配置文件。

获得了 `ApplicationContext` 的实例，就获得了 IoC 容器的引用。从 `ApplicationContext` 中我们可以根据 Bean 的 ID 获取 Bean，但更多的时候我们根据 Bean 的类型获取 Bean 的引用：

```java
UserService userService = context.getBean(UserService.class);
```



Spring 还提供另一种 IoC 容器叫 `BeanFactory`，使用方式和`ApplicationContext`类似：

```java
BeanFactory factory = new XmlBeanFactory(new ClassPathResource("application.xml"));
MailService mailService = factory.getBean(MailService.class);
```

`BeanFactory `和 `ApplicationContext` 的区别在于，`BeanFactory` 的实现是按需创建，即第一次获取 Bean 时才创建这个 Bean，而 `ApplicationContext` 会一次性创建所有的 Bean。实际上，`ApplicationContext` 接口是从 `BeanFactory` 接口继承而来的，并且，`ApplicationContext`提供了一些额外的功能，包括国际化支持、事件和通知机制等。

> 通常情况下，我们总是使用 `ApplicationContext`，很少会考虑使用 `BeanFactory`



Spring Boot 可在不同的环境下自动创建正确的 IoC 容器:

- AnnotationConfigApplicationContext：默认创建的IoC容器
- AnnotationConfigServletWebServerApplicationContext：在Web应用下创建的IoC容器
- AnnotationConfigReactiveWebServerApplicationContext：在响应式Web应用下创建的IoC容器



## Bean 配置

![img](https://img-note.langyastudio.com/202112101426630.webp?x-oss-process=style/watermark)



### 注解配置

使用 XML 的缺点是写起来非常繁琐，每增加一个组件，就必须把新的 Bean 配置到 XML 中。

可以使用 **Annotation** 配置，可以完全不需要XML，让 Spring 自动扫描 Bean 并组装它们。

- 使用 Annotation 可以大幅简化配置，每个Bean通过 `@Component` 和 `@Autowired` 注入

- 必须合理设计包的层次结构，才能发挥 `@ComponentScan` 的威力



当类注解为 @Component、@Service、@Repository 或 @Controller 时，Spring 容器会自动扫描（通过**@ComponentScan** 实现，Spring Boot 已经做好了配置），并将它们注册成受容器管理的 Bean。

- @Component 被注解类是“组件”

- @Controller 被注解类是“控制器”

- @Service 被注解类是“服务”

- @Repository 被注解类是“数据仓库”


> 在类上注解 @Configuration（@Component 的特例，会被容器自动扫描），可使类成为配置类



给 `MailService` 添加一个 `@Component` 注解：

```java
@Component
public class MailService {
    ...
}
```

这个 `@Component` 注解就相当于定义了一个 Bean，它有一个可选的名称，默认是 `mailService`，即小写开头的类名。

然后，我们给 `UserService` 添加一个 `@Component` 注解和一个 `@Autowired` 注解：

```java
@Component
public class UserService {
    @Autowired
    MailService mailService;

    ...
}
```

使用 `@Autowired` 就相当于把指定类型的 Bean 注入到指定的字段中。和 XML 配置相比，`@Autowired` 大幅简化了注入，因为它不但可以写在 `set()` 方法上，还可以直接写在字段上，甚至可以写在构造方法的函数或参数中：

```java
@Component
public class UserService {
    MailService mailService;

    //or @Autowired
    public UserService(@Autowired MailService mailService) {
        this.mailService = mailService;
    }
    ...
}
```

> 如果 Bean 只有一个构造器，则可以直接省略 @Autowired 注解。若 Bean 有多个构造器，则需注解一个构造器用来注入



最后，编写启动容器类：

```java
@Configuration
@ComponentScan
public class AppConfig {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        UserService userService = context.getBean(UserService.class);      
    }
}
```

除了 `main()` 方法外，`AppConfig` 标注了 `@Configuration`，表示它是一个配置类，因为我们创建`ApplicationContext` 时：

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
```

使用的实现类是 `AnnotationConfigApplicationContext`，必须传入一个标注了 `@Configuration` 的类名。

此外，`AppConfig` 还标注了 `@ComponentScan`，它告诉容器，自动搜索当前类所在的包以及子包，把所有标注为`@Component` 的Bean 自动创建出来，并根据 `@Autowired` 进行装配。



使用 Annotation 配合自动扫描能大幅简化 Spring 的配置，我们只需要保证：

- 每个 Bean 被标注为 `@Component`
- 使用 `@Autowired` 注入（基于构造函数、类属性等）
- **配置类被标注为 `@Configuration` 和 `@ComponentScan`**
- 所有Bean均在指定包以及子包内



### Bean 定制

- Spring 默认使用 Singleton 创建 Bean，也可指定 Scope 为 Prototype

- 可将相同类型的 Bean 注入 `List`

- 可用 `@Autowired(required=false)` 允许可选注入

  适合有定义就使用定义，没有就使用默认值的情况，否则会抛出 `NoSuchBeanDefinitionException` 异常

- **可用带 `@Bean` 标注的方法创建 Bean**

- **相同类型的 Bean 只能有一个指定为  `@Primary` ，其他必须用 `@Quanlifier("beanName")` 指定别名**

  注入时，可通过别名 `@Quanlifier("beanName")` 指定使用某个 Bean



#### Java创建Bean

在 `@Configuration` 类中编写一个 Java 方法创建并返回它，并标记 `@Bean` 注解：

```java
@Configuration
@ComponentScan
public class AppConfig {
    // 创建第三方Bean:
    @Bean
    ZoneId createZoneId() {
        return ZoneId.of("Z");
    }
}
```



#### Scope

一个 Bean 标记为 `@Component` 后，它就会自动为我们创建一个单例（Singleton），即容器初始化时创建 Bean，容器关闭前销毁 Bean。

如果想每次请求 Bean 时都会创建一个实例，这种Bean称为 Prototype（原型），采用`@Scope`标记。

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE) // @Scope("prototype")
public class MailSession {
    ...
}
```



#### Qualifier 使用别名

有些时候，我们需要对一种类型的 Bean 创建多个实例。例如，同时连接多个数据库，就必须创建多个 `DataSource` 实例。

可以用 `@Bean("name")` 指定别名，也可以用 `@Bean`+`@Qualifier("name")` 指定别名。

```java
@Configuration
@ComponentScan
public class AppConfig {
    @Bean
    @Primary
    DataSource createMasterDataSource() {
        ...
    }

    @Bean
    @Qualifier("slave")
    DataSource createSlaveDataSource() {
        ...
    }
}
```

> 在注入时，如果没有指出Bean的名字，Spring会注入标记有`@Primary`的Bean。这种方式也很常用例如对于主从两个数据源，通常将主数据源定义为`@Primary`。一般 `@Primary` 与 `@Qualifier` 配合使用效果更佳。
>
> 但是当全局有多个同类型的 Bean 且没有标记 `@Primary` 会提示 “required a single bean, but n were found”



#### 注入List

有些时候，我们会有一系列接口相同，不同实现类的 Bean。例如，注册用户时，我们要对 email、name 这2个变量进行验证。为了便于扩展，我们先定义验证接口：

```java
public interface Validator {
    void validate(String email, String name);
}
```

然后，分别使用2个`Validator`对用户参数进行验证：

```java
@Component
@Order(1)
public class EmailValidator implements Validator {
    public void validate(String email, String name) {
        if (!email.matches("^[a-z0-9]+\\@[a-z0-9]+\\.[a-z]{2,10}$")) {
            throw new IllegalArgumentException("invalid email: " + email);
        }
    }
}

@Component
@Order(2)
public class NameValidator implements Validator {
    public void validate(String email, String name) {
        if (name == null || name.isBlank() || name.length() > 20) {
            throw new IllegalArgumentException("invalid name: " + name);
        }
    }
}
```

最后，我们通过一个`Validators`作为入口进行验证：

```java
@Component
public class Validators {
    @Autowired
    List<Validator> validators;

    public void validate(String email, String name) {
        for (var validator : this.validators) {
            validator.validate(email, name);
        }
    }
}
```

Spring 会**自动**把所有类型为 `Validator` 的 Bean 装配为一个 `List` 注入进来，这样一来每新增一个 `Validator` 类型，就自动被 Spring 装配到 `Validators` 中，非常方便。

> 因为 Spring 是通过扫描 classpath 获取到所有的 Bean，而 `List` 是有序的，要指定 `List` 中 Bean 的顺序，可以加上 `@Order` 注解



### Bean 生命周期

#### `@PostConstruct` 初始化和销毁

通常会定义一个 `init()` 方法进行初始化（标记 `@PostConstruct`），定义一个 `shutdown()` 方法进行清理（标记`@PreDestroy`）。

需要引入 JSR-250 定义的Annotation：

```xml
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```

初始化流程：

- 调用构造方法创建实例
- 根据 `@Autowired` 进行注入
- 调用标记有 `@PostConstruct` 的 `init()` 方法进行初始化

- 销毁时，容器会首先调用标记有 `@PreDestroy` 的 `shutdown()` 方法



**@Bean 创建与销毁**

如果使用的是 @Bean 的方式创建的 Bean，需要使用 @Bean 的 initMethod 和 destroyMethod 

Bean的定义如下

![image-20210817111131302](https://img-note.langyastudio.com/20210817111131.png?x-oss-process=style/watermark)

在JavaConfig中配置下面的代码

![image-20210729094450208](https://img-note.langyastudio.com/20210729094450.png?x-oss-process=style/watermark)



#### @Lazy 延迟初始化

只要在 Bean 上注解了 @Lazy，那么 Bean 在被调用时才会被初始化。它可以和 @Component 类注解或 @Bean 一起使用。



#### @DependsOn 依赖顺序

设置 Bean lifeService2 依赖于 lifeService，让 lifeService 先初始化，可以用 @DependsOn 来实现。

![image-20210729094810833](https://img-note.langyastudio.com/20210729094810.png?x-oss-process=style/watermark)



### 条件装配

- Spring 允许通过 `@Profile` 配置不同的 Bean

- Spring 还提供了 `@Conditional` 来进行条件装配。Spring Boot 在此基础上进一步提供了基于配置、Class、Bean等条件进行装配

开发应用程序时会使用开发环境，使用内存数据库以便快速启动。而运行在生产环境时会使用生产环境，例如使用MySQL数据库。如果应用程序可以根据自身的环境做一些适配，无疑会更加灵活。



#### Profile 场景

可以通过@Profile注解指定当前的运行场景。@Profile 可以和 @Component、@Configuration、@Bean等一起使用，当然也分别限制了@Profile生效的Bean的分组。



Spring为应用程序准备了Profile这一概念，用来表示不同的环境。例如分别定义开发、测试和生产这3个环境：

- native
- test
- production

创建某个 Bean 时，Spring容器可以根据注解`@Profile`来决定是否创建。例如，以下配置：

```java
@Configuration
@ComponentScan
public class AppConfig {
    @Bean
    @Profile("!test")
    ZoneId createZoneId() {
        return ZoneId.systemDefault();
    }

    @Bean
    @Profile("test")
    ZoneId createZoneIdForTest() {
        return ZoneId.of("America/New_York");
    }
}
```

如果当前的 Profile 设置为 `test`，则 Spring 容器会调用 `createZoneIdForTest()` 创建 `ZoneId`，否则调用`createZoneId() `创建 `ZoneId`。

在运行程序时，加上 JVM 参数 `-Dspring.profiles.active=test` 就可以指定以 `test` 环境启动。

实际上，Spring 允许指定多个Profile，例如：

```java
-Dspring.profiles.active=test,master
```

可以表示 `test` 环境，并使用 `master` 分支代码。

```java
@Bean
@Profile({ "test", "master" }) // 同时满足test和master
ZoneId createZoneId() {
    ...
}
```



#### Conditional 条件装配

除了根据 `@Profile` 条件来决定是否创建某个 Bean 外，Spring 还可以根据 `@Conditional` 决定是否创建某个 Bean。@Conditional 同样可以和 @Component、@Configuration、@Bean 一起使用，进而指定条件起作用的范围。

**Spring 只提供了 `@Conditional` 注解**，具体判断逻辑还需要我们自己实现。

例如，我们对 `SmtpMailService `添加如下注解：

```java
@Component
@Conditional(OnSmtpEnvCondition.class)
public class SmtpMailService implements MailService {
    ...
}
```

它的意思是，如果满足 `OnSmtpEnvCondition` 的条件，才会创建 `SmtpMailService` 这个Bean。`OnSmtpEnvCondition`的条件是什么呢？我们看一下代码：

```java
public class OnSmtpEnvCondition implements Condition {
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return "true".equalsIgnoreCase(System.getenv("smtp"));
    }
}
```

因此，`OnSmtpEnvCondition `的条件是存在环境变量 `smtp`，值为 `true`。这样，我们就可以通过环境变量来控制是否创建 `SmtpMailService`。



#### spring boot 条件装配

Spring Boot 则为我们准备好了几个非常有用的条件：

- @ConditionalOnProperty：如果有指定的配置

- @ConditionalOnBean：如果有指定的Bean

- @ConditionalOnMissingBean：如果没有指定的Bean

- @ConditionalOnMissingClass：如果没有指定的Class

- @ConditionalOnWebApplication：在Web环境中

- @ConditionalOnExpression：根据表达式判断条件




以最常用的 `@ConditionalOnProperty` 为例

首先定义配置 `storage.type=xxx`，用来判断条件，默认为 `local`：

```yml
storage:
  type: ${STORAGE_TYPE:local}
```

设定为 `local` 时，启用 `LocalStorageService`：

```java
@Component
@ConditionalOnProperty(value = "storage.type", havingValue = "local", matchIfMissing = true)
public class LocalStorageService implements StorageService {
    ...
}
```

设定为 `aws` 时，启用 `AwsStorageService`：

```java
@Component
@ConditionalOnProperty(value = "storage.type", havingValue = "aws")
public class AwsStorageService implements StorageService {
    ...
}
```

设定为 `aliyun` 时，启用 `AliyunStorageService`：

```java
@Component
@ConditionalOnProperty(value = "storage.type", havingValue = "aliyun")
public class AliyunStorageService implements StorageService {
    ...
}
```

注意到 `LocalStorageService` 的注解，当指定配置为 `local` 或者配置不存在，均启用 `LocalStorageService`。

其他需要存储的服务则注入 `Uploader`：

```java
@Componentpublic class UserImageService { 
    @Autowired    
    StorageService storageService;
}
```

当应用程序检测到配置文件存在 `storage.type=aws` 时，自动使用 `AwsStorageService`，如果存在配置 `storage.type=local`，或者配置 `storage.type `不存在，则使用 `LocalStorageService`。

可见，Spring Boot 提供的条件装配使得应用程序更加具有灵活性。



如果当前 classpath 中存在类 `javax.mail.Transport`，则创建 `MailService`：

```java
@Component
@ConditionalOnClass(name = "javax.mail.Transport")
public class MailService {
    ...
}
```



### 开启配置

@Enable 会自动对相应的功能进行自动配置，如 @EnableWebMvc、@EnableCaching、@EnableScheduling、@EnableAsync、@EnableWebSocket、@EnableJpaRepositories、@EnableTransactionManagement、@EnableJpaAuditing 和 @EnableAspectJAutoProxy等。



@Enable* 的开启配置的功能依赖于 @Import 注解，@Import 注解支持导入如下配置：

- 直接导入@Configuration配置类

- 配置类选择器 ImportSelector 的实现

- 动态注册器 ImportBeanDefinitionRegistrar 的实现

- 混合以上三种



### CommandLineRunner

在 Spring Boot 下可以注册一个 CommandLineRunner 的 Bean，在容器启动后这个 Bean 可用来执行一些专门的任务，如在 JavaConfig 里。

> CommandLineRunner 是一个函数接口，输入的参数为 main 方法里接收的 args 参数
>
> CommandLineRunner 有个姊妹接口叫作 ApplicationRunner

```java
@Bean
CommandLineRunner asyncTaskClr(AsyncTask asyncTask)
{
    return (args) -> {
        for (int ix=0; ix<10; ix++)
        {
            asyncTask.loopPrint(ix);
        }
    };
}
```



## 配置文件

- Environment 代表当前运行的应用环境
- Spring 容器可以通过 `@PropertySource` 自动读取配置文件，并以 `@Value("${key}")` 的形式注入；可以通过 `${key:defaultValue}` 指定默认值

- 以  `#{bean.property}` 形式注入时，Spring 容器自动把指定 Bean 的指定属性值注入

  

**Spring EL**（Spring Expression Language，Spring 表达式语言）是 Spring 生态下的通用语言，在运行时使用表达式查询属性信息（使用符号$）或操作 Java 对象（使用符号#），主要用在 XML 或注解上

​    

### 配置文件

在开发应用程序时，经常需要读取配置文件。最常用的配置方法是以 `key=value` 的形式写在 `.properties` 文件中。

例如，`MailService `根据配置的 `app.zone=Asia/Shanghai` 来决定使用哪个时区

只需要在 `@Configuration `配置类上再添加一个注解：

```java
@Configuration
@ComponentScan
@PropertySource("app.properties") // 表示读取classpath的app.properties
public class AppConfig {
    @Value("${app.zone:Z}")
    String zoneId;

    @Bean
    ZoneId createZoneId() {
        return ZoneId.of(zoneId);
    }
}
```

> @PropertySource 支持 XML 格式和 properties 格式，不支持Spring Boot下的 YAML 格式
>
> 当有多个外部配置时，可以用 @PropertySources 指定
>
> 如果想支持YAML：
>
> https://blog.csdn.net/qq_37960603/article/details/122971803 
>
> https://blog.csdn.net/feinifi/article/details/127580474



还可以把注入的注解写到方法参数中：

```java
@Bean
ZoneId createZoneId(@Value("${app.zone:Z}") String zoneId) {
    return ZoneId.of(zoneId);
}
```



另一种注入配置的方式是先通过一个简单的JavaBean持有所有的配置，例如，一个`SmtpConfig`：

```java
@Component
public class SmtpConfig {
    @Value("${smtp.host}")
    private String host;

    @Value("${smtp.port:25}")
    private int port;

    public String getHost() {
        return host;
    }

    public int getPort() {
        return port;
    }
}
```

然后，在需要读取的地方，使用 `#{smtpConfig.host}` 注入：

```java
@Component
public class MailService {
    @Value("#{smtpConfig.host}")
    private String smtpHost;

    @Value("#{smtpConfig.port}")
    private int smtpPort;
}
```

> 它和 `${key}` 不同的是，`#{}` 表示从 JavaBean 读取属性。`"#{smtpConfig.host}" `的意思是，从名称为`smtpConfig `的 Bean 读取 `host` 属性，即调用 `getHost()` 方法。一个 Class 名为 `SmtpConfig` 的 Bean，它在Spring 容器中的默认名称就是 `smtpConfig`，除非用 `@Qualifier` 指定了名称



### 使用Resource

Spring 提供了一个 `org.springframework.core.io.Resource`（注意不是`javax.annotation.Resource`），它可以像`String`、`int` 一样使用 `@Value` 注入。

```java
@Component
public class AppService {
    @Value("classpath:/logo.txt")
    private Resource resource;

    @PostConstruct
    public void init() throws IOException {
        try (var reader = new BufferedReader(
                new InputStreamReader(resource.getInputStream(), StandardCharsets.UTF_8))) {
            String logo = reader.lines().collect(Collectors.joining("\n"));
        }
    }
}
```

> 最常用的注入是通过 classpath 以 `classpath:/path/to/file` 的形式注入
>
> 使用 Maven 的标准目录结构，所有资源文件放入 `src/main/resources` 即可



也可以直接指定文件的路径，例如：

```java
@Value("file:/path/to/logo.txt")
private Resource resource;
```



## BeanPostProcessor

注解本身只是元数据，即描述数据的数据，被描述的数据可以是类、方法、属性、参数或构造器等。注解本身是没有任何可执行功能的代码，但是只要标注了注解，我们就能得到想要的功能。

在 BeanPostProcessor 实现中，只要名称为 *AnnotationBeanPostProcessor，就都是针对处理注解的，即对容器内标注了指定注解的Bean进行功能处理：

- AutowiredAnnotationBeanPostProcessor： 让@Autowired、@Value和@Inject注解生效
- CommonAnnotationBeanPostProcessor：让@PostConstruct和@PreDestroy注解生效
- AsyncAnnotationBeanPostProcessor：让@Async或@Asynchronous注解生效
- ScheduledAnnotationBeanPostProcessor：让@Scheduled注解生效
- PersistenceAnnotationBeanPostProcessor：让@PersistenceUnit和@PersistenceContext注解生效
- JmsListenerAnnotationBeanPostProcessor：让@JmsListener注解生效



可以通过实现 BeanPostProcessor 接口，在构造时对容器内所有或者部分指定 Bean 进行处理。和 @PostConstruct 与@PreDestroy 不同的是，**它针对的是 IoC 容器里的所有的 Bean**。

![image-20210729102457706](https://img-note.langyastudio.com/20210729102457.png?x-oss-process=style/watermark)

通过覆写 postProcessBeforeInitialization 和 postProcessAfterInitialization 方法，所有的 Bean 在初始化之前都会执行 postProcessBeforeInitialization 里的处理逻辑，在初始化之后都会执行 postProcessAfterInitialization 里的处理逻辑。执行结果如图3-14所示。



如果想要缩小 Processor 的处理范围，则可以通过判断 Bean 类型来实现。

![image-20210729102601946](https://img-note.langyastudio.com/20210729102602.png?x-oss-process=style/watermark)



### BeanFactoryPostProcessor

针对 Bean 的配置元数据（注解等）进行处理操作的，它属于 BeanFactory 的职责范畴。

◎ConfigurationClassPostProcessor：使@PropertySource、@ComponentScan、@Component类、Configuration、@Bean、@Import 和 @ImportResource 注解生效

◎EventListenerMethodProcessor：使@EventListener注解生效



## Bean 事件通讯

如果 Bean 之间需要通信，比如说 BeanA 完成了处理后需要告知 BeanB，通知 BeanB 继续处理，那么我们称 BeanA 为Publisher，称 BeanB 为Listener。



Publisher和Listener之间传递的事件数据通过继承 `ApplicationEvent` 来实现。

**Publisher的实现方式如下：**

- 通过 ApplicationEventPublisherAware 注入 ApplicationEventPublisher 发布事件

- 直接注入 ApplicationEventPublisher 发布事件

- 直接注入 ApplicationContext 发布事件



**Listener的实现方式如下：**

- 实现 ApplicationListener 接口

- 注解 **@EventListener** 的方法接收事件

  

事件数据：

![image-20210729105434001](https://img-note.langyastudio.com/20210729105434.png?x-oss-process=style/watermark)



发布者：

![image-20210729105457354](https://img-note.langyastudio.com/20210729105457.png?x-oss-process=style/watermark)

![image-20210729105739781](https://img-note.langyastudio.com/20210729105739.png?x-oss-process=style/watermark)



监听者：

![image-20210729105514961](https://img-note.langyastudio.com/20210729105515.png?x-oss-process=style/watermark)

> EventListenerService 实现了 ApplicationListener<MessageEvent>接口，泛型 MessageEvent 可缩小监听事件的范围，通过覆写 onApplicationEvent 方法来监听事件。



监听者还可以注解 @EventListener 方法来监听事件。

![image-20210729105548098](https://img-note.langyastudio.com/20210729105548.png?x-oss-process=style/watermark)

**推荐使用 @EventListener 注解方式，耦合度更低。**



除能监听自定义发布的事件外，还可以监听系统发布的事件：

主要的系统事件如下。

- ContextRefreshedEvent：当ApplicationContext被初始化或刷新时发布该事件

- ContextStartedEvent：当ApplicationContext开始时发布该事件

- ContextStoppedEvent：当ApplicationContext停止时发布该事件

- ContextClosedEvent：当ApplicationContext关闭时发布该事件
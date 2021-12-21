> 本文来自廖雪峰，郎涯进行简单排版与补充



## 装配AOP

AOP 是 Aspect Oriented Programming，即面向切面编程。AOP 把系统分解为不同的关注点或者称之为切面（Aspect）。Spring 的 AOP 实现就是**基于 JVM 的动态代理，让我们把一些常用功能如权限检查、事务等，从每个业务方法中剥离出来**。

在 Spring 容器中使用 AOP 非常简单，只需要定义执行方法，并用 **AspectJ** 的注解标注应该在何处触发并执行。

Spring 通过 CGLIB 动态创建子类等方式来实现 AOP 代理模式，大大简化了代码。



在学习AOP之前，先来熟悉下面的概念。

- 切面：Aspect，编写额外行为的地方

- 连接点：Join Point，被拦截的方法

- 切点：PointCut，通过条件匹配一批连接点

- 建言：Advice，对于每个连接点需要做的行为
- 目标对象：符合指定条件的Bean



### 装配AOP

> **在实际项目中，这种写法其实很少使用，更多使用注解装配AOP**
>
> 因为容易污染，即很多不需要AOP代理的Bean也被自动代理了

首先，通过 Maven 引入 Spring 对 AOP 的支持：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>${spring.version}</version>
</dependency>
```



定义一个`LoggingAspect`：

```java
@Aspect
@Component
public class LoggingAspect {
    // 在执行UserService的每个方法前执行:
    @Before("execution(public * com.itranswarp.learnjava.service.UserService.*(..))")
    public void doAccessCheck() {
        System.err.println("[Before] do access check...");
    }

    // 在执行MailService的每个方法前后执行:
    @Around("execution(public * com.itranswarp.learnjava.service.MailService.*(..))")
    public Object doLogging(ProceedingJoinPoint pjp) throws Throwable {
        System.err.println("[Around] start " + pjp.getSignature());
        Object retVal = pjp.proceed();
        System.err.println("[Around] done " + pjp.getSignature());
        return retVal;
    }
}
```

> `@Around` 可以决定是否执行目标方法



紧接着，我们需要给 `@Configuration` 类加上一个 `@EnableAspectJAutoProxy` 注解来开启对 AspectJ 的支持（SpringBoot 已经自动做了配置，所以无须额外声明）：

```java
@Configuration
@ComponentScan
@EnableAspectJAutoProxy
public class AppConfig {
    ...
}
```

> Spring 的 IoC 容器看到这个注解，就会自动查找带有 `@Aspect` 的Bean



### 拦截器类型

顾名思义，拦截器有以下类型：

- @Before：这种拦截器先执行拦截代码，再执行目标代码。如果拦截器抛异常，那么目标代码就不执行了
- @After：这种拦截器先执行目标代码，再执行拦截器代码。无论目标代码是否抛异常，拦截器代码都会执行
- @AfterReturning：和@After不同的是，只有当目标代码正常返回时，才执行拦截器代码
- @AfterThrowing：和@After不同的是，只有当目标代码抛出了异常时，才执行拦截器代码
- @Around：能完全控制目标代码是否执行，并可以在执行前后、抛异常后执行任意拦截代码，可以说是包含了上面所有功能



AspectJ的注入语法则比较复杂，请参考[Spring文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-pointcuts-examples)。



## 使用注解配置AOP

使用注解实现 AOP 需要先定义注解，然后使用 `@Around("@annotation(paramname)")` 实现装配；

使用注解既简单，又能明确标识 AOP 装配，是使用 AOP **推荐** 的方式。



演示如何使用注解实现 AOP。为了监控应用程序的性能，我们定义一个性能监控的注解：

```java
@Target(METHOD)
@Retention(RUNTIME)
public @interface MetricTime {
    String value();
}
```

在需要被监控的关键方法上标注该注解：

```java
@Component
public class UserService {
    @MetricTime("register")
    public User register(String email, String password, String name) {
        ...
    }
    ...
}
```

最后定义 `MetricAspect`：

```java
@Aspect
@Component
public class MetricAspect {
    @Around("@annotation(metricTimeParam)")
    public Object metric(ProceedingJoinPoint joinPoint, MetricTime metricTimeParam) throws Throwable {
        String name = metricTimeParam.value();
        long  start = System.currentTimeMillis();
        try {
            return joinPoint.proceed();
        } finally {
            long t = System.currentTimeMillis() - start;
           
            // 写入日志或发送至JMX:
            System.err.println("[Metrics] " + name + ": " + t + "ms");
        }
    }
}
```

注意 `metric()` 方法标注了 `@Around("@annotation(metricTime)")`，它的意思是符合条件的目标方法是带有`@MetricTime` 注解的方法。

有了 `@MetricTime` 注解，再配合 `MetricAspect`，任何 Bean，只要方法标注了 `@MetricTime` 注解，就可以自动实现性能监控。



## AOP 逼坑指南

- 由于 Spring 通过 CGLIB 实现代理类，我们要避免直接访问 Bean 的字段以及由 `final` 方法带来的“未代理”问题

- 遇到 CglibAopProxy 的相关日志，务必要仔细检查，防止因为 AOP 出现 NPE 异常

无论是使用 AspectJ 语法，还是配合 Annotation，使用 AOP，实际上就是让 Spring 自动为我们创建一个 **Proxy**，使得调用方能无感知地调用指定方法，但运行期却动态“织入”了其他逻辑，因此，AOP 本质上就是一个[代理模式](https://www.liaoxuefeng.com/wiki/1252599548343744/1281319432618017)。



Spring 使用 CGLIB 构造的 Proxy 类（基于子类实现代理功能），是直接生成字节码， 并没有源码-编译-字节码这个步骤，所以不会调用 `super()` 方法，因此：



**不会初始化代理类自身继承的任何成员变量，包括 final 类型的成员变量！**



因此，正确使用 AOP，我们需要一个避坑指南：

- 访问被注入的 Bean 时，总是调用方法而非直接访问字段

- 编写 Bean 时，如果可能会被代理，就不要编写 `public final` 方法

这样才能保证有没有 AOP，代码都能正常工作
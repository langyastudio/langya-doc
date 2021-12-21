> 先看 [spring boot 2.x 深度理解定时任务schedule](https://mp.weixin.qq.com/s/XHAOY8T5wYkyQECGat3WIg)
>
> 基于上述代码修改
>
> 源码：https://github.com/langyastudio/langya-tech/tree/springboot/async
>
> 官方文档： https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#scheduling-annotation-support



工作中经常涉及异步任务，通常是使用多线程技术，比如线程池 ThreadPoolExecutor，它的执行规则如下（图片来自网络）：

![img](https://img-note.langyastudio.com/20210816221915.webp?x-oss-process=style/watermark)



当使用 Spring 进行开发时，除使用 **`@EnableAsync` 和 `@Async`** 注解外，还需定义类型为 `TaskExecutor` 的 Bean。庆幸的是 Spring Boot 提供了自动配置 `TaskExecutionAutoConfiguration`，它自动注册了一个 Bean（名称为 applicationTaskExecutor）的 **ThreadPoolTaskExecutor**（TaskExecutor 的实现类），所以在 Spring Boot 中只需使用`@EnableAsync` 和 `@Async` 两个注解即可完成多线程异步的操作。



### @Async 使用步骤

Spring boot 提供了异步多线程任务的功能，只需通过：

- `@EnableAsync`

  开启对异步多线程任务的支持，可以作用在在配置类或者 Main 类上

- `@Async`

  注解需要执行的异步多线程任务。可以作用在类上或者方法上，作用在类上代表这个类的所有方法都是异步方法。



在入口类开启对异步多线程任务的支持:

```java
@SpringBootApplication
@EnableAsync
public class Application
{
    public static void main(String[] args)
    {
        ...
    }
}
```

定义一个包含异步多线程任务的类:

```kotlin
@Component
@Log4j2
public class AsyncTask
{
    @Async
    public void loopPrint(Integer i)
    {
        log.info("async task:" + i);
    }
}
```

通过 CommandLineRunner 进行检验:

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

运行程序执行结果如下：

```
[task-1] com.langyastudio.springboot.common.middleware.task.AsyncTask : async task:0
[task-1] com.langyastudio.springboot.common.middleware.task.AsyncTask : async task:8
[task-1] com.langyastudio.springboot.common.middleware.task.AsyncTask : async task:9
[task-2] com.langyastudio.springboot.common.middleware.task.AsyncTask : async task:1
[task-3] com.langyastudio.springboot.common.middleware.task.AsyncTask : async task:2
[task-7] com.langyastudio.springboot.common.middleware.task.AsyncTask : async task:6
[task-6] com.langyastudio.springboot.common.middleware.task.AsyncTask : async task:5
[task-8] com.langyastudio.springboot.common.middleware.task.AsyncTask : async task:7
[task-5] com.langyastudio.springboot.common.middleware.task.AsyncTask : async task:4
[task-4] com.langyastudio.springboot.common.middleware.task.AsyncTask : async task:3
```

从结果可以看出，异步任务使用的不是主线程 `restartedMain`，而是一个线程池，默认有 8 个线程，线程名以 `task-` 开头。执行结果是乱序的，这意味着任务是并发执行的。



### TaskExecutor

线程池的线程数、线程名前缀等信息如何自定义呢？答案是 `ThreadPoolTaskExecutor`，通过它可以自定义具体异步多线程任务的线程数、线程名称前缀等信息。

当系统中有多个不同的异步多线程任务时，通过这种方式可以给特定的任务指定 `ThreadPoolTaskExecutor`。Spring Boot 提供了 TaskExecutorBuilder 的 Bean，可以用它新建我们定制的 ThreadPoolTaskExecutor。

示例：

```java
@Bean
ThreadPoolTaskExecutor customTaskExecutor(TaskExecutorBuilder builder)
{
    return builder.threadNamePrefix("langyatask")
            .corePoolSize(5)
            .build();
}
```

在多线程异步任务的注解中指定 TaskExecutor 即可：

```java
@Component
@Log4j2
public class AsyncTask
{
    @Async("customTaskExecutor")
    public void loopPrint(Integer i)
    {
        log.info("async task:" + i);
    }
}
```

此时线程数、线程名称前缀等将按照 `customTaskExecutor` 定义的值执行。



### Future 

有时候我们不止希望异步执行任务，还希望任务执行完成后会有一个返回值，在 java 中提供了 Future 泛型接口，用来接收任务执行结果，spring boot 也提供了此类支持，使用实现了 ListenableFuture 接口的类如 AsyncResult 来作为返回值的载体。

#### ListenableFuture

比如希望返回一个类型为 String 类型的值，可以将返回值改造为:

```java
@Async("customTaskExecutor")
public ListenableFuture<String> loopPrint(Integer i)
{
    String res = "async task:" + i;
    log.info(res);
    
    return new AsyncResult<>(res);
}
```



#### 调用返回值

```kotlin
@Autowired
private AsyncTask asyncTask;

// 阻塞调用
asyncTask.loopPrint(1).get();

// 限时调用
asyncTask.loopPrint(2).get(1, TimeUnit.SECONDS)
```



#### 构造函数

You can not use `@Async` in conjunction with lifecycle callbacks such as `@PostConstruct`. To asynchronously initialize Spring beans, you currently have to use a **separate initializing Spring bean** that then invokes the `@Async` annotated method on the target, as the following example shows:

```java
public class SampleBeanImpl implements SampleBean {
    @Async
    void doSomething() {
        // ...
    }
}

public class SampleBeanInitializer {

    private final SampleBean bean;

    public SampleBeanInitializer(SampleBean bean) {
        this.bean = bean;
    }

    @PostConstruct
    public void initialize() {
        bean.doSomething();
    }
}
```



> 参考文档：廖雪峰、《从企业级开发到云原生微服务：Spring Boot实战 》等

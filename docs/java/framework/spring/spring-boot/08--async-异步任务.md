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



### 案例分析

> 来自 https://mp.weixin.qq.com/s/CqwgrvzIUrpa987kNx3eRg

在很久很久之前，我有一段痛苦的记忆。那种被故障所驱使的感觉，在我脑海里久久无法驱散。

原因无它，有小伙伴开启了线程池的暴力使用模式。没错，就是下面这篇文章。

[夺命故障 ! 炸出了投资人！](https://mp.weixin.qq.com/s?__biz=MzA4MTc4NTUxNQ==&mid=2650523924&idx=1&sn=b247bae74ccea19290cbbc7b3b43edbc&scene=21#wechat_redirect)

我有必要简单的复述一下。其主要原因，就是开发人员，在每一次方法调用里，都创建了一个单独的线程池去处理。这样的话，如果请求量一增加，整个操作系统的压力就会耗尽，最终所有的业务都无法响应。

![图片](https://img-note.langyastudio.com/202203041018455.png?x-oss-process=style/watermark)

我一直认为这是一个非常偶发的低级错误，发生频率非常的低。但随着这样的故障越来越多，xjjdog 认识到这是一个普遍的现象。

以异步性能优化为目的，反而带来的整体业务不可用的结果，是非常打脸的一种优化。



#### Spring 的异步代码

Spring 作为 Java 届的杠把子框架，其过度封装的 API 深得开发人员的喜爱。根据语义化编程的逻辑，只要某些关键字在语言层面上过得去，我们就可以把它给加上去。比如 `@Async` 注解。

我永远想不通是什么给了开发人员勇气，去加上这个 `@Async` 注解，因为这种涉及到多线程的东西，即使是自己去创建线程，也是心怀敬畏，唯恐扰了操作系统的安宁。`@Async` 这样的黑盒，真的可以那么顺畅的使用么？

我们不妨 debug 一下代码，让子弹飞一会儿。

首先，生成一个小小的项目，然后在主类上加上必须的注解。嗯，别忘了这一环，否则你后面加的注解将没什么用处

```java
@SpringBootApplication
@EnableAsync
public class DemoApplication {}
```

创造一个带 @Async 注解的方法

```java
@Component
public class AsyncService {
    @Async
    public void async(){
        try {
            Thread.sleep(1000);
            System.out.println(Thread.currentThread());
        }catch (Exception ex){
            ex.printStackTrace();
        }
    }
}
```

然后，做一个对应的 test 接口，访问时会调用这个 async 方法

```java
@ResponseBody
@GetMapping("test")
public void test(){
    service.async();
}
```

访问时，直接打个断点，即可获取执行异步线程的线程池

![图片](https://img-note.langyastudio.com/202203041018902.png?x-oss-process=style/watermark)

可以看到，异步任务使用了一个线程池，它的 corePoolSize=8, **阻塞队列采用了无界队列 LinkedBlockingQueue**。一旦采用了这样的组合，最大线程数就会形同虚设，因为超出 8 个线程的任务，将全部会被放到无界队列里。使得下面的代码变成了摆设。

```java
throw new TaskRejectedException("Executor [" + executor + "] did not accept task: " + task, var4);
```

如果你的访问量非常大，这些任务将全部堆积在 LinkedBlockingQueue 里。情况好一点的，这些任务的执行会变得延迟很大；情况坏一点的，任务太多将直接造成内存溢出 OOM！

你可能会说，我可以自己指定另外一个 ThreadPoolExceute，然后使用 @Async 注解来声明啊。说这话的同学，一定是能力比较强，或者 Review 的代码比较少，没有经过猪队友的洗礼。



#### SpringBoot 救了你

SpringBoot 是个好东西。

在 TaskExecutionAutoConfiguration中，通过生成 ThreadPoolTaskExecutor 的Bean，来提供默认的 Executor。

```java
@ConditionalOnMissingBean({Executor.class})
public ThreadPoolTaskExecutor applicationTaskExecutor(TaskExecutorBuilder builder) {
    return builder.build();
}
```

也就是我们上面所说的那个。如果**没有 SpringBoot 的助力，Spring 将默认使用 SimpleAsyncTaskExecutor**。

参见 org.springframework.aop.interceptor.AsyncExecutionInterceptor

```java
@Override
@Nullable
protected Executor getDefaultExecutor(@Nullable BeanFactory beanFactory) {
 Executor defaultExecutor = super.getDefaultExecutor(beanFactory);
 return (defaultExecutor != null ? defaultExecutor : new SimpleAsyncTaskExecutor());
}
```

这就是 Spring 大仙所干的事。

**SimpleAsyncTaskExecutor 类设计的非常操蛋，因为它每执行一次，都会创建一个单独的线程**，根本没有共用线程池。比如你的 TPS 是1000，异步执行了任务，那么你每秒将会生成 1000 个线程！

这明显是想要累死操作系统的节奏。

```java
protected void doExecute(Runnable task) {
 Thread thread = (this.threadFactory != null ? this.threadFactory.newThread(task) : createThread(task));
 thread.start();
}
```



#### End

明眼人一看，这种使用 new 线程的处理方式将会是非常可怕的。但就拿 Spring 本身来说，引用SimpleAsyncTaskExecutor 这个类的地方还不少，包括比较流行的 AsyncRestTemplate。

![图片](https://img-note.langyastudio.com/202203041018576.png?x-oss-process=style/watermark)

这暴露了很多风险，尤其是竟然在这些列表中看到了 redis 的身影。这个类的设计，使得任务的执行变的非常的不可控。

看这个 API，我感觉 Spring 是进入了设计的魔怔状态。

这个东西的隐藏 bug 可能还会更深！比如 org.springframework.context.event.EventListener 注解，用于实现 DDD 那套所谓的事件驱动模式，有不少框架直接 set 了 SimpleAsyncTaskExecutor，那么就等死吧。

赶紧把 SimpleAsyncTaskExecutor 加入你的API黑名单，或者埋坑清单吧！

创建线程有那么难么？需要使用 Spring 创建的线程？有时候我实在是想不通，暴露出这样的接口目的是为了什么。

就连原生的线程池我们还没搞明白呢，你还给包了一层，这是方便我们甩锅啊！



> 参考文档：廖雪峰、《从企业级开发到云原生微服务：Spring Boot实战 》等

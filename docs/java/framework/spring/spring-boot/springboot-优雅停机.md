> https://mp.weixin.qq.com/s/y733CyxK-LqhhPfXemUEKA

## 介绍

什么叫优雅停机？

简单的说，就是向应用进程发出停止指令之后，能**保证正在执行的业务操作不受影响**，直到操作运行完毕之后再停止服务。应用程序接收到停止指令之后，会进行如下操作：

- 停止接收新的访问请求
- 正在处理的请求，等待请求处理完毕；对于内部正在执行的其他任务，比如定时任务、mq 消费等等，也要等当前正在执行的任务执行完毕，并且不再启动新的任务
- 当应用准备关闭的时候，按需向外发出信号，告知其他应用服务准备接手，以保证服务高可用

如果暴力的关闭应用程序，比如通过 `kill -9 <pid>` 命令强制直接关闭应用程序进程，可能会导致正在执行的任务数据丢失或者错乱，也可能会导致任务所持有的全局资源等不到释放，比如当前任务持有 redis 的锁，并且没有设置过期时间，当任务突然被终止并且没有主动释放锁，会导致其他进程因无法获取锁而不能处理业务。

那么如何在不影响正在执行的业务的情况下，将应用程序安全的进行关闭呢？



## 方案实践

SpringBoot 官方文档上，已经告诉开发者只需要实现特定接口即可监听到项目启动成功与关闭时的事件，相关接口如下：

- `CommandLineRunner` 接口：当应用启动成功后但在开始接受流量之前，会回调此接口的实现类，也可以实现`ApplicationRunner` 接口，工作的方式与 `CommandLineRunner` 与之类似
- `DisposableBean` 接口：当应用正要被销毁前，会回调此接口的实现类，也可以使用 `@PreDestroy` 注解，被标记的方法也会被调用

基于此流程，我们可以创建一个服务监听类，用于监听到项目启动成功与关闭时的回调服务，示例代码如下：

```java
@Component
public class AppListener implements CommandLineRunner, DisposableBean {

    @Override
    public void run(String... args) throws Exception {
        System.out.println("应用启动成功，预加载相关数据");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("应用正在关闭，清理相关数据");
    }

}
```

每一个 `SpringApplication` 在启用的时候，都会向 JVM 注册一个关闭钩子 `shutdown hook`，以确保`ApplicationContext` 在退出的时候，通过这个勾子通知 JVM，实现服务正常的关闭，以下介绍的所有关闭服务的方法，都是基于这一原理进行实现的。



### 通过 Actuator 的 Endpoint 机制关闭服务

使用此方法，需要先添加 `spring-boot-starter-actuator` 监控服务依赖包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

默认配置下，`shutdown` 端点是关闭的，需要在 `application.properties` 里配置里面开启：

```java
management.endpoint.shutdown.enabled=true
```

虽然 `Actuator` 的端点，支持通过 `JMX` 或 `HTTP` 进行远程访问。而 `shutdown` 默认配置下是不支持 `HTTP` 进行 `Web` 访问的，所以使用 `HTTP` 请求进行关闭时的配置，也需要开启：

```java
management.endpoints.web.exposure.include=shutdown
```

最后将 `SpringBoot` 服务启动之后，使用 `POST` 请求类型，调用以下接口，即可实现关闭服务！

```java
http://127.0.0.1:8080/actuator/shutdown
```

![图片](https://img-note.langyastudio.com/202302181513412.png?x-oss-process=style/watermark)



### 使用 ApplicationContext 的 close 方法关闭服务

如果你不想添加 `spring-boot-starter-actuator` 监控服务依赖包来关停服务，也可以使用 `ApplicationContext`的 `close` 方法来关停服务，他会自动销毁 `bean` 对象并关停服务。

只需要在应用启用的时候，获取 `ApplicationContext` 对象，然后在相关的位置调用 `close` 方法，就可以关闭服务。

示例代码如下：

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
      ConfigurableApplicationContext context = SpringApplication.run(Application.class, args);

      try {
         TimeUnit.SECONDS.sleep(10);
      } catch (InterruptedException e) {
         e.printStackTrace();
      }
      //启动10秒以后，自动关闭
      context.close();
    }
}
```

当然我们也可以自己写一个 `Controller`，获取对应的 `ApplicationContext` 对象，通过 `api` 操作调用 `close` 方法关停服务，示例代码如下：

```java
@RestController
public class ShutdownController implements ApplicationContextAware {

    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.context = applicationContext;
    }

    /**
     * 关闭服务
     */
    @GetMapping("/shutdown")
    public void shutdownContext() {
        ((ConfigurableApplicationContext) context).close();
    }
}
```



### 监听服务 pid，通过 kill 方式关闭服务

通过 `api` 方式来关停服务，在很多人看来并不安全，因为一旦接口泄漏了，意味着用户可以随便请求这个接口来关闭服务，其影响不言而喻，因此很多人建议在服务端，通过其他的方式来关闭服务，比如通过进程命令方式来关停。

在 `springboot` 启动的时候将应用进程 ID 写入一个 `app.pid` 文件，生成的路径可以指定，然后通过脚本命令方式来关闭服务。

启动示例代码如下：

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(Application.class);
        application.addListeners(new ApplicationPidFileWriter("/home/app/project1/app.pid"));
        application.run();
    }
}
```

通过如下命令方式，可以安全的关闭服务。

```java
cat /home/app/project1/app.pid | xargs kill
```

这种方式，也是目前在 `linux` 操作系统中，使用较为普遍的一种解决方案，区别在于实现的方式可能不同，有的不用写文件，通过其他方式来获取应用进程 ID。

**如果使用 `kill -9 <pid>` 的方式关闭服务，服务的监听器不会收到任何消息，类似于直接强杀应用进程，此方法不可取**！



### 使用 SpringApplication 的 exit 方法关闭服务

通过调用一个 `SpringApplication.exit() `方法也可以安全的退出程序，同时会返回一个退出码，这个退出码可以传递给所有的 `context`，最后通过调用 `System.exit()` 可以将这个错误码也传给 `JVM`。

示例代码如下：

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Application.class, args);

        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //5秒后，关闭服务
        exitApplication(context);
    }

    public static void exitApplication(ConfigurableApplicationContext context) {
     //获取退出码
        int exitCode = SpringApplication.exit(context, (ExitCodeGenerator) () -> 0);
        //退出码传递给jvm，安全退出程序
        System.exit(exitCode);
    }

}
```



## 其他监听介绍

### ApplicationListener

如果有些服务，比如定时任务，我们想在 `SpringBoot` 关闭数据源连接池之前，将其关闭，可以通过实现`ApplicationListener` 接口，监听 `bean` 对象的变化情况，在 `bean` 对象销毁之前，执行相关的关闭任务。

示例代码如下：

```java
@Component
public class JobTaskListener implements ApplicationListener {

    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent) {
        // 在spring bean容器销毁之前执行的事件，防止数据库连接池在任务终止前销毁
        if (applicationEvent instanceof ContextClosedEvent) {
            System.out.println("关闭相关的定时任务");
        }
    }
}
```



### PreDestroy

上文中，我们提到了实现 `DisposableBean` 接口，可以监听应用关闭前的回调处理，其实在自定义的方法上加`@PreDestroy` 注解，也可以实现相同的效果。

示例代码如下：

```java
@Component
public class AppDestroyConfig {

    @PreDestroy
    public void PreDestroy(){
        System.out.println("应用程序正在关闭。。。");
    }
}
```


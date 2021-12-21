> 本文来自廖雪峰，郎涯进行简单排版与补充



Spring 内置定时任务和 Cron 任务的支持，编写调度任务十分方便

在很多应用程序中，经常需要执行定时任务。例如，每天或每月给用户发送账户汇总报表，定期检查并发送系统状态报告，等等。

无需额外的依赖，我们可以直接在 `AppConfig` 中加上 **`@EnableScheduling`** 就开启了定时任务的支持：

```java
@Configuration
@ComponentScan
@EnableWebMvc
@EnableScheduling
public class AppConfig {
    ...
}
```



### fixedRate fixedDelay

接下来，我们可以直接在一个 Bean 中编写一个 `public void` 无参数方法，然后加上 **`@Scheduled`** 注解：

```java
@Component
public class TaskService {
    final Logger logger = LoggerFactory.getLogger(getClass());

    @Scheduled(initialDelay = 60_000, fixedRate = 60_000)
    public void checkSystemStatusEveryMinute() {
        logger.info("Start check system status...");
    }
}
```

上述注解指定了启动延迟 60 秒，并以 60 秒的间隔执行任务。现在，我们直接运行应用程序，就可以在控制台看到定时任务打印的日志：

```
2020-06-03 18:47:32 INFO  [pool-1-thread-1] c.i.learnjava.service.TaskService - Start check system status...
2020-06-03 18:48:32 INFO  [pool-1-thread-1] c.i.learnjava.service.TaskService - Start check system status...
2020-06-03 18:49:32 INFO  [pool-1-thread-1] c.i.learnjava.service.TaskService - Start check system status...
```

如果没有看到定时任务的日志，需要检查：

- 是否忘记了在 `AppConfig` 中标注 `@EnableScheduling`
- 是否忘记了在定时任务的方法所在的 class 标注 `@Component`

> 除了可以使用 `fixedRate` 外，还可以使用 `fixedDelay`



有的童鞋在实际开发中会遇到一个问题，因为 Java 的注解全部是常量，写死了 `fixedDelay=30000`，如果根据实际情况要改成 60 秒怎么办，只能重新编译？

我们可以把定时任务的配置放到配置文件中，例如 `task.properties`：

```ini
task.checkDiskSpace=30000
```

这样就可以随时修改配置文件而无需动代码。但是在代码中，我们需要用 `fixedDelayString` 取代 `fixedDelay`：

```java
@Component
public class TaskService {
    ...

    @Scheduled(initialDelay = 30_000, fixedDelayString = "${task.checkDiskSpace:30000}")
    public void checkDiskSpaceEveryMinute() {
        logger.info("Start check disk space...");
    }
}
```

注意到上述代码的注解参数 `fixedDelayString` 是一个属性占位符，并配有默认值 30000，Spring 在处理 `@Scheduled` 注解时，如果遇到 `String`，会根据占位符自动用配置项替换，这样就可以灵活地修改定时任务的配置。



此外，`fixedDelayString` 还可以使用更易读的 `Duration`，例如：

```java
@Scheduled(initialDelay = 30_000, fixedDelayString = "${task.checkDiskSpace:PT2M30S}")
```

以字符串 `PT2M30S` 表示的 `Duration` 就是2分30秒，请参考 [LocalDateTime ](https://www.liaoxuefeng.com/wiki/1252599548343744/1303871087444002) 一节的 Duration 相关部分。

多个 `@Scheduled` 方法完全可以放到一个 Bean 中，这样便于统一管理各类定时任务。



### Cron

还有一类定时任务，它不是简单的重复执行，而是按时间触发，我们把这类任务称为Cron任务，例如：

- 每天凌晨 2:15 执行报表任务
- 每个工作日 12:00 执行特定任务
- ……

Cron 源自 Unix/Linux 系统自带的crond守护进程，以一个简洁的表达式定义任务触发时间。在 Spring 中，也可以使用Cron 表达式来执行 Cron 任务，在 Spring 中，它的格式是：

```ini
秒 分 小时 天 月份 星期 年
```

年是可以忽略的，通常不写。每天凌晨 2:15 执行的 Cron 表达式就是：

```ini
0 15 2 * * *
```

每个工作日 12:00 执行的 Cron 表达式就是：

```ini
0 0 12 * * MON-FRI
```

每个月 1 号，2 号，3 号和 10 号 12:00 执行的 Cron 表达式就是：

```ini
0 0 12 1-3,10 * *
```



在 Spring 中，我们定义一个每天凌晨 2:15 执行的任务：

```java
@Component
public class TaskService {
    ...

    @Scheduled(cron = "${task.report:0 15 2 * * *}")
    public void cronDailyReport() {
        logger.info("Start daily report task...");
    }
}
```

Cron 任务同样可以使用属性占位符，这样修改起来更加方便。

Cron 表达式还可以表达每 10 分钟执行，例如：

```
0 */10 * * * *
```

这样，在每个小时的0:00，10:00，20:00，30:00，40:00，50:00均会执行任务，实际上它可以取代`fixedRate`类型的定时任务。



### Quartz

在 Spring 中使用定时任务和 Cron 任务都十分简单，但是要注意到，这些任务的调度都是在每个 JVM 进程中的。如果在本机启动两个进程，或者在多台机器上启动应用，这些进程的定时任务和 Cron 任务都是独立运行的，互不影响。

如果一些定时任务要以集群的方式运行，例如每天 23:00 执行检查任务，只需要集群中的一台运行即可，这个时候，可以考虑使用 [Quartz](https://www.quartz-scheduler.org/)。

Quartz 可以配置一个 JDBC 数据源，以便存储所有的任务调度计划以及任务执行状态。也可以使用内存来调度任务，但这样配置就和使用 Spring 的调度没啥区别了，额外集成Quartz的意义就不大。

Quartz 的 JDBC 配置比较复杂，Spring 对其也有一定的支持。要详细了解 Quartz 的集成，请参考 [Spring的文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#scheduling-quartz)。
> 先看 [spring boot 2.x actuator 监控/健康检查/审计/统计](https://mp.weixin.qq.com/s/v1E8vEh73zfH3CyyMIwewQ)
>
> 基于上述代码修改
>
> 源码：https://github.com/langyastudio/langya-tech/tree/springboot/scheduling
>
> 官方文档： https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#scheduling-annotation-support



在很多应用程序中经常需要执行定时任务。例如每天给用户发送报表，定时检查系统状态等。在 Spring + SpringMVC 环境中，一般来说要实现定时任务有两种方案：

- 一种是使用 Spring 自带的定时任务处理器 **@Scheduled** 注解
- 另一种是使用第三方框架如 Quartz 



**Spring 自带方案**

 Spring 内置定时任务和 Cron 任务的支持，因此不需要手写大量代码，只需要使用 2 个注解即可：

- 通过 `@EnableScheduling` 开启对计划任务的支持（基于类注解）

- 使用 `@Scheduled` 注解需要计划执行的任务（基于函数注解）

  

## Scheduled

### **@Scheduled 步骤**

添加 `@EnableScheduling` 开启定时任务的支持：

```java
@SpringBootApplication
@EnableScheduling
public class Application
{
    public static void main(String[] args)
    {
       ...
    }
}
```

接下来定义计划任务的类（标记 `@Component`）并使用 `@Scheduled` 注解计划执行的任务：

```java
@Log4j2
@Component
public class SchedulingService
{
    @Scheduled(fixedRate = 60000)
    public void  fixedRateSchedule()
    {
        log.info("每 60 秒钟执行fixedRateSchedule...");
    }


    /**
     * initialDelay 延迟 5 秒后开始执行该任务
     */
    @Scheduled(initialDelay = 5000, fixedDelay = 10000)
    public void  fixedDelaySchedule()
    {
        log.info("在上次任务完成 10 秒后执行fixedDelaySchedule...");
    }
}
```



**重点参数说明：**

- fixedRate 

  每隔固定的时间执行一次，**无论上次任务是否完成**

- fixedDelay 

  在上次任务执行完成后，在指定时间后执行新任务

- initialDelay 

  表示首次任务启动的延迟时间

- **所有时间的单位都是毫秒**



例如上述的 `fixedDelaySchedule` 函数的注解指定了启动延迟 5 秒，并以 10 秒的间隔执行任务。可以在控制台看到定时任务打印的日志：

```
2021-08-16 18:30:14.729 INFO [scheduling-1] com.langyastudio.springboot.common.middleware.task.SchedulingService : 在上次任务完成 10 秒后执行fixedDelay...
2021-08-16 18:30:24.730 INFO [scheduling-1] com.langyastudio.springboot.common.middleware.task.SchedulingService : 在上次任务完成 10 秒后执行fixedDelay...
2021-08-16 18:30:34.732 INFO [scheduling-1] com.langyastudio.springboot.common.middleware.task.SchedulingService : 在上次任务完成 10 秒后执行fixedDelay...
```



### **动态修改参数值**

例如 `fixedDelay=10000`，如果根据实际情况要改成 60 秒怎么办，难道只能重新编译？

可以看到 `Scheduled` 注解提供了 `fixedDelayString` 、`fixedRateString`、`initialDelayString` 等参数，可以把定时任务的配置值放到配置文件中，从而达到动态修改计划任务时间的目的。

例如定义配置文件：

```ini
langyastudio:
  task:
    fixedDelaySchedule: 60000
```

再通过 `xxxString` 替代 `xxx` 的方式实现动态修改参数值的目的：

```java
@Scheduled(initialDelay = 5000, fixedDelayString = "${langyastudio.task.fixedDelaySchedule:10000}")
public void  fixedDelaySchedule()
{
    log.info("在上次任务完成 10 秒后执行fixedDelay...");
}
```

上述的 `fixedDelayString` ，表示先从配置文件中获取 `langyastudio.task.fixedDelaySchedule` 值，如果没有则使用默认值 10000。



### **配置计划任务** 

示例如下:

```yml
spring:
  task:
    scheduling:
      pool:
        size: 5        
      shutdown:
        #关闭应用程序时，是否等待任务执行完毕
        await-termination: true
        await-termination-period: 60000 
```

通过配置参数，可以定制计划任务在应用关闭时采用的措施，如 `await-termination` 表示等待**执行完毕再关闭**。



## Cron

Cron 源自 Unix/Linux 系统自带的 crond 守护进程，以一个简洁的表达式定义任务触发时间（**按时间触发，不同于重复计划任务**）。

> 推荐一个在线 Cron 表达式生成器： https://crontab.guru/



在 Spring 中可以使用 Cron 表达式来执行 Cron 任务，它的格式如下：

```
 ┌───────────── second (0-59)
 │ ┌───────────── minute (0 - 59)
 │ │ ┌───────────── hour (0 - 23)
 │ │ │ ┌───────────── day of the month (1 - 31)
 │ │ │ │ ┌───────────── month (1 - 12) (or JAN-DEC)
 │ │ │ │ │ ┌───────────── day of the week (0 - 7)
 │ │ │ │ │ │          (0 or 7 is Sunday, or MON-SUN)
 │ │ │ │ │ │
 * * * * * *
  秒 分 小时 天 月份 星期 年 
```



**具体取值如下：**

| 序号 | 说明 | 是否必填 | 允许填写的值    | 允许的通配符 |
| :--- | :--- | :------- | :-------------- | :----------- |
| 1    | 秒   | 是       | 0-59            | - * /        |
| 2    | 分   | 是       | 0-59            | - * /        |
| 3    | 时   | 是       | 0-23            | - * /        |
| 4    | 日   | 是       | 1-31            | - * ? / L W  |
| 5    | 月   | 是       | 1-12 or JAN-DEC | - * /        |
| 6    | 周   | 是       | 1-7 or SUN-SAT  | - * ? / L #  |
| 7    | 年   | **否**   | 1970-2099       | - * /        |



**示例：**

周一至周五的 12:00 执行的 Cron 表达式：

```ini
0 0 12 * * MON-FRI
```

每个月 1-3 号和 10 号 12:00 执行的 Cron 表达式：

```ini
0 0 12 1-3,10 * *
```

在 spring 中定义一个每 30 秒执行的任务：

```java
@Scheduled(cron = "*/30 * * * * *")
public void  cronchedule()
{
    log.info("每30秒执行cron...");
}
```

此时 `xx:xx:00`、`xx:xx:30` 执行上述任务

> cron 也可以使用配置文件，不再详述



## Quartz

在 Spring 中使用定时任务和 Cron 任务都十分简单，但是要注意到，**这些任务的调度都是在每个 JVM 进程中的**。如果在本机启动两个进程或者在多台机器上启动应用，这些进程的定时任务和 Cron 任务都是独立运行的，互不影响。

如果一些定时任务要以集群的方式运行，例如每天 23:00 执行检查任务，只需要集群中的一台运行即可，这个时候可以考虑使用 [Quartz](https://www.quartz-scheduler.org/)。

Quartz 可以配置一个 JDBC 数据源，以便存储所有的任务调度计划以及任务执行状态。详细了解 Quartz 的集成，请参考 [Spring的文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#scheduling-quartz)。


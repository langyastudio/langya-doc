> 先看 [spring boot 2.x 日志log4j2会用吗？](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484719&idx=1&sn=3ec483a1f17bd9a4a0842b8c2d1d074b&chksm=c03a578ef74dde98290c5b683307b88a9a5dd5c144b0fbdc902e9fa3b63ef603650aa4965894&scene=21#wechat_redirect)
>
> 基于上述代码修改
>
> 源码：https://github.com/langyastudio/langya-tech/tree/springboot/actuator
>
> 官方文档：https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints



Spring Boot 提供了一个 Actuator，可以帮助你**监控和管理 Spring Boot 应用**，比如健康检查、审计、统计和 HTTP 追踪等。所有的这些特性可以通过 JMX 或者 HTTP endpoints 来获得。

endpoints 可以分为三大类：

- 应用配置类

  获取应用程序中加载的应用配置、环境变量、自动化配置报告等与 Spring Boot 应用密切相关的配置类信息

- 度量指标类

  获取应用程序运行过程中用于监控的度量指标，比如：内存信息、线程池信息、HTTP 请求统计等

- 操作控制类

  提供了对应用的关闭等操作类功能



Actuator 同时还可以与外部应用监控系统整合，比如 [Prometheus](https://prometheus.io/), [Graphite](https://graphiteapp.org/), [DataDog](https://www.datadoghq.com/), [Influx](https://www.influxdata.com/), [Wavefront](https://www.wavefront.com/), [New Relic](https://newrelic.com/)等。这些系统提供了非常好的仪表盘、图标、分析和告警等功能，使得你可以通过统一的接口轻松的监控和管理你的应用。



## 访问点

**pom.xml 引入 actuator**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



### 配置访问点

Actuator 默认把所有访问点暴露给 JMX，但出于安全原因默认只有 `health` 和 `info` 会暴露给 Web。如果需要暴露更多的访问点给 Web，需要在 `application.yml` 中加上配置（不建议），例如：

```yml
management:
  endpoints:
    web:
      exposure:
        include: health, info, beans, env, conditions, configprops, loggers, metrics
```

> 要特别注意暴露的 URL 的安全性，例如，`/actuator/env` 可以获取当前机器的所有环境变量，不可暴露给公网



### 常用访问点

| Endpoint ID    | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| health         | 应用的健康状态  |
| info           | 应用的基本信息  |
| shutdown       | 关闭应用 （开启和关闭可以通过 `management.endpoint.端点名.enabled` 来设置）|
| env            | 应用的环境信息，包含Profile、系统环境变量和应用的properties信息 |
| beans     | 应用的所有Bean |
| conditions     | 应用的自动配置报告，包含匹配的自动配置类（positiveMatches）、不匹配的自动配置类（negativeMatches）和非条件配置类（unconditionalClasses） |
|httptrace      | 显示 HTTP 足迹，最近 100 个HTTP request/repsponse       |
|configprops | 所有注解 @ConfigurationProperties 的Bean|
| threaddump     | Java 虚拟机线程信息 |
| loggers        | 应用中所有的 logger |
| mappings       | 所有的 @RequestMapping  |
| metrics        | 应用指标信息，包含内存使用情况等 |
| scheduledtasks | 应用中的调度任务 |



## web 访问

`/actuator` 是所有端点的前缀，访问 http://localhost:8080/actuator，可显示所有已暴露且功能已开启的端点访问信息

![image-20210813102135501](https://img-note.langyastudio.com/20210813102135.png?x-oss-process=style/watermark)



> 这里介绍部分访问点

**health**

> 许多网关作为反向代理需要一个 URL 来探测后端集群应用是否存活，这个 URL 就可以提供给网关使用

Actuator 可以通过URL `/actuator/` + 访问点进行访问，例如输入 `http://localhost:8080/actuator/health`， 可以查看应用程序当前状态：

```json
{
    "status": "UP"
}
```
如果想要看到健康信息的明细，可在 `application.yml` 中配置：

```yml
management:
  endpoint:
    health:
      show-details: always
```

此时重启应用后，可以看到明细的 health 信息：

```json
{
    "status": "UP",
    "components": {
        "diskSpace": {
            "status": "UP",
            "details": {
                "total": 64424505344,
                "free": 23276142592,
                "threshold": 10485760,
                "exists": true
            }
        },
        "ping": {
            "status": "UP"
        }
    }
}
```



**info**

可在 `application.yml` 中配置，使用 `info.*` 可设置任意信息，例如：

```yml
info:
  name: spring boot app
  version: 1.0.1
  maintainer: langyastudio
```

此时访问 `http://localhost:8080/actuator/info` 可以看到应用信息如下：

```json
{
    "name": "spring boot app",
    "version": "1.0.1",
    "maintainer": "langyastudio"
}
```



**httptrace**

用于显示请求的追踪信息，**但从 Spring 2.2.x 开始，我们必须定制实现 `HttpTraceRepository` 的 Bean**，让它来实现存储追踪和查询信息的功能，这样才能启用此端点。



## 自定义

### 修改访问地址

可在 `application.yml` 中配置

```yml
endpoints:
  web:
    exposure:
      #health, info
      include: health, info
    base-path: /monitor
    path-mapping:
      health: check-health
```

- 访问地址由 `actuator` 改为 `monitor`
- health 端点的地址改为 check-health，如 `http://localhost:8080/monitor/check-health`



### 自定义健康指示器

使用 `HealthIndicator` 定义健康指示器，把前面端点的状态数据作为健康依据。如果状态不为空，则为健康（UP）；如果状态为空，则不健康（DOWN）。

```java
@Component
public class MonitorHealthIndicator implements HealthIndicator
{
    @Override
    public Health health()
    {
        //Health.down();
       return  Health.up()
                .withDetail("error", "no")
                .build();
    }
}
```

此时访问 `health` 端点的结果如下：

![image-20210813113057031](https://img-note.langyastudio.com/20210813113057.png?x-oss-process=style/watermark)



### 自定义端点

在 Bean上注解 @Endpoint、@WebEndpoint 或 @EndpointWebExtension，可以将 Bean 通过 HTTP 暴露为端点。

自定义端点支持以下三种操作并通过 @Selector 接收参数：

- @ReadOperation：GET（查询请求）
- @WriteOperation：POST（保存请求）
- @DeleteOperation：DELETE（删除请求）

这里不做详细介绍



### 自定义指标

Spring Boot Actuator 还提供了基于 `Micrometer` 的基本指标和自动配置。

这里不做详细介绍


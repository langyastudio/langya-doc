## Sentinel 简介

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 是面向分布式服务架构的流量控制组件，主要以流量为切入点，从**流量控制、熔断降级、系统自适应保护**等多个维度来帮助您保障微服务的稳定性。

![sentinel-opensource-cloud-native-landscape-202006](https://img-note.langyastudio.com/202201102146156.png?x-oss-process=style/watermark)



### 特性

- 丰富的应用场景：承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀，可以实时熔断下游不可用应用
- 完备的实时监控：同时提供实时的监控功能。可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况
- 广泛的开源生态：提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Boot/Spring Cloud、Dubbo、gRPC、Feign、Spring Cloud Gateway、RocketMQ 的整合
- 完善的 SPI 扩展点：提供简单易用、完善的 SPI 扩展点。您可以通过实现扩展点，快速的定制逻辑



### 核心概念

**资源**

资源是 Sentinel 的关键概念。它可以是 Java 应用程序中的任何内容，例如，由应用程序提供的服务，或由应用程序调用的其它应用提供的服务，甚至可以是一段代码。在接下来的文档中，都会用资源来描述代码块。

只要通过 Sentinel API 定义的代码，就是资源，能够被 Sentinel 保护起来。大部分情况下，可以使用方法签名，URL，甚至服务名称作为资源名来标示资源。

**规则**

围绕资源的实时状态设定的规则，可以包括流量控制规则、熔断降级规则以及系统保护规则。所有规则可以动态实时调整。



### 基本原理

在 Sentinel 里面，所有的资源都对应一个资源名称以及一个 Entry。Entry 可以通过对主流框架的适配自动创建，也可以通过注解的方式或调用 API 显式创建；每一个 Entry 创建的时候，同时也会创建一系列功能插槽（slot chain）。这些插槽有不同的职责，例如:

- `NodeSelectorSlot` 负责收集资源的路径，并将这些资源的调用路径，以树状结构存储起来，用于根据调用路径来限流降级
- `ClusterBuilderSlot` 则用于存储资源的统计信息以及调用者信息，例如该资源的 RT, QPS, thread count 等等，这些信息将用作为多维度限流，降级的依据
- `StatisticSlot` 则用于记录、统计不同纬度的 runtime 指标监控信息
- `FlowSlot` 则用于根据预设的限流规则以及前面 slot 统计的状态，来进行流量控制
- `AuthoritySlot` 则根据配置的黑白名单和调用来源信息，来做黑白名单控制
- `DegradeSlot` 则通过统计信息以及预设的规则，来做熔断降级
- `SystemSlot` 则通过系统的状态，例如 load1 等，来控制总的入口流量

![arch overview](https://sentinelguard.io/docs/zh-cn/img/sentinel-slot-chain-architecture.png)



## Sentinel 功能和设计理念

### 流量控制

流量控制在网络传输中是一个常用的概念，它用于调整网络包的发送数据。然而，从系统稳定性角度考虑，在处理请求的速度上，也有非常多的讲究。任意时间到来的请求往往是随机不可控的，而系统的处理能力是有限的。需要根据系统的处理能力对流量进行控制。Sentinel 作为一个调配器，可以根据需要把**随机的请求调整成合适的形状**，如下图所示：

![arch](https://sentinelguard.io/docs/zh-cn/img/sentinel-flow-overview.jpg)

流量控制有以下几个角度:

- 资源的调用关系，例如资源的调用链路，资源和资源之间的关系
- 运行指标，例如 QPS、线程池、系统负载等
- 控制的效果，例如直接限流、冷启动、排队等

Sentinel 的设计理念是让您自由选择控制的角度，并进行灵活组合，从而达到想要的效果。



### 熔断降级

#### 什么是熔断降级

除了流量控制以外，降低调用链路中的不稳定资源也是 Sentinel 的使命之一。由于调用关系的复杂性，如果调用链路中的某个资源出现了不稳定，最终会导致**请求发生堆积**。

![image](https://user-images.githubusercontent.com/9434884/62410811-cd871680-b61d-11e9-9df7-3ee41c618644.png)

Sentinel 和 Hystrix 的原则是一致的:  当调用链路中某个资源出现不稳定，例如，表现为 timeout，异常比例升高的时候，则对这个资源的调用进行限制，并让请求快速失败，避免影响到其它的资源，最终产生雪崩的效果。



#### 熔断降级设计理念

在限制的手段上，Sentinel 和 Hystrix 采取了完全不一样的方法。

Hystrix 通过[线程池](https://github.com/Netflix/Hystrix/wiki/How-it-Works#benefits-of-thread-pools)的方式，来对依赖(在的概念中对应资源)进行了隔离。这样做的好处是资源和资源之间做到了最彻底的隔离。缺点是除了增加了线程切换的成本，还需要预先给各个资源做线程池大小的分配。

Sentinel 对这个问题采取了两种手段:

- 通过**并发线程数**进行限制

和资源池隔离的方法不同，Sentinel 通过限制资源并发线程的数量，来减少不稳定资源对其它资源的影响。这样不但没有线程切换的损耗，也不需要您预先分配线程池的大小。当某个资源出现不稳定的情况下，例如响应时间变长，对资源的直接影响就是会造成线程数的逐步堆积。当线程数在特定资源上堆积到一定的数量之后，对该资源的新请求就会被拒绝。堆积的线程完成任务后才开始继续接收请求。

- 通过**响应时间**对资源进行降级

除了对并发线程数进行控制以外，Sentinel 还可以通过响应时间来快速降级不稳定的资源。当依赖的资源出现响应时间过长后，所有对该资源的访问都会被直接拒绝，直到过了指定的时间窗口之后才重新恢复。



### 系统负载保护

Sentinel 同时提供[系统维度的自适应保护能力](https://sentinelguard.io/zh-cn/docs/system-adaptive-protection.html)。**防止雪崩**，是系统防护中重要的一环。当系统负载较高的时候，如果还持续让请求进入，可能会导致系统崩溃，无法响应。在集群环境下，网络负载均衡会把本应这台机器承载的流量转发到其它的机器上去。如果这个时候其它的机器也处在一个边缘状态的时候，这个增加的流量就会导致这台机器也崩溃，最后导致整个集群不可用。

针对这个情况，Sentinel 提供了对应的保护机制，让系统的入口流量和系统的负载达到一个平衡，保证系统在能力范围之内处理最多的请求。



## [Sentinel 控制台](https://sentinelguard.io/zh-cn/docs/dashboard.html)

> 安装与使用可以查看官方文档：[https://sentinelguard.io/zh-cn/docs/dashboard.html](https://sentinelguard.io/zh-cn/docs/dashboard.html)

Sentinel 提供一个轻量级的开源控制台，它提供机器发现以及健康情况管理、监控（单机和集群），规则管理和推送的功能。

Sentinel 控制台包含如下功能:

- 查看机器列表以及健康情况：收集 Sentinel 客户端发送的心跳包，用于判断机器是否在线
- 监控 (单机和集群聚合)：通过 Sentinel 客户端暴露的监控 API，定期拉取并且聚合应用监控信息，最终可以实现秒级的实时监控
- 规则管理和推送：统一管理推送规则
- 鉴权：生产环境中鉴权非常重要。这里每个开发者需要根据自己的实际情况进行定制



**docker 部署脚本：**

`bladex/sentinel-dashboard`， 默认账号密码为  [sentinel sentinel]

```yml
version: '3.5'

services:
  sentinel-dashboard:
    image: bladex/sentinel-dashboard:1.8.0
    container_name: sentinel-dashboard
    #account and password: [sentinel sentinel]
    #修改默认密码为 123456    
    environment:
      - sentinel.dashboard.auth.password=123456
    restart: always
    ports:
      - 8858:8858
```

![image-20220107114701821](https://img-note.langyastudio.com/202201071147041.png?x-oss-process=style/watermark)



## Sentinel 实战

源码地址：[https://github.com/langyastudio/langya-tech/tree/master/spring-cloud](https://github.com/langyastudio/langya-tech/tree/master/spring-cloud)



### 规则的种类

#### FlowRule 流量控制规则

| Field           | 说明                                                         | 默认值                        |
| --------------- | ------------------------------------------------------------ | ----------------------------- |
| resource        | 资源名，资源名是限流规则的作用对象                           |                               |
| count           | 限流阈值                                                     |                               |
| grade           | 限流阈值类型，QPS 模式（1）或并发线程数模式（0）             | QPS 模式                      |
| limitApp        | 流控针对的调用来源                                           | `default`，代表不区分调用来源 |
| strategy        | 调用关系限流策略：直接、链路、关联                           | 根据资源本身（直接）          |
| controlBehavior | 流控效果（直接拒绝/WarmUp/匀速+排队等待），不支持按调用关系限流 | 直接拒绝                      |
| clusterMode     | 是否集群限流                                                 | 否                            |



#### DegradeRule 熔断降级规则

| Field              | 说明                                                         | 默认值     |
| ------------------ | ------------------------------------------------------------ | ---------- |
| resource           | 资源名，即规则的作用对象                                     |            |
| grade              | 熔断策略，支持慢调用比例/异常比例/异常数策略                 | 慢调用比例 |
| count              | 慢调用比例模式下为慢调用临界 RT（超出该值计为慢调用）；异常比例/异常数模式下为对应的阈值 |            |
| timeWindow         | 熔断时长，单位为 s                                           |            |
| minRequestAmount   | 熔断触发的最小请求数，请求数小于该值时即使异常比率超出阈值也不会熔断（1.7.0 引入） | 5          |
| statIntervalMs     | 统计时长（单位为 ms），如 60*1000 代表分钟级（1.8.0 引入）   | 1000 ms    |
| slowRatioThreshold | 慢调用比例阈值，仅慢调用比例模式有效（1.8.0 引入）           |            |



#### SystemRule 系统保护规则

Sentinel 系统自适应限流从整体维度对应用入口流量进行控制，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

| Field             | 说明                                   | 默认值      |
| ----------------- | -------------------------------------- | ----------- |
| highestSystemLoad | `load1` 触发值，用于触发自适应控制阶段 | -1 (不生效) |
| avgRt             | 所有入口流量的平均响应时间             | -1 (不生效) |
| maxThread         | 入口流量的最大并发数                   | -1 (不生效) |
| qps               | 所有入口资源的 QPS                     | -1 (不生效) |
| highestCpuUsage   | 当前系统的 CPU 使用率（0.0-1.0）       | -1 (不生效) |



#### AuthorityRule 访问控制规则

很多时候，我们需要根据调用方来限制资源是否通过，这时候可以使用 Sentinel 的访问控制（黑白名单）的功能。黑白名单根据资源的请求来源（`origin`）限制资源是否通过，若配置白名单则只有请求来源位于白名单内时才可通过；若配置黑名单则请求来源位于黑名单时不通过，其余的请求通过。

授权规则，即黑白名单规则（`AuthorityRule`）非常简单，主要有以下配置项：

- `resource`：资源名，即规则的作用对象
- `limitApp`：对应的黑名单/白名单，不同 origin 用 `,` 分隔，如 `appA,appB`
- `strategy`：限制模式，`AUTHORITY_WHITE` 为白名单模式，`AUTHORITY_BLACK` 为黑名单模式，默认为白名单模式



#### ParamFlowRule 热点参数规则

| 属性              | 说明                                                         | 默认值   |
| ----------------- | ------------------------------------------------------------ | -------- |
| resource          | 资源名，必填                                                 |          |
| count             | 限流阈值，必填                                               |          |
| grade             | 限流模式                                                     | QPS 模式 |
| durationInSec     | 统计窗口时间长度（单位为秒），1.6.0 版本开始支持             | 1s       |
| controlBehavior   | 流控效果（支持快速失败和匀速排队模式），1.6.0 版本开始支持   | 快速失败 |
| maxQueueingTimeMs | 最大排队等待时长（仅在匀速排队模式生效），1.6.0 版本开始支持 | 0ms      |
| paramIdx          | 热点参数的索引，必填，对应 `SphU.entry(xxx, args)` 中的参数索引位置 |          |
| paramFlowItemList | 参数例外项，可以针对指定的参数值单独设置限流阈值，不受前面 `count` 阈值的限制。**仅支持基本类型和字符串类型** |          |
| clusterMode       | 是否是集群参数流控规则                                       | `false`  |
| clusterConfig     | 集群流控相关配置                                             |          |



#### 判断限流降级异常

在 Sentinel 中所有流控降级相关的异常都是异常类 `BlockException` 的子类：

- 流控异常：`FlowException`
- 熔断降级异常：`DegradeException`
- 系统保护异常：`SystemBlockException`
- 热点参数限流异常：`ParamFlowException`

我们可以通过以下函数判断是否为 Sentinel 的流控降级异常：

```java
BlockException.isBlockException(Throwable t);
```



### 创建 sentinel-server 模块

> 这里创建一个 sentinel-server 模块，用于演示 Sentinel 的熔断与限流功能

在 pom.xml 中添加相关依赖，这里使用 Nacos 作为注册中心，所以需要同时添加 Nacos 的依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

在 application.yml 中添加相关配置，主要是配置了 Nacos 和 Sentinel 控制台的地址

```yaml
server:
  port: 20001

spring:
  application:
    name: sentinel-server

  cloud:
    #Nacos
    nacos:
      username: nacos
      password: nacos
      discovery:
        server-addr: 192.168.123.22:8848

    #配置sentinel dashboard地址
    sentinel:
      transport:
        dashboard: 192.168.123.22:8858
        port: 8719


management:
  endpoints:
    health:
      show-details: always
    web:
      exposure:
        include: '*'
```



### 限流功能

> Sentinel Starter 默认为所有的 HTTP 服务提供了限流埋点，也可以通过使用 @SentinelResource 来自定义一些限流行为

#### 创建 RateLimitController 类

> 用于测试熔断和限流功能

```java
/**
 * 限流功能
 */
@RestController
@RequestMapping("/rateLimit")
public class RateLimitController
{
    /**
     * 按资源名称限流，需要指定限流处理逻辑
     */
    @GetMapping("/byResource")
    @SentinelResource(value = "byResource", blockHandler = "handleException")
    public ResponseEntity byResource()
    {
        return new ResponseEntity("按资源名称限流", HttpStatus.OK);
    }

    /**
     * 按URL限流，有默认的限流处理逻辑
     */
    @GetMapping("/byUrl")
    @SentinelResource(value = "byUrl", blockHandler = "handleException")
    public ResponseEntity byUrl()
    {
        return new ResponseEntity("按url限流", HttpStatus.OK);
    }

    public ResponseEntity handleException(BlockException exception)
    {
        return new ResponseEntity(exception.getClass().getCanonicalName(), HttpStatus.OK);
    }
}
```



#### 根据资源名称限流

> 可以根据 @SentinelResource 注解中定义的 value（资源名称）来进行限流操作，但是需要指定限流处理逻辑

- 由于使用了 Nacos 注册中心，先启动 Nacos 和 sentinel-server
- 由于 Sentinel 采用的**懒加载规则**，需要先访问下接口，Sentinel 控制台中才会有对应服务信息，先访问下该接口：http://localhost:20001/rateLimit/byResource
- 流控规则可以在 Sentinel 控制台进行配置。在 Sentinel 控制台配置流控规则，根据 @SentinelResource 注解的 value 值

![image-20220107154119609](https://img-note.langyastudio.com/202201071541859.png?x-oss-process=style/watermark)



快速访问上面的接口，可以发现返回了自己定义的限流处理信息

![image-20220107154257063](https://img-note.langyastudio.com/202201071542114.png?x-oss-process=style/watermark)

#### 根据 URL 限流

> 还可以通过访问的 URL 来限流，会返回默认的限流处理信息

在 Sentinel 控制台配置流控规则，使用访问的 URL

![img](https://img-note.langyastudio.com/202201191149094.png?x-oss-process=style/watermark)



多次访问该接口，会返回默认的限流处理结果：http://localhost:20001/rateLimit/byUrl

```bash
Blocked by Sentinel (flow limiting)
```



#### 自定义限流处理逻辑

> 可以自定义通用的限流处理逻辑，然后在 @SentinelResource 中指定

创建 CustomBlockHandler 类用于自定义限流处理逻辑

```java
public class CustomBlockHandler
{
    public ResponseEntity handleException(BlockException exception)
    {
        return new ResponseEntity("自定义限流信息", HttpStatus.OK);
    }
}
```

在 RateLimitController 中使用自定义限流处理逻辑

```java
/**
 * 自定义通用的限流处理逻辑
 */
@GetMapping("/customBlockHandler")
@SentinelResource(value = "customBlockHandler", blockHandler = "handleException",
        blockHandlerClass = CustomBlockHandler.class)
public ResponseEntity blockHandler()
{
    return new ResponseEntity("限流成功", HttpStatus.OK);
}
```



### 熔断功能

> Sentinel 支持对服务间调用进行保护，对故障应用进行熔断操作，这里调用 nacos-discovery-provider 服务所提供的接口来演示下该功能

#### RestTemplete 实现

首先需要使用 @SentinelRestTemplate 来包装下 RestTemplate 实例

```java
@Configuration
public class RestTemplateConfig
{
    @Bean
    @SentinelRestTemplate
    public RestTemplate restTemplate()
    {
        return new RestTemplate();
    }
}
```

添加 CircleBreakerController 类，定义对 nacos-discovery-provider 提供接口的调用

```java
/**
 * 熔断功能
 */
@Log4j2
@RestController
@RequestMapping("/breaker")
public class CircleBreakerController
{
    @Autowired
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-discovery-provider}")
    private String providerServiceUrl;

    @RequestMapping("/fallback/{id}")
    @SentinelResource(value = "fallback", fallback = "handleFallback")
    public ResultInfo fallback(@PathVariable String id)
    {
        return restTemplate.getForObject(providerServiceUrl + "/echoex/" + id, ResultInfo.class);
    }

    /**
     * 忽略 NullPointerException 异常的降级
     */
    @RequestMapping("/fallbackex/{id}")
    @SentinelResource(value = "fallbackex", fallback = "handleFallback", exceptionsToIgnore =
            {NullPointerException.class})
    public ResultInfo fallbackEx(@PathVariable String id)
    {
        if (Objects.equals(id, "1"))
        {
            throw new NullPointerException();
        }

        return restTemplate.getForObject(providerServiceUrl + "/echoex/" + id, ResultInfo.class);
    }

    /*
     * 熔断降级逻辑
     */
    public ResultInfo handleFallback(String id)
    {
        return ResultInfo.data("handleFallback 服务降级返回");
    }
}
```

- 启动 nacos-discovery-provider 和 sentinel-service 服务

- 访问 http://localhost:20001/breaker/fallback/0 由于调用的 `nacos-discovery-provider` 服务输出了 `NullPointerException` 异常，所以引发熔断服务 - 调用 `handleFallback` 函数的内容进行返回

- 访问 http://localhost:20001/breaker/fallbackex/1 由于使用了 exceptionsToIgnore 参数忽略了 `NullPointerException`，所以访问接口报空指针时不会发生服务降级

  

#### Feign 实现

> Sentinel 也适配了 Feign 组件，使用 Feign 进行服务间调用时，也可以使用它来进行熔断

首先需要在 pom.xml 中添加 Feign 相关依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

在 application.yml 中打开 Sentinel 对 Feign 的支持

```yaml
feign:
  sentinel:
    #打开sentinel对feign的支持
    enabled: true 
```

在应用启动类上添加 @EnableFeignClients 启动 Feign 的功能

```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class SentinelServerApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(SentinelServerApplication.class, args);
    }
}
```

创建一个 EchoService 接口，用于定义对 nacos-discovery-provider 服务的调用

```java
@FeignClient(name = "${service-url.nacos-discovery-provider}", fallback = EchoServiceImpl.class,
        configuration = FeignConfig.class)
public interface EchoService
{
    @GetMapping("/echo/{str}")
    ResultInfo echoex(@PathVariable("str") String str);
}
```

创建 EchoServiceImpl 类实现 EchoService 接口，用于处理**服务降级**逻辑

```java
public class EchoServiceImpl implements EchoService
{
    @Override
    public ResultInfo echoex(@PathVariable("str") String str)
    {
        return ResultInfo.data("EchoService 服务降级返回");
    }
}
```

在 EchoFeignController 中使用 EchoService 通过 Feign 调用 nacos-discovery-service 服务中的接口

```java
@RestController
@RequestMapping("/feign")
public class EchoFeignController
{
    @Autowired
    private EchoService echoService;

    @GetMapping("/echo/{str}")
    public ResultInfo feign(@PathVariable String str)
    {
        return echoService.echoex(str);
    }
}
```

调用如下接口会发生服务降级，返回服务降级处理信息：http://localhost:20001/feign/echo/0

```json
{
    "code": 111111,
    "msg": "请求成功",
    "time": 1651806857944,
    "result": "EchoService 服务降级返回"
}
```



### Nacos 规则持久化

默认情况下，当在 Sentinel 控制台中配置规则时，控制台推送规则方式是通过 API 将规则推送至客户端并直接更新到内存中。**一旦重启应用，规则将消失**。下面介绍下如何将配置规则进行持久化，以存储到 Nacos 为例

推荐的推送规则的做法：**配置中心控制台/Sentinel 控制台 → 配置中心 → Sentinel 数据源 → Sentinel**，这样的流程就非常清晰了：

![Remote push rules to config center](https://user-images.githubusercontent.com/9434884/53381986-a0b73f00-39ad-11e9-90cf-b49158ae4b6f.png)



先在 pom.xml 中添加相关依赖

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

修改 application.yml 配置文件，添加 Nacos 数据源配置

```yaml
spring:
  cloud:
    sentinel:
      datasource:
          ds1:
            nacos:
              server-addr: localhost:8848
              username: nacos
              password: nacos
              dataId: ${spring.application.name}-sentinel
              #namespace: public
              groupId: DEFAULT_GROUP
              data-type: json
              rule-type: flow
```

在 Nacos 中添加配置

![image-20220110173902058](https://img-note.langyastudio.com/202201101739299.png?x-oss-process=style/watermark)

添加配置信息如下

```json
[
    {
        "resource": "/rateLimit/byUrl",
        "limitApp": "default",
        "grade": 1,
        "count": 1,
        "strategy": 0,
        "controlBehavior": 0,
        "clusterMode": false
    }
]
```

相关参数解释：
- resource：资源名称
- limitApp：来源应用
- grade：阈值类型，0表示线程数，1表示QPS
- count：单机阈值
- strategy：流控模式，0表示直接，1表示关联，2表示链路
- controlBehavior：流控效果，0表示快速失败，1表示Warm Up，2表示排队等待
- clusterMode：是否集群

 

发现 Sentinel 控制台已经有了如下限流规则：

![image-20220110174447810](https://img-note.langyastudio.com/202201101744901.png?x-oss-process=style/watermark)



### GateWay支持

若想跟 Sentinel Starter 配合使用，需要加上 `spring-cloud-alibaba-sentinel-gateway` 依赖，同时需要添加 `spring-cloud-starter-gateway` 依赖来让 `spring-cloud-alibaba-sentinel-gateway` 模块里的 Spring Cloud Gateway 自动化配置类生效：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>

<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

同时请将 `spring.cloud.sentinel.filter.enabled` 配置项置为 false（若在网关流控控制台上看到了 URL 资源，就是此配置项没有置为 false）。Sentinel 网关流控默认的粒度是 route 维度以及自定义 API 分组维度，默认**不支持 URL 粒度**。

**注意**：网关流控规则数据源类型是 **`gw-flow`**，若将网关流控规则数据源指定为 flow 则不生效。



## 参考

[Sentinel 官方文档](https://sentinelguard.io/zh-cn/docs/introduction.html)

[Sentinel  wiki](https://github.com/alibaba/Sentinel/wiki)

[spring-cloud-alibaba wiki](https://github.com/alibaba/spring-cloud-alibaba/wiki)

[Spring Cloud Alibaba：Sentinel 实现熔断与限流](http://www.macrozheng.com/#/cloud/sentinel)

[阿里限流神器Sentinel夺命连环 17 问](https://mp.weixin.qq.com/s/w8lhJfhLdh7POpPw2MyPwA)


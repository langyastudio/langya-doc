`Spring` 在因 `Netflix` 开源流产事件后，在不断的更换 `Netflix` 相关的组件，比如：`Eureka`、`Zuul`、`Feign`、`Ribbon` 等，`Zuul` 的替代产品就是 `SpringCloud Gateway`，这是 `Spring` 团队研发的网关组件，可以实现**安全认证、限流、重试、支持长连接**等新特性。



## 背景说明

如果有三个服务 `account-service`，`product-service`，`order-service`。现在有客户端 `WEB应用` 或 `APP应用` 需要访问后端服务获取数据那么就需要在客户端维护好三个服务的访问路径。

![图片](https://img-note.langyastudio.com/202201061138527.webp?x-oss-process=style/watermark)

这样的架构会有如下几个典型的问题：

- 每个微服务都需要开通**外网访问**权限，配置单独的访问域名，每新增一个服务都需要先让运维人员配置好域名映射
- 客户端需要维护所有微服务的访问地址，试想一下如果微服务有几十几百个呢
- 当服务需要对接口进行权限控制，必须要认证用户才能调用，那么所有的权限逻辑在服务端都要重新编写一套
- 。。。



所以需要在微服务之前加一个网关服务，让所有的客户端只要访问网关，网关负责对请求进行转发；将权限校验逻辑放到网关的过滤器中，后端服务不需要再关注权限校验的代码；只需要对外提供一个可供外网访问的域名地址，新增服务后也不需要再让运维人员进行网络配置了，这样上面的架构就变成了如下所示：

![图片](https://img-note.langyastudio.com/202201061139967.webp?x-oss-process=style/watermark)

![图片](https://img-note.langyastudio.com/202208311019779.png?x-oss-process=style/watermark)

这张图展示了一个多层 Gateway 架构，其中有一个总的 Gateway 接入所有的流量(**流量网关** )，并分发给不同的子系统，还有第二级 Gateway 用于做各个子系统的接入 Gateway(**业务网关** )。



## 常见网关

- **Nginx+lua** ：OpenResty、Kong、Orange、Abtesting gateway 等
- **Java** ：Zuul/Zuul2、Spring Cloud Gateway、Kaazing KWG、gravitee、Dromara soul 等
- **Go** ：Janus、fagongzi、Grpc-gateway
- **Dotnet** ：Ocelot
- **NodeJS** ：Express Gateway、Micro Gateway

按照使用数量、成熟度等来划分，主流的有 5个：

- OpenResty
- Kong
- Zuul、Zuul2
- Spring Cloud Gateway



## GateWay 介绍

在 SpringCloud 体系架构中，需要部署一个单独的网关服务对外提供访问入口，然后网关服务根据配置好的规则将请求转发至具体的后端服务。

`Spring Cloud Gateway` 是 `SpringCloud` 的全新子项目，该项目基于 `Spring5.x`、`SpringBoot2.x` 技术版本进行编写，意在提供简单方便、可扩展的统一 API 路由管理方式。



### 基本概念

- Route（路由）

  路由是网关的基本单元，由 ID、URI、一组 Predicate、一组 Filter 组成，根据 Predicate 进行匹配转发

- Predicate（断言）

  指的是 Java 8 的 Function Predicate，输入类型是 Spring 框架中的 ServerWebExchange。作为路由转发的判断条件，目前 `SpringCloud Gateway` 支持多种方式，常见如：`Path`、`Query`、`Method`、`Header`等

- Filter（过滤器）

  过滤器是路由转发请求时所经过的过滤逻辑，GatewayFilter 的实例可用于修改请求、响应内容



### 工作流程

![4461954-e8f9d2b9eae8fcab](https://img-note.langyastudio.com/202201051431057.png?x-oss-process=style/watermark)

客户端向 `Spring Cloud Gateway` 发出请求。如果网关处理程序映射确定请求与路由匹配，则将其发送到网关 Web 处理程序。此处理程序运行时通过特定于请求的筛选链发送请求。过滤器被虚线分隔的原因是过滤器可以在发送代理请求之前或之后执行逻辑。



### Predicates 断言

当满足这种条件后才会被转发，如果是多个，那就是都满足的情况下被转发。

| 匹配方式   | 说明             | 样例                                                         |
| ---------- | ---------------- | ------------------------------------------------------------ |
| Before     | 某一个时间点之前 | Before=2019-05-01T00:00:00+08:00[Asia/Shanghai]              |
| After      | 某一个时间点之后 | After=2019-04-29T00:00:00+08:00[Asia/Shanghai]               |
| Between    | Before +After    | Between=2019-04-29T00:00:00+08:00[Asia/Shanghai], 2019-05-01T00:00:00+08:00[Asia/Shanghai] |
| Cookie     | Cookie 值        | Cookie=hacfin, langyastudio                                  |
| Header     | Header 值        | Header=X-Request-Id, \d+                                     |
| Host       | 主机名           | Host=**.langyastudio.com                                     |
| Method     | 请求方式         | Method=POST                                                  |
| Query      | 请求参数         | Query=xxx, zzz                                               |
| Path       | 请求路径         | Path=/article/{articleId}                                    |
| RemoteAddr | 请求IP           | RemoteAddr=192.168.1.56/24                                   |
| Weight     | 权重             | Weight=group1, 8                                             |

**Weight 示例：**

80% 的请求会被路由到 localhost:8201，20% 会被路由到 localhost:8202

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: http://localhost:8201
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: http://localhost:8202
        predicates:
        - Weight=group1, 2
```



### Filter 过滤器 

路由过滤器可用于修改进入的 HTTP 请求和返回的 HTTP 响应。Spring Cloud Gateway 内置了多种路由过滤器，他们都由 GatewayFilter 的工厂类来产生，下面介绍下常用路由过滤器的用法。

#### AddRequestParameter 添加参数

给请求添加参数的过滤器

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: add_request_parameter_route
          uri: http://localhost:8201
          filters:
            - AddRequestParameter=username, langyastudio
          predicates:
            - Method=GET
```

以上配置会对 GET 请求添加 `username=langyastudio` 的请求参数，通过 curl 工具使用以下命令进行测试

```bash
curl http://localhost:9201/user/getByUsername
```

相当于发起该请求：

```bash
curl http://localhost:8201/user/getByUsername?username=langyastudio
```



#### StripPrefix 前缀去除

对指定数量的路径前缀去除的过滤器

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: strip_prefix_route
        uri: http://localhost:8201
        predicates:
        - Path=/user-service/**
        filters:
        - StripPrefix=2
```

以上配置会把以 `/user-service/` 开头的请求的路径去除两位，通过 curl 工具使用以下命令进行测试

```bash
curl http://localhost:9201/user-service/a/user/1
```

相当于发起该请求：

```bash
curl http://localhost:8201/user/1
```



#### PrefixPath 前缀增加

与 StripPrefix 过滤器恰好相反，会对原有路径前缀增加操作的过滤器

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefix_path_route
        uri: http://localhost:8201
        predicates:
        - Method=GET
        filters:
        - PrefixPath=/user
```

以上配置会对所有 GET 请求添加 `/user` 路径前缀，通过 curl 工具使用以下命令进行测试

```bash
curl http://localhost:9201/1
```

相当于发起该请求：

```bash
curl http://localhost:8201/user/1
```



#### RequestRateLimiter 限流

RequestRateLimiter 过滤器可以用于限流，使用 RateLimiter 实现来确定是否允许当前请求继续进行，如果请求太大默认会返回 HTTP 429 -太多请求状态。

在 pom.xml 中添加相关依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

添加限流策略的配置类，这里有两种策略一种是根据请求参数中的 username 进行限流，另一种是根据访问 IP 进行限流

```java
@Configuration
public class RedisRateLimiterConfig {
    @Bean
    KeyResolver userKeyResolver() {
        return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("username"));
    }

    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
    }
}
```

使用 Redis 来进行限流，所以需要添加 Redis 和 RequestRateLimiter 的配置，这里对所有的 GET 请求都进行了按 IP来限流的操作

```yaml
server:
  port: 9201
spring:
  redis:
    host: localhost
    password: 123456
    port: 6379
  cloud:
    gateway:
      routes:
        - id: requestratelimiter_route
          uri: http://localhost:8201
          filters:
            - name: RequestRateLimiter
              args:
                #每秒允许处理的请求数量
                redis-rate-limiter.replenishRate: 1 
                #令牌桶的容量，允许在一秒钟内完成的最大请求数
                redis-rate-limiter.burstCapacity: 2 
                #限流策略，对应策略的Bean
                #SpEL 表达式根据#{@beanName}从 Spring 容器中获取 Bean 对象
                key-resolver: "#{@ipKeyResolver}" 
          predicates:
            - Method=GET
logging:
  level:
    org.springframework.cloud.gateway: debug
```

多次请求该地址：http://localhost:9201/user/1 ，会返回状态码为 429 的错误

![img](https://img-note.langyastudio.com/202201051659043.png?x-oss-process=style/watermark)



#### Retry 重试

对路由请求进行重试的过滤器，可以根据路由请求返回的 HTTP 状态码来确定是否进行重试

修改配置文件：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: retry_route
        uri: http://localhost:8201
        predicates:
        - Method=GET
        filters:
        - name: Retry
          args:
            retries: 1 #需要进行重试的次数
            statuses: BAD_GATEWAY #返回哪个状态码需要进行重试，返回状态码为5XX进行重试
            backoff:
              firstBackoff: 10ms
              maxBackoff: 50ms
              factor: 2
              basedOnPreviousValue: false
```

当调用返回 500 时会进行重试，访问测试地址：http://localhost:9201/user/111

可以发现 user-service 控制台报错 2 次，说明进行了一次重试

```bash
2019-10-27 14:08:53.435 ERROR 2280 --- [nio-8201-exec-2] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.NullPointerException] with root cause
```



#### 自定义过滤器

例如：使用 SpringCloud 架构后希望所有的请求都需要经过网关才能访问，在不作任何处理的情况下是可以绕过网关直接访问后端服务的。

防止绕过网关直接请求后端服务的解决方案主要有三种：

- 网络隔离

  后端普通服务都部署在**内网**，通过防火墙策略限制只允许网关应用访问后端服务

- 应用层拦截

  请求后端服务时通过拦截器**校验**请求是否来自网关，如果不来自网关则提示不允许访问

- 使用 Kubernetes 部署

  在使用 Kubernetes 部署 SpringCloud 架构时给网关的 Service 配置 NodePort，其他后端服务的 Service 使用ClusterIp，这样在集群外就只能访问到网关了

如果采用应用层拦截，在请求经过网关时添加额外的 Header 示例：

```java
@Component
@Order(0)
public class GatewayRequestFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        byte[] token = Base64Utils.encode("GATEWAY_TOKEN_VALUE".getBytes());
        String[] headerValues = {new String(token)};        
        ServerHttpRequest build = exchange.getRequest()
                .mutate()
                .header("geteway_token", headerValues)
                .build();
        ServerWebExchange newExchange = exchange.mutate().request(build).build();
        
        return chain.filter(newExchange);
    }
}
```



## GateWay 实战

源码地址：[https://github.com/langyastudio/langya-tech/tree/master/spring-cloud](https://github.com/langyastudio/langya-tech/tree/master/spring-cloud)

使用 Nacos Discovery Starter 、 Spring Cloud Gateway Starter 完成 Spring Cloud 服务路由。

- [Nacos](https://github.com/alibaba/Nacos) 是阿里巴巴开源的一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台
- [Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway) 是 spring cloud 官方开源的一个在 SpringMVC 上可以构建 API 网关的库



### 如何接入

> 通过修改官方示例 [nacos-gateway-example](https://github.com/alibaba/spring-cloud-alibaba/tree/2020.0.0/spring-cloud-alibaba-examples/nacos-example/nacos-gateway-example) 来演示 API 网关的功能

修改 pom.xml 文件，引入 Nacos Discovery Starter、Spring Cloud Gateway Starter 依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```



配置文件中配置 Nacos Server 地址 与 Spring Cloud Gateway 路由

`Spring Cloud Gateway` 目前有两种方式进行配置：

- `application.yml` 配置文件方式
- 通过 @Bean 注解 `RouteLocator` 方法返回值

```yml
spring:
  main:
    #springcloudgateway 的内部是通过 netty+webflux 实现的
    #webflux 实现和 spring-boot-starter-web 依赖冲突
    web-application-type: reactive

  application:
    name: nacos-gateway-discovery

  cloud:
    #Nacos config
    nacos:
      username: nacos
      password: nacos
      discovery:
        server-addr: 127.0.0.1:8848

    #spring cloud gateway config
    gateway:
      discovery:
        locator:
          #设为true便开启通过服务中心的自动根据 serviceId 创建路由的功能
          enabled: true 
          #服务名使用小写
          lowerCaseServiceId: true 
          #网关访问时应用名作为上下文
          filters[0]: PreserveHostHeader
      routes:
        - id: nacos-gateway  
          uri: lb://nacos-discovery-provider
          # 网关的 /nacos 映射为 nacos-discovery-provider 服务
          predicates:
            - Path=/nacos/**
          filters:
            - StripPrefix=1
```



使用 `@EnableDiscoveryClient` 注解开启服务注册与发现功能

```java
@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```



### 服务启动

- 启动 nacos-discovery-provider 服务
- 启动本实例的网关服务

此时执行 `http://192.168.123.100:18061/nacos/**` 请求时，其实转发到 nacos-discovery-provider 服务，如下图所示：

```bash
#由于使用了 StripPrefix=1
#实际转发到 nacos-discovery-provider 服务的 /echo/aaa 
curl 'http://192.168.123.100:18061/nacos/echo/aaa' 
 
hello Nacos Discovery aaa
```



### 全局异常处理

在SpringCloud gateway中默认使用 `DefaultErrorWebExceptionHandler` 来处理异常。这个可以通过配置类 `ErrorWebFluxAutoConfiguration` 得之。

在 `DefaultErrorWebExceptionHandler` 类中的默认异常处理逻辑如下：

```java
public class DefaultErrorWebExceptionHandler extends AbstractErrorWebExceptionHandler {
 ...
    protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {
        return RouterFunctions.route(this.acceptsTextHtml(), this::renderErrorView).andRoute(RequestPredicates.all(), this::renderErrorResponse);
    }
   ...
}
```

根据请求头确认返回什么资源格式。

返回的数据内容在 `DefaultErrorAttributes` 类中构建而成。

```java
public class DefaultErrorAttributes implements ErrorAttributes {
 ...
    public Map<String, Object> getErrorAttributes(ServerRequest request, boolean includeStackTrace) {
        Map<String, Object> errorAttributes = new LinkedHashMap();
        errorAttributes.put("timestamp", new Date());
        errorAttributes.put("path", request.path());
        Throwable error = this.getError(request);
        MergedAnnotation<ResponseStatus> responseStatusAnnotation = MergedAnnotations.from(error.getClass(), SearchStrategy.TYPE_HIERARCHY).get(ResponseStatus.class);
        HttpStatus errorStatus = this.determineHttpStatus(error, responseStatusAnnotation);
        errorAttributes.put("status", errorStatus.value());
        errorAttributes.put("error", errorStatus.getReasonPhrase());
        errorAttributes.put("message", this.determineMessage(error, responseStatusAnnotation));
        errorAttributes.put("requestId", request.exchange().getRequest().getId());
        this.handleException(errorAttributes, this.determineException(error), includeStackTrace);
        return errorAttributes;
    }
 ...
}
```

阅读到这里就可以看到为什么上面会返回那样的数据格式，接下来需要改写返回格式。



这里可以自定义一个 `CustomErrorWebExceptionHandler` 类用来继承 `DefaultErrorWebExceptionHandler`，然后修改生成前端响应数据的逻辑。再然后定义一个配置类，写法可以参考 `ErrorWebFluxAutoConfiguration`，简单将异常类替换成 `CustomErrorWebExceptionHandler`类即可。

这种方法大家请自行研究，基本都是复制代码，改写不复杂，这种方法就不演示了，这里给大家介绍另外一种写法：

定义一个全局异常类 `GlobalErrorWebExceptionHandler` 让其直接实现顶级接口 `ErrorWebExceptionHandler` 重写 `handler()`方法，在 `handler()`方法中返回自定义的响应类。但是需要注意重写的实现类优先级一定要小于内置 `ResponseStatusExceptionHandler` 经过它处理的获取对应错误类的响应码。

代码如下：

```java
/**
 * 网关全局异常处理
 */
@Slf4j
@Order(-1)
@Configuration
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
public class GlobalErrorWebExceptionHandler implements ErrorWebExceptionHandler {

    private final ObjectMapper objectMapper;

    @Override
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex) {
        ServerHttpResponse response = exchange.getResponse();
        if (response.isCommitted()) {
            return Mono.error(ex);
        }

        // 设置返回JSON
        response.getHeaders().setContentType(MediaType.APPLICATION_JSON);
        if (ex instanceof ResponseStatusException) {
            response.setStatusCode(((ResponseStatusException) ex).getStatus());
        }

        return response.writeWith(Mono.fromSupplier(() -> {
            DataBufferFactory bufferFactory = response.bufferFactory();
            try {
                //返回响应结果
                return bufferFactory.wrap(objectMapper.writeValueAsBytes(ResultData.fail(500,ex.getMessage())));
            }
            catch (JsonProcessingException e) {
                log.error("Error writing response", ex);
                return bufferFactory.wrap(new byte[0]);
            }
        }));
    }
}
```



### 隐私接口禁止外部访问

SpringCloud 体系中如何防止内部隐私接口被网关调用？解决方案主要有：

**黑名单机制**

即将这些接口放入“黑名单”中存储起来，在网关启动时读取黑名单配置，然后校验是否在黑名单中



**接口路径**

即给接口指定访问路径时采用这样的格式 : **/访问控制/接口**。访问控制可以有以下几个规则（参考JAVA包规范），可根据业务需要进行扩展。

```
pb - public 所有请求均可访问

pt - protected 需要进行token认证通过后方可访问

pv - private 无法通过网关访问，只能微服务内部调用

df - default 网关请求token认证，并且请求参数和返回结果进行加解密

...
```

有了这套接口规范以后，就可以灵活控制接口访问权限，然后在网关对接口路径进行校验，如果命中对应的访问控制规则就进行对应的逻辑处理。

```java
@Component
@Order(0)
@Slf4j
public class GatewayRequestFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        //获取请求路径
        String rawPath = exchange.getRequest().getURI().getRawPath();

        if(isPv(rawPath)){
            throw new HttpServerErrorException(HttpStatus.FORBIDDEN,"can't access private API");
        }
        
        return chain.filter(newExchange);
    }

    /**
     * 判断是否内部私有方法
     * @param requestURI 请求路径
     * @return boolean
     */
    private boolean isPv(String requestURI) {
        return isAccess(requestURI,"/pv");
    }

    /**
     * 网关访问控制校验
     */
    private boolean isAccess(String requestURI, String access) {
        //后端标准请求路径为 /访问控制/请求路径
        int index = requestURI.indexOf(access);
        return index >= 0 && StringUtils.countOccurrencesOf(requestURI.substring(0,index),"/") < 1;
    }
}
```



### 灰度发布

- 通过实现 ServiceInstanceListSupplier 来自定义服务筛选逻辑，可以直接继承 DelegatingServiceInstanceListSupplier 来实现

```java
/**
 * 参考：org.springframework.cloud.loadbalancer.core.ZonePreferenceServiceInstanceListSupplier
 */
@Log4j2
public class VersionServiceInstanceListSupplier extends DelegatingServiceInstanceListSupplier {


    public VersionServiceInstanceListSupplier(ServiceInstanceListSupplier delegate) {
        super(delegate);
    }


    @Override
    public Flux<List<ServiceInstance>> get() {
        return delegate.get();
    }

    @Override
    public Flux<List<ServiceInstance>> get(Request request) {
        return delegate.get(request).map(instances -> filteredByVersion(instances,getVersion(request.getContext())));
    }


    /**
     * filter instance by requestVersion
     */
    private List<ServiceInstance> filteredByVersion(List<ServiceInstance> instances, String requestVersion) {
        log.info("request version is {}",requestVersion);
        if(StringUtils.isEmpty(requestVersion)){
            return instances;
        }

        List<ServiceInstance> filteredInstances = instances.stream()
                .filter(instance -> requestVersion.equalsIgnoreCase(instance.getMetadata().getOrDefault("version","")))
                .collect(Collectors.toList());

        if (filteredInstances.size() > 0) {
            return filteredInstances;
        }

        return instances;
    }

    private String getVersion(Object requestContext) {
        if (requestContext == null) {
            return null;
        }
        String version = null;
        if (requestContext instanceof RequestDataContext) {
            version = getVersionFromHeader((RequestDataContext) requestContext);
        }
        return version;
    }

    /**
     * get version from header
     */
    private String getVersionFromHeader(RequestDataContext context) {
        if (context.getClientRequest() != null) {
            HttpHeaders headers = context.getClientRequest().getHeaders();
            if (headers != null) {
                //could extract to the properties
                return headers.getFirst("version");
            }
        }
        
        return null;
    }
}
```

实现原理跟自定义负载均衡策略一样，根据 version 匹配符合要求的服务实例。

- 编写配置类 `VersionServiceInstanceListSupplierConfiguration`，用于替换默认服务实例筛选逻辑

```java
public class VersionServiceInstanceListSupplierConfiguration {
    @Bean
    ServiceInstanceListSupplier serviceInstanceListSupplier(ConfigurableApplicationContext context) {
        ServiceInstanceListSupplier delegate = ServiceInstanceListSupplier.builder()
                .withDiscoveryClient()
                .withCaching()
                .build(context);
        return new VersionServiceInstanceListSupplier(delegate);
    }
}
```

- 在网关启动类使用注解 @LoadBalancerClient 指定哪些服务使用自定义负载均衡算法
  通过 `@LoadBalancerClient(value = "nacos-discovery-provider", configuration = VersionServiceInstanceListSupplierConfiguration.class)`，对于 nacos-discovery-provider 启用自定义负载均衡算法 或通过 `@LoadBalancerClients(defaultConfiguration = VersionServiceInstanceListSupplierConfiguration.class)` 为所有服务启用自定义负载均衡算法



## 参考

[Spring Cloud GateWay 路由转发规则介绍](https://segmentfault.com/a/1190000019101829)

[Spring Cloud Gateway：新一代API网关服务](http://www.macrozheng.com/#/cloud/gateway?id=spring-cloud-gateway：新一代api网关服务)

[nacos-gateway-example](https://github.com/alibaba/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/nacos-example/nacos-gateway-example)

[隐私接口禁止外部访问](https://mp.weixin.qq.com/s?__biz=MzAwMTk4NjM1MA==&mid=2247493894&idx=1&sn=eedad2851638c9ac9b8215adc5a0987d&chksm=9ad3f347ada47a5199bce3225969a24a0ddfe9a063d8a8a5838d3fea85ef885bd9b2887d4ce8&token=1863605670&lang=zh_CN#rd)

[实现网关的灰度发布](https://mp.weixin.qq.com/s?__biz=MzAwMTk4NjM1MA==&mid=2247491931&idx=1&sn=16faf7f21b8606569dbc68242c94b570&chksm=9ad3fb1aada4720c33a937e9311e5cf5776256fe706afcb31da2c4a9757605a570f36137f0a2&scene=178&cur_album_id=1418244755364134912#rd)

[5 种 API 网关技术选型](https://mp.weixin.qq.com/s/KWEH5qBCLW8WXa6S55SNkw)
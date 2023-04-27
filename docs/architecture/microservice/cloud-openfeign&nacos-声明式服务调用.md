源码地址：[https://github.com/langyastudio/langya-tech/tree/master/spring-cloud](https://github.com/langyastudio/langya-tech/tree/master/spring-cloud)

> 从 `SpringCloud 2020` 版本开始 ribbon 默认被 **移除**，替代品为 [spring-cloud-loadbalancer](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-loadbalancer)



## 工作流程

openFeign 服务调用（spring cloud 2020 开始使用 spring-cloud-loadbalancer 作为负载均衡组件）

![img](https://img-note.langyastudio.com/202201051650195.png?x-oss-process=style/watermark)



## 如何接入

> 通过修改官方示例 [nacos-discovery-consumer-example](https://github.com/alibaba/spring-cloud-alibaba/tree/2020.0.0/spring-cloud-alibaba-examples/nacos-example/nacos-discovery-example/nacos-discovery-consumer-example) 来演示服务发现的功能
>
> 使用 FeignClient 作为服务调用方的客户端

### 引入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```



### openFeign 

![img](https://img-note.langyastudio.com/202201052236494.png?x-oss-process=style/watermark)

FeignClient 已经默认集成了 **Ribbon** 或者 **spring-cloud-loadbalancer**（spring cloud 2020），此处演示如何配置一个 FeignClient

Ribbon（老版本）：openfeign 中 ribbon 默认负载算法为**轮询**、默认重试机制为当前请求实例 **重试1次**



### 创建服务提供者

既然是微服务之间的相互调用，那么一定会有服务提供者了，创建`openFeign-provider9005`，注册进入Nacos中，配置如下：

```yaml
server:
  port: 9005
spring:
  application:
    ## 指定服务名称，在nacos中的名字
    name: nacos-discovery-provider
  cloud:
    nacos:
      discovery:
        # nacos的服务地址，nacos-server中IP地址:端口号
        server-addr: 127.0.0.1:8848
management:
  endpoints:
    web:
      exposure:
        ## yml文件中存在特殊字符，必须用单引号包含，否则启动报错
        include: '*'
```

**注意**：此处的 `spring.application.name` 指定的名称将会在 openFeign 接口调用中使用



### 创建服务消费者

#### 添加注解@EnableFeignClients开启openFeign功能

老套路了，在Spring boot 主启动类上添加一个注解`@EnableFeignClients`，开启openFeign功能，如下：

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class OpenFeignConsumer9006Application
{
    public static void main(String[] args) {
        SpringApplication.run(OpenFeignConsumer9006Application.class, args);
    }
}
```



#### 新建openFeign接口

```java
@FeignClient(name = "nacos-discovery-provider")
public interface EchoService
{
    @GetMapping("/echo/{str}")
    String echo(@PathVariable("str") String str);

    @GetMapping("/divide")
    String divide(@RequestParam("a") Integer a, @RequestParam("b") Integer b);
}
```

- 使用 @FeignClient 注解将 EchoService 这个接口包装成一个 FeignClient
  - name 与 value 相同，name 属性会作为**微服务的名称**，用于服务发现。属性 name 对应服务名 nacos-discovery-provider
  - url: 一般用于调试，可以手动指定 @FeignClient 调用的地址
  - fallback: 定义容错的处理类，当调用远程接口失败或超时时，会调用对应接口的容错逻辑，fallback 指定的类必须实现 @FeignClient 标记的接口
  - configuration: Feign 配置类，可以自定义 Feign 的 Encoder、Decoder、LogLevel、Contract
  - fallbackFactory:  工厂类，用于生成 fallback 类示例，通过这个属性我们可以实现每个接口通用的容错逻辑，减少重复的代码
- echo 方法上的 @RequestMapping 注解将 echo 方法与 URL "/echo/{str}" 相对应，@PathVariable 注解将 URL 路径中的 `{str}` 对应成 echo 方法的参数 str



#### 测试代码

完成以上配置后，将两者自动注入到 TestController 中

```java
 @RestController
 public class TestController { 
     @Autowired
     private EchoService echoService; 

     @GetMapping(value = "/echo-feign/{str}")
     public String feign(@PathVariable String str) {
         return echoService.echo(str);
     }
 }
```

再配置 nacos 与端口即可



#### 服务启动

启动应用，支持 IDE 直接启动和编译打包后启动。

多次调用接口：  http://127.0.0.1:18091/echo-feign/12345 可以看到浏览器显示 nacos-discovery-provider 返回的消息 "hello Nacos Discovery 1234"，证明服务发现生效

可以发现两个 nacos-discovery-provider 的控制台交替打印如下信息（负载均衡生效）：

```bash
2022-01-04 19:26:08.643  INFO 16516 --- [io-18082-exec-4] c.l.c.c.EchoController                   : 1234
```



## openFeign 如何传参

开发中接口传参的方式有很多，但是在openFeign中的传参是有一定规则的，下面详细介绍。

### 传递  JSON 数据

这个也是接口开发中常用的传参规则，在Spring Boot 中通过`@RequestBody`标识入参。

provider接口中JSON传参方法如下：

```java
@RestController
@RequestMapping("/openfeign/provider")
public class OpenFeignProviderController {
    @PostMapping("/order2")
    public Order createOrder2(@RequestBody Order order){
        return order;
    }
}
```

consumer中openFeign接口中传参代码如下：

```java
@FeignClient(value = "openFeign-provider")
public interface OpenFeignService {
    /**
     * 参数默认是@RequestBody标注的，这里的@RequestBody可以不填
     * 方法名称任意
     */
    @PostMapping("/openfeign/provider/order2")
    Order createOrder2(@RequestBody Order order);
}
```

注意：`openFeign`默认的传参方式就是JSON传参（`@RequestBody`），因此定义接口的时候可以不用`@RequestBody`注解标注，不过为了规范，一般都填上。



### POJO 表单传参

这种传参方式也是比较常用，参数使用POJO对象接收。

provider服务提供者代码如下：

```java
@RestController
@RequestMapping("/openfeign/provider")
public class OpenFeignProviderController {
    @PostMapping("/order1")
    public Order createOrder1(Order order){
        return order;
    }
}
```

consumer消费者openFeign代码如下：

```java
@FeignClient(value = "openFeign-provider")
public interface OpenFeignService {
    /**
     * 参数默认是@RequestBody标注的，如果通过POJO表单传参的，使用@SpringQueryMap标注
     */
    @PostMapping("/openfeign/provider/order1")
    Order createOrder1(@SpringQueryMap Order order);
}
```

网上很多人疑惑POJO表单方式如何传参，官方文档明确给出了解决方案，如下：

![图片](https://img-note.langyastudio.com/202304172104337.png?x-oss-process=style/watermark)

openFeign提供了一个注解`@SpringQueryMap`完美解决POJO表单传参。



### URL 中携带参数

此种方式针对restful方式中的GET请求，也是比较常用请求方式。

provider服务提供者代码如下：

```java
@RestController
@RequestMapping("/openfeign/provider")
public class OpenFeignProviderController {

    @GetMapping("/test/{id}")
    public String test(@PathVariable("id")Integer id){
        return "accept one msg id="+id;
}
```

consumer消费者openFeign接口如下：

```java
@FeignClient(value = "openFeign-provider")
public interface OpenFeignService {

    @GetMapping("/openfeign/provider/test/{id}")
    String get(@PathVariable("id")Integer id);
}
```

使用注解`@PathVariable`接收url中的占位符，这种方式很好理解。



### 另类传参

此种方式传参不建议使用，但是也有很多开发在用。

provider服务提供者代码如下：

```java
@RestController
@RequestMapping("/openfeign/provider")
public class OpenFeignProviderController {
    @PostMapping("/test2")
    public String test2(String id,String name){
        return MessageFormat.format("accept on msg id={0}，name={1}",id,name);
    }
}
```

consumer消费者openFeign接口传参如下：

```java
@FeignClient(value = "openFeign-provider")
public interface OpenFeignService {
    /**
     * 必须要@RequestParam注解标注，且value属性必须填上参数名
     * 方法参数名可以任意，但是@RequestParam注解中的value属性必须和provider中的参数名相同
     */
    @PostMapping("/openfeign/provider/test2")
    String test(@RequestParam("id") String arg1,@RequestParam("name") String arg2);
}
```



## 超时如何处理

想要理解超时处理，先看一个例子：我将provider服务接口睡眠3秒钟，接口如下：

```java
@PostMapping("/test2")
public String test2(String id,String name) throws InterruptedException {
        Thread.sleep(3000);
        return MessageFormat.format("accept on msg id={0}，name={1}",id,name);
}
```

此时，我们调用consumer的openFeign接口返回结果如下图：

![图片](https://img-note.langyastudio.com/202304172104332.png?x-oss-process=style/watermark)

很明显的看出程序异常了，返回了接口调用超时。what？why？...........

openFeign其实是有默认的超时时间的，默认分别是连接超时时间`10秒`、读超时时间`60秒`，源码在`feign.Request.Options#Options()`这个方法中，如下图：

![图片](https://img-note.langyastudio.com/202304172104302.png?x-oss-process=style/watermark)

那么问题来了：**为什么我只设置了睡眠3秒就报超时呢？**

其实openFeign集成了Ribbon，Ribbon的默认超时连接时间、读超时时间都是是1秒，源码在`org.springframework.cloud.openfeign.ribbon.FeignLoadBalancer#execute()`方法中，如下图：

![图片](https://img-note.langyastudio.com/202304172104491.png?x-oss-process=style/watermark)

**源码大致意思**：如果openFeign没有设置对应得超时时间，那么将会采用Ribbon的默认超时时间。

理解了超时设置的原理，由之产生两种方案也是很明了了，如下：

- 设置openFeign的超时时间
- 设置Ribbon的超时时间



### 设置Ribbon的超时时间（不推荐）

设置很简单，在配置文件中添加如下设置：

```yaml
ribbon:
  # 值的是建立链接所用的时间，适用于网络状况正常的情况下， 两端链接所用的时间
  ReadTimeout: 5000
  # 指的是建立链接后从服务器读取可用资源所用的时间
  ConectTimeout: 5000
```



### 设置openFeign的超时时间（推荐）

openFeign设置超时时间非常简单，只需要在配置文件中配置，如下：

```yaml
feign:
  client:
    config:
      ## default 设置的全局超时时间，指定服务名称可以设置单个服务的超时时间
      default:
        connectTimeout: 5000
        readTimeout: 5000
```

> default设置的是全局超时时间，对所有的openFeign接口服务都生效

但是正常的业务逻辑中可能涉及到多个openFeign接口的调用，如下图：

![图片](https://img-note.langyastudio.com/202304172104946.png?x-oss-process=style/watermark)

上图中的伪代码如下：

```java
public T invoke(){
    //1. 调用serviceA
    serviceA();
    
    //2. 调用serviceA
    serviceB();
    
    //3. 调用serviceA
    serviceC();
}
```

那么上面配置的全局超时时间能不能通过呢？很显然是`serviceA`、`serviceB`能够成功调用，但是`serviceC`并不能成功执行，肯定报超时。

此时我们可以给`serviceC`这个服务单独配置一个超时时间，配置如下：

```java
feign:
  client:
    config:
      ## default 设置的全局超时时间，指定服务名称可以设置单个服务的超时时间
      default:
        connectTimeout: 5000
        readTimeout: 5000
      ## 为serviceC这个服务单独配置超时时间
      serviceC:
        connectTimeout: 30000
        readTimeout: 30000
```

> **注意**：单个配置的超时时间将会覆盖全局配置。



## 如何开启日志增强

openFeign虽然提供了日志增强功能，但是默认是不显示任何日志的，不过开发者在调试阶段可以自己配置日志的级别。

openFeign的日志级别如下：

- **NONE**：默认的，不显示任何日志;
- **BASIC**：仅记录请求方法、URL、响应状态码及执行时间;
- **HEADERS**：除了BASIC中定义的信息之外，还有请求和响应的头信息;
- **FULL**：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据。

配置起来也很简单，步骤如下：



### 配置类中配置日志级别

需要自定义一个配置类，在其中设置日志级别，如下：

![图片](https://img-note.langyastudio.com/202304172104524.png?x-oss-process=style/watermark)

> **注意**：这里的logger是feign包里的。



### yaml文件中设置接口日志级别

只需要在配置文件中调整指定包或者openFeign的接口日志级别，如下：

```yaml
logging:
  level:
    cn.myjszl.service: debug
```

这里的`cn.myjszl.service`是openFeign接口所在的包名，当然你也可以配置一个特定的openFeign接口。



### 演示效果

上述步骤将日志设置成了`FULL`，此时发出请求，日志效果如下图：

![图片](https://img-note.langyastudio.com/202304172104070.png?x-oss-process=style/watermark)

日志中详细的打印出了请求头、请求体的内容。



## 如何替换默认的 httpclient

Feign在默认情况下使用的是JDK原生的**URLConnection**发送HTTP请求，没有连接池，但是对每个地址会保持一个长连接，即利用HTTP的persistence connection。

在生产环境中，通常不使用默认的http client，通常有如下两种选择：

- 使用**ApacheHttpClient**
- 使用**OkHttp**

至于哪个更好，其实各有千秋，我比较倾向于ApacheHttpClient，毕竟老牌子了，稳定性不在话下。

那么如何替换掉呢？其实很简单，下面演示使用ApacheHttpClient替换。

### 添加ApacheHttpClient依赖

在openFeign接口服务的pom文件添加如下依赖：

```xml
<!--     使用Apache HttpClient替换Feign原生httpclient-->
    <dependency>
      <groupId>org.apache.httpcomponents</groupId>
      <artifactId>httpclient</artifactId>
    </dependency>
    
    <dependency>
      <groupId>io.github.openfeign</groupId>
      <artifactId>feign-httpclient</artifactId>
    </dependency>
```

为什么要添加上面的依赖呢？从源码中不难看出，请看`org.springframework.cloud.openfeign.FeignAutoConfiguration.HttpClientFeignConfiguration`这个类，代码如下：

![图片](https://img-note.langyastudio.com/202304172104984.png?x-oss-process=style/watermark)

上述红色框中的生成条件，其中的`@ConditionalOnClass(ApacheHttpClient.class)`，必须要有`ApacheHttpClient`这个类才会生效，并且`feign.httpclient.enabled`这个配置要设置为`true`。



### 配置文件中开启

在配置文件中要配置开启，代码如下：

```yaml
feign:
  client:
    httpclient:
      # 开启 Http Client
      enabled: true
```



### 如何验证已经替换成功

其实很简单，在`feign.SynchronousMethodHandler#executeAndDecode()`这个方法中可以清楚的看出调用哪个client，如下图：

![图片](https://img-note.langyastudio.com/202304172104540.png?x-oss-process=style/watermark)

上图中可以看到最终调用的是`ApacheHttpClient`。



## 如何通讯优化

在讲如何优化之前先来看一下**GZIP** 压缩算法，概念如下：

> gzip是一种数据格式，采用用deflate算法压缩数据；gzip是一种流行的数据压缩算法，应用十分广泛，尤其是在Linux平台。

**当GZIP压缩到一个纯文本数据时，效果是非常明显的，大约可以减少70％以上的数据大小。**

网络数据经过压缩后实际上降低了网络传输的字节数，最明显的好处就是可以加快网页加载的速度。网页加载速度加快的好处不言而喻，除了节省流量，改善用户的浏览体验外，另一个潜在的好处是GZIP与搜索引擎的抓取工具有着更好的关系。例如 Google就可以通过直接读取GZIP文件来比普通手工抓取更快地检索网页。

GZIP压缩传输的原理如下图：

![图片](https://img-note.langyastudio.com/202304172104648.png?x-oss-process=style/watermark)

按照上图拆解出的步骤如下：

- 客户端向服务器请求头中带有：`Accept-Encoding:gzip,deflate` 字段，向服务器表示，客户端支持的压缩格式（gzip或者deflate)，如果不发送该消息头，服务器是不会压缩的。
- 服务端在收到请求之后，如果发现请求头中含有`Accept-Encoding`字段，并且支持该类型的压缩，就对响应报文压缩之后返回给客户端，并且携带`Content-Encoding:gzip`消息头，表示响应报文是根据该格式压缩过的。
- 客户端接收到响应之后，先判断是否有Content-Encoding消息头，如果有，按该格式解压报文。否则按正常报文处理。

openFeign支持**请求/响应**开启GZIP压缩，整体的流程如下图：

![图片](https://img-note.langyastudio.com/202304172104785.png?x-oss-process=style/watermark)

上图中涉及到GZIP传输的只有两块，分别是**Application client -> Application Service**、 **Application Service->Application client**。

**注意**：openFeign支持的GZIP仅仅是在openFeign接口的请求和响应，即是openFeign消费者调用服务提供者的接口。

openFeign开启GZIP步骤也是很简单，只需要在配置文件中开启如下配置：

```yaml
feign:
  ## 开启压缩
  compression:
    request:
      enabled: true
      ## 开启压缩的阈值，单位字节，默认2048，即是2k，这里为了演示效果设置成10字节
      min-request-size: 10
      mime-types: text/xml,application/xml,application/json
    response:
      enabled: true
```

上述配置完成之后，发出请求，可以清楚看到请求头中已经携带了GZIP压缩，如下图：

![图片](https://img-note.langyastudio.com/202304172104983.png?x-oss-process=style/watermark)



## 异常处理

当 feign 调用结果为非 200 的响应码时就触发了 Feign 的异常解析，Feign 的异常解析器会将其包装成 FeignException，即在我们业务异常的基础上再包装一次。

很显然，这个包装后的异常我们并不需要，我们应该直接将捕获到的生产者的业务异常直接抛给前端，只需要重写 Feign的异常解析器，重新实现 decode 逻辑，返回正常的 Exception 即可，而后全局异常拦截器又会捕获 Exception

- 重写 Feign 异常解析器

```java
@Log4j2
public class OpenFeignErrorDecoder implements ErrorDecoder
{
    /**
     * Feign 异常解析
     */
    @Override
    public Exception decode(String methodKey, Response response)
    {
        log.error("feign client error,response is {}:", response);
        try
        {
            //获取数据
            String body = Util.toString(response.body().asReader(Charset.defaultCharset()));

            ResultInfo<?> resultData = JSON.parseObject(body, ResultInfo.class);
            if (!resultData.isSuccess())
            {
                return new MyException(resultData.getStatus(), resultData.getMessage());
            }

        }
        catch (IOException e)
        {
            e.printStackTrace();
        }

        return new MyException("Feign client 调用异常");
    }
}
```

- 在 Feign 配置文件中注入自定义的异常解码器

```java
@ConditionalOnClass(Feign.class)
@Configuration
public class OpenFeignConfig {  
    /**
      * 自定义异常解码器
      */
    @Bean
    public ErrorDecoder errorDecoder()
    {
        return new OpenFeignErrorDecoder();
    }
}
```



## 如何熔断降级

常见的熔断降级框架有`Hystrix`、`Sentinel`，openFeign默认支持的就是`Hystrix`，这个在官方文档上就有体现，毕竟是一奶同胞嘛，哈哈...........

但是阿里的Sentinel无论是功能特性、简单易上手等各方面都完全秒杀Hystrix，因此此章节就使用**openFeign+Sentinel**进行整合实现服务降级。

> **说明**：此处并不着重介绍Sentinel，陈某打算放在下一篇文章详细介绍Sentinel的强大之处。

### 添加Sentinel依赖

在`openFeign-consumer9006`消费者的pom文件添加sentinel依赖（由于使用了聚合模块，不指定版本号），如下：

```xml
<dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```



### 配置文件中开启sentinel熔断降级

要想openFeign使用sentinel的降级功能，还需要在配置文件中开启，添加如下配置：

```xml
feign:
  sentinel:
    enabled: true
```



### 添加降级回调类

这个类一定要和openFeign接口实现同一个类，如下图：

![图片](https://img-note.langyastudio.com/202304172104971.png?x-oss-process=style/watermark)

`OpenFeignFallbackService`这个是降级回调的类，一旦`OpenFeignService`中对应得接口出现了异常则会调用这个类中对应得方法进行降级处理。



### 添加fallback属性

在`@FeignClient`中添加`fallback`属性，属性值是降级回调的类，如下：

```java
@FeignClient(value = "openFeign-provider",fallback = OpenFeignFallbackService.class)
public interface OpenFeignService {}
```



### 演示

经过如上4个步骤，openFeign的熔断降级已经设置完成了，此时演示下效果。

通过postman调用`http://localhost:9006/openfeign/order3`这个接口，正常逻辑返回如下图：

![图片](https://img-note.langyastudio.com/202304172104238.png?x-oss-process=style/watermark)

现在手动造个异常，在服务提供的接口中抛出异常，如下图：

![图片](https://img-note.langyastudio.com/202304172104717.png?x-oss-process=style/watermark)

此时重新调用`http://localhost:9006/openfeign/order3`，返回如下图：

![图片](https://img-note.langyastudio.com/202304172104551.png?x-oss-process=style/watermark)



## 参考

[Nacos 官方文档](https://nacos.io/zh-cn/docs/what-is-nacos.html)

[spring-cloud-feign 官方文档](https://cloud.spring.io/spring-cloud-openfeign/reference/html/#spring-cloud-feign)

[nacos-example/nacos-discovery-example](https://github.com/alibaba/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/nacos-example/nacos-discovery-example)

[Openfeign与Ribbon](https://www.cnblogs.com/unchain/p/13405814.html)

[使用Feign的一些问题以及如何解决](https://mp.weixin.qq.com/s?__biz=MzAwMTk4NjM1MA==&mid=2247499532&idx=1&sn=3e1fdfaa339480b3b09632155e03a60a&chksm=9ad3e54dada46c5ba536975726b4c7dbba6f4010da8ffe80be0724171fbb0b9039843faf5d7f&token=1863605670&lang=zh_CN#rd)

[Feign传递Access_Token](https://mp.weixin.qq.com/s?__biz=MzAwMTk4NjM1MA==&mid=2247484278&idx=1&sn=8386577cefc184c4945ec3f448e972c4&chksm=9ad01937ada790210ec86d2c3231108e9378f3ef86941e07e1ac21ea91c6804c55a27810b96b&token=1863605670&lang=zh_CN#rd)

[一个注解优雅的实现 Feign 的重试调用](https://mp.weixin.qq.com/s/9p_uBRzDu5q70hzHpPY3Xg)

[OpenFeign的9个坑](https://mp.weixin.qq.com/s/8knF04uSYFKavxmkeitqSA)

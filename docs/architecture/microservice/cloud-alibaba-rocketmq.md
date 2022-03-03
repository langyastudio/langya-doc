## RocketMQ 介绍

详解了解可以查看如下文档：[rocketmq 基础知识](../../middleware/msg-queue/rocketmq/rocketmq-intro.md)

[RocketMQ](https://rocketmq.apache.org/) 是一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的**消息发布与订阅**服务。同时，广泛应用于多个领域，包括异步通信解耦、企业解决方案、金融支付、电信、电子商务、快递物流、广告营销、社交、即时通信、移动应用、手游、视频、物联网、车联网等。

具有以下特点：

- 能够保证严格的消息顺序
- 提供丰富的消息拉取模式
- 高效的订阅者水平扩展能力
- 实时的消息订阅机制
- 亿级消息堆积能力



## 如何选择 RocketMQ 实现

![6.png](https://img-note.langyastudio.com/202202181454939.png?x-oss-process=style/watermark)



##  Spring Cloud Stream

[Spring Cloud Stream](https%3A%2F%2Fgithub.com%2Fspring-cloud%2Fspring-cloud-stream) 是一个用于构建基于**消息**的微服务应用框架，使用 [Spring Integration](https%3A%2F%2Fwww.oschina.net%2Fp%2Fspring%2Bintegration) 与 Broker 进行连接。

> 一般来说，消息队列中间件都有一个 **Broker Server**（代理服务器），消息中转角色，负责存储消息、转发消息
>
> 例如在 RocketMQ 中，Broker 负责接收从生产者发送来的消息并存储、同时为消费者的拉取请求作准备。另外，Broker 也存储消息相关的元数据，包括消费者组、消费进度偏移和主题和队列消息等

Spring Cloud Stream 提供了消息中间件的**统一抽象**，推出了 publish-subscribe、consumer groups、partition 这些统一的概念。

Spring Cloud Stream 内部有两个概念：**Binder** 和 **Binding**

- Binder，**跟消息中间件集成的组件**，用来创建对应的 Binding。各消息中间件都有自己的 Binder 具体实现

  - Kafka 实现了 KafkaMessageChannelBinder
  - RabbitMQ 实现了 RabbitMessageChannelBinder
  - RocketMQ 实现了 RocketMQMessageChannelBinder

- Binding，包括 Input Binding 和 Output Binding。Binding 在消息中间件与应用程序提供的 Provider 和 Consumer 之间提供了一个**桥梁**，实现了开发者只需使用应用程序的 Provider 或 Consumer 生产或消费数据即可，屏蔽了开发者与底层消息中间件的接触

  ![img](https://img-note.langyastudio.com/202201201712740.webp?x-oss-process=style/watermark)

![TB1v8rcbUY1gK0jSZFCXXcwqXXa 1236 773](https://img-note.langyastudio.com/202201201609448.png?x-oss-process=style/watermark)



## RocketMQ 基本使用

本项目演示如何使用 RocketMQ Binder 完成 Spring Cloud 应用消息的订阅和发布。

源码地址：[https://github.com/langyastudio/langya-tech/tree/master/spring-cloud](https://github.com/langyastudio/langya-tech/tree/master/spring-cloud)

### 依赖环境

引入 Spring Cloud Alibaba RocketMQ 相关依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
</dependency>
```



### Producer 生产者

#### 配置文件

```yml
server:
  port: 28081

spring:
  application:
    name: rocketmq-produce

  cloud:
    stream:
      # 对应 BindingProperties Map
      bindings:
        output-common:
          #RocketMQ Topic
          destination: topic-common-01
          content-type: application/json
        output-tx:
          destination: topic-tx-01
          content-type: application/json

      # Spring Cloud Stream RocketMQ 配置项
      rocketmq:
        # 对应 RocketMQBinderConfigurationProperties 类
        binder:
          name-server: 192.168.123.22:9876

        # RocketMQ 自定义 Binding 配置项，对应 RocketMQBindingProperties Map
        bindings:
          output-common:
            # RocketMQ Producer 配置项，对应 RocketMQProducerProperties 类
            producer:
              #生产者分组
              group: group-common
              #同步发送消息
              sync: true
              #最大字节数
              maxMessageSize: 8249344
              #超时时间
              sendMessageTimeout: 3000
          tx-output:
            producer:
              #group name
              group: group-tx
              #发送事务消息
              transactional: true

logging:
  level:
    com:
      alibaba:
        cloud:
          stream:
            binder:
              rocketmq: DEBUG
```

- `spring.cloud.stream.bindings` 为 Binding 配置项

  Binding 分成 Input 和 Output 两种类型，但是在配置项中并不能体现出来，而是要在稍后搭配 **`@Input` or `@Output`** 注解，才会区分

- `spring.cloud.stream.rocketmq.binder` 为 RocketMQ Binder 配置项。

  `name-server`：RocketMQ Namesrv 地址。名称服务充当路由消息的提供者。生产者或消费者能够通过**名字服务**查找各主题相应的 Broker IP 列表。多个 Namesrv 实例组成集群，但相互独立，没有信息交换。

  

#### @Output 发送消息

```kotlin
public interface OutputSource
{
    @Output("output-common")
    MessageChannel sendCommon();

    @Output("output-tx")
    MessageChannel sendTx();
}
```

通过 `@Output` 注解，声明了一个名字为 `output-common` 的 Output Binding。注意，这个名字要和配置文件中的 `spring.cloud.stream.bindings` 配置项对应上

同时，`@Output` 注解的方法的返回结果为 MessageChannel 类型，可以使用它**发送消息**。



#### SenderService 消息发送服务

通过组装 `Message` 消息体并调用 `OutputSource` 接口，实现消息发送的逻辑

```java
@Service
public class SenderService
{
    @Autowired
    private OutputSource source;

    /**
     * 发送消息
     *
     * @param msg 消息内容
     * @param tag 标签
     * @param delay 设置延迟级别为x秒后消费
     * @return
     * @throws Exception
     */
    public <T> boolean sendObject(T msg, String tag, Integer delay) throws Exception
    {
        Message<T> message = MessageBuilder.withPayload(msg)
                .setHeader(MessageConst.PROPERTY_TAGS, tag)
                .setHeader(MessageConst.PROPERTY_DELAY_TIME_LEVEL, delay)
                .setHeader(MessageHeaders.CONTENT_TYPE, MimeTypeUtils.APPLICATION_JSON)
                .build();

        return source.sendCommon().send(message);
    }

    public <T> boolean sendTransactionalMsg(T msg, int num) throws Exception
    {
        Message<T> message  = MessageBuilder.withPayload(msg)
                .setHeader("tx-state", String.valueOf(num))
                .setHeader(MessageConst.PROPERTY_TAGS, "binder")
                .setHeader(MessageHeaders.CONTENT_TYPE, MimeTypeUtils.APPLICATION_JSON)
                .build();

        return source.sendTx().send(message);
    }
}
```



#### Controller 接口定义


提供发送消息的 HTTP 接口。代码如下：

```java
@RestController
@RequestMapping("/produce")
public class ProduceController
{
    @Autowired
    private SenderService senderService;

    @GetMapping("/send1")
    public boolean output1(@RequestParam("msg") String msg) throws Exception
    {
        int msgId = new SecureRandom().nextInt();
        return senderService.sendObject(new FooMsg(msgId, msg), "tagObj", 0);
    }

    @GetMapping("/send3")
    public boolean output3() throws Exception
    {
        // unknown message
        senderService.sendTransactionalMsg("transactional-msg1", 1);
        // rollback message
        senderService.sendTransactionalMsg("transactional-msg2", 2);
        // commit message
        senderService.sendTransactionalMsg("transactional-msg3", 3);

        return true;
    }
}
```



#### Application 入口

启动应用。代码如下：

```kotlin
@SpringBootApplication
@EnableBinding({OutputSource.class})
public class RocketMQProduceApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(RocketMQProduceApplication.class, args);
    }
}
```

使用 `@EnableBinding`注解，声明指定接口开启 Binding 功能，扫描其 `@Input` 和 `@Output` 注解。



### 消费者

#### 配置文件

```yml
server:
  # 随机端口，方便启动多个消费者
  port: ${random.int[10000,19999]}

spring:
  application:
    name: rocketmq-consume

  cloud:
    # Spring Cloud Stream 配置项，对应 BindingServiceProperties 类
    stream:
      # Binding 配置项，对应 BindingProperties Map
      bindings:
        input-common-1:
          # 目的地。这里使用 RocketMQ Topic
          destination: topic-common-01
          content-type: application/json
          ## 消费者分组, 命名规则：group+topic名+xx
          group: group-common-1
        input-common-2:
          destination: topic-common-01
          content-type: application/json
          consumer:
            concurrency: 20
            maxAttempts: 1
          group: group-common-2
        input-common-3:
          destination: topic-common-01
          content-type: application/json
          consumer:
            concurrency: 20
          group: group-common-3
        input-tx-1:
          destination: topic-tx-01
          content-type: text/json
          consumer:
            concurrency: 5
          group: group-tx-1

      # Spring Cloud Stream RocketMQ 配置项
      rocketmq:
        binder:
          name-server: 192.168.123.22:9876

        bindings:
          input-common-1:
            # RocketMQ Consumer 配置项
            consumer:
              # 是否开启消费，默认为 true
              enabled: true
              # 是否使用广播消费，默认为 false 使用集群消费，如果要使用广播消费值设成true
              broadcasting: false
              orderly: true
          input-common-2:
            consumer:
              orderly: false
          input-common-3:
            consumer:
              tags: tagObj
```

`spring.cloud.stream.rocketmq.bindings` 

- `enabled`：是否开启消费，默认为 `true`。在日常开发时，如果在本地环境不想消费，可以通过设置 `enabled`为 `false` 进行关闭

- `broadcasting`： 是否使用广播消费，默认为 `false` 使用的是集群消费。

  **集群消费（Clustering）**：集群消费模式下，相同 Consumer Group 的每个 Consumer 实例**平均分摊消息**

  **广播消费（Broadcasting）**：广播消费模式下，相同 Consumer Group 的每个 Consumer 实例都接收**全量消息**



#### @Input 接收消息

```kotlin
public interface InputBinding
{
    @Input("input-common-1")
    SubscribableChannel inputCommon1();

    @Input("input-common-2")
    SubscribableChannel inputCommon2();

    @Input("input-common-3")
    SubscribableChannel inputCommon3();

    @Input("input-tx-1")
    SubscribableChannel inputTx1();
}
```

通过 `@Input` 注解，声明了一个名字为 `input-common-1`  Input Binding。注意，这个名字要和配置文件中的`spring.cloud.stream.bindings` 配置项对应上。

同时，`@Input` 注解的方法的返回结果为 SubscribableChannel 类型，可以使用它订阅消息来消费。

```java
public interface SubscribableChannel extends MessageChannel {

 boolean subscribe(MessageHandler handler); // 订阅
 boolean unsubscribe(MessageHandler handler); // 取消订阅

}
```



#### ReceiveService 接收消息服务

通过组装 `StreamListener` ，实现消息接收的逻辑

```kotlin
@Service
public class ReceiveService
{
    @StreamListener("input-common-1")
    public void receiveInput1(@Payload FooMsg receiveMsg)
    {
        System.out.println("input1 receive: " + receiveMsg);
    }

    @StreamListener("input-common-2")
    public void receiveInput2(@Payload FooMsg receiveMsg)
    {
        System.out.println("input2 receive: " + receiveMsg);
    }

    @StreamListener("input-common-3")
    public void receiveInput3(@Payload FooMsg receiveMsg)
    {
        System.out.println("input3 receive: " + receiveMsg);
    }

    @StreamListener("input-tx-1")
    public void receiveTransactionalMsg(String transactionMsg)
    {
        System.out.println("input tx receive transaction msg: " + transactionMsg);
    }

}
```

在方法上，添加 `@StreamListener` 注解，声明对应的 Input Binding。

因为消费的消息是 POJO 类型，所以需要添加 `@Payload` 注解，声明需要进行反序列化成 POJO 对象。



#### Application 入口

启动应用。代码如下：

```kotlin
@SpringBootApplication
@EnableBinding({InputBinding.class})
public class RocketMQConsumerApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(RocketMQConsumerApplication.class, args);
    }
}
```

使用 `@EnableBinding` 注解，声明指定接口开启 Binding 功能，扫描其 `@Input` 和 `@Output` 注解



### 测试单集群多实例的场景

- 执行 ProducerApplication，启动**生产者**的实例
- 执行 RocketMQConsumerApplication，启动**消费者**的实例

之后，请求 [http://localhost:28081/produce/send1?msg=发送短信-0248](http://localhost:28081/produce/send1?msg=发送短信-0248) 接口三次，发送三条消息。此时在 IDEA 控制台看到消费者打印日志如下：

```swift
[onMessage][input-common-2 线程编号:99 消息内容：FooMsg(id=-1996543838, bar=发送短信-0248)] 
[onMessage][input-common-3 线程编号:98 消息内容：FooMsg(id=-1996543838, bar=发送短信-0248)] 
[onMessage][input-common-1 线程编号:97 消息内容：FooMsg(id=-1996543838, bar=发送短信-0248)] 
```

符合预期。从日志可以看出，每条消息被同一个分组仅被消费一次



## 定时消息

定时消息，是指消息发到 Broker 后，不能立刻被 Consumer 消费，要**等待特定的时间后**才能被消费

RocketMQ 暂时不支持任意的时间精度的延迟，而是固化了 18 个延迟级别。如下表格：

| 延迟级别 | 时间 | 延迟级别 | 时间 | 延迟级别 | 时间 |
| -------- | ---- | -------- | ---- | -------- | ---- |
| 1        | 1s   | 7        | 3m   | 13       | 9m   |
| 2        | 5s   | 8        | 4m   | 14       | 10m  |
| 3        | 10s  | 9        | 5m   | 15       | 20m  |
| 4        | 30s  | 10       | 6m   | 16       | 30m  |
| 5        | 1m   | 11       | 7m   | 17       | 1h   |
| 6        | 2m   | 12       | 8m   | 18       | 2h   |



## 消费重试

> 不过要注意，只有 **集群消费** 模式下，才有消息重试

RocketMQ 提供**消费重试**的机制。在消息**消费失败**的时候，RocketMQ 会通过**消费重试**机制，重新投递该消息给 Consumer ，让 Consumer 有机会重新消费消息，实现消费成功。

当然，RocketMQ 并不会无限重新投递消息给 Consumer 重新消费，而是在默认情况下，达到 16 次重试次数时，Consumer 还是消费失败时，该消息就会进入到**死信队列**。



**max-attempts** 次数

因为 Spring Cloud Stream 提供的重试间隔，是通过 sleep 实现，会**占掉当前线程**，影响 Consumer 的消费速度，所以这里并**不推荐使用**，因此设置 max-attempts 配置项为 1，禁用 Spring Cloud Stream 提供的重试功能，使用 RocketMQ 提供的重试功能。



**delay-level-when-next-consume** 策略

- -1：不重复，直接放入死信队列
- 0：RocketMQ Broker 控制重试策略
- \>0：RocketMQ Consumer 控制重试策略

每条消息的失败重试，是有一定的间隔时间。第一次重试消费按照延迟级别为 **3** 开始。所以，默认为 16 次重试消费，也非常好理解，毕竟延迟级别最高为 18 呀。



## 消费异常处理机制

> 如果异常处理方法成功，没有重新抛出异常，会认定为该消息被消费成功，所以就不会进行消费重试。

在 Spring Cloud Stream 中，提供了通用的消费异常处理机制，可以拦截到消费者消费消息时发生的异常，进行自定义的处理逻辑。

有**两种**方式来实现异常处理：

- **局部**的异常处理：通过订阅**指定错误 Channel** - `<destination>.<group>.errors`
- **全局**的异常处理：通过订阅**全局错误 Channel** - errorChannel

消费消息发生异常时，会发送错误消息 ErrorMessage 到对应的错误 Channel 中。同时，所有错误 Channel 都桥接到了 Spring Integration 定义的全局错误 Channel。

```java
    //<destination>.<group>.errors
    //局部错误
    @ServiceActivator(inputChannel = "topic-common-01.group-common-3.errors")
    public void handleError(ErrorMessage errorMessage)
    {
        System.out.printf("[handleError][payload：%s] %n", ExceptionUtils.getRootCauseMessage(errorMessage.getPayload()));
        System.out.printf("[handleError][originalMessage：%s] %n", errorMessage.getOriginalMessage());
        System.out.printf("[handleError][headers：%s] %n", errorMessage.getHeaders());
    }
    // errorChannel
    // 全局错误
    @StreamListener(IntegrationContextUtils.ERROR_CHANNEL_BEAN_NAME)
    public void globalHandleError(ErrorMessage errorMessage)
    {
        System.out.printf("[globalHandleError][payload：%s] %n",
                          ExceptionUtils.getRootCauseMessage(errorMessage.getPayload()));
        System.out.printf("[globalHandleError][originalMessage：%s] %n", errorMessage.getOriginalMessage());
        System.out.printf("[globalHandleError][headers：%s] %n", errorMessage.getHeaders());
    }
```



## 广播消息

广播消费模式下，相同 Consumer Group 的每个 Consumer 实例都接收全量的消息。

例如说，我们基于 WebSocket 实现了 IM 聊天，在我们给用户主动发送消息时，因为我们不知道用户连接的是哪个提供 WebSocket 的应用，所以可以通过 RocketMQ 广播消费，每个应用判断当前用户是否是和自己提供的 WebSocket 服务连接，如果是，则推送消息给用户。

**设置 `broadcasting` 配置项为 `true`**



## 顺序消息

RocketMQ 提供了两种顺序级别：

- 普通顺序消息：Producer 将相关联的消息发送到相同的消息队列
- 完全严格顺序：在【普通顺序消息】的基础上，Consumer 严格顺序消费

消息有序，指的是一类消息消费时，能按照发送的顺序来消费。例如：一个订单产生了三条消息分别是订单创建、订单付款、订单完成。消费时要按照这个顺序消费才能有意义，但是同时订单之间是可以并行消费的。RocketMQ 可以严格的保证消息有序。

分区顺序就是普通顺序消息，全局顺序就是完全严格顺序

- 全局顺序：对于指定的一个 Topic，所有消息按照严格的先入先出（FIFO）的顺序进行发布和消费。适用场景：性能要求不高，所有的消息严格按照 FIFO 原则进行消息发布和消费的场景
- 分区顺序：对于指定的一个 Topic，所有消息**根据 Sharding key 进行区块分区**。 同一个分区内的消息按照严格的 FIFO 顺序进行发布和消费。Sharding key 是顺序消息中用来区分不同分区的关键字段，和普通消息的 Key 是完全不同的概念。适用场景：性能要求高，以 Sharding key 作为分区字段，在同一个区块中严格的按照 FIFO 原则进行消息发布和消费的场景



**分区顺序：**

- 生产者

  添加 `partition-key-expression` 配置项，设置 Producer 发送顺序消息的 Sharding key

  `sync: true`  是否同步发送消息，默认为 false 异步

  ```yml
  server:
    port: 28081
  
  spring:
    application:
      name: rocketmq-produce
  
    cloud:
      stream:
        # Spring Cloud Stream 配置项，对应 BindingProperties Map
        bindings:
          output-common:
            #RocketMQ Topic
            destination: topic-common-01
            content-type: application/json
  
            # Producer 配置项，对应 ProducerProperties 类
            producer:
              # 分区 key 表达式。该表达式基于 Spring EL，从消息中获得分区 key
              partition-key-expression: payload['id']
  
        # Spring Cloud Stream RocketMQ 配置项
        rocketmq:
          # RocketMQ Binder 配置项，对应 RocketMQBinderConfigurationProperties 类
          binder:
            name-server: 192.168.123.22:9876
  
          # RocketMQ 自定义 Binding 配置项，对应 RocketMQBindingProperties Map
          bindings:
            output-common:
              # RocketMQ Producer 配置项，对应 RocketMQProducerProperties 类
              producer:
                #生产者分组
                group: group-common
                #同步发送消息
                sync: true
  ```

- 消费者

  添加 `orderly` 配置项，设置 Consumer 顺序消费消息

  ```yml
  server:
    # 随机端口，方便启动多个消费者
    port: ${random.int[10000,19999]}
  
  spring:
    application:
      name: rocketmq-consume
  
    cloud:
      # Spring Cloud Stream 配置项，对应 BindingServiceProperties 类
      stream:
        # Binding 配置项，对应 BindingProperties Map
        bindings:
          input-common-1:
            #重试次数
            max-attempts: 1
            # 目的地。这里使用 RocketMQ Topic
            destination: topic-common-01
            content-type: application/json
            ## 消费者分组, 命名规则：group+topic名+xx
            group: group-common-1
  
        # Spring Cloud Stream RocketMQ 配置项
        rocketmq:
          binder:
            name-server: 192.168.123.22:9876
  
          bindings:
            input-common-1:
              # RocketMQ Consumer 配置项
              consumer:
                # 顺序接收消息
                orderly: true
  ```



## 消息过滤

RocketMQ 提供了两种方式给 Consumer 进行消息的过滤：

- 基于 Tag 过滤

  > **标签（Tag）**：为消息设置的标志，用于同一主题下区分不同类型的消息。来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签。标签能够有效地保持代码的清晰度和连贯性，并优化 RocketMQ 提供的查询系统。消费者可以根据 Tag 实现对不同子主题的不同消费逻辑，实现更好的扩展性

- 基于 [SQL92](https%3A%2F%2Frocketmq.apache.org%2Frocketmq%2Ffilter-messages-by-sql92-in-rocketmq%2F) 过滤，示例：https://www.jianshu.com/p/5b13868f4451



Tag 过滤比较常见，需要设置如下：

- 生产者

  发送带 tag 的消息，如：

  ```java
  public <T> boolean sendObject(T msg, String tag) throws Exception
  {
      Message<T> message = MessageBuilder.withPayload(msg)
              .setHeader(MessageConst.PROPERTY_TAGS, tag)
              .setHeader(MessageHeaders.CONTENT_TYPE, MimeTypeUtils.APPLICATION_JSON)
              .build();
  
      return source.sendCommon().send(message);
  }
  ```

- 消费者

  订阅 tag 过滤的消息，如：

  ```yml
  server:
    # 随机端口，方便启动多个消费者
    port: ${random.int[10000,19999]}
  
  spring:
    application:
      name: rocketmq-consume
  
    cloud:
      # Spring Cloud Stream 配置项，对应 BindingServiceProperties 类
      stream:
        # Binding 配置项，对应 BindingProperties Map
        bindings:
          input-common-3:
            destination: topic-common-01
            content-type: application/json
            consumer:
              concurrency: 20
            group: group-common-3
  
        # Spring Cloud Stream RocketMQ 配置项
        rocketmq:
          binder:
            name-server: 192.168.123.22:9876
  
          bindings:
            input-common-3:
              consumer:
                # 基于 Tag 订阅，多个 Tag 使用 || 分隔，默认为空
                tags: tagObj
  ```



## [事务消息](https://www.jianshu.com/p/3dca1dabda45)

在分布式消息队列中，目前唯一提供**完整的**事务消息的，只有 RocketMQ 。关于这一点，还是可以鼓吹下的。

考虑一个极端的情况，在**本地数据库事务已经提交**的时时候，如果因为网络原因，又或者崩溃等等意外，导致事务消息没有被 commit ，最终导致这条事务消息丢失，分布式事务出现问题。

相比来说，RocketMQ 提供事务回查机制，如果应用超过一定时长未 commit 或 rollback 这条事务消息，RocketMQ 会主动回查应用，询问这条事务消息是 commit 还是 rollback ，从而实现事务消息的状态最终能够被 commit 或是 rollback ，达到最终事务的一致性。



## 配置选项

### RocketMQ Binder Properties

- spring.cloud.stream.rocketmq.binder.name-server

  RocketMQ NameServer 地址(老版本使用 namesrv-addr 配置项)。Default: `127.0.0.1:9876`

- spring.cloud.stream.rocketmq.binder.access-key

  阿里云账号 AccessKey。Default: null

- spring.cloud.stream.rocketmq.binder.secret-key

  阿里云账号 SecretKey。Default: null

- spring.cloud.stream.rocketmq.binder.enable-msg-trace

  是否为 Producer 和 Consumer 开启消息轨迹功能 Default: `true`

- spring.cloud.stream.rocketmq.binder.customized-trace-topic

  消息轨迹开启后存储的 topic 名称。Default: `RMQ_SYS_TRACE_TOPIC`




### RocketMQ Provider Properties

下面的这些配置是以 `spring.cloud.stream.rocketmq.bindings.<channelName>.producer.` 为前缀的 RocketMQ Producer 相关的配置。

- enable

  是否启用 Producer。默认值: `true`

- group

  Producer group name。默认值: empty

- maxMessageSize

  消息发送的最大字节数。默认值: `8249344`

- transactional

  是否发送事务消息。默认值: `false`

- sync

  是否使用同步得方式发送消息。默认值: `false`

- vipChannelEnabled

  是否在 Vip Channel 上发送消息。默认值: `true`

- sendMessageTimeout

  发送消息的超时时间(毫秒)。默认值: `3000`

- compressMessageBodyThreshold

  消息体压缩阀值(当消息体超过 4k 的时候会被压缩)。默认值: `4096`

- retryTimesWhenSendFailed

  在同步发送消息的模式下，消息发送失败的重试次数。默认值: `2`

- retryTimesWhenSendAsyncFailed

  在异步发送消息的模式下，消息发送失败的重试次数。默认值: `2`

- retryNextServer

  消息发送失败的情况下是否重试其它的 broker。默认值: `false`



### RocketMQ Consumer Properties

下面的这些配置是以 `spring.cloud.stream.rocketmq.bindings.<channelName>.consumer.` 为前缀的 RocketMQ Consumer 相关的配置。

- enable

  是否启用 Consumer。默认值: `true`.

- tags

  Consumer 基于 TAGS 订阅，多个 tag 以 `||` 分割。默认值: empty.

- sql

  Consumer 基于 SQL 订阅。默认值: empty.

- broadcasting

  Consumer 是否是广播消费模式。如果想让所有的订阅者都能接收到消息，可以使用广播模式。默认值: `false`.

- orderly

  Consumer 是否同步消费消息模式。默认值: `false`.

- delayLevelWhenNextConsume

  异步消费消息模式下消费失败重试策略：-1,不重复，直接放入死信队列0,broker 控制重试策略>0,client 控制重试策略默认值: `0`.

- suspendCurrentQueueTimeMillis

  同步消费消息模式下消费失败后再次消费的时间间隔。默认值: `1000`.




## 参考

[RocketMQ-Docker安装](https://baiyp.ren/RocketMQ-Docker%E5%AE%89%E8%A3%85.html)

[Spring Cloud Alibaba RocketMQ](https://github.com/alibaba/spring-cloud-alibaba/wiki/RocketMQ)

[RocketMQ Example](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-examples/rocketmq-example/readme-zh.md)

[RocketMQ 与 Spring Cloud Stream整合](https://www.jianshu.com/p/7f8fd90564ca)
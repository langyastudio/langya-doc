[Nacos 官方文档](https://nacos.io/zh-cn/docs/what-is-nacos.html)

![nacos_landscape.png](https://img-note.langyastudio.com/202112301703229.png?x-oss-process=style/watermark)



## Nacos 简介

Nacos 致力于帮助您**发现、配置和管理**微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理：如 [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)、[gRPC](https://grpc.io/docs/guides/concepts.html#service-definition) & [Dubbo RPC Service](https://dubbo.incubator.apache.org/)、[Spring Cloud RESTful Service](https://spring.io/understanding/REST)



**Nacos 具有如下特性:**

- 服务发现和服务健康监测

  支持基于 DNS 和基于 RPC 的服务发现，支持对服务的实时的健康检查，阻止向不健康的主机或服务实例发送请求

- 动态配置服务

  动态配置服务可以让您以中心化、外部化和动态化的方式管理所有环境的应用配置和服务配置

- 动态 DNS 服务

  动态 DNS 服务支持权重路由，让您更容易地实现中间层负载均衡、更灵活的路由策略、流量控制以及数据中心内网的简单 DNS 解析服务

- 服务及其元数据管理

  支持从微服务平台建设的视角管理数据中心的所有服务及元数据



Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建**以 “服务” 为中心**的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。

![nacos_map](https://nacos.io/img/nacosMap.jpg)



## [Nacos 重要概念](https://nacos.io/zh-cn/docs/concepts.html)

**命名空间**：用于进行**租户粒度**的配置隔离。不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。Namespace 的常用场景之一是**不同环境的配置**的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等

**配置快照：**Nacos 的客户端 SDK 会在**本地**生成配置的快照。当客户端无法连接到 Nacos Server 时，可以使用配置快照显示系统的整体容灾能力。配置快照类似于 Git 中的本地 commit，也类似于缓存，会在适当的时机更新，但是并没有缓存过期（expiration）的概念

**虚拟集群：**同一个服务下的所有服务实例组成一个默认集群, 集群可以被进一步按需求划分，划分的单位可以是虚拟集群

**健康保护阈值：**为了防止因过多实例 (Instance) 不健康导致流量全部流向健康实例 (Instance) ，继而造成流量压力把健康实例 (Instance) 压垮并形成**雪崩**效应，应将健康保护阈值定义为一个 0 到 1 之间的浮点数。当域名健康实例数 (Instance) 占总服务实例数 (Instance) 的比例小于该值时，无论实例 (Instance) 是否健康，都会将这个实例 (Instance) 返回给客户端。这样做虽然损失了一部分流量，但是保证了集群中剩余健康实例 (Instance) 能正常工作



## [Nacos 架构](https://nacos.io/zh-cn/docs/architecture.html)

![nacos_arch.jpg](https://img-note.langyastudio.com/202112312248882.jpeg?x-oss-process=style/watermark)

### 领域模型

#### 数据模型

Nacos 数据模型 Key 由三元组唯一确定, Namespace默认是空串，公共命名空间（public），分组默认是 DEFAULT_GROUP。

![nacos_data_model](https://img-note.langyastudio.com/202201051658139.jpeg?x-oss-process=style/watermark)

#### 服务领域模型

![nacos_naming_data_model](https://img-note.langyastudio.com/202201051658366.jpeg?x-oss-process=style/watermark)



#### 配置领域模型

围绕配置，主要有两个关联的实体，一个是配置变更历史，一个是服务标签（用于打标分类，方便索引），由 ID 关联。

![nacos_config_er](https://img-note.langyastudio.com/202201051658396.jpeg?x-oss-process=style/watermark)



### 构建物、部署及启动模式

![undefined](https://img-note.langyastudio.com/202201051658974.png?x-oss-process=style/watermark)

**两种交付工件**

Nacos 支持标准 Docker 镜像(TODO: 0.2版本开始支持）及 zip(tar.gz)压缩包的构建物。

**两种启动模式**

Nacos 支持将注册中心(Service Registry）与配置中心(Config Center) 在一个进程合并部署或者将2者分离部署的两种模式。

**免费的公有云服务模式**

除了您自己部署和启动 Nacos 服务之外，在云计算时代，Nacos 也支持公有云模式，在阿里云公有云的商业产品（如[ACM](https://www.aliyun.com/product/acm), [EDAS](https://www.aliyun.com/product/edas)) 中会提供 Nacos 的免费的公有云服务。我们也欢迎和支持其他的公有云提供商提供 Nacos 的公有云服务。



## Nacos 实战

源码地址：[https://github.com/langyastudio/langya-tech/tree/master/spring-cloud](https://github.com/langyastudio/langya-tech/tree/master/spring-cloud)



![echo service](https://img-note.langyastudio.com/202201051658700.png?x-oss-process=style/watermark)

### 安装

[基于 docker 安装 Nacos](https://nacos.io/zh-cn/docs/quick-start-docker.html)

运行成功后，访问 `http://localhost:8848/nacos` 可以查看 Nacos 的主页，默认账号密码都是 nacos



### 服务注册

Spring Cloud Nacos Discovery 遵循了 spring cloud common 标准，实现了 AutoServiceRegistration、ServiceRegistry、Registration 这三个接口。

在 spring cloud 应用的启动阶段，监听了 WebServerInitializedEvent 事件，当Web容器初始化完成后，即收到 WebServerInitializedEvent 事件后，会触发注册的动作，调用 ServiceRegistry 的 register 方法，将服务注册到 Nacos Server。

#### 如何接入

> 通过修改官方示例 [nacos-discovery-provider-example](https://github.com/alibaba/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/nacos-example/nacos-discovery-example/nacos-discovery-provider-example) 来演示服务注册的功能

如果使用 Spring Cloud Alibaba 的组件需要在 pom.xml 中添加如下的配置

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2021.1</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

引入 Nacos Discovery Starter

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

配置文件中配置 Nacos Server 地址

```yaml
server:
  port: 18081

spring:
  application:
    #服务名称
    name: nacos-discovery-provider

  cloud:
    #Nacos 配置
    nacos:
      username: nacos
      password: nacos
      discovery:
        #Nacos 的安装地址
        server-addr: 127.0.0.1:8848
        namespace: public
        cluster-name: DEFAULT
        group: DEFAULT_GROUP

management:
  endpoints:
    health:
      show-details: always
    web:
      exposure:
        include: '*'
```

使用 @EnableDiscoveryClient 注解开启服务注册与发现功能

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ProviderApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```



#### 服务启动

运行两个 nacos-discovery-provider，在 Nacos 页面上可以看到如下信息：

![image-20220101063625405](https://img-note.langyastudio.com/202201010636872.png?x-oss-process=style/watermark)



**查询服务**

在浏览器输入此地址 `http://127.0.0.1:8848/nacos/v1/ns/catalog/instances?serviceName=nacos-discovery-provider&clusterName=DEFAULT&pageSize=10&pageNo=1&namespaceId=`，并点击跳转，可以看到服务节点已经成功注册到 Nacos Server

![image-20220101063755401](https://img-note.langyastudio.com/202201010637838.png?x-oss-process=style/watermark)



### 服务发现

> 从 `SpringCloud 2020` 版本开始 ribbon 默认被 **移除**，替代品为 [spring-cloud-loadbalancer](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-loadbalancer)

NacosServerList 实现了 com.netflix.loadbalancer.ServerList 接口，并在 @ConditionOnMissingBean 的条件下进行自动注入，默认集成了 Ribbon。如果需要有更加自定义的可以使用 @Autowired 注入一个 NacosRegistration 实例，通过其持有的 NamingService 字段内容直接调用 Nacos API。

**Nacos Discovery Starter 默认集成了 Ribbon** ，所以对于使用了 Ribbon 做负载均衡的组件，可以直接使用 Nacos 的服务发现。



#### 工作流程

restTemplate 服务调用

![img](https://img-note.langyastudio.com/202201051650571.png?x-oss-process=style/watermark)



#### 如何接入

> 通过修改官方示例 [nacos-discovery-consumer-example](https://github.com/alibaba/spring-cloud-alibaba/tree/2020.0.0/spring-cloud-alibaba-examples/nacos-example/nacos-discovery-example/nacos-discovery-consumer-example) 来演示服务发现的功能
>
> 使用 RestTemplate 作为服务调用方的客户端

##### 引入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```



##### restTemplate 

添加 @LoadBlanced 注解，使得 RestTemplate 接入负载均衡

```java
 @Bean
 @LoadBalanced
 public RestTemplate restTemplate() {
     return new RestTemplate();
 }
```



##### 测试代码

完成以上配置后，将两者自动注入到 TestController 中

```java
 @RestController
 public class TestController { 
     @Autowired
     private RestTemplate restTemplate;    
 
     @GetMapping(value = "/echo-rest/{str}")
     public String rest(@PathVariable String str) {
         return restTemplate.getForObject("http://nacos-discovery-provider/echo/" + str, String.class);
     }   
 }
```

再配置 nacos 与端口即可



#### 服务启动

启动应用，支持 IDE 直接启动和编译打包后启动。

多次调用接口： http://127.0.0.1:18091/echo-rest/1234 可以看到浏览器显示 nacos-discovery-provider 返回的消息 "hello Nacos Discovery 1234"，证明服务发现生效

可以发现两个 nacos-discovery-provider 的控制台交替打印如下信息（负载均衡生效）：

```bash
2022-01-04 19:26:08.643  INFO 16516 --- [io-18082-exec-4] c.l.c.c.EchoController                   : 1234
```



### 更多配置

| 配置项         | key                                            | 默认值                  | 说明                                                         |
| -------------- | ---------------------------------------------- | ----------------------- | ------------------------------------------------------------ |
| 服务端地址     | spring.cloud.nacos.discovery.server-addr       |                         |                                                              |
| 服务名         | spring.cloud.nacos.discovery.service           | spring.application.name |                                                              |
| 权重           | spring.cloud.nacos.discovery.weight            | 1                       | 取值范围 1 到 100，数值越大，权重越大                        |
| 网卡名         | spring.cloud.nacos.discovery.network-interface |                         | 当IP未配置时，注册的IP为此网卡所对应的IP地址，如果此项也未配置，则默认取第一块网卡的地址 |
| 注册的IP地址   | spring.cloud.nacos.discovery.ip                |                         | 优先级最高                                                   |
| 注册的端口     | spring.cloud.nacos.discovery.port              | -1                      | 默认情况下不用配置，会自动探测                               |
| 命名空间       | spring.cloud.nacos.discovery.namespace         |                         | 常用场景之一是不同环境的注册的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。 |
| AccessKey      | spring.cloud.nacos.discovery.access-key        |                         |                                                              |
| SecretKey      | spring.cloud.nacos.discovery.secret-key        |                         |                                                              |
| Metadata       | spring.cloud.nacos.discovery.metadata          |                         | 使用Map格式配置                                              |
| 日志文件名     | spring.cloud.nacos.discovery.log-name          |                         |                                                              |
| 接入点         | spring.cloud.nacos.discovery.endpoint          | UTF-8                   | 地域的某个服务的入口域名，通过此域名可以动态地拿到服务端地址 |
| 是否集成Ribbon | ribbon.nacos.enabled                           | true                    |                                                              |



## 参考

[Nacos 官方文档](https://nacos.io/zh-cn/docs/what-is-nacos.html)

[nacos-example/nacos-discovery-example](https://github.com/alibaba/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/nacos-example/nacos-discovery-example)




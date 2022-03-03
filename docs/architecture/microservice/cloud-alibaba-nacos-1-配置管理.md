源码地址：[https://github.com/langyastudio/langya-tech/tree/master/spring-cloud](https://github.com/langyastudio/langya-tech/tree/master/spring-cloud)

![nacos_config_er](https://img-note.langyastudio.com/202201051657613.jpeg?x-oss-process=style/watermark)



## 服务配置

Nacos Config 主要通过 dataId 和 group 来唯一确定一条配置，Nacos Client 从 Nacos Server 端获取数据时，调用的是此接口 `ConfigService.getConfig(String dataId, String group, long timeoutMs)`。

`group` 默认为 `DEFAULT_GROUP`，可以通过 `spring.cloud.nacos.config.group` 配置。



### Spring Cloud 

Nacos Config Starter 实现了 `org.springframework.cloud.bootstrap.config.PropertySourceLocator` 接口，并将**优先级设置成了最高**。

在 Spring Cloud 应用启动阶段，会主动从 Nacos Server 端获取对应的数据，并将获取到的数据转换成 PropertySource 且注入到 **Environment** 的 PropertySources 属性中，所以使用 @Value 注解也能直接获取 Nacos Server 端配置的内容。

在 Nacos Config Starter 中，dataId 的拼接格式如下

```bash
${prefix} - ${spring.profiles.active} . ${file-extension}
```

比如说我们现在要获取应用名称为 `nacos-config` 的应用在 `dev` 环境下的 `yml` 配置，dataid 如下：

```bash
nacos-config-dev.yml
```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix` 来配置

- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)

  注意，当 active profile 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}`.`${file-extension}`

- `file-extension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置

  

### 如何接入

> 通过修改官方示例 [nacos-config-example](https://github.com/alibaba/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/nacos-example/nacos-config-example) 来演示服务配置管理的功能

在 pom.xml 中添加 Nacos Config Starter 依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    
    <!--spring cloud 2020 需要引入-->
    <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-bootstrap</artifactId>
    </dependency>
</dependency>
```

bootstrap.yml 文件中配置 Nacos Config 元数据。spring.cloud.nacos.config 相关的 server-addr 和 file-extension 要放到 **bootstrap.yml** 中，而不是 application.yml 中

```yml
spring:
  application:
    name: nacos-config

  cloud:
    #Nacos
    nacos:
      username: nacos
      password: nacos
      config:
        server-addr: 192.168.123.22:8848
        #namespace: public
        group: DEFAULT_GROUP
        file-extension: yml
```



完成上述两步后，应用会从 Nacos Config 中获取相应的配置，并添加在 Spring Environment 的 PropertySources 中。这里我们使用 @Value 注解来将对应的配置注入到 SampleController 的 userName 和 age 字段，并添加 **@RefreshScope 打开动态刷新功能**

```java
 @RefreshScope
 class SampleController {

 	@Value("${user.name}")
 	String userName;

 	@Value("${user.age}")
 	int age;
 }
```

Nacos 中新建配置示意图：

![image-20220104231156791](https://img-note.langyastudio.com/202201042311860.png?x-oss-process=style/watermark)



### 服务启动

启动 nacos-config，调用接口查看配置信息：http://192.168.123.134:18071/user

```bash
Hello Nacos Config!Hello langyastudio 90 [UserConfig]: UserConfig{age=90, name='langyastudio', map={location=jinan, phone=15589933912}!com.alibaba.nacos.client.config.NacosConfigService@1a064999
```



### 动态刷新配置

Nacos Config Starter 默认为所有获取数据成功的 Nacos 的配置项添加了监听功能，在监听到服务端配置发生变化时会实时触发 `org.springframework.cloud.context.refresh.ContextRefresher` 的 refresh 方法 。

如果需要对 Bean 进行动态刷新，请参照 Spring 和 Spring Cloud 规范。推荐给类添加 **`@RefreshScope`** 或 `@ConfigurationProperties ` 注解。

只要修改下 Nacos 中对应的配置信息，再次调用查看配置的接口，就会发现配置已经刷新，Nacos 和 Consul 一样都支持动态刷新配置。



### 更多配置

| 配置项                   | key                                       | 默认值        | 说明                                                         |
| ------------------------ | ----------------------------------------- | ------------- | ------------------------------------------------------------ |
| 服务端地址               | spring.cloud.nacos.config.server-addr     |               |                                                              |
| DataId前缀               | spring.cloud.nacos.config.prefix          |               | spring.application.name                                      |
| Group                    | spring.cloud.nacos.config.group           | DEFAULT_GROUP |                                                              |
| dataID后缀及内容文件格式 | spring.cloud.nacos.config.file-extension  | properties    | dataId的后缀，同时也是配置内容的文件格式，目前只支持 properties |
| 配置内容的编码方式       | spring.cloud.nacos.config.encode          | UTF-8         | 配置的编码                                                   |
| 获取配置的超时时间       | spring.cloud.nacos.config.timeout         | 3000          | 单位为 ms                                                    |
| 配置的命名空间           | spring.cloud.nacos.config.namespace       |               | 常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源隔离等。 |
| AccessKey                | spring.cloud.nacos.config.access-key      |               |                                                              |
| SecretKey                | spring.cloud.nacos.config.secret-key      |               |                                                              |
| 相对路径                 | spring.cloud.nacos.config.context-path    |               | 服务端 API 的相对路径                                        |
| 接入点                   | spring.cloud.nacos.config.endpoint        | UTF-8         | 地域的某个服务的入口域名，通过此域名可以动态地拿到服务端地址 |
| 是否开启监听和自动刷新   | spring.cloud.nacos.config.refresh-enabled | true          |                                                              |



## 参考

[nacos-config-example](https://github.com/alibaba/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/nacos-example/nacos-config-example)

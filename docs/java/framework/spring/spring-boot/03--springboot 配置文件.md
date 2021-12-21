> 先看 [spring boot 2.x restful web应用](https://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484615&idx=1&sn=db2294154cd0a9b249a0124ad1d0ffc8&chksm=c03a5666f74ddf70c7c8700514068c34d053502a56d87d7b5a0bc3911392ef24f644c9d65754&scene=21#wechat_redirect)
>
> 基于上述代码修改

- Spring Boot 允许在一个配置文件中针对不同 Profile 进行配置

- Spring Boot 在未指定 Profile 时默认为 `default`

- Spring Boot 提供了 `@ConfigurationProperties` 注解，可以非常方便地把一段配置加载到一个 Bean 中

- 官方文档：https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config

  

> 源码地址：https://github.com/langyastudio/langya-tech/tree/springboot/properties

## 加载自动配置

Spring Boot 的入口类是一个简单的包含可执行 main() 方法的 Java 类。SpringApplication 的作用是新建一个 Spring IoC容器。

- 在非Web环境中，它可以新建一个 AnnotationConfigApplicationContext
- 在Web环境中，它可以新建一个 AnnotationConfigServletWebServerApplicationContext
- 在响应式Web环境中，它可以新建一个 AnnotationConfigReactiveWebServerApplicationContext

由此看来，SpringApplication并没有什么特别的神奇之处。



再来看看 @SpringBootApplication 的定义:

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
}
```

- 组合获得 @SpringBootConfiguration 注解的功能，而这个注解又组合了 @Configuration 元注解，这意味着入口类是一个配置类
- @EnableAutoConfiguration 开启自动配置
- 注解 @ComponentScan 获得自动扫描 Bean 的功能



接着看看 @EnableAutoConfiguration 的定义:

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {

}
```

**@Import 注解导入一个自动配置选择器** —— AutoConfigurationImportSelector。下面聚焦AutoConfigurationImportSelector 所实现的接口 ImportSelector 的覆写方法 selectImports，整体的调用流程如下:

- getAutoConfigurationEntry(autoConfigurationMetadata, annotationMetadata)

  获得应导入的自动配置

- getCandidateConfigurations(annotationMetadata,attributes)

  获得自动配置类的名称

- SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),getBeanClassLoader())

  从类路径下获取所有 META-INF/spring.factories（如spring-boot-autoconfigure-2.2.x.RELEASE.jar/META-INF/spring.factories）文件，并查找类型为 getSpringFactoriesLoaderFactoryClass() 的工厂，即 org.springframework.boot.autoconfigure.EnableAutoConfiguration，找到符合自动加载的配置

> 上述主要阐述加载自动配置类，如配置文件中定义的各种组件 aop、amqp、cache、mongo、jdbc等



### 修改默认配置

**自动扫描配置**

在默认情况下，Spring Boot 会自动扫描入口类所在包及其下级包中所有的 Bean。而 @SpringBootApplication 注解组合了 @ComponentScan 元注解，所以对自动扫描的配置是通过 @SpringBootApplication 来完成的。

```java
@SpringBootApplication(scanBasePackages = {"com.langyastudio.springboot"})
```



**关闭自动配置**

```java
@SpringBootApplication(exclude = SpringApplicationAdminJmxAutoConfiguration.class)
```



**覆盖Bean**

我们可以用重新定义的 Bean 来覆盖已经自动配置的 Bean。如 Spring Boot 提供自动配置的 ObjectMapper 的 Bean。



**使用Customizer来定制**

Spring Boot 提供了大量的 Customizer 来修改或定制默认行为，我们可以继承或实现这些 Customizer。



## 应用配置

### SpringApplication

可以通过 SpringApplication 类对应用启动行为进行配置。SpringApplication 支持很多和容器相关的配置，它们都是以set 和 add 开头的方法：

```java
@SpringBootApplication
public class Application
{
    public static void main(String[] args)
    {
        SpringApplication app = new SpringApplication(Application.class);

        app.setBannerMode(Banner.Mode.OFF);
        app.addListeners(new MyListener());

        app.run(args);
    }
}
```

> 也可以通过 SpringApplicationBuilder 来定制应用启动。它是一个建造者模式的类，和 Stream 运算很像，设置为中间运算，用一个终结运算来执行。
>
> SpringApplication 配置与 SpringApplicationBuilder 配置是等同的，前者的方法名去掉前缀（set和add）即为后者的方法名。



### 外部配置

可以通过外部配置（可以是命令行、系统环境变量或 **application.properties**）来定制应用启动行为。例如比较常用的通过 application.properties 配置文件即可定制应用的启动行为

在 Spring Boot下，**外部配置属性的加载顺序**：先列的属性配置优先级最高，先列的配置属性可覆盖后列的配置属性。

- 命令行参数
- SPRING_APPLICATION_JSON
- ServletConfig 初始化参数
- ServletContext 初始化参数
- JNDI（java:comp/env）
- Java 系统属性（System.getProperties()）
- 操作系统变量
- RandomValuePropertySource随机值
- 应用部署 jar 包外部的 application-{profile}.properties/yml
- 应用部署 jar 包内部的 application-{profile}.properties/yml
- 应用部署 jar 包外部的 application.properties/yml
- 应用部署 jar 包内部的 application.properties/yml
- @PropertySource
- SpringApplication.setDefaultProperties

> Spring Boot 提供了对 Environment 和 SpringApplication 进行定制的 EnvironmentPostProcessor



### 其他默认配置

Spring Boot 除做了大量的自动配置外，还提供了一些其他默认配置。

如应用监听器：在类路径下文件 META-INF/spring.factories 中的工厂名为org.springframework.context.ApplicationListener 的所有监听器

![image-20210812110511848](https://img-note.langyastudio.com/20210812110512.png?x-oss-process=style/watermark)



## 读取配置文件

### 配置样例

这里使用 `application.yml` yml 类型的配置文件，而不是 `application.properties` ini 类型的配置文件。

```yml
langyastudio:
  disk:
    local:
      root: /mnt/volume/edu/pms/
      #只有在windows运行时才生效
      #use for test on windows OS
      win-root: Z:/volume/edu/pms/
      #文件存储路径
      root-file: ${langyastudio.disk.local.root}file/
      #转码路径自动映射为mts/
      root-mts: ${langyastudio.disk.local.root}mts/
      #文件存储临时路径
      root-tmp: ${langyastudio.disk.local.root}temp/upload/
      #-图片-可直接浏览的文件大小10M
      browse-img-max-size: 10485760
      #-图片-可转码的文件大小20M
      cvt-img-max-size: 20971520
      # 最大文件大小，默认10M
      max-size: 10485760
      # 是否允许空文件:
      allow-empty: false
      # 允许的文件类型:
      allow-types: jpg, png, gif
```



### @Value

读取配置文件可以直接使用注解 `@Value`，例如在某个类里需要获取 `root` 配置项，可使用 `@Value` 注入：

```java
@RestController
@RequestMapping("/config")
public class ConfigController
{
    @Value("${langyastudio.disk.local.root}")
    String root;
    
    @Value("${langyastudio.disk.local.root:/mnt/volume/edu/pms/}")
    String root;
	
    ...   
}
```

> 这里演示的是在 control 层注入配置项，在其他类文件中注入配置项需要使用 @Component
>
> 可以使用 `${langyastudio.disk.local.root:/mnt/volume/edu/pms/}` 设置缺省值



### @ConfigurationProperties

**定义 Bean**

为了更好地管理配置，Spring Boot 允许创建一个 Bean 直接注入一组配置项。可以首先定义一个  Java Bean，需要确保 Java Bean 的属性名称与配置一致即可：

```java
@Data
@ConfigurationProperties(prefix = "langyastudio.disk.local")
public class DiskLocalConfig
{
    private String       root;
    private String       winRoot;
    private String       rootFile;
    private String       rootMts;
    private String       rootTmp;
    private Integer      browseImgMaxSize;
    private Integer      cvtImgMaxSize;
    private Integer      maxSize;
    private Boolean      allowEmpty;
    private List<String> allowTypes;
}
```

注意到 `@ConfigurationProperties(prefix = "langyastudio.disk.local")`  表示将从配置项 `langyastudio.disk.local` 读取该项的所有子项配置并一一映射到属性字段中。



**定义 ConfigurationPropertiesScan**

表示对那些类文件进行属性扫描组装为 Bean。`com.langyastudio.springboot.common.*`表示类所在的位置。

```java
@ConfigurationPropertiesScan("com.langyastudio.springboot.common.*")
@Configuration
public class WebConfig implements WebMvcConfigurer
{

}
```

> 也可以使用 `@Configuration` 替代



**使用配置项**

```java
@Autowired
DiskLocalConfig diskLocalConfig;

---- 实际数据
{
    "root": "/mnt/volume/edu/pms/",
    "winRoot": "Z:/volume/edu/pms/",
    "rootFile": "/mnt/volume/edu/pms/file/",
    "rootMts": "/mnt/volume/edu/pms/mts/",
    "rootTmp": "/mnt/volume/edu/pms/temp/upload/",
    "browseImgMaxSize": 10485760,
    "cvtImgMaxSize": 20971520,
    "maxSize": 10485760,
    "allowEmpty": false,
    "allowTypes": [
        "jpg",
        "png",
        "gif"
    ]
}
```

这样就很方面的使用配置项，不需要使用一堆 `@Value`



## profiles 配置环境

即解决如何在研发、测试、上线等不同环境中使用不同的配置？通过 Profile 可以实现一套代码在不同环境启用不同的配置和功能。

我们可以三个配置文件：

- application.yml
- application-dev.yml
- application-pro.yml

`application-dev.yml` 或 `application-pro.yml` 配置文件会自动覆盖 `application.yml` 文件的已有配置项。

在 java 程序运行时，可以通过  `--spring.profiles.active=dev` 指定使用哪个配置文件，如要以 `dev` 环境启动，可输入如下命令：

```shell
$ java -Dspring.profiles.active=dev -jar xxx.jar
```



当然如果只使用一个配置文件，可以实现多文档分区，即同样的多配置功能。在一个 yml 文件中，通过 `---` 分隔多个不同配置，根据 spring.profiles.active 的值来决定启用哪个配置。

此时可以通过 @Profile 指定不同的配置项组实现同样的效果。这里不做详细介绍

```yml
#公共配置
spring:
  profiles:
    # 指定使用哪个文档块
    active: pro
---
spring:
  profiles: dev

server:
  port: 8080
---
spring:
  profiles: pro

server:
  port: 8081
```


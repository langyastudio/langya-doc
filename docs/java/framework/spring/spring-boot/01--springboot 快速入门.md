![img](https://img-note.langyastudio.com/20210819155010.png?x-oss-process=style/watermark)



Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can “just run”...Most Spring Boot applications need very little Spring configuration.

> `Spring Boot` 和 `Spring` 的关系就是整车和零部件的关系，它们不是取代关系。



## spring boot 简介

Spring 框架一直是 Java EE 开发的王者，但是由于其有大量的配置，因而导致学习曲线较为陡峭。Spring 在 2014 年推出了 Spring Boot，Spring Boot 提供了如下功能来简化 Spring 的开发：

- AutoConfiguration 自动配置

  Spring Boot 为绝大多数的常用开发组件提供了自动配置，通过**自动扫描+条件装配**实现的。如 JDBC、JPA、Kafka、Elasticsearch、Spring MVC、Spring Security、Spring Integration 及 Spring Batch

- starter 项目

  Spring Boot 是一个基于 Spring 的套件，它帮我们预组装了 Spring 的一系列组件，以便以尽可能少的代码和配置来开发基于 Spring 的 Java 应用程序

- 依赖版本管理

  Spring Boot 提供了**全局依赖版本支持**，只需声明 Spring Boot 的版本号即可，无须对 Spring Boot 支持的组件技术声明版本信息，依赖组件会直接得到最佳的依赖版本

- 独立运行

  Spring Boot 支持将整个应用打包成 jar 包形式，jar 包中内嵌了 Servlet 容器（Tomcat、Jetty等），可独立运行

- 开发者工具

  Spring Boot 提供了开发者工具，只要添加 spring-boot-devtools 依赖，就可以在开发过程中提供自动重启功能，减少编译等待时间

- Spring Boot Actuator

  为生产时对应用的监控提供了支持



## 新建 spring boot 项目

### 环境要求

JDK 1.8+

[Gradle 4+](http://www.gradle.org/downloads) or [Maven 3.2+](https://maven.apache.org/download.cgi)

 

### 新建项目

初始化 spring boot 项目，有多种方式，例如：

#### 官方

https://start.spring.io/

![image-20210727114407490](https://img-note.langyastudio.com/20210727114407.png?x-oss-process=style/watermark)

#### 阿里

https://start.aliyun.com/bootstrap.html 

提供功能更丰富的在线配置项

![image-20210727114920401](https://img-note.langyastudio.com/20210727114920.png?x-oss-process=style/watermark)



#### [IDEA 编辑器（采用该方案）](https://www.jetbrains.com/idea/)

选择 `File -> New -> Project`菜单，创建项目

- name 

  项目名称

- location 

  代码存储位置

- type 

  项目构建工具，Maven or Gradle

- java

  版本，例如 `8`、`11`、`16`，这里采用的 `11`
  
- group

  组织域名

- artifact

  应用名称

![](https://img-note.langyastudio.com/20210727123212.png?x-oss-process=style/watermark)



点击下一步，选择：

- spring boot 版本号为 `2.5.3`
- 依赖组件 `Spring Web`

再点击完成即可。

![image-20210727124733946](https://img-note.langyastudio.com/20210727124734.png?x-oss-process=style/watermark)



### 项目结构

此时新建了一个 `springboot` 的初始化项目，标准的 `Maven` 目录结构如下：

```java
springboot
├── pom.xml
├── src
│   └── main
│       ├── java
│       └── com.langyastudio.springboot
│       └── ├── Application.java
│       └── resources
│           ├── application.yml
│           ├── static
│           └── templates
└── target
```

注意到几个文件：

**application.yml**

Spring Boot 默认的**配置文件**，文件名必须是  `application.yml` 而不是其他名称。YAML格式比 `key=value` 格式的 `.properties` 文件更易读



**static 文件夹**

**静态资源**目录，如 js、css、img 等



**template 文件夹**

html **模板**目录



**Application.java** 

**应用程序入口**，Spring Boot 要求 `main()` 方法所在的启动类必须放到根 package 下，命名不做要求

```java
/**
 * 应用程序入口
 */
@SpringBootApplication
public class Application
{
    public static void main(String[] args)
    {
        SpringApplication.run(Application.class, args);
    }
}
```

启动 Spring Boot 应用程序只需要一行代码加上一个注解 `@SpringBootApplication`，该注解实际上包含了：

- @SpringBootConfiguration
  - @Configuration
- @EnableAutoConfiguration
  - @AutoConfigurationPackage
- @ComponentScan

这样一个注解就相当于启动了 `自动配置`和`自动扫描`。



**pom.xml**

maven 配置文件

- parent

  继承的父组件

- propeties

  属性描述，如 java 版本号。涉及属性都可以统一定义在这里

- dependencies

  依赖库。引入`spring-boot-starter-web`时，其实自动创建了（AutoConfiguration ）：

  - `ServletWebServerFactoryAutoConfiguration`：自动创建一个嵌入式Web服务器，默认是Tomcat
  - `DispatcherServletAutoConfiguration`：自动创建一个`DispatcherServlet`
  - `HttpEncodingAutoConfiguration`：自动创建一个`CharacterEncodingFilter`
  - `WebMvcAutoConfiguration`：自动创建若干与MVC相关的Bean
  - ...

![image-20210727140927844](https://img-note.langyastudio.com/20210727140927.png?x-oss-process=style/watermark)



### 运行项目

点击 IDE 工具栏上的 run or debug，即可运行 spring boot 服务

![image-20210727142950814](https://img-note.langyastudio.com/20210727142950.png?x-oss-process=style/watermark)



Console 控制台显示结果如下：

![image-20210727142913397](https://img-note.langyastudio.com/20210727142913.png?x-oss-process=style/watermark)



> 参考文档：廖雪峰、《从企业级开发到云原生微服务：Spring Boot实战 》等

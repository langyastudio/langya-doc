源码地址：[https://github.com/langyastudio/langya-tech/tree/master/spring-cloud](https://github.com/langyastudio/langya-tech/tree/master/spring-cloud)

## [Spring Boot Admin](https://github.com/codecentric/spring-boot-admin) 简介

SpringBoot 应用可以通过 **Actuator** 来暴露应用运行过程中的各项指标，Spring Boot Admin 通过这些指标来监控SpringBoot 应用，然后通过图形化界面呈现出来。Spring Boot Admin 不仅可以监控单体应用，还可以和 Spring Cloud的注册中心相结合来**监控微服务**应用。

Spring Boot Admin 可以提供应用的以下监控信息：

- 监控应用运行过程中的概览信息
- 度量指标信息，比如 JVM、内存、Tomcat 及进程信息
- 环境变量信息，比如系统属性、系统环境变量以及应用配置信息
- 查看所有创建的 Bean 信息
- 查看应用中的所有配置信息
- 查看应用运行日志信息
- 查看 JVM 信息
- 查看可以访问的 Web 端点
- 查看 HTTP 跟踪信息
- 查看审计事件
- 查看计划任务
- 支持 Spring Cloud 的 postable /env- &/refresh-endpoint



## 基本使用

### 添加 admin-server 模块

> 这里我们创建一个 admin-server 模块来作为**监控中心**演示其功能

在 pom.xml 中添加相关依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
</dependency>
```

在 application.yml 中进行配置

```yaml
server:
  port: 19001

spring:
  application:
    name: admin-server

management:
  endpoints:
    health:
      show-details: always
    web:
      exposure:
        include: '*'
```

在启动类上添加 @EnableAdminServer 来启用 admin-server 功能

```java
@EnableAdminServer
@SpringBootApplication
public class AdminServerApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(AdminServerApplication.class, args);
    }
}
```



### 添加 admin-client 模块

> 这里我们创建一个 admin-client 模块作为**客户端**注册到 admin-server

在pom.xml中添加相关依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
</dependency>
```

在application.yml中进行配置

```yaml
server:
  port: 19002


spring:
  application:
    name: admin-client
  boot:
    admin:
      client:
        #配置admin-server地址
        url: http://localhost:19001


management:
  endpoints:
    web:
      exposure:
        include: '*'
  endpoint:
    health:
      show-details: always
```

启动 admin-server 和 admin-client 服务



### 监控信息展示

访问如下地址打开 Spring Boot Admin 的主页：[http://localhost:19001](http://localhost:19001/)

![image-20220106152827324](https://img-note.langyastudio.com/202201061528401.png?x-oss-process=style/watermark)



点击 wallboard 按钮，选择 admin-client 查看监控信息

监控信息概览

![image-20220106153053898](https://img-note.langyastudio.com/202201061530041.png?x-oss-process=style/watermark)



度量指标信息，比如 JVM、Tomcat 及进程信息；

![image-20220106153159021](https://img-note.langyastudio.com/202201061531099.png?x-oss-process=style/watermark)



环境变量信息，比如系统属性、系统环境变量以及应用配置信息；

![image-20220106153320256](https://img-note.langyastudio.com/202201061533353.png?x-oss-process=style/watermark)



查看应用中的所有配置信息；

![image-20220106153545210](https://img-note.langyastudio.com/202201061535314.png?x-oss-process=style/watermark)



查看日志信息，需要添加以下配置才能开启；

```yaml
logging:
  file:
    name: admin-client.log 
```

![image-20220106153910159](https://img-note.langyastudio.com/202201061539248.png?x-oss-process=style/watermark)



查看可以访问的 Web 端点

![image-20220106154022123](https://img-note.langyastudio.com/202201061540221.png?x-oss-process=style/watermark)



## 结合注册中心使用

Spring Boot Admin 结合 Spring Cloud Alibaba Nacos 注册中心使用，只需将 admin-server 和注册中心整合即可，admin-server 会**自动从注册中心获取服务**列表，然后挨个获取监控信息。这里以 **Nacos** 注册中心为例来介绍下该功能

### admin-server 修改 

在 pom.xml 中添加相关依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

在 application.yml 中进行配置，只需添加注册中心配置即可

```yaml
server:
  port: 19001

spring:
  application:
    name: admin-server

  cloud:
    #Nacos
    nacos:
      username: nacos
      password: nacos
      discovery:
        server-addr: localhost:8848       
```

在启动类上添加 @EnableDiscoveryClient 来启用服务注册功能

```java
@EnableDiscoveryClient
@EnableAdminServer
@SpringBootApplication
public class AdminServerApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(AdminServerApplication.class, args);
    }
}
```



### admin-client 修改 

在 pom.xml 中添加相关依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

在 application.yml 中进行配置，删除原来的 admin-server 地址配置，添加注册中心配置即可

```yaml
server:
  port: 19002

spring:
  application:
    name: admin-client
  #Nacos
  nacos:
    username: nacos
    password: nacos
    discovery:
      server-addr: localhost:8848

management:
  endpoints:
    web:
      exposure:
        include: '*'
  endpoint:
    health:
      show-details: always


#添加开启admin的日志监控
logging:
  file:
    name: admin-client.log
```

在启动类上添加 @EnableDiscoveryClient 来启用服务注册功能

```java
@EnableDiscoveryClient
@SpringBootApplication
public class AdminClientApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(AdminClientApplication.class, args);
    }
}
```



### 功能演示

- 启动 nacos-server
- 启动 admin-server、admin-client

查看注册中心发现服务均已注册：http://localhost:8848/

![image-20220106154930427](https://img-note.langyastudio.com/202201061549539.png?x-oss-process=style/watermark)

查看 Spring Boot Admin 主页发现可以看到服务信息：[http://localhost:19001](http://localhost:19001/)



## [登录认证](https://github.com/codecentric/spring-boot-admin/tree/master/spring-boot-admin-samples/spring-boot-admin-sample-servlet)

> 我们可以通过给 admin-server 添加 Spring Security 支持来获得登录认证功能

### 添加 admin-server-security 模块

在 pom.xml 中添加相关依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

在 application.yml 中进行配置，配置登录用户名和密码，忽略 admin-server-security 的监控信息

```yaml
server:
  port: 19011

spring:
  application:
    name: admin-security-server

  cloud:
    #Nacos
    nacos:
      username: nacos
      password: nacos
      discovery:
        server-addr: localhost:8848

  # 配置登录用户名和密码
  security:
    user:
      name: admin
      password: 123456
```

对 SpringSecurity 进行配置，以便 admin-client 可以注册：

```java
@Configuration
public class SecuritySecureConfig extends WebSecurityConfigurerAdapter
{
    private final String adminContextPath;

    public SecuritySecureConfig(AdminServerProperties adminServerProperties)
    {
        this.adminContextPath = adminServerProperties.getContextPath();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception
    {
        SavedRequestAwareAuthenticationSuccessHandler successHandler =
                new SavedRequestAwareAuthenticationSuccessHandler();
        successHandler.setTargetUrlParameter("redirectTo");
        successHandler.setDefaultTargetUrl(adminContextPath + "/");

        http.authorizeRequests()
                //1.授予对所有静态资产和登录页面的公共访问权限
                .antMatchers(adminContextPath + "/assets/**").permitAll()
                .antMatchers(adminContextPath + "/actuator/info").permitAll()
                .antMatchers(adminContextPath + "/actuator/health").permitAll()
                .antMatchers(adminContextPath + "/login").permitAll()

                //2.其他请求必须经过身份验证
                .anyRequest().authenticated()
                .and()

                //2.登录
                .formLogin().loginPage(adminContextPath + "/login").successHandler(successHandler)
                .and()

                //3.登出
                .logout().logoutUrl(adminContextPath + "/logout")
                .and()

                //4.开启http basic支持
                .httpBasic()
                .and()

                //5.开启基于cookie的csrf保护
                .csrf()
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())

                //6.忽略这些路径的csrf保护以便admin-client注册
                .ignoringRequestMatchers(
                        new AntPathRequestMatcher(adminContextPath + "/instances",
                                                  HttpMethod.POST.toString()),
                        new AntPathRequestMatcher(adminContextPath + "/instances/*",
                                                  HttpMethod.DELETE.toString()),
                        new AntPathRequestMatcher(adminContextPath + "/actuator/**")
                )
                .and()

                .rememberMe(rememberMe -> rememberMe.key(UUID.randomUUID().toString()).tokenValiditySeconds(1209600));
    }
}
```

修改启动类，开启 AdminServer 及注册发现功能：

```java
@EnableDiscoveryClient
@EnableAdminServer
@SpringBootApplication
public class AdminServerSecurityApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(AdminServerSecurityApplication.class, args);
    }
}
```

启动 nacos-server，admin-server-security，访问 Spring Boot Admin 主页发现需要登录才能访问：[http://localhost:19011](http://localhost:19011/)

![image-20220106161240838](https://img-note.langyastudio.com/202201061612969.png?x-oss-process=style/watermark)



## [定制化](https://codecentric.github.io/spring-boot-admin/2.5.1/#customizing)

可以自定义通知、自定义Logo、标题、语言、增加跳转链接、移除页头菜单等。

例如定制登录标题、不显示 about 菜单、不显示 admin-server 的监控信息：

```yml
spring:
  application:
    name: admin-security-server

  # 配置admin
  boot:
    admin:
      ui:
        #页头左侧
        brand: <img src="assets/img/icon-spring-boot-admin.svg"><span>服务监控</span>
        #登录的标题
        title: 服务监控
        #隐藏 about 菜单
        view-settings:
          - name: about
            enabled: false
      discovery:
        #不显示admin-server的监控信息
        ignored-services: ${spring.application.name}
```

![image-20220106172925570](https://img-note.langyastudio.com/202201061729658.png?x-oss-process=style/watermark)



## 参考

[spring-boot-admin 官方文档](https://codecentric.github.io/spring-boot-admin/2.5.1/#getting-started)

[http://www.macrozheng.com/#/cloud/admin](http://www.macrozheng.com/#/cloud/admin)


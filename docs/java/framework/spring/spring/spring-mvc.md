> 本文来自廖雪峰，郎涯进行简单排版与补充



## Spring MVC

Java Web 的基础：

- Servlet：能处理HTTP请求并将HTTP响应返回
- JSP：一种嵌套Java代码的HTML，将被编译为Servlet
- Filter：能过滤指定的URL以实现拦截功能
- Listener：监听指定的事件，如ServletContext、HttpSession的创建和销毁



### Spring MVC 的工作流程

使用 Spring MVC 时，整个 **Web 应用程序按如下顺序启动：**

1. 启动 Tomcat 服务器
2. Tomcat 读取 web.xml 并初始化 DispatcherServlet
3. DispatcherServlet 创建 IoC 容器并自动注册到 ServletContext 中

启动后，浏览器发出的 HTTP 请求全部由 DispatcherServlet 接收，并根据配置转发到指定 Controller 的指定方法处理

> 在 Spring Boot 中，通过 DispatcherServletAutoConfiguration 来定义 DispatcherServlet 的 Bean，并通过DispatcherServletRegistrationBean 的 Bean 来注册 DispatcherServlet 到 Servlet 容器中



DispatcherServlet 在处理（doDispatch 方法）时，将主要的功能都代理给了下面的两个 Bean:

**HandlerMapping：RequestMappingHandlerMapping**

- 它主要负责获取 Web 请求与 Handler（控制器方法）之间的映射

  当前使用的实现为RequestMappingHandlerMapping，它获取 RequestMappingInfo 作为请求和 Handler 方法之间的映射，RequestMappingInfo 信息是从 @RequestMapping 注解中获取的

- 负责 PathMatchConfigurer（路径匹配）的设置

- 负责 Interceptor（拦截器）的设置

- 负责 CORS（跨域资源共享）的设置

> 使用 PathMatchConfigurer.setUseSuffixPatternMatch(Boolean suffixPatternMatch) 可以设置是否使用后缀匹配。若设置为 true，则路径 /xx 和 /xx.* 是等效的；在 Spring Boot下，默认是 false



**HandlerAdapter：RequestMappingHandlerAdapter**

- 它主要负责从映射中获取 Handler 并调用

  当前使用的实现为 RequestMappingHandlerAdapter，它通过 HandlerMapping 的 getHandler 方法从RequestMappingInfo 中获取并执行 HandlerMethod，HandlerMethod 即我们实际要执行的控制器方法

- 负责 HttpMessageConverter 的设置

- 负责 WebBindingInitializer 的设置

- 负责 HandlerMethodArgumentResolver（控制器方法的参数）的设置

- 负责 HandlerMethodReturnValueHandler（控制器方法返回值）的设置



DispatcherServlet 支持多个 HandlerMapping，从 HandlerMapping 的 getHandler 方法可以获得不同类型的Handler。HandlerAdapter 的 supports 方法声明只处理符合它支持的 Handler 类型；RequestMappingHandlerAdapter 只支持类型为 HandlerMethod 的 Handler。

- @RequestParam：RequestParamMethodArgumentResolver
- @PathVariable：PathVariableMethodArgumentResolver
- @RequestPart：RequestPartMethodArgumentResolver
- @RequestHeader：RequestHeaderMethodArgumentResolver
- ServletRequest：ServletRequestMethodArgumentResolver
- ServletResponse：ServletResponseMethodArgumentResolver



### Spring MVC 配置

> 自 Spring 5.0 开始，Spring 全面支持 Java 8，因而 WebMvcConfigurerAdapter 的实现内容已经被WebMvcConfigurer 的默认方法替代，**WebMvcConfigurerAdapter 将被废弃**

可以通过实现 WebMvcConfigurer 接口来定制 HandlerMapping 和 HandlerAdapter



首先创建基于 Web 的 Maven 工程，引入如下依赖：

```ini
- org.springframework:spring-context:5.2.0.RELEASE
- org.springframework:spring-webmvc:5.2.0.RELEASE
- org.springframework:spring-jdbc:5.2.0.RELEASE
- javax.annotation:javax.annotation-api:1.3.2
- io.pebbletemplates:pebble-spring5:3.1.2
- ch.qos.logback:logback-core:1.2.3
- ch.qos.logback:logback-classic:1.2.3

#以及`provided`依赖：
- org.apache.tomcat.embed:tomcat-embed-core:9.0.26
- org.apache.tomcat.embed:tomcat-embed-jasper:9.0.2
```



这个标准的 Maven Web 工程目录结构如下：

```
spring-web-mvc
├── pom.xml
└── src
    └── main
        ├── java
        │   └── com
        │       └── itranswarp
        │           └── learnjava
        │               ├── AppConfig.java
        │               ├── entity
        │               │   └── User.java
        │               ├── service
        │               │   └── UserService.java
        │               └── web
        │                   └── UserController.java
        ├── resources
        │   ├── jdbc.properties
        │   └── logback.xml
        └── webapp
            ├── WEB-INF
            │   ├── templates
            │   │   ├── _base.html
            │   │   ├── index.html
            │   │   ├── profile.html
            │   │   ├── register.html
            │   │   └── signin.html
            │   └── web.xml
            └── static
                ├── css
                │   └── bootstrap.css
                └── js
                    └── jquery.js
```



新增了一个 `logback.xml` 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<appender name="STDOUT"
		class="ch.qos.logback.core.ConsoleAppender">
		<layout class="ch.qos.logback.classic.PatternLayout">
			<Pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</Pattern>
		</layout>
	</appender>

	<logger name="com.itranswarp.learnjava" level="info" additivity="false">
		<appender-ref ref="STDOUT" />
	</logger>

	<root level="info">
		<appender-ref ref="STDOUT" />
	</root>
</configuration>
```



只需加上 **`@EnableWebMvc`** 注解，就“激活”了 Spring MVC：

```java
@Configuration
@ComponentScan
@EnableWebMvc // 启用Spring MVC
@EnableTransactionManagement
@PropertySource("classpath:/jdbc.properties")
public class AppConfig {
    ...
}
```

 

除了创建 `DataSource`、`JdbcTemplate`、`PlatformTransactionManager` 外，`AppConfig` 需要额外创建几个用于Spring MVC 的 Bean：

```java
@Bean
WebMvcConfigurer createWebMvcConfigurer() {
    return new WebMvcConfigurer() {
        @Override 
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            registry.addResourceHandler("/static/**").addResourceLocations("/static/");
        }
    };
}
```

`WebMvcConfigurer` 并不是必须的，但我们在这里创建一个默认的 `WebMvcConfigurer`，只覆写`addResourceHandlers()`，目的是让 Spring MVC 自动处理静态文件，并且映射路径为 `/static/**`。



另一个必须要创建的 Bean 是 `ViewResolver`，因为 Spring MVC 允许集成任何模板引擎，使用哪个模板引擎，就实例化一个对应的 `ViewResolver`：

```java
@Bean
ViewResolver createViewResolver(@Autowired ServletContext servletContext) {
    PebbleEngine engine = new PebbleEngine.Builder().autoEscaping(true)
            .cacheActive(false)
            .loader(new ServletLoader(servletContext))
            .extension(new SpringExtension())
            .build();
    
    PebbleViewResolver viewResolver = new PebbleViewResolver();
    viewResolver.setPrefix("/WEB-INF/templates/");
    viewResolver.setSuffix("");
    viewResolver.setPebbleEngine(engine);
    return viewResolver;
}
```

`ViewResolver `通过指定 prefix 和 suffix 来确定如何查找 View。上述配置使用 Pebble 引擎，指定模板文件存放在`/WEB-INF/templates/`目录下。



剩下的 Bean 都是普通的 `@Component`，但 Controller 必须标记为 `@Controller`，例如：

```java
@Controller
public class UserController {
    // 正常使用@Autowired注入:
    @Autowired
    UserService userService;

    // 处理一个URL映射:
    @GetMapping("/")
    public ModelAndView index() {
        ...
    }
    ...
}
```



现在是 Web 应用程序，而 Web 应用程序总是由 Servlet 容器创建，那么 Spring 容器应该由谁创建？在什么时候创建？Spring 容器中的 Controller 又是如何通过 Servlet 调用的？

在 Web 应用中启动 Spring 容器有很多种方法，可以通过 Listener 启动，也可以通过 Servlet 启动，可以使用 XML 配置，也可以使用注解配置。这里，我们只介绍一种最简单的启动 Spring 容器的方式。

第一步，我们在`web.xml` 中配置 Spring MVC 提供的 `DispatcherServlet`：

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
        </init-param>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.itranswarp.learnjava.AppConfig</param-value>
        </init-param>
        <load-on-startup>0</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
</web-app>
```

初始化参数 `contextClass` 指定使用注解配置的 `AnnotationConfigWebApplicationContext`，配置文件的位置参数`contextConfigLocation` 指向`AppConfig`的完整类名，最后把这个Servlet映射到 `/*`，即处理所有URL。

上述配置可以看作一个样板配置，有了这个配置，Servlet 容器会首先初始化 Spring MVC 的 `DispatcherServlet`，在`DispatcherServlet `启动时，它根据配置 `AppConfig` 创建了一个类型是 WebApplicationContext 的 IoC 容器，完成所有 Bean 的初始化，并将容器绑到 ServletContext 上。

因为 `DispatcherServlet` 持有 IoC 容器，能从 IoC 容器中获取所有 `@Controller` 的 Bean，因此，`DispatcherServlet` 接收到所有 HTTP 请求后，根据 Controller 方法配置的路径，就可以正确地把请求转发到指定方法，并根据返回的 `ModelAndView` 决定如何渲染页面。

> 最后在 `AppConfig` 中通过 `main()` 方法启动嵌入式 Tomcat



### Controller 编写

有了 Web 应用程序的最基本的结构，我们的重点就可以放在如何编写 Controller 上。Spring MVC 对 Controller 没有固定的要求，也不需要实现特定的接口。以 UserController 为例，编写 Controller 只需要遵循以下要点：

总是标记 `@Controller` 而不是 `@Component`：

```java
@Controller
public class UserController {
    ...
}
```



一个方法对应一个 HTTP 请求路径，用 `@GetMapping` 或 `@PostMapping` 表示 GET 或 POST 请求：

```java
@PostMapping("/signin")
public ModelAndView doSignin(
        @RequestParam("email") String email,
        @RequestParam("password") String password,
        HttpSession session) {
    ...
}
```

需要接收的HTTP参数以 `@RequestParam()` 标注，可以设置默认值。如果方法参数需要传入 `HttpServletRequest`、`HttpServletResponse `或者 `HttpSession`，直接添加这个类型的参数即可，Spring MVC 会自动按类型传入。



返回的 ModelAndView 通常包含 View 的路径和一个 Map 作为 Model，但也可以没有 Model，例如：

```java
return new ModelAndView("signin.html"); // 仅View，没有Model
```



返回重定向时既可以写 `new ModelAndView("redirect:/signin")`，也可以直接返回String：

```java
public String index() {
    if (...) {
        return "redirect:/signin";
    } else {
        return "redirect:/profile";
    }
}
```



如果在方法内部直接操作 `HttpServletResponse` 发送响应，**返回 `null` 表示无需进一步处理**：

```java
public ModelAndView download(HttpServletResponse response) {
    byte[] data = ...
    response.setContentType("application/octet-stream");
    OutputStream output = response.getOutputStream();
    output.write(data);
    output.flush();
    
    return null;
}
```



对 URL 进行分组，每组对应一个 Controller 是一种很好的组织形式，并可以在 Controller 的 class 定义出添加URL前缀，例如：

```java
@Controller
@RequestMapping("/user")
public class UserController {
    // 注意实际URL映射是/user/profile
    @GetMapping("/profile")
    public ModelAndView profile() {
        ...
    }

    // 注意实际URL映射是/user/changePassword
    @GetMapping("/changePassword")
    public ModelAndView changePassword() {
        ...
    }
}
```

实际方法的 URL 映射总是 **前缀+路径**，这种形式还可以有效避免不小心导致的重复的URL映射



### Servlet 容器配置

我们可以使用 Servlet 注解 @WebServlet、@WebFilter 和 @WebListener 来注册 Servlet、Filter 和 Listener 的Bean，即通过 @ServletComponentScan 将它们注册成Bean。



**外部属性配置**

- 网络配置：server.port、server.address等

- 用户会话配置：server.servlet.session.

- 错误配置：server.error.*

- HTTP压缩：server.compression.

  支持HTML、XML、CSS、JS、JSON和text。默认是关闭的，可用server.compression.enabled:true开启。

- SSL配置：server.ssl.*

- Tomcat专有配置：server.tomcat.*

- Jetty专有配置：server.jetty.*

- Undertow专有配置：server.undertow.*

- Servlet相关配置：server.servlet.*

> Spring Boot提供了 WebServerFactoryCustomizer 来定制Servlet容器



**ConfigurableServletWebServerFactory**

在默认情况下，Spring Boot 通过 ServletWebServerApplicationContext 自动寻找配置 ServletWebServerFactory 的Bean；Tomcat 的为 TomcatServletWebServerFactory，Jetty 的为 ServletWebServerFactory，Undertow 的为UndertowServletWebServerFactory。

我们可以通过定义 ConfigurableServletWebServerFactory 的 Bean 来定制Servlet容器



## REST

- 使用 `@RestController` 可以方便地编写 REST 服务，Spring 默认使用 JSON 作为输入和输出

- 要控制序列化和反序列化，可以使用 Jackson 提供的 `@JsonIgnore` 和 `@JsonProperty` 注解  

- ReuqestBody 主要是处理json串格式的请求参数，要求使用方指定header `content-type:application/json`

- RequestBody 通常要求调用方使用post请求



直接在 Controller 中处理 JSON 是可以的，因为 Spring MVC 的 `@GetMapping` 和 `@PostMapping` 都支持指定输入和输出的格式。如果我们想接收 JSON，输出 JSON，那么可以这样写：

```java
@PostMapping(value = "/rest",
             consumes = "application/json;charset=UTF-8",
             produces = "application/json;charset=UTF-8")
@ResponseBody
public String rest(@RequestBody User user) {
    return "{\"restSupport\":true}";
}
```

对应的 Maven 工程需要加入 Jackson 这个依赖：`com.fasterxml.jackson.core:jackson-databind:2.11.0`



**`@ResponseBody` 表示返回的 `String` 无需额外处理，直接作为输出内容写入 `HttpServletResponse`。输入的 JSON 则根据注解 `@RequestBody` 直接被 Spring 反序列化为 `User` 这个 JavaBean**



使用curl命令测试一下：

```shell
$ curl -v -H "Content-Type: application/json" -d '{"email":"bob@example.com"}' http://localhost:8080/rest      
> POST /rest HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.64.1
> Accept: */*
> Content-Type: application/json
> Content-Length: 27
> 
< HTTP/1.1 200 
< Content-Type: application/json;charset=utf-8
< Content-Length: 20
< Date: Sun, 10 May 2020 09:56:01 GMT
< 
{"restSupport":true}
```

输出正是我们写入的字符串。



直接用 Spring 的 Controller 配合一大堆注解写 REST 太麻烦了，因此，Spring 还额外提供了一个 `@RestController`注解，使用 `@RestController` 替代 `@Controller` 后，**每个方法自动变成 API 接口方法**。我们还是以实际代码举例，编写`ApiController`如下：

```java
@RestController
@RequestMapping("/api")
public class ApiController {
    @Autowired
    UserService userService;

    @GetMapping("/users")
    public List<User> users() {
        return userService.getUsers();
    }

    @GetMapping("/users/{id}")
    public User user(@PathVariable("id") long id) {
        return userService.getUserById(id);
    }

    @PostMapping("/signin")
    public Map<String, Object> signin(@RequestBody SignInRequest signinRequest) {
        try {
            User user = userService.signin(signinRequest.email, signinRequest.password);
            return Map.of("user", user);
        } catch (Exception e) {
            return Map.of("error", "SIGNIN_FAILED", "message", e.getMessage());
        }
    }

    public static class SignInRequest {
        public String email;
        public String password;
    }
}
```

编写 REST 接口只需要定义 `@RestController`，然后，每个方法都是一个 API 接口，输入和输出只要能被 Jackson 序列化或反序列化为 JSON 就没有问题。我们用浏览器测试 GET 请求，可直接显示 JSON 响应：

![user-api](https://img-note.langyastudio.com/20210402084029.png?x-oss-process=style/watermark)



要避免输出 `password` 属性，可以直接在 `User` 的 `password` 属性定义处加上 `@JsonIgnore` 表示完全忽略该属性：

```java
public class User {
    ...

    @JsonIgnore
    public String getPassword() {
        return password;
    }

    ...
}
```

但是这样一来，如果写一个 `register(User user)` 方法，那么该方法的 User 对象也拿不到注册时用户传入的密码了。如果要允许输入 `password`，但不允许输出 `password`，即在 JSON 序列化和反序列化时，允许写属性，禁用读属性，可以更精细地控制如下：

```java
public class User {
    ...

    @JsonProperty(access = Access.WRITE_ONLY)
    public String getPassword() {
        return password;
    }

    ...
}
```

> 可以使用 `@JsonProperty(access = Access.READ_ONLY)` 允许输出，不允许输入



## CORS

- CORS 可以控制指定域的页面 JavaScript 能否访问API



在 JavaScript 与 REST 交互的时候，有很多安全限制。默认情况下，浏览器按同源策略放行 JavaScript 调用 API，即：

- 如果 A 站在域名 `a.com` 页面的 JavaScript 调用 A 站自己的 API 时，没有问题
- 如果 A 站在域名 `a.com` 页面的 JavaScript 调用 B 站 `b.com` 的 API 时，将被浏览器拒绝访问，因为不满足同源策略

> 同源要求域名要完全相同（`a.com `和 `www.a.com` 不同），协议要相同（`http` 和 `https` 不同），端口要相同



那么，在域名 `a.com` 页面的 JavaScript 要调用 B 站 `b.com` 的API时，还有没有办法？

办法是有的，那就是 CORS，全称 Cross-Origin Resource Sharing，是 HTML5 规范定义的如何跨域访问资源。如果 A 站的 JavaScript 访问 B 站 API 的时候，B 站能够返回响应头 `Access-Control-Allow-Origin: http://a.com`，那么，浏览器就允许 A 站的 JavaScript 访问 B 站的 API。

注意到跨域访问能否成功，取决于 B 站是否愿意给 A 站返回一个正确的 `Access-Control-Allow-Origin` 响应头，所以决定权永远在提供 API 的服务方手中。

关于 CORS 的详细信息可以参考[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)，这里不再详述。



有好几种方法设置CORS，我们来一一介绍。

### @CrossOrigin 使用

第一种方法是使用 `@CrossOrigin` 注解，可以在 `@RestController` 的 class 级别或方法级别定义一个`@CrossOrigin`，例如：

```java
@CrossOrigin(origins = "http://local.liaoxuefeng.com:8080")
@RestController
@RequestMapping("/api")
public class ApiController {
    ...
}
```

上述定义在 `ApiController` 处的 `@CrossOrigin` 指定了只允许来自 `local.liaoxuefeng.com` 跨域访问，允许多个域访问需要写成数组形式，例如 `origins = {"http://a.com", "https://www.b.com"})` 。如果要允许任何域访问，写成 `origins = "*"` 即可。



### CorsRegistry 使用

第二种方法是在 `WebMvcConfigurer` 中定义一个全局 CORS 配置，下面是一个示例：

```java
@Bean
WebMvcConfigurer createWebMvcConfigurer() {
    return new WebMvcConfigurer() {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/api/**")
                    .allowedOrigins("http://local.liaoxuefeng.com:8080")
                    .allowedMethods("GET", "POST")
                    .maxAge(3600);
            
            // 可以继续添加其他URL规则:
            // registry.addMapping("/rest/v2/**")...
        }
    };
}
```

这种方式可以创建一个全局 CORS 配置，如果仔细地设计 URL 结构，那么可以一目了然地看到各个 URL 的 CORS 规则，**推荐使用这种方式配置 CORS。**



### CorsFilter 使用

第三种方法是使用 Spring 提供的 `CorsFilter`，我们在[集成Filter](https://www.liaoxuefeng.com/wiki/1252599548343744/1282384114745378/)中详细介绍了将 Spring 容器内置的 Bean 暴露为Servlet 容器的 Filter 的方法，由于这种配置方式需要修改 `web.xml`，也比较繁琐，所以推荐使用第二种方式。



## Filter

- 当一个 Filter 作为 Spring 容器管理的 Bean 存在时，可以通过 `DelegatingFilterProxy` 间接地引用它并使其生效。

> 在 Spring MVC 中，`DispatcherServlet` 只需要固定配置到 `web.xml` 中，剩下的工作主要是专注于编写 Controller



有的童鞋在上一节的 Web 应用中可能发现了，如果注册时输入中文会导致乱码，因为 Servlet 默认按非 UTF-8 编码读取参数。为了修复这一问题，我们可以简单地使用一个 EncodingFilter，在全局范围类给 `HttpServletRequest` 和`HttpServletResponse ` 强制设置为UTF-8编码。

可以自己编写一个 EncodingFilter，也可以直接使用 Spring MVC 自带的一个 `CharacterEncodingFilter`。配置Filter时，只需在`web.xml`中声明即可：

```xml
<web-app>
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

因为这种 Filter 和我们业务关系不大，注意到 `CharacterEncodingFilter` 其实和 Spring 的 IoC 容器没有任何关系，两者均互不知晓对方的存在，因此，配置这种 Filter 十分简单。



我们再考虑这样一个问题：如果允许用户使用 Basic 模式进行用户验证，即在 HTTP 请求中添加头`Authorization: Basic email:password`，这个需求如何实现？

编写一个 `AuthFilter` 是最简单的实现方式：

```java
@Component
public class AuthFilter implements Filter {
    @Autowired
    UserService userService;

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        
        // 获取Authorization头:
        String authHeader = req.getHeader("Authorization");
        if (authHeader != null && authHeader.startsWith("Basic ")) {
            // 从Header中提取email和password:
            String email = prefixFrom(authHeader);
            String password = suffixFrom(authHeader);
           
            // 登录:
            User user = userService.signin(email, password);
            
            // 放入Session:
            req.getSession().setAttribute(UserController.KEY_USER, user);
        }
        
        // 继续处理请求:
        chain.doFilter(request, response);
    }
}
```



现在问题来了：在Spring中创建的这个 `AuthFilter` 是一个普通 Bean，**Servlet 容器并不知道**，所以它不会起作用。

如果我们直接在 `web.xml` 中声明这个 `AuthFilter`，注意到 `AuthFilter` 的实例将由 Servlet 容器而不是 Spring 容器初始化，因此 `@Autowire` 根本不生效，用于登录的 `UserService` 成员变量永远是 `null`。

所以，得通过一种方式，让 Servlet 容器实例化的 Filter，**间接引用 Spring 容器实例化的 `AuthFilter`**。Spring MVC 提供了一个  `DelegatingFilterProxy`，专门来干这个事情：

```xml
<web-app>
    <filter>
        <filter-name>authFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>authFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

我们来看实现原理：

1. Servlet 容器从 `web.xml` 中读取配置，实例化  `DelegatingFilterProxy`，注意命名是 `authFilter`
2. Spring 容器通过扫描  `@Component` 实例化 `AuthFilter`

当 `DelegatingFilterProxy` 生效后，它会自动查找注册在 `ServletContext`上的 Spring 容器，再试图从容器中查找名为 `authFilter` 的 Bean，也就是我们用 `@Component` 声明的 `AuthFilter`



`DelegatingFilterProxy `将请求代理给 `AuthFilter`，核心代码如下：

```java
public class DelegatingFilterProxy implements Filter {
    private Filter delegate;
    public void doFilter(...) throws ... {
        if (delegate == null) {
            delegate = findBeanFromSpringContainer();
        }
        delegate.doFilter(req, resp, chain);
    }
}
```

这就是一个 [代理模式 ](https://www.liaoxuefeng.com/wiki/1252599548343744/1281319432618017)的简单应用。我们画个图表示它们之间的引用关系如下：

```ascii
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐ ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
  ┌─────────────────────┐        ┌───────────┐   │
│ │DelegatingFilterProxy│─│─│─ ─>│AuthFilter │
  └─────────────────────┘        └───────────┘   │
│ ┌─────────────────────┐ │ │    ┌───────────┐
  │  DispatcherServlet  │─ ─ ─ ─>│Controllers│   │
│ └─────────────────────┘ │ │    └───────────┘
                                                 │
│    Servlet Container    │ │  Spring Container
 ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```



如果在 `web.xml `中配置的 Filter 名字和 Spring 容器的 Bean 的名字不一致，那么需要指定Bean的名字：

```xml
<filter>
    <filter-name>basicAuthFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <!-- 指定Bean的名字 -->
    <init-param>
        <param-name>targetBeanName</param-name>
        <param-value>authFilter</param-value>
    </init-param>
</filter>
```

实际应用时，尽量保持名字一致，以减少不必要的配置



## Async

- 在 Spring MVC 中异步处理请求需要正确配置 `web.xml`，并返回 `Callable` 或 `DeferredResult` 对象



在 Servlet 模型中，每个请求都是由某个线程处理，然后，将响应写入IO流，发送给客户端。从开始处理请求，到写入响应完成，都是在同一个线程中处理的。

这种线程模型非常重要，因为 Spring 的 JDBC 事务是基于 `ThreadLocal` 实现的，如果在处理过程中，一会由线程 A 处理，一会又由线程 B 处理，那事务就全乱套了。此外，很多安全认证，也是基于 `ThreadLocal` 实现的，可以保证在处理请求的过程中，各个线程互不影响。

但是，如果一个请求处理的时间较长，例如几秒钟甚至更长，那么，这种基于线程池的同步模型很快就会把所有**线程耗尽**，导致服务器无法响应新的请求。如果把长时间处理的请求改为异步处理，那么线程池的利用率就会大大提高。



**Servlet 从 3.0 规范开始添加了异步支持，允许对一个请求进行异步处理**。



我们先来看看在 Spring MVC 中如何实现对请求进行异步处理的逻辑。首先建立一个 Web 工程，然后编辑 `web.xml `文件如下：

```xml
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
    version="3.1">
    <display-name>Archetype Created Web Application</display-name>

    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
        </init-param>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.itranswarp.learnjava.AppConfig</param-value>
        </init-param>
        <load-on-startup>0</load-on-startup>
            
        <async-supported>true</async-supported>
            
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
</web-app>
```

和前面普通的 MVC 程序相比，这个 `web.xml` 主要有几点不同：

- 不能再使用 `<!DOCTYPE ...web-app_2_3.dtd">` 的 DTD 声明，**必须用新的支持 Servlet 3.1 规范的 XSD 声明**，照抄即可
- 对 `DispatcherServlet ` 的配置多了一个 `<async-supported>`，默认值是 `false`，必须明确写成 `true`，这样Servlet 容器才会支持 async 处理



### async 实现

下一步就是在 Controller 中编写 async 处理逻辑

- 第一种 async 处理方式是返回一个 `Callable`

Spring MVC 自动把返回的 `Callable` 放入线程池执行，等待结果返回后再写入响应：

```java
@GetMapping("/users")
public Callable<List<User>> users() {
    return () -> {
        // 模拟3秒耗时:
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
        }
        return userService.getUsers();
    };
}
```



- **第二种** async 处理方式是返回一个 `DeferredResult` 对象

    然后在另一个线程中，设置此对象的值并写入响应：

```java
@GetMapping("/users/{id}")
public DeferredResult<User> user(@PathVariable("id") long id) {
    DeferredResult<User> result = new DeferredResult<>(3000L); // 3秒超时
   
    new Thread(() -> {
        // 等待1秒:
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
        }
        try {
            User user = userService.getUserById(id);
            // 设置正常结果并由Spring MVC写入Response:
            result.setResult(user);
        } catch (Exception e) {
            // 设置错误结果并由Spring MVC写入Response:
            result.setErrorResult(Map.of("error", e.getClass().getSimpleName(), "message", e.getMessage()));
        }
    }).start();
    
    return result;
}
```

使用 `DeferredResult` 时，可以设置超时，超时会自动返回超时错误响应。在另一个线程中，可以调用 `setResult()` 写入结果，也可以调用 `setErrorResult()` 写入一个错误结果。

> DeferredResult 和 Callable 只能异步返回单个值；如果想要异步返回多个值，则可以用 HTTPStreaming 来实现
>
> HTTP Streaming 是一种推送形式的数据传输技术，它通过无限期开放的 HTTP 连接让 Web 服务器（Tomcat）能持续向客户端（浏览器）传送数据，如 ResponseBodyEmitter、SSE、StreamingResponseBody



### Filter 使用

我们用两个Filter：SyncFilter 和 AsyncFilter 分别测试：**async-supported 需要声明为true**

```xml
<web-app ...>
    ...
    <filter>
        <filter-name>sync-filter</filter-name>
        <filter-class>com.itranswarp.learnjava.web.SyncFilter</filter-class>
    </filter>

    <filter>
        <filter-name>async-filter</filter-name>
        <filter-class>com.itranswarp.learnjava.web.AsyncFilter</filter-class>
        <async-supported>true</async-supported>
    </filter>

    <filter-mapping>
        <filter-name>sync-filter</filter-name>
        <url-pattern>/api/version</url-pattern>
    </filter-mapping>

    <filter-mapping>
        <filter-name>async-filter</filter-name>
        <url-pattern>/api/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

一个声明为支持 `<async-supported>` 的 Filter 既可以过滤 async 处理请求，也可以过滤正常的同步处理请求，而未声明`<async-supported>` 的 Filter 无法支持 async 请求，如果一个普通的 Filter 遇到 async 请求时，会直接报错，因此，务必注意普通 Filter 的 `<url-pattern>` 不要匹配 async 请求路径。



在 `logback.xml `配置文件中，我们把输出格式加上 `[%thread]`，可以输出当前线程的名称：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</Pattern>
        </layout>
    </appender>
    ...
</configuration>
```

> 在实际使用时，经常用到的就是 `DeferredResult`，因为返回 `DeferredResult` 时，可以设置超时、正常结果和错误结果，易于编写比较灵活的逻辑。



**使用 async 异步处理响应时，要时刻牢记，在另一个异步线程中的事务和 Controller 方法中执行的事务不是同一个事务，在 Controller 中绑定的 `ThreadLocal` 信息也无法在异步线程中获取。**



此外，Servlet 3.0 规范添加的异步支持是针对同步模型打了一个“补丁”，虽然可以异步处理请求，但高并发异步请求时，它的处理效率并不高，因为这种异步模型并没有用到真正的“原生”异步。

Java 标准库提供了封装操作系统的异步 IO 包 `java.nio`，是真正的多路复用 IO 模型，可以用少量线程支持大量并发。使用 NIO 编程复杂度比同步 IO 高很多，因此我们很少直接使用 NIO。

> 大部分需要高性能异步 IO 的应用程序会选择 [Netty](https://netty.io/) 这样的框架，它基于 NIO 提供了更易于使用的 API，方便开发异步应用程序



## HttpMessageConverter

在 Spring MVC 中，请求（@RequestBody、RequestEntity 等）和返回（@Responsebody、ResponseEntity 等）都是通过 HttpMessageConverter 来实现数据转换的。



Spring MVC 自动注册了下列 HttpMessageConverter：

- ByteArrayHttpMessageConverter：二进制数组转换

- StringHttpMessageConverter：字符串转换，默认支持所有的媒体类型

- ResourceHttpMessageConverter：org.springframework.core.io.Resource类型转换

- SourceHttpMessageConverter：javax.xml.transform.Source类型转换

- 各种 JSON 库的 HttpMessageConverter



## i18n

- 多语言支持需要从 HTTP 请求中解析用户的 Locale，然后针对不同 Locale 显示不同的语言

- Spring MVC 应用程序通过 `MessageSource` 和 `LocaleResolver` ，配合 View 实现国际化 

在开发应用程序的时候，经常会遇到支持多语言的需求，这种支持多语言的功能称之为国际化，英文是**internationalization，缩写为 i18n**（因为首字母i和末字母n中间有18个字母）。

还有针对特定地区的本地化功能，英文是 localization，缩写为 **L10n**，本地化是指根据地区调整类似姓名、日期的显示等。

也有把上面两者合称为全球化，英文是 globalization，缩写为 **g11n**。



### 获取 Locale

实现国际化的第一步是获取到用户的 `Locale`。在 Web 应用程序中，HTTP 规范规定了浏览器会在请求中携带 `Accept-Language` 头，用来指示用户浏览器设定的语言顺序，如：

```ini
Accept-Language: zh-CN,zh;q=0.8,en;q=0.2
```

上述 HTTP 请求头表示优先选择简体中文，其次选择中文，最后选择英文。`q`表示权重，解析后我们可获得一个根据优先级排序的语言列表，把它转换为 Java 的 `Locale`，即获得了用户的 `Locale`。大多数框架通常只返回权重最高的`Locale`。



Spring MVC 通过 `LocaleResolver` 来自动从 `HttpServletRequest`中获取 `Locale`。有多种 `LocaleResolver` 的实现类，其中最常用的是 `CookieLocaleResolver`：

```java
@Bean
LocaleResolver createLocaleResolver() {
    var clr = new CookieLocaleResolver();
    clr.setDefaultLocale(Locale.ENGLISH);
    clr.setDefaultTimeZone(TimeZone.getDefault());
    return clr;
}
```

`CookieLocaleResolver ` 从 `HttpServletRequest` 中获取 `Locale` 时，首先根据一个特定的 Cookie 判断是否指定了`Locale`，如果没有，就从 HTTP 头获取，如果还没有，就返回默认的 `Locale`。

当用户第一次访问网站时，`CookieLocaleResolver` 只能从 HTTP 头获取 `Locale`，即使用浏览器的默认语言。通常网站也允许用户自己选择语言，此时，`CookieLocaleResolver` 就会把用户选择的语言存放到 Cookie 中，下一次访问时，就会返回用户上次选择的语言而不是浏览器默认语言。



### 提取资源文件

第二步是把写死在模板中的字符串以资源文件的方式存储在外部。对于多语言，主文件名如果命名为 `messages`，那么资源文件必须按如下方式命名并放入 classpath 中：

- 默认语言，文件名必须为`messages.properties`
- 简体中文，Locale是`zh_CN`，文件名必须为`messages_zh_CN.properties`
- 日文，Locale是`ja_JP`，文件名必须为`messages_ja_JP.properties`
- 其它更多语言……

每个资源文件都有相同的key，例如，默认语言是英文，文件 `messages.properties` 内容如下：

```ini
language.select=Language
home=Home
signin=Sign In
copyright=Copyright©{0,number,#}
```

文件 `messages_zh_CN.properties` 内容如下：

```ini
language.select=语言
home=首页
signin=登录
copyright=版权所有©{0,number,#}
```



### 创建MessageSource

第三步是创建一个 Spring 提供的 `MessageSource` 实例，它自动读取所有的 `.properties` 文件，并提供一个统一接口来实现“翻译”：

```java
// code, arguments, locale:
String text = messageSource.getMessage("signin", null, locale);
```

其中，`signin` 是我们在 `.properties` 文件中定义的 key，第二个参数是 `Object[]` 数组作为格式化时传入的参数，最后一个参数就是获取的用户 `Locale` 实例。



创建 `MessageSource` 如下：

```java
@Bean("i18n")
MessageSource createMessageSource() {
    var messageSource = new ResourceBundleMessageSource();
    // 指定文件是UTF-8编码:
    messageSource.setDefaultEncoding("UTF-8");
    // 指定主文件名:
    messageSource.setBasename("messages");
    return messageSource;
}
```

注意到 `ResourceBundleMessageSource` 会自动根据主文件名自动把所有相关语言的资源文件都读进来。

再注意到 Spring 容器会创建不只一个 `MessageSource` 实例，我们自己创建的这个 `MessageSource` 是专门给页面国际化使用的，因此命名为 `i18n`，不会与其它 `MessageSource` 实例冲突。



### 实现多语言

要在 View 中使用 `MessageSource` 加上 `Locale` 输出多语言，我们通过编写一个 `MvcInterceptor`，把相关资源注入到`ModelAndView `中：

```java
@Component
public class MvcInterceptor implements HandlerInterceptor {
    @Autowired
    LocaleResolver localeResolver;

    // 注意注入的MessageSource名称是i18n:
    @Autowired
    @Qualifier("i18n")
    MessageSource messageSource;

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        if (modelAndView != null) {
            // 解析用户的Locale:
            Locale locale = localeResolver.resolveLocale(request);
           
            // 放入Model:
            modelAndView.addObject("__messageSource__", messageSource);
            modelAndView.addObject("__locale__", locale);
        }
    }
}
```

不要忘了在 `WebMvcConfigurer `中注册 `MvcInterceptor`。现在，就可以在View中调用`MessageSource.getMessage()` 方法来实现多语言：

```xml
<a href="/signin">{{ __messageSource__.getMessage('signin', null, __locale__) }}</a>
```



上述这种写法虽然可行，但格式太复杂了。使用 View 时，要根据每个特定的 View 引擎定制国际化函数。在 Pebble 中，我们可以封装一个国际化函数，名称就是下划线 `_`，改造一下创建 `ViewResolver` 的代码：

```java
@Bean
ViewResolver createViewResolver(@Autowired ServletContext servletContext, @Autowired @Qualifier("i18n") MessageSource messageSource) {
    PebbleEngine engine = new PebbleEngine.Builder()
            .autoEscaping(true)
            .cacheActive(false)
            .loader(new ServletLoader(servletContext))
            // 添加扩展:
            .extension(createExtension(messageSource))
            .build();
    PebbleViewResolver viewResolver = new PebbleViewResolver();
    viewResolver.setPrefix("/WEB-INF/templates/");
    viewResolver.setSuffix("");
    viewResolver.setPebbleEngine(engine);
    return viewResolver;
}

private Extension createExtension(MessageSource messageSource) {
    return new AbstractExtension() {
        @Override
        public Map<String, Function> getFunctions() {
            return Map.of("_", new Function() {
                public Object execute(Map<String, Object> args, PebbleTemplate self, EvaluationContext context, int lineNumber) {
                    String key = (String) args.get("0");
                    List<Object> arguments = this.extractArguments(args);
                    Locale locale = (Locale) context.getVariable("__locale__");
                    return messageSource.getMessage(key, arguments.toArray(), "???" + key + "???", locale);
                }
                private List<Object> extractArguments(Map<String, Object> args) {
                    int i = 1;
                    List<Object> arguments = new ArrayList<>();
                    while (args.containsKey(String.valueOf(i))) {
                        Object param = args.get(String.valueOf(i));
                        arguments.add(param);
                        i++;
                    }
                    return arguments;
                }
                public List<String> getArgumentNames() {
                    return null;
                }
            });
        }
    };
}
```

这样，我们可以把多语言页面改写为：

```xml
<a href="/signin">{{ _('signin') }}</a>
```

如果是带参数的多语言，需要把参数传进去：

```xml
<h5>{{ _('copyright', 2020) }}</h5>
```

使用其它 View 引擎时，也应当根据引擎接口实现更方便的语法。



### Locale切换

最后，我们需要允许用户手动切换 `Locale`，编写一个 `LocaleController` 来实现该功能：

```java
@Controller
public class LocaleController {
    final Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    LocaleResolver localeResolver;

    @GetMapping("/locale/{lo}")
    public String setLocale(@PathVariable("lo") String lo, HttpServletRequest request, HttpServletResponse response) {
        // 根据传入的lo创建Locale实例:
        Locale locale = null;
        int pos = lo.indexOf('_');
        if (pos > 0) {
            String lang = lo.substring(0, pos);
            String country = lo.substring(pos + 1);
            locale = new Locale(lang, country);
        } else {
            locale = new Locale(lo);
        }
        
        // 设定此Locale:
        localeResolver.setLocale(request, response, locale);
        logger.info("locale is set to {}.", locale);
        
        // 刷新页面:
        String referer = request.getHeader("Referer");
        return "redirect:" + (referer == null ? "/" : referer);
    }
}
```



## Interceptor

> Servlet 提供了 filter（过滤器）来预处理和后处理每一个 Web 请求，Spring MVC 提供了 Interceptor（拦截器）来预处理和后处理每一个 Web 请求，它的优势是能使用 IoC 容器的一些功能，使用时要注意 Interceptor 的作用范围



在 Web 程序中，注意到使用 Filter 的时候，Filter由 Servlet 容器管理，它在 Spring MVC 的Web应用程序中作用范围如下：

```ascii
         │   ▲
         ▼   │
       ┌───────┐
       │Filter1│
       └───────┘
         │   ▲
         ▼   │
       ┌───────┐
┌ ─ ─ ─│Filter2│─ ─ ─ ─ ─ ─ ─ ─ ┐
       └───────┘
│        │   ▲                  │
         ▼   │
│ ┌─────────────────┐           │
  │DispatcherServlet│<───┐
│ └─────────────────┘    │      │
   │              ┌────────────┐
│  │              │ModelAndView││
   │              └────────────┘
│  │                     ▲      │
   │    ┌───────────┐    │
│  ├───>│Controller1│────┤      │
   │    └───────────┘    │
│  │                     │      │
   │    ┌───────────┐    │
│  └───>│Controller2│────┘      │
        └───────────┘
└ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

上图虚线框就是 Filter2 的拦截范围，Filter 组件实际上并不知道后续内部处理是通过 Spring MVC 提供的`DispatcherServlet` 还是其他 Servlet 组件，因为 **Filter 是 Servlet 规范定义的标准组件**，它可以应用在任何基于 Servlet 的程序中。



如果只基于 Spring MVC 开发应用程序，还可以使用 Spring MVC 提供的一种功能类似 Filter 的拦截器：Interceptor。和 Filter 相比，**Interceptor 拦截范围不是后续整个处理流程，而是仅针对 Controller 拦截**：

```ascii
       │   ▲
       ▼   │
     ┌───────┐
     │Filter1│
     └───────┘
       │   ▲
       ▼   │
     ┌───────┐
     │Filter2│
     └───────┘
       │   ▲
       ▼   │
┌─────────────────┐
│DispatcherServlet│<───┐
└─────────────────┘    │
 │              ┌────────────┐
 │              │ModelAndView│
 │              └────────────┘
 │ ┌ ─ ─ ─ ─ ─ ─ ─ ─ ┐ ▲
 │    ┌───────────┐    │
 ├─┼─>│Controller1│──┼─┤
 │    └───────────┘    │
 │ │                 │ │
 │    ┌───────────┐    │
 └─┼─>│Controller2│──┼─┘
      └───────────┘
   └ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

返回 `ModelAndView` 后，后续对 View 的渲染就脱离了 Interceptor 的拦截范围。



使用 Interceptor 的好处是 Interceptor 本身是 Spring 管理的 Bean，因此注入任意 Bean 都非常简单。此外，可以应用多个 Interceptor，并通过简单的 `@Order` 指定顺序。我们先写一个 `LoggerInterceptor`：

```java
@Order(1)
@Component
public class LoggerInterceptor implements HandlerInterceptor {

    final Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        logger.info("preHandle {}...", request.getRequestURI());
        if (request.getParameter("debug") != null) {
            PrintWriter pw = response.getWriter();
            pw.write("<p>DEBUG MODE</p>");
            pw.flush();
            return false;
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        logger.info("postHandle {}.", request.getRequestURI());
        if (modelAndView != null) {
            modelAndView.addObject("__time__", LocalDateTime.now());
        }
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        logger.info("afterCompletion {}: exception = {}", request.getRequestURI(), ex);
    }
}
```



一个 Interceptor 必须实现 `HandlerInterceptor` 接口，可以选择实现`preHandle()`、`postHandle()`和`afterCompletion()`方法。

- `preHandle() `是 Controller 方法调用前执行
- `postHandle() `是 Controller 方法**正常返回后执行**
- `afterCompletion()` 无论Controller方法是否抛异常都会执行，参数`ex`就是Controller方法抛出的异常（未抛出异常是`null`）

在 `preHandle() `中，也可以直接处理响应，然后返回 `false` 表示无需调用 Controller 方法继续处理了，通常在认证或者安全检查失败时直接返回错误响应。

在 `postHandle()` 中，因为捕获了 Controller 方法返回的 `ModelAndView`，所以可以继续往 `ModelAndView` 里添加一些通用数据，很多页面需要的全局数据如 Copyright 信息等都可以放到这里，无需在每个 Controller 方法中重复添加。



最后，要让拦截器生效，我们在 `WebMvcConfigurer` 中注册所有的 Interceptor：

```java
@Bean
WebMvcConfigurer createWebMvcConfigurer(@Autowired HandlerInterceptor[] interceptors) {
    return new WebMvcConfigurer() {
        public void addInterceptors(InterceptorRegistry registry) {
            for (var interceptor : interceptors) {
                registry.addInterceptor(interceptor);
            }
        }
        ...
    };
}
```

 **如果拦截器没有生效，请检查是否忘了在 WebMvcConfigurer 中注册**



### 处理异常

在 Controller 中，Spring MVC 还允许定义基于 `@ExceptionHandler` 注解的异常处理方法。我们来看具体的示例代码：

```java
@Controller
public class UserController {
    @ExceptionHandler(RuntimeException.class)
    public ModelAndView handleUnknowException(Exception ex) {
        return new ModelAndView("500.html", Map.of("error", ex.getClass().getSimpleName(), "message", ex.getMessage()));
    }
    ...
}
```

异常处理方法没有固定的方法签名，可以传入`Exception`、`HttpServletRequest`等，返回值可以是 `void`，也可以是`ModelAndView`，上述代码通过 `@ExceptionHandler(RuntimeException.class)`表示当发生 `RuntimeException` 的时候，就自动调用此方法处理。



注意到我们返回了一个新的 `ModelAndView`，这样在应用程序内部如果发生了预料之外的异常，可以给用户显示一个出错页面，而不是简单的500 Internal Server Error或404 Not Found。例如B站的错误页：

![500](https://www.liaoxuefeng.com/files/attachments/1347185673240641/l)

可以编写多个错误处理方法，每个方法针对特定的异常。例如，处理`LoginException`使得页面可以自动跳转到登录页



## Formatter

Spring 为我们提供了 ConversionService 接口来做**类型转换**，它是 Spring 类型转换系统的入口。例如，注册的Formatter 的 FormattingConversionService 类就是它的实现类。

FormattingConversionService 支持注册 Formatter、Converter 和 AnnotationFormatterFactory，它属于配置初始化数据绑定的一部分。



### Formatter

org.springframework.format.Formatter 是 Spring 提供用来替代 PropertyEditor 的，PropertyEditor 不是线程安全的，每个Web请求都会通过 WebDataBinder 注册创建新的 PropertyEditor 实例；而 Formatter 是线程安全的。



定义一个类似于 PersonEditor 的 PersonFormmater，它可以实现 Formatter 接口：

![image-20210819094212878](https://img-note.langyastudio.com/20210819094214.png?x-oss-process=style/watermark)



在 Spring MVC 下，通过 WebConfiguration 覆写 WebMvcConfigurer 接口的 addFormatters 方法来注册 Formatter。

![image-20210819094239686](https://img-note.langyastudio.com/20210819094242.png?x-oss-process=style/watermark)



Spring Boot 为我们提供了更简单的配置方式，只要注册 Formatter 的 Bean 即可，无须使用 addFormatters 方法。

![image-20210819094311133](https://img-note.langyastudio.com/20210819094312.png?x-oss-process=style/watermark)



此时还要注释掉 InitBinderAdvice 中关于代码注册（registerPersonEditor）的代码，否则 PersonEditor 和PersonFormatter 会产生冲突。

除用 addFormatters 方法注册外，还可以通过 InitBinderAdvice中@InitBinder 注解的方法来注册 Formatter。

![image-20210819094344175](https://img-note.langyastudio.com/20210819094346.png?x-oss-process=style/watermark)



用控制器验证，结果如图5-26所示。

![image-20210819094405400](https://img-note.langyastudio.com/20210819094407.png?x-oss-process=style/watermark)



![image-20210819094423616](https://img-note.langyastudio.com/20210819094425.png?x-oss-process=style/watermark)



###  AnnotationFormatterFactory

AnnotationFormatterFactory 创建 Formatter，用来格式化标记特定注解的属性或参数值。

- NumberFormatAnnotationFormatterFactory

  使用 @NumberFormat 注解创建 NumberStyle Formatter、CurrencyStyleFormatter 和 PercentStyleFormatter 这些 Formatter 来格式化注解的属性

- DateTimeFormatAnnotationFormatterFactory

  使用 @DateTimeFormat 注解创建 DateFormatter 的来格式化日期


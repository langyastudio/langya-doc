> 本文来自廖雪峰，郎涯进行简单排版与补充



## Web 基础

最早的 JavaEE 的名称是 J2EE：Java 2 Platform Enterprise Edition，后来改名为 JavaEE。由于 Oracle 将 JavaEE 移交给[Eclipse](https://www.eclipse.org/) 开源组织时，不允许他们继续使用 Java 商标，所以 JavaEE 再次改名为 [Jakarta EE](https://jakarta.ee/)。因为这个拼写比较复杂而且难记，所以我们后面还是用 JavaEE 这个缩写。

JavaEE 并不是一个软件产品，它更多的是一种软件架构和设计思想。我们可以把 JavaEE 看作是在 JavaSE 的基础上，**开发的一系列基于服务器的组件、API 标准和通用架构**。

JavaEE 最核心的组件就是基于 Servlet 标准的 Web 服务器，开发者编写的应用程序是基于 Servlet API 并运行在 Web 服务器内部的：

```ascii
┌─────────────┐
│┌───────────┐│
││ User App  ││
│├───────────┤│
││Servlet API││
│└───────────┘│
│ Web Server  │
├─────────────┤
│   JavaSE    │
└─────────────┘
```

此外，JavaEE 还有一系列技术标准：

- EJB：Enterprise JavaBean，企业级 JavaBean，早期经常用于实现应用程序的业务逻辑，现在基本被轻量级框架如Spring 所取代
- JAAS：Java Authentication and Authorization Service，一个标准的认证和授权服务，常用于企业内部，Web 程序通常使用更轻量级的自定义认证
- JCA：JavaEE Connector Architecture，用于连接企业内部的 EIS 系统等
- JMS：Java Message Service，用于消息服务
- JTA：Java Transaction API，用于分布式事务
- JAX-WS：Java API for XML Web Services，用于构建基于 XML 的 Web 服务
- ...

目前流行的基于 Spring 的轻量级 JavaEE 开发架构，使用最广泛的是 Servlet 和 JMS，以及一系列开源组件。



## Servlet

在 JavaEE 平台上，**解析 HTTP 协议、处理 TCP 连接等底层工作**统统扔给现成的 Web 服务器去做，我们只需要把自己的应用程序跑在 Web 服务器上。为了实现这一目的，JavaEE 提供了 Servlet API，我们使用 Servlet API 编写自己的 Servlet来处理 HTTP 请求，Web 服务器实现 Servlet API 接口，实现底层功能：

```ascii
                 ┌───────────┐
                 │My Servlet │
                 ├───────────┤
                 │Servlet API│
┌───────┐  HTTP  ├───────────┤
│Browser│<──────>│Web Server │
└───────┘        └───────────┘
```



### 第一个Servlet程序

- 编写 Web 应用程序就是编写 Servlet 处理 HTTP 请求

- Servlet API 提供了 `HttpServletRequest` 和 `HttpServletResponse` 两个高级接口来封装HTTP请求和响应

- Web 应用程序必须按固定结构组织并打包为 `.war` 文件

- 需要启动 Web 服务器来加载我们的 war 包来运行Servlet



一个 Servlet 总是继承自 `HttpServlet`，然后覆写 `doGet()` 或 `doPost()` 方法。

```java
// WebServlet注解表示这是一个Servlet，并映射到地址/:
@WebServlet(urlPatterns = "/")
public class HelloServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        // 设置响应类型:
        resp.setContentType("text/html");
        
        // 获取输出流:
        PrintWriter pw = resp.getWriter();
        // 写入响应:
        pw.write("<h1>Hello, world!</h1>");
        
        // 最后不要忘记flush强制输出:
        pw.flush();
    }
}
```

> 一个 Servlet 如果映射到 `/hello`，那么所有请求方法都会由这个 Servlet 处理，至于能不能返回 200 成功响应，要看有没有覆写对应的请求方法（post、get、put等）。



浏览器发出的 HTTP 请求总是由 Web Server 先接收，然后，根据 Servlet 配置的映射，不同的路径转发到不同的Servlet：

```ascii
               ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐

               │            /hello    ┌───────────────┐│
                          ┌──────────>│ HelloServlet  │
               │          │           └───────────────┘│
┌───────┐    ┌──────────┐ │ /signin   ┌───────────────┐
│Browser│───>│Dispatcher│─┼──────────>│ SignInServlet ││
└───────┘    └──────────┘ │           └───────────────┘
               │          │ /         ┌───────────────┐│
                          └──────────>│ IndexServlet  │
               │                      └───────────────┘│
                              Web Server
               └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

映射到 `/` 的 `IndexServlet` 比较特殊，它实际上会接收所有 **未匹配** 的路径，相当于 `/*`



Servlet API 是一个 jar 包，我们需要通过 Maven 来引入它，才能正常编译。编写 `pom.xml ` 文件如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.itranswarp.learnjava</groupId>
    <artifactId>web-servlet-hello</artifactId>
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>hello</finalName>
    </build>
</project>
```

打包类型不是 `jar`，而是 `war`，表示 Java Web Application Archive：

```xml
<packaging>war</packaging>
```



我们还需要在工程目录下创建一个 `web.xml` 描述文件，放到 `src/main/webapp/WEB-INF` 目录下（固定目录结构，不要修改路径，注意大小写）。文件内容可以固定如下：

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd">
<web-app>
  <display-name>Archetype Created Web Application</display-name>
</web-app>
```

整个工程结构如下：

```ascii
web-servlet-hello
├── pom.xml
└── src
    └── main
        ├── java
        │   └── com
        │       └── itranswarp
        │           └── learnjava
        │               └── servlet
        │                   └── HelloServlet.java
        ├── resources
        └── webapp
            └── WEB-INF
                └── web.xml
```

运行 Maven 命令 `mvn clean package`，在 `target` 目录下得到一个 `hello.war` 文件，需要借助 web 服务器才能运行。



常用的服务器有：

- [Tomcat](https://tomcat.apache.org/)：由 Apache 开发的开源免费服务器
- [Jetty](https://www.eclipse.org/jetty/)：由 Eclipse 开发的开源免费服务器
- [GlassFish](https://javaee.github.io/glassfish/)：一个开源的全功能 JavaEE 服务器

还有一些收费的商用服务器，如 Oracle 的 [WebLogic](https://www.oracle.com/middleware/weblogic/)，IBM 的 [WebSphere](https://www.ibm.com/cloud/websphere-application-platform/)。



实际上，类似 Tomcat 这样的服务器也是 Java 编写的，启动 Tomcat 服务器实际上是启动 Java 虚拟机，执行 Tomcat 的`main() `法，然后由 Tomcat 负责加载我们的 `.war` 文件，并创建一个 `HelloServlet` 实例，最后以多线程的模式来处理HTTP 请求。

> 把 `hello.war` 复制到 Tomcat 的 `webapps` 目录下，在浏览器输入 `http://localhost:8080/hello/` 即可看到`HelloServlet` 的输出



### Debug

通过 `main()` 方法启动 Tomcat 服务器并加载我们自己的 webapp 有如下好处：

- 启动简单，无需下载 Tomcat 或安装任何 IDE 插件
- 调试方便，可在 IDE 中使用断点调试
- 使用 Maven 创建 war 包后，也可以正常部署到独立的 Tomcat 服务器中



编写`pom.xml`如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itranswarp.learnjava</groupId>
    <artifactId>web-servlet-embedded</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <java.version>11</java.version>
        <tomcat.version>9.0.26</tomcat.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
            <version>${tomcat.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <version>${tomcat.version}</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
</project>
```

其中，`<packaging>` 类型仍然为 `war`，引入依赖 `tomcat-embed-core` 和 `tomcat-embed-jasper`，引入的 Tomcat 版本 `<tomcat.version>` 为 `9.0.26`。

不必引入 Servlet API，因为引入 Tomcat 依赖后自动引入了 Servlet API。

正常编写 Servlet 后。再编写一个 `main()` 方法，启动 Tomcat 服务器：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 启动Tomcat:
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(Integer.getInteger("port", 8080));
        tomcat.getConnector();
       
        // 创建webapp:
        Context ctx = tomcat.addWebapp("", new File("src/main/webapp").getAbsolutePath());
        WebResourceRoot resources = new StandardRoot(ctx);
        resources.addPreResources(
                new DirResourceSet(resources, "/WEB-INF/classes", new File("target/classes").getAbsolutePath(), "/"));
        ctx.setResources(resources);
        
        tomcat.start();
        tomcat.getServer().await();
    }
}
```

这样，我们直接运行 `main()` 方法，即可启动嵌入式 Tomcat 服务器



**错误: 无法初始化主类 Main  java.lang.NoClassDefFoundError: org/apache/catalina/WebResourceRoot**

在 IDEA 中，maven 配置 `<scope>provided</scope>`，就告诉了 IDEA 程序会在运行的时候提供这个依赖，但是实际上却并没有提供这个依赖。

解决方案：

1. 打开idea的Run/Debug Configurations

2. 选择Application - Main

3. 右侧Configuration：Use classpath of module

4. 钩上☑︎Include dependencies with "Provided" scope

    

### HttpServlet

**一个 Servlet 类在服务器中只有一个实例**。Web 服务器通过多线程处理 HTTP 请求，一个 Servlet 的处理方法可以由多线程并发执行。

**HttpServletRequest**

`HttpServletRequest` 封装了一个 HTTP 请求，它实际上是从 `ServletRequest` 继承而来。最早设计 Servlet 时，设计者希望 Servlet 不仅能处理 HTTP，也能处理类似 SMTP 等其他协议，因此，单独抽出了 `ServletRequest` 接口，但实际上除了 HTTP 外，并没有其他协议会用 Servlet 处理，所以这是一个过度设计。

我们通过 `HttpServletRequest` 提供的接口方法可以拿到 HTTP 请求的几乎全部信息，常用的方法有：

- getMethod()：返回请求方法，例如，`"GET"`，`"POST"`
- getRequestURI()：返回请求路径，但不包括请求参数，例如，`"/hello"`
- getQueryString()：返回请求参数，例如，`"name=Bob&a=1&b=2"`
- getParameter(name)：返回请求参数，GET请求从URL读取参数，POST请求从Body中读取参数
- getContentType()：获取请求Body的类型，例如，`"application/x-www-form-urlencoded"`
- getContextPath()：获取当前Webapp挂载的路径，对于ROOT来说，总是返回空字符串`""`
- getCookies()：返回请求携带的所有Cookie
- getHeader(name)：获取指定的Header，对Header名称不区分大小写
- getHeaderNames()：返回所有Header名称
- getInputStream()：如果该请求带有HTTP Body，该方法将打开一个输入流用于读取Body
- getReader()：和getInputStream()类似，但打开的是Reader
- getRemoteAddr()：返回客户端的IP地址
- getScheme()：返回协议类型，例如，`"http"`，`"https"`

此外，`HttpServletRequest` 还有两个方法：**`setAttribute()` 和 `getAttribute()`**，可以给当前`HttpServletRequest` 对象附加多个 Key-Value，相当于把 `HttpServletRequest` 当作一个  `Map<String, Object> ` 使用。



**HttpServletResponse**

`HttpServletResponse` 封装了一个 HTTP 响应。由于 HTTP 响应必须先发送 Header，再发送 Body，所以，操作`HttpServletResponse `对象时，必须先调用设置 Header 的方法，最后调用发送Body 的方法。

常用的设置 Header 的方法有：

- setStatus(sc)：设置响应代码，默认是`200`
- setContentType(type)：设置Body的类型，例如，`"text/html"`
- setCharacterEncoding(charset)：设置字符编码，例如，`"UTF-8"`
- setHeader(name, value)：设置一个Header的值
- addCookie(cookie)：给响应添加一个Cookie
- addHeader(name, value)：给响应添加一个Header，因为HTTP协议允许有多个相同的Header

写入响应时，需要通过 `getOutputStream()` 获取写入流，或者通过 `getWriter()` 获取字符流，二者只能获取其中一个。

> 写入完毕后对输出流调用flush()而不是close()方法！(复用此TCP连接)



### 重定向与转发

使用重定向时，**浏览器知道重定向规则**，并且会自动发起新的 HTTP 请求

使用转发时，**浏览器并不知道服务器内部的转发逻辑**



**Redirect**

重定向是指当浏览器请求一个 URL 时，服务器返回一个重定向指令，告诉浏览器地址已经变了，麻烦使用新的 URL 再重新发送新请求。

例如，我们已经编写了一个能处理 `/hello` 的 `HelloServlet`，如果收到的路径为 `/hi`，希望能重定向到 `/hello`，可以再编写一个 `RedirectServlet`：

```java
@WebServlet(urlPatterns = "/hi")
public class RedirectServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 构造重定向的路径:
        String query = req.getQueryString();
        String redirectToUrl = "/hello" + (query == null ? "" : "?" + query);
        
        // 发送重定向响应:
        resp.sendRedirect(redirectToUrl);
    }
}
```

如果浏览器发送 `GET /hi` 请求，`RedirectServlet ` 将处理此请求。由于 `RedirectServlet` 在内部又发送了重定向响应，因此，浏览器会收到如下响应：

```
HTTP/1.1 302 Found
Location: /hello
```

**重定向有两种：**

一种是302响应，称为临时重定向

一种是301响应，称为永久重定向

两者的区别是，如果服务器发送 301 永久重定向响应，浏览器会缓存 `/hi` 到 `/hello` 这个重定向的关联，下次请求 `/hi` 的时候，浏览器就直接发送 `/hello` 请求了。



**Forward**

Forward 是指内部转发。当一个 Servlet 处理请求的时候，它可以决定自己不继续处理，而是转发给另一个 Servlet 处理。

例如，我们已经编写了一个能处理 `/hello` 的 `HelloServlet`，继续编写一个能处理 `/morning` 的 `ForwardServlet`：

```java
@WebServlet(urlPatterns = "/morning")
public class ForwardServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.getRequestDispatcher("/hello").forward(req, resp);
    }
}
```

`ForwardServlet ` 在收到请求后，它并不自己发送响应，而是把请求和响应都转发给路径为 `/hello` 的 Servlet

```ascii
                          ┌────────────────────────┐
                          │      ┌───────────────┐ │
                          │ ────>│ForwardServlet │ │
┌───────┐  GET /morning   │      └───────────────┘ │
│Browser│ ──────────────> │              │         │
│       │ <────────────── │              ▼         │
└───────┘    200 <html>   │      ┌───────────────┐ │
                          │ <────│ HelloServlet  │ │
                          │      └───────────────┘ │
                          │       Web Server       │
                          └────────────────────────┘
```



### Session与Cookie

**Session**

Servlet 容器提供了 Session 机制以跟踪用户

```java
req.getSession().setAttribute("web-user", "abc");
```



使用多台服务器构成集群时，使用 Session 会遇到一些额外的问题：一个用户在 Web Server 1 存储的 Session 信息，在Web Server 2和3上并不存在

```ascii
                                     ┌────────────┐
                                ┌───>│Web Server 1│
                                │    └────────────┘
┌───────┐     ┌─────────────┐   │    ┌────────────┐
│Browser│────>│Reverse Proxy│───┼───>│Web Server 2│
└───────┘     └─────────────┘   │    └────────────┘
                                │    ┌────────────┐
                                └───>│Web Server 3│
                                     └────────────┘
```

解决方案：

- 采用粘滞会话（Sticky Session）机制，即反向代理在转发请求的时候，总是根据 JSESSIONID 的值判断，相同的JSESSIONID 总是转发到固定的 Web Server，但这需要反向代理的支持
- 采用第三方存储例如 Redis 进行多服务器共享机制

> 对于大型 Web 应用程序来说，通常需要避免使用 Session 机制



**Cookie**

浏览器在请求某个 URL 时，是否携带指定的 Cookie，取决于 Cookie 是否满足以下所有要求：

- URL 前缀是设置 Cookie 时的 Path
- Cookie 在有效期内
- Cookie 设置了 secure 时必须以 https 访问

```java
// 创建一个新的Cookie:
Cookie cookie = new Cookie("lang", lang);

// 该Cookie生效的路径范围:
cookie.setPath("/");
// 该Cookie有效期:
cookie.setMaxAge(8640000); // 8640000秒=100天
// 将该Cookie添加到响应:
resp.addCookie(cookie);
```



## JSP开发

JSP 是在 HTML 中写 Java 代码，JSP 本身目前已经很少使用，我们只需要了解其基本用法即可。

它的文件必须放到 `/src/main/webapp` 下，文件名必须以 `.jsp` 结尾，整个文件与 HTML 并无太大区别，但是稍有不同：

- 包含在 `<%--`和`--%>` 之间的是 JSP 的注释，它们会被完全忽略

- 包含在 `<%`和`%> `之间的是 Java 代码，可以编写任意 Java 代码

- 如果使用 `<%= xxx %>` 则可以快捷输出一个变量的值

    

JSP 页面内置了几个变量：

- out：表示 HttpServletResponse 的 PrintWriter
- session：表示当前 HttpSession 对象
- request：表示 HttpServletRequest 对象

这几个变量可以直接使用。

```jsp
<html>
<head>
    <title>Hello World - JSP</title>
</head>
<body>
    <%-- JSP Comment --%>
    <h1>Hello World!</h1>
    <p>
    <%
         out.println("Your IP address is ");
    %>
    <span style="color:red">
        <%= request.getRemoteAddr() %>
    </span>
    </p>
</body>
</html>
```

**JSP 本质上就是一个 Servlet，因为 JSP 在执行前首先被编译成一个 Servlet。**

访问 JSP 页面时，直接指定完整路径。例如，`http://localhost:8080/hello.jsp`



**JSP 高级功能**

JSP 的指令非常复杂，除了 `<% ... %>` 外，JSP 页面本身可以通过 `page` 指令引入 Java 类：

```jsp
<%@ page import="java.io.*" %>
<%@ page import="java.util.*" %>
```

这样后续的 Java 代码才能引用简单类名而不是完整类名。

使用 `include` 指令可以引入另一个 JSP 文件：

```jsp
<html>
<body>
    <%@ include file="header.jsp"%>
    <h1>Index Page</h1>
    <%@ include file="footer.jsp"%>
</body>
```



## MVC开发

MVC 模式是一种分离业务逻辑和显示逻辑的设计模式，广泛应用在 Web 和桌面应用程序。

```ascii
                   ┌───────────────────────┐
             ┌────>│Controller: UserServlet│
             │     └───────────────────────┘
             │                 │
┌───────┐    │           ┌─────┴─────┐
│Browser│────┘           │Model: User│
│       │<───┐           └─────┬─────┘
└───────┘    │                 │
             │                 ▼
             │     ┌───────────────────────┐
             └─────│    View: user.jsp     │
                   └───────────────────────┘
```

使用 MVC 模式的好处是，Controller 专注于业务处理，它的处理结果就是 Model。Controller 只负责把 Model 传递给View，View 只负责把 Model 给“渲染”出来，这样，三者职责明确，且开发更简单。



直接把 MVC 搭在 Servlet 和 JSP 之上还是不太好，原因如下：

- Servlet 提供的接口仍然偏底层，需要实现 Servlet 调用相关接口
- JSP 对页面开发不友好，更好的替代品是模板引擎
- 业务逻辑最好由纯粹的 Java 类实现，而不是强迫继承自Servlet

简单设计的 MVC 的架构如下：

```ascii
   HTTP Request    ┌─────────────────┐
──────────────────>│DispatcherServlet│
                   └─────────────────┘
                            │
               ┌────────────┼────────────┐
               ▼            ▼            ▼
         ┌───────────┐┌───────────┐┌───────────┐
         │Controller1││Controller2││Controller3│
         └───────────┘└───────────┘└───────────┘
               │            │            │
               └────────────┼────────────┘
                            ▼
   HTTP Response ┌────────────────────┐
<────────────────│render(ModelAndView)│
                 └────────────────────┘
```

Java有很多开源的模板引擎，常用的有：

- [Thymeleaf](https://www.thymeleaf.org/)
- [FreeMarker](https://freemarker.apache.org/)
- [Velocity](https://velocity.apache.org/)

他们的用法都大同小异。这里我们推荐一个使用[Jinja](https://palletsprojects.com/p/jinja/)语法的模板引擎[Pebble](https://pebbletemplates.io/)，它的特点是语法简单，支持模板继承



## Filter

为了把一些公用逻辑从各个 Servlet 中抽离出来，JavaEE 的 Servlet 规范还提供了一种 Filter 组件，即过滤器，它的作用是，在 HTTP 请求**到达 Servlet 之前，可以被一个或多个 Filter 预处理**，类似打印日志、登录检查等逻辑，完全可以放到Filter 中。



例如，我们编写一个最简单的 EncodingFilter，它强制把输入和输出的编码设置为 UTF-8：

```java
@WebFilter(urlPatterns = "/*")
public class EncodingFilter implements Filter {
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("EncodingFilter:doFilter");
        request.setCharacterEncoding("UTF-8");
        response.setCharacterEncoding("UTF-8");
        
        chain.doFilter(request, response);
    }
}
```

用 `@WebFilter` 注解标注该 Filter 需要过滤的 URL。这里的 `/*` 表示所有路径。

> 如果 Filter 要使请求继续被处理，就一定要调用 chain.doFilter()！



还可以继续添加其他 Filter，例如 LogFilter：

```java
@WebFilter("/*")
public class LogFilter implements Filter {
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("LogFilter: process " + ((HttpServletRequest) request).getRequestURI());
        
        chain.doFilter(request, response);
    }
}
```

多个 Filter 会组成一个链，每个请求都被链上的 Filter 依次处理：

```ascii
                                        ┌────────┐
                                     ┌─>│ServletA│
                                     │  └────────┘
    ┌──────────────┐    ┌─────────┐  │  ┌────────┐
───>│EncodingFilter│───>│LogFilter│──┼─>│ServletB│
    └──────────────┘    └─────────┘  │  └────────┘
                                     │  ┌────────┐
                                     └─>│ServletC│
                                        └────────┘
```

Servlet 规范并没有对 `@WebFilter` 注解标注的 Filter 规定顺序。如果一定要给每个 Filter 指定顺序，就必须在 `web.xml`文件中对这些 Filter 再配置一遍。



#### 配置filter顺序

Filter注解（urlPatterns在web.xml配置即可）：

```java
@WebFilter(filterName = "EncodingFilter")
@WebFilter(filterName = "LogFilter")
```

web.xml里:

```xml
<!-- 过滤顺序：谁的写在上面，谁先被过滤 -->
  <filter-mapping>
    <filter-name>EncodingFilter</filter-name>
    <url-pattern>/*</url-pattern> 
  </filter-mapping>

  <filter-mapping>
    <filter-name>LogFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```



#### 修改请求

例如，保证文件上传的完整性怎么办？我们知道，如果在上传文件的同时，把文件的哈希也传过来，服务器端做一个验证，就可以确保用户上传的文件一定是完整的。这个验证逻辑非常适合写在 `ValidateUploadFilter` 中，因为它可以复用。

```java
@WebFilter("/upload/*")
public class ValidateUploadFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse resp = (HttpServletResponse) response;
        
        // 获取客户端传入的签名方法和签名:
        String digest = req.getHeader("Signature-Method");
        String signature = req.getHeader("Signature");
        if (digest == null || digest.isEmpty() || signature == null || signature.isEmpty())         {
            sendErrorPage(resp, "Missing signature.");
            return;
        }
        
        // 读取Request的Body并验证签名:
        MessageDigest md = getMessageDigest(digest);
        InputStream input = new DigestInputStream(request.getInputStream(), md);
        byte[] buffer = new byte[1024];
        for (;;) {
            int len = input.read(buffer);
            if (len == -1) {
                break;
            }
        }
        String actual = toHexString(md.digest());
        if (!signature.equals(actual)) {
            sendErrorPage(resp, "Invalid signature.");
            return;
        }
        
        // 验证成功后继续处理:
        chain.doFilter(request, response);
    }

    // 将byte[]转换为hex string:
    private String toHexString(byte[] digest) {
        StringBuilder sb = new StringBuilder();
        for (byte b : digest) {
            sb.append(String.format("%02x", b));
        }
        return sb.toString();
    }

    // 根据名称创建MessageDigest:
    private MessageDigest getMessageDigest(String name) throws ServletException {
        try {
            return MessageDigest.getInstance(name);
        } catch (NoSuchAlgorithmException e) {
            throw new ServletException(e);
        }
    }

    // 发送一个错误响应:
    private void sendErrorPage(HttpServletResponse resp, String errorMessage) throws IOException {
        resp.setStatus(HttpServletResponse.SC_BAD_REQUEST);
        PrintWriter pw = resp.getWriter();
        pw.write("<html><body><h1>");
        pw.write(errorMessage);
        pw.write("</h1></body></html>");
        pw.flush();
    }
}
```

以上代码存在的问题是：对 `HttpServletRequest` 进行读取时，**只能读取一次**。如果 Filter 调用 `getInputStream()`读取了一次数据，后续 Servlet 处理时，再次读取，将无法读到任何数据。



这个时候，我们需要一个“伪造”的`HttpServletRequest`，具体做法是使用代理模式，对 `getInputStream()` 和`getReader()` 返回一个新的流：

```java
class ReReadableHttpServletRequest extends HttpServletRequestWrapper {
    private byte[] body;
    private boolean open = false;

    public ReReadableHttpServletRequest(HttpServletRequest request, byte[] body) {
        super(request);
        this.body = body;
    }

    // 返回InputStream:
    public ServletInputStream getInputStream() throws IOException {
        if (open) {
            throw new IllegalStateException("Cannot re-open input stream!");
        }
        
        open = true;
        return new ServletInputStream() {
            private int offset = 0;

            public boolean isFinished() {
                return offset >= body.length;
            }

            public boolean isReady() {
                return true;
            }

            public void setReadListener(ReadListener listener) {
            }

            public int read() throws IOException {
                if (offset >= body.length) {
                    return -1;
                }
                int n = body[offset] & 0xff;
                offset++;
                return n;
            }
        };
    }

    // 返回Reader:
    public BufferedReader getReader() throws IOException {
        if (open) {
            throw new IllegalStateException("Cannot re-open reader!");
        }
        
        open = true;
        return new BufferedReader(new InputStreamReader(getInputStream(), "UTF-8"));
    }
}
```

注意观察 `ReReadableHttpServletRequest` 的构造方法，它保存了 `ValidateUploadFilter` 读取的 `byte[]` 内容，并在调用 `getInputStream()` 时通过 `byte[]`构 造了一个新的 `ServletInputStream`。

然后，我们在 `ValidateUploadFilter` 中，把 `doFilter()` 调用时传给下一个处理者的 `HttpServletRequest` 替换为我们自己“伪造”的 `ReReadableHttpServletRequest`：

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        throws IOException, ServletException {
    ...
    chain.doFilter(new ReReadableHttpServletRequest(req, output.toByteArray()), response);
}
```

借助 `HttpServletRequestWrapper`，我们可以在 Filter 中实现对原始 `HttpServletRequest` 的修改。对`HttpServletRequest` 接口进行代理的步骤：

- 从 `HttpServletRequestWrapper` 继承一个 `XxxHttpServletRequest`，需要传入原始的 `HttpServletRequest`实例

- 覆写某些方法，使得新的 `XxxHttpServletRequest` 实例看上去“改变”了原始的 `HttpServletRequest` 实例

- 在 `doFilter()` 中传入新的 `XxxHttpServletRequest` 实例

虽然整个 Filter 的代码比较复杂，但它的好处在于：这个 Filter 在整个处理链中实现了灵活的**可插拔**特性，即是否启用对Web 应用程序的其他组件（Filter、Servlet）完全没有影响



#### 修改响应

借助 `HttpServletResponseWrapper`，我们可以在 Filter 中实现对原始 `HttpServletRequest` 的修改。

假设我们编写了一个 Servlet，但由于业务逻辑比较复杂，处理该请求需要耗费很长的时间：好消息是每次返回的响应内容是固定的，因此，如果我们能使用缓存将结果缓存起来，就可以大大提高Web应用程序的运行效率。缓存逻辑最好不要在 Servlet 内部实现，因为我们希望能复用缓存逻辑，所以，编写一个 `CacheFilter` 最合适：

```java
@WebFilter("/slow/*")
public class CacheFilter implements Filter {
    // Path到byte[]的缓存:
    private Map<String, byte[]> cache = new ConcurrentHashMap<>();

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse resp = (HttpServletResponse) response;
        
        // 获取Path:
        String url = req.getRequestURI();
        // 获取缓存内容:
        byte[] data = this.cache.get(url);
        resp.setHeader("X-Cache-Hit", data == null ? "No" : "Yes");
        
        if (data == null) {
            // 缓存未找到,构造一个伪造的Response:
            CachedHttpServletResponse wrapper = new CachedHttpServletResponse(resp);
            // 让下游组件写入数据到伪造的Response:
            chain.doFilter(request, wrapper);
            
            // 从伪造的Response中读取写入的内容并放入缓存:
            data = wrapper.getContent();
            cache.put(url, data);
        }
        
        // 写入到原始的Response:
        ServletOutputStream output = resp.getOutputStream();
        output.write(data);
        output.flush();
    }
}
```

这个 `CachedHttpServletResponse` 实现如下：

```java
class CachedHttpServletResponse extends HttpServletResponseWrapper {
    private boolean open = false;
    private ByteArrayOutputStream output = new ByteArrayOutputStream();

    public CachedHttpServletResponse(HttpServletResponse response) {
        super(response);
    }

    // 获取Writer:
    public PrintWriter getWriter() throws IOException {
        if (open) {
            throw new IllegalStateException("Cannot re-open writer!");
        }
        
        open = true;
        return new PrintWriter(output, false, StandardCharsets.UTF_8);
    }

    // 获取OutputStream:
    public ServletOutputStream getOutputStream() throws IOException {
        if (open) {
            throw new IllegalStateException("Cannot re-open output stream!");
        }
        
        open = true;
        return new ServletOutputStream() {
            public boolean isReady() {
                return true;
            }

            public void setWriteListener(WriteListener listener) {
            }

            // 实际写入ByteArrayOutputStream:
            public void write(int b) throws IOException {
                output.write(b);
            }
        };
    }

    // 返回写入的byte[]:
    public byte[] getContent() {
        return output.toByteArray();
    }
}
```



## Listener

通过 Listener 我们可以**监听 Web 应用程序的生命周期**，获取 `HttpSession` 等创建和销毁的事件；其中最常用的是`ServletContextListener`。我们编写一个实现了 `ServletContextListener` 接口的类如下：

```java
@WebListener
public class AppListener implements ServletContextListener {
    // 在此初始化WebApp,例如打开数据库连接池等:
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("WebApp initialized.");
    }

    // 在此清理WebApp,例如关闭数据库连接池等:
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("WebApp destroyed.");
    }
}
```

任何标注为 `@WebListener`，且实现了特定接口的类会被 Web 服务器自动初始化。

除了 `ServletContextListener` 外，还有几种 Listener：

- HttpSessionListener：监听HttpSession的创建和销毁事件
- ServletRequestListener：监听ServletRequest请求的创建和销毁事件
- ServletRequestAttributeListener：监听ServletRequest请求的属性变化事件（即调用`ServletRequest.setAttribute()`方法）
- ServletContextAttributeListener：监听ServletContext的属性变化事件（即调用`ServletContext.setAttribute()`方法）



**ServletContext**

一个 Web 服务器可以运行一个或多个 WebApp，对于每个 WebApp，Web 服务器都会为其创建一个**全局唯一**的`ServletContext` 实例（可用于设置和共享配置信息），我们在 `AppListener` 里面编写的两个回调方法实际上对应的就是 `ServletContext` 实例的创建和销毁：

```java
public void contextInitialized(ServletContextEvent sce) {
    System.out.println("WebApp initialized: ServletContext = " + sce.getServletContext());
}
```

`ServletRequest`、`HttpSession` 等很多对象也提供 `getServletContext()` 方法获取到同一个 `ServletContext` 实例。

> 此外，`ServletContext` 还提供了动态添加 Servlet、Filter、Listener 等功能，它允许应用程序在运行期间动态添加一个组件，虽然这个功能不是很常用



## 部署

部署 Web 应用程序时，要设计合理的目录结构，同时考虑开发模式需要便捷性，生产模式需要高性能。

合理组织文件结构非常重要。我们以一个具体的 Web 应用程序为例：

```ascii
webapp
├── pom.xml
└── src
    └── main
        ├── java
        │   └── com
        │       └── itranswarp
        │           └── learnjava
        │               ├── Main.java
        │               ├── filter
        │               │   └── EncodingFilter.java
        │               └── servlet
        │                   ├── FileServlet.java
        │                   └── HelloServlet.java
        ├── resources
        └── webapp
            ├── WEB-INF
            │   └── web.xml
            ├── favicon.ico
            └── static
                └── bootstrap.css
```



我们把所有的静态资源文件放入 `/static/` 目录，在开发阶段，有些 Web 服务器会自动为我们加一个专门负责处理静态文件的 Servlet，但如果 `IndexServlet` 映射路径为 `/`，会屏蔽掉处理静态文件的 Servlet 映射。因此，我们需要自己编写一个处理静态文件的 `FileServlet`：

```java
@WebServlet(urlPatterns = "/static/*")
public class FileServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext ctx = req.getServletContext();
        
        // RequestURI包含ContextPath,需要去掉:
        String urlPath = req.getRequestURI().substring(ctx.getContextPath().length());
       
        // 获取真实文件路径:
        String filepath = ctx.getRealPath(urlPath);
        if (filepath == null) {
            // 无法获取到路径:
            resp.sendError(HttpServletResponse.SC_NOT_FOUND);
            return;
        }
        
        Path path = Paths.get(filepath);
        if (!path.toFile().isFile()) {
            // 文件不存在:
            resp.sendError(HttpServletResponse.SC_NOT_FOUND);
            return;
        }
        
        // 根据文件名猜测Content-Type:
        String mime = Files.probeContentType(path);
        if (mime == null) {
            mime = "application/octet-stream";
        }        
        resp.setContentType(mime);
        
        // 读取文件并写入Response:
        OutputStream output = resp.getOutputStream();
        try (InputStream input = new BufferedInputStream(new FileInputStream(filepath))) {
            input.transferTo(output);
        }
        output.flush();
    }
}
```



通常，我们在生产环境部署时，总是使用类似 Nginx 这样的服务器充当反向代理和静态服务器，只有动态请求才会放行给应用服务器，所以，部署架构如下：

使用 Nginx+Tomcat 部署

```ascii
             ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐

             │  /static/*            │
┌───────┐      ┌──────────> file
│Browser├────┼─┤                     │    ┌ ─ ─ ─ ─ ─ ─ ┐
└───────┘      │/          proxy_pass
             │ └─────────────────────┼───>│  Web Server │
                       Nginx                   tomcat
             └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘    └ ─ ─ ─ ─ ─ ─ ┘
```

实现上述功能的Nginx配置文件如下：

```nginx
server {
    listen 80;

    server_name www.local.liaoxuefeng.com;

    # 静态文件根目录:
    root /path/to/src/main/webapp;

    access_log /var/log/nginx/webapp_access_log;
    error_log  /var/log/nginx/webapp_error_log;

    # 处理静态文件请求:
    location /static {
    }

    # 处理静态文件请求:
    location /favicon.ico {
    }

    # 不允许请求/WEB-INF:
    location /WEB-INF {
        return 404;
    }

    # 其他请求转发给Tomcat:
    location / {
        proxy_pass       http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

使用 Nginx 配合 Tomcat 服务器，可以充分发挥 Nginx 作为网关的优势，既可以高效处理静态文件，也可以把 https、防火墙、限速、反爬虫等功能放到 Nginx 中，使得我们自己的 WebApp 能专注于业务逻辑。



## JDBC

### 简介

数据库（Database）是专门用于集中存储和查询的软件。

**数据库类别**

付费的商用数据库：

- Oracle，典型的高富帅

- SQL Server，微软自家产品，Windows定制专款

- DB2，IBM的产品，听起来挺高端

免费的开源数据库：

- MySQL，大家都在用，一般错不了

- PostgreSQL，学术气息有点重，其实挺不错，但知名度没有MySQL高

- sqlite，嵌入式数据库，适合桌面和移动应用

    

**JDBC**

JDBC 是 **Java DataBase Connectivity **的缩写，它是 Java 程序访问数据库的**标准接口**。

使用 JDBC 的好处是：

- 各数据库厂商使用相同的接口，Java 代码不需要针对不同数据库分别开发
- Java 程序编译期仅依赖 java.sql 包，不依赖具体数据库的 jar 包
- 可随时替换底层数据库，访问数据库的 Java 代码基本不变

具体的 JDBC 驱动是由数据库厂商提供的

```ascii
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐

│  ┌───────────────┐  │
   │   Java App    │
│  └───────────────┘  │
           │
│          ▼          │
   ┌───────────────┐
│  │JDBC Interface │<─┼─── JDK
   └───────────────┘
│          │          │
           ▼
│  ┌───────────────┐  │
   │  JDBC Driver  │<───── Vendor
│  └───────────────┘  │
           │
└ ─ ─ ─ ─ ─│─ ─ ─ ─ ─ ┘
           ▼
   ┌───────────────┐
   │   Database    │
   └───────────────┘
```



### 查询与更新

#### 查询

- JDBC 接口的 `Connection` 代表一个 JDBC 连接

- 使用 JDBC 查询时，总是使用 `PreparedStatement` 进行查询而不是 `Statement`

- 查询结果总是 `ResultSet`，即使使用聚合查询也不例外



某个数据库实现了 JDBC 接口的 jar 包称为 JDBC 驱动

直接添加一个 Maven 依赖就可以了，例如 MySQL：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
    <scope>runtime</scope>
</dependency>
```



**JDBC 连接**

获取数据库连接，使用如下代码：

```java
// JDBC连接的URL, 不同数据库有不同的格式:
String JDBC_URL = "jdbc:mysql://localhost:3306/test";
String JDBC_USER = "root";
String JDBC_PASSWORD = "password";

// 获取连接:
Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD);
// TODO: 访问数据库...
// 关闭连接:
conn.close();
```



获取到 JDBC 连接后，下一步我们就可以查询数据库了。查询数据库分以下几步：

第一步，通过`Connection`提供的`createStatement()`方法创建一个`Statement`对象，用于执行一个查询

第二步，执行`Statement`对象提供的`executeQuery("SELECT * FROM students")`并传入SQL语句，执行查询并获得返回的结果集，使用`ResultSet`来引用这个结果集

第三步，反复调用`ResultSet`的`next()`方法并读取每一行结果

完整查询代码如下：

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (Statement stmt = conn.createStatement()) {
        try (ResultSet rs = stmt.executeQuery("SELECT id, grade, name, gender FROM students WHERE gender=1")) {
            while (rs.next()) {
                long id = rs.getLong(1); // 注意：索引从1开始
                long grade = rs.getLong(2);
                String name = rs.getString(3);
                int gender = rs.getInt(4);
            }
        }
    }
}
```



**SQL注入**

使用 `PreparedStatement` 可以 **完全避免SQL注入** 的问题，因为 `PreparedStatement` 始终使用 `?` 作为占位符，并且把数据连同 SQL 本身传给数据库，这样可以保证每次传给数据库的SQL语句是相同的，只是占位符的数据不同，还能高效利用数据库本身对查询的缓存。上述登录SQL如果用`PreparedStatement`可以改写如下：

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement("SELECT id, grade, name, gender FROM students WHERE gender=? AND grade=?")) {
        ps.setObject(1, "M"); // 注意：索引从1开始
        ps.setObject(2, 3);
        
        try (ResultSet rs = ps.executeQuery()) {
            while (rs.next()) {
                long id = rs.getLong("id");
                long grade = rs.getLong("grade");
                String name = rs.getString("name");
                String gender = rs.getString("gender");
            }
        }
    }
}
```

 

**数据类型**

JDBC在 `java.sql.Types` 定义了一组常量来表示如何映射SQL数据类型，但是平时我们使用的类型通常也就以下几种：

| SQL数据类型       | Java数据类型             |
| :---------------- | :----------------------- |
| BIT, BOOL         | boolean                  |
| INTEGER           | int                      |
| BIGINT            | long                     |
| **REAL**          | **float**                |
| **FLOAT, DOUBLE** | **double**               |
| CHAR, VARCHAR     | String                   |
| DECIMAL           | BigDecimal               |
| DATE              | java.sql.Date, LocalDate |
| TIME              | java.sql.Time, LocalTime |

> 注意：只有最新的 JDBC 驱动才支持`LocalDate`和`LocalTime`



#### 更新

- 使用 **JDBC 执行 `INSERT`、`UPDATE`和`DELETE` 都可视为更新操作**

- 更新操作使用 `PreparedStatement` 的 `executeUpdate()` 进行，返回受影响的行数



**插入并获取主键**

插入操作是 `INSERT`，即插入一条新记录。通过 JDBC 进行插入，本质上也是用`PreparedStatement`执行一条SQL语句，不过最后执行的不是 `executeQuery()`，而是 `executeUpdate()`。示例代码如下：

获取自增主键的正确写法是在创建`PreparedStatement`的时候，指定一个`RETURN_GENERATED_KEYS`标志位，表示JDBC驱动必须返回插入的自增主键。示例代码如下：

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement(
            "INSERT INTO students (grade, name, gender) VALUES (?,?,?)",
            Statement.RETURN_GENERATED_KEYS)) {
        ps.setObject(1, 1); // grade
        ps.setObject(2, "Bob"); // name
        ps.setObject(3, "M"); // gender
        
        int n = ps.executeUpdate(); // 1
        try (ResultSet rs = ps.getGeneratedKeys()) {
            if (rs.next()) {
                long id = rs.getLong(1); // 注意：索引从1开始
            }
        }
    }
}
```

更新与删除操作与插入类似



### 连接池

为了**避免频繁地创建和销毁 JDBC 连接**，我们可以通过连接池（Connection Pool）复用已经创建好的连接。

- 数据库连接池是一种复用 `Connection` 的组件，它可以避免反复创建新连接，提高 JDBC 代码的运行效率

- 可以配置连接池的详细参数并监控连接池

JDBC 连接池有一个标准的接口 `javax.sql.DataSource`，注意这个类位于Java标准库中，但仅仅是接口。要使用 JDBC连接池，我们必须选择一个 JDBC 连接池的实现。常用的 JDBC 连接池有：

- HikariCP
- C3P0
- BoneCP
- **Druid**

> 推荐使用Druid，因为国内用的多



目前使用最广泛的是 HikariCP。我们以 HikariCP 为例，要使用 JDBC 连接池，先添加 HikariCP 的依赖如下：

```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>2.7.1</version>
</dependency>
```

紧接着，我们需要创建一个`DataSource`实例，这个实例就是连接池：

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/test");
config.setUsername("root");
config.setPassword("password");
config.addDataSourceProperty("connectionTimeout", "1000"); // 连接超时：1秒
config.addDataSourceProperty("idleTimeout", "60000"); // 空闲超时：60秒
config.addDataSourceProperty("maximumPoolSize", "10"); // 最大连接数：10
DataSource ds = new HikariDataSource(config);
```

注意创建 `DataSource` 也是一个非常昂贵的操作，所以通常 `DataSource` 实例总是作为一个全局变量存储，并贯穿整个应用程序的生命周期。

有了连接池以后，我们如何使用它呢？和前面的代码类似，只是获取 `Connection` 时，把`DriverManage.getConnection()` 改为 `ds.getConnection()`：

```java
try (Connection conn = ds.getConnection()) { // 在此获取连接
    ...
} // 在此“关闭”连接
```



### 事务与Batch

#### 事务

数据库事务（Transaction）是由若干个SQL语句构成的一个操作序列，有点类似于 Java 的 `synchronized` 同步。具有ACID 特性：

- Atomicity：原子性
- Consistency：一致性
- Isolation：隔离性
- Durability：持久性 



SQL标准定义了**4种隔离级别**，分别对应可能出现的数据不一致的情况：

| Isolation Level  | 脏读（Dirty Read） | 不可重复读（Non Repeatable Read） | 幻读（Phantom Read） |
| :--------------- | :----------------- | :-------------------------------- | :------------------- |
| Read Uncommitted | Yes                | Yes                               | Yes                  |
| Read Committed   | -                  | Yes                               | Yes                  |
| Repeatable Read  | -                  | -                                 | Yes                  |
| Serializable     | -                  | -                                 | -                    |



要在JDBC中执行事务，本质上就是如何把多条SQL包裹在一个数据库事务中执行。我们来看JDBC的事务代码：

```java
Connection conn = openConnection();
try {
    // 关闭自动提交:
    conn.setAutoCommit(false);
    
    // 执行多条SQL语句:
    insert(); update(); delete();
    
    // 提交事务:
    conn.commit();
} catch (SQLException e) {
    // 回滚事务:
    conn.rollback();
} finally {    
    conn.setAutoCommit(true);    
    conn.close();
}
```



如果要设定事务的隔离级别，可以使用如下代码：

```java
// 设定隔离级别为READ COMMITTED:
conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
```

如果没有调用上述方法，那么会使用数据库的默认隔离级别。MySQL 的默认隔离级别是 `REPEATABLE READ`。



#### Batch

使用 JDBC 的 batch 操作会大大提高执行效率，对**内容相同，参数不同的SQL**，要优先考虑 batch 操作。

把同一个 SQL 但参数不同的若干次操作合并为一个 batch 执行。我们以**批量插入**为例，示例代码如下：

```java
try (PreparedStatement ps = conn.prepareStatement("INSERT INTO students (name, gender, grade, score) VALUES (?, ?, ?, ?)")) {
    // 对同一个PreparedStatement反复设置参数并调用addBatch():
    for (String name : names) {
        ps.setString(1, name);
        ps.setBoolean(2, gender);
        ps.setInt(3, grade);
        ps.setInt(4, score);
        
        ps.addBatch(); // 添加到batch
    }
    
    // 执行batch:
    int[] ns = ps.executeBatch();
    for (int n : ns) {
        System.out.println(n + " inserted."); // batch中每个SQL执行的结果数量
    }
}
```

- 执行 batch 和执行一个 SQL 不同点在于，需要对同一个 `PreparedStatement` 反复设置参数并调用 `addBatch()`，这样就相当于给一个 SQL 加上了多组参数，相当于变成了“多行” SQL

- 第二个不同点是调用的不是 `executeUpdate()`，而是 `executeBatch()`，因为我们设置了多组参数，相应地，返回结果也是多个 `int `值，因此返回类型是 `int[]`，循环 `int[]` 数组即可获取每组参数执行后影响的结果数量

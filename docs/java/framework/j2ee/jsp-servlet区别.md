Servlet 编程是纯粹的 java 编程，而 jsp 则是 html 和 java 编程的中庸形式。**jsp 就是在 html 里面写 java 代码，servlet 就是在 java 里面写 html 代码**。其实 jsp 经过容器解释之后就是 servlet。只是我们自己写代码的时候尽量能让它们各司其职，jsp 更注重前端显示，servlet 更注重模型和业务逻辑。不要写出万能的 jsp 或 servlet 来即可。




#### 不同之处

- Servlet 在 Java 代码中通过 HttpServletResponse 对象动态输出 HTML 内容

- JSP 在静态 HTML 内容中嵌入 Java 代码，Java 代码被动态执行后生成 HTML 内容

    

#### 各自的特点

- Servlet 能够很好地组织业务逻辑代码，但是在 Java 源文件中通过字符串拼接的方式生成动态 HTML 内容会导致代码维护困难、可读性差

- JSP 虽然规避了 Servlet 在生成 HTML 内容方面的劣势，但是在 HTML 中混入大量、复杂的业务逻辑同样也是不可取的

    


#### 通过MVC双剑合璧
既然 JSP 和 Servlet 都有自身的适用环境，那么能否扬长避短，让它们发挥各自的优势呢？
答案是肯定的——MVC(Model-View-Controller)模式非常适合解决这一问题。

MVC模式（Model-View-Controller）是软件工程中的一种软件架构模式，把软件系统分为三个基本部分：模型（Model）、视图（View）和控制器（Controller）：

- Controller——负责转发请求，对请求进行处理

- View——负责界面显示

- Model——业务功能编写（例如算法实现）、数据库设计以及数据存取操作实现


![在这里插入图片描述](https://img-note.langyastudio.com/202111091458695.png?x-oss-process=style/watermark)



#####  在 JSP/Servlet 开发的软件系统中，这三个部分的描述如下所示：

Web 浏览器发送 HTTP 请求到服务端，被 Controller(Servlet) 获取并进行处理（例如参数解析、请求转发）
Controller(Servlet) 调用核心业务逻辑—— Model 部分，获得结果
Controller(Servlet) 将逻辑处理结果交给 View（JSP），动态输出 HTML 内容
动态生成的 HTML 内容返回到浏览器显示

MVC 模式在 Web 开发中的好处是非常明显，它规避了 JSP 与 Servlet 各自的短板，Servlet 只负责业务逻辑而不会通过 out.append() 动态生成 HTML 代码；JSP 中也不会充斥着大量的业务代码。这大大提高了代码的可读性和可维护性。

> `Spring Boot` 有多流行，JSP 就有多落寞，这两者的流行程度成反比。Spring Boot 支持 JSP，但是有一些限制。例如内嵌容器用 Jetty 和 Tomcat 就只能打成 War 包，不能用 Undertow 。其实，Spring Boot 最佳的使用方式就是不用 JSP，而是通过 Restfull API 和前端框架配合

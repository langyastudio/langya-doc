> 先看 [spring boot 2.x properties 配置文件详解](https://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484692&idx=1&sn=d4133a364c5121e51844258499be7b10&chksm=c03a57b5f74ddea350b4fcc5f0c52cdc1211cff7941b6a1634a45c2a6b099804a5c81689fa02&scene=21#wechat_redirect)
>
> 基于上述代码修改
>
> 日志本质上属于 jdk 的范畴

日志是进行调试和分析的重要工具。Logback、Log4j2 等都可以作为日志提供者。



## JDK Logging

Java 标准库内置了日志包 `java.util.logging` 。

定义了 7 个日志级别，默认级别是 INFO，从严重到普通：

- SEVERE
- WARNING
- INFO
- CONFIG
- FINE
- FINER
- FINEST



**缺点：** **难用！**

- Logging 系统在 JVM 启动时读取配置文件并完成初始化，一旦开始运行 `main()`方法，就无法修改配置

- 配置修改需要在 JVM 启动时传递参数 `-Djava.util.logging.config.file=<config-file-name>`

  缺省的配置文件是 JRE 中 `lib/logging.properties` 或 jdk 下`conf/logging.properties`文件

- 如果想生成文件，需要修改配置文件的配置项为 `handlers= java.util.logging.FileHandler` or `handlers= java.util.logging.FileHandler, java.util.logging.ConsoleHandler`



## Commons Logging

Commons Logging 是 **Apache 创建的日志模块**，它可以挂接不同的日志系统，并通过配置文件指定挂接的日志系统。即 Commons Logging 相当于提供标准的 **API 接口**，再由日志系统提供方去实现这些接口。

Commons Logging 默认情况下自动搜索并使用 Log4j，如果没有找到 Log4j，再使用 JDK Logging。所以这种自动搜索的机制与简洁的 API 接口导致 Commons Logging 日志模块使用非常广泛。



 Commons Logging 定义了6个日志级别：默认级别是`INFO`。

- FATAL
- ERROR
- WARNING
- INFO
- DEBUG
- TRACE



日志使用只有两步：

- 通过 `LogFactory` 获取 `Log` 类的实例

- 使用 `Log` 实例的方法打日志

```java
public class Main {
    public static void main(String[] args) {
        Log log = LogFactory.getLog(Main.class);
        log.info("start...");
        log.warn("end.");
    }
}
```



### **SLF4J 是什么**

- 对 Commons Logging 的 API 接口不满意，有人就搞了 `SLF4J`

- 对 Log4j 的性能不满意，有人就搞了 `Logback`



## Log4j2 使用

通过 Commons Logging 实现日志，只需要进行相关配置，不需要修改代码即可使用 Log4j。

**Log4j 的架构大致如下：**

```ascii
log.info("User signed in.");
 │
 │   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
 ├──>│ Appender │───>│  Filter  │───>│  Layout  │───>│ Console  │
 │   └──────────┘    └──────────┘    └──────────┘    └──────────┘
 │
 │   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
 ├──>│ Appender │───>│  Filter  │───>│  Layout  │───>│   File   │
 │   └──────────┘    └──────────┘    └──────────┘    └──────────┘
 │
 │   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
 └──>│ Appender │───>│  Filter  │───>│  Layout  │───>│  Socket  │
     └──────────┘    └──────────┘    └──────────┘    └──────────┘
```

当我们使用 Log4j 输出一条日志时，Log4j 自动通过不同的 Appender 把同一条日志输出到不同的目的地。例如：

- console：输出到屏幕
- file：输出到文件
- socket：通过网络输出到远程计算机



**pom.xml 引入 log4j2**

由于 **spring boot 2.x 默认使用 slf4j 作为日志的 API 接口**，所以需要排除默认的日志自动配置并引入 `spring-boot-starter-log4j2`

```xml
<!--spring log4j-->
<!--https://docs.spring.io/spring-boot/docs/1.5.19.RELEASE/reference/htmlsingle/#howto-configure-log4j-for-logging-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>

<!-- 排除 Spring-boot-starter 默认的日志配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```



**定义配置文件**

在 `resources` 根目录下创建 `log4j2-spring.xml` 配置文件，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--monitorInterval：Log4j2 能够自动检测修改的配置文件并重新加载配置，设置检测的间隔秒数-->
<Configuration monitorInterval="5">
    <!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL-->
    <Properties>
        <!-- 定义日志格式 -->
        <Property name="log.pattern">%d{yyyy-MM-dd HH:mm:ss.SSS} %highlight{%-5level}[%thread] %style{%logger{36}}{cyan} : %msg%n</Property>

        <!-- 定义文件名变量 -->
        <!-- 定义日志存储的路径，不要配置相对路径 -->
        <property name="FILE_PATH" value="logs/" />
        <property name="FILE_NAME" value="log" />
    </Properties>

    <!-- 定义Appender，即目的地 -->
    <Appenders>
        <!-- 定义输出到控制台 -->
        <Console name="console" target="SYSTEM_OUT">
            <!-- 日志格式引用上面定义的log.pattern -->
            <PatternLayout pattern="${log.pattern}" disableAnsi="false" noConsoleNoAnsi="false"/>

            <!--控制台只输出level及其以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="DEBUG" onMatch="ACCEPT" onMismatch="DENY"/>
        </Console>

        <!-- 这个会打印出所有的error及以下级别的信息，
            每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <RollingFile name="err" bufferedIO="true" fileName="${FILE_PATH}/error.log" filePattern="${FILE_PATH}/${FILE_NAME}-error-%d{yyyy-MM-dd}_%i.log.gz">
            <PatternLayout pattern="${log.pattern}" disableAnsi="false" noConsoleNoAnsi="false"/>
            <!--只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>

            <Policies>
                <!--interval属性用来指定多久滚动一次，默认是1 hour-->
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                <!-- 根据文件大小自动切割日志 -->
                <SizeBasedTriggeringPolicy size="20MB" />
            </Policies>
            <!-- 保留最近20份 -->
            <DefaultRolloverStrategy max="20" />
        </RollingFile>
    </Appenders>

    <Loggers>
        <!--监控系统信息-->
        <!--若是additivity设为false，则子Logger只会在自己的appender里输出，而不会在父Logger的appender里输出。-->
        <Logger name="org.springframework.web" level="info" additivity="false">
            <AppenderRef ref="console"/>
        </Logger>

        <!--日志配置-->
        <Root level="info">
            <!-- 对info级别的日志，输出到console -->
            <AppenderRef ref="console" level="DEBUG" />
            <!-- 对error级别的日志，输出到err，即上面定义的RollingFile -->
            <AppenderRef ref="err" level="error" />
        </Root>
    </Loggers>
</Configuration>
```

这里面核心的是配置了 spring 的日志:

```xml
<Logger name="org.springframework.web" level="info" additivity="false">
    <AppenderRef ref="console"/>
</Logger>
```

表示对 `org.springframework.web` 包按照 info 的级别输出日志到控制台。



**使用日志**

使用 `lombok` 插件，通过 `@Log4j2` 注解后，内部直接使用 `log.xx` 写日志，是不是超简单！示例如下：

```java
import lombok.extern.log4j.Log4j2;

@Log4j2
@Component
public class ShutDownHook
{  
    public void onApplicationClosed()
    {        
        log.info("后台-系统已关闭");
        ....
    }
}
```



> 源代码：https://github.com/langyastudio/langya-tech/tree/springboot/log

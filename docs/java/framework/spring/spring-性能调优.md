### 增加内嵌 Tomcat 的最大连接数

```java
@Configuration  
public class TomcatConfig {  
    @Bean  
    public ConfigurableServletWebServerFactory webServerFactory() {  
        TomcatServletWebServerFactory tomcatFactory = new TomcatServletWebServerFactory();  
        tomcatFactory.addConnectorCustomizers(new MyTomcatConnectorCustomizer());  
        tomcatFactory.setPort(8005);  
        tomcatFactory.setContextPath("/api-g");  
        return tomcatFactory;  
    }  
    class MyTomcatConnectorCustomizer implements TomcatConnectorCustomizer {  
        public void customize(Connector connector) {  
            Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();  
            //设置最大连接数  
            protocol.setMaxConnections(20000);  
            //设置最大线程数  
            protocol.setMaxThreads(2000);  
            protocol.setConnectionTimeout(30000);  
        }  
    }  
  
}  
```



### 默认 Tomcat 容器改为 Undertow

默认 Tomcat 容器改为 Undertow（Jboss 下的服务器，Tomcat 吞吐量 5000，Undertow 吞吐量 8000）

```xml
<exclusions>  
  <exclusion>  
     <groupId>org.springframework.boot</groupId>  
     <artifactId>spring-boot-starter-tomcat</artifactId>  
  </exclusion>  
</exclusions>  

<dependency>  
  <groupId>org.springframework.boot</groupId>  
  <artifactId>spring-boot-starter-undertow</artifactId>  
</dependency>  
```


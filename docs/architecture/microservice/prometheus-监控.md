## 应用接入

- 添加依赖包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

- 修改 application.yml 文件

```yml
#endpoint
management:
  endpoint:
    #显示健康具体信息，默认不会显示详细信息
    health:
      show-details: always
    prometheus:
      enabled: true
  metrics:
    export:
      prometheus:
        enabled: true
        #step: 1ms
        descriptions: true
    tags:
      application: ${spring.application.name}
  endpoints:
    web:
      exposure:
        # 暴露所有节点
        #'prometheus,health'
        include: '*'
        exclude: "shutdown"
  health:
    db:
      enabled: false
    elasticsearch:
      enabled: false
```

- 解决缺失 jvm 信息的问题

需要配置扫描该类所在的包

```java
import io.micrometer.prometheus.PrometheusMeterRegistry;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 解决 Prometheus 无法获取 jvm 信息的问题
 */
@Configuration
public class ActuatorMetricsConfig {
    @Bean
    InitializingBean forcePrometheusPostProcessor(BeanPostProcessor meterRegistryPostProcessor, PrometheusMeterRegistry registry) {
        return () -> meterRegistryPostProcessor.postProcessAfterInitialization(registry, "");
    }
}
```

- 访问 `ip:port/xx/actuator/prometheus` 即可访问监控信息

![image-20221228194512287](https://img-note.langyastudio.com/202212281945546.png?x-oss-process=style/watermark)
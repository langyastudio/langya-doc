### 依赖库

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.5</version>
</dependency>
```



### 连接池配置

```yml
spring:
  datasource:
    url: jdbc:mysql://192.168.123.22:3306/hc_opens
    username: root
    password: daemon
    driver-class-name: org.mariadb.jdbc.Driver
    
    druid:    
	  # 初始化发生在显示调用init方法，或者第一次getConnection时
      initial-size: 10
      #最大连接池数量
      max-active: 20
      #最小连接池数量
      min-idle: 10
      #获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。建议配置2000ms
      max-wait: 2000
      #是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql5.5以下的版本中没有PSCache功能，建议关闭掉。5.5及以上版本有PSCache，建议开启。
      #pool-prepared-statements:
      #要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100
      #max-pool-prepared-statement-per-connection-size:
      #用来检测连接是否有效的sql，要求是一个查询语句，常用select 'x'。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会起作用。
      validation-query: SELECT 1
      #validation-query-timeout:
      #申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。建议值：false
      test-on-borrow: false
      #归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。建议值：false
      test-on-return: false
      #建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
      test-while-idle: true
      #1) Destroy线程定时监测的间隔， Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于minEvictableIdleTimeMillis则关闭物理连接。2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明
      time-between-eviction-runs-millis: 60000
      #连接保持空闲而不被驱逐的最长时间。建议值：5* timeBetweenEvictionRunsMillis
      min-evictable-idle-time-millis: 300000
      #max-evictable-idle-time-millis:
      #属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有：监控统计用的filter:stat，日志用的filter:log4j， 防御sql注入的filter:wall
      filters: stat,wall  
      #建立新连接时将发送到JDBC驱动程序的连接属性。字符串的格式必须为[propertyName = property;] *注 - “用户”和“密码”属性将被明确传递，因此不需要在此处包含。
      connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=2000 
     
```



- 配置项说明

| 配置                          | 缺省值             | 说明                                                         |
| ----------------------------- | ------------------ | ------------------------------------------------------------ |
| name                          |                    | 配置这个属性的意义在于，如果存在多个数据源，监控的时候可以通过名字来区分开来。  如果没有配置，将会生成一个名字，格式是："DataSource-" + System.identityHashCode(this) |
| jdbcUrl                       |                    | 连接数据库的url，不同数据库不一样。例如：  mysql : jdbc:mysql://10.20.153.104:3306/druid2  oracle : jdbc:oracle:thin:@10.20.149.85:1521:ocnauto |
| username                      |                    | 连接数据库的用户名                                           |
| password                      |                    | 连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter。详细看这里：https://github.com/alibaba/druid/wiki/%E4%BD%BF%E7%94%A8ConfigFilter |
| driverClassName               | 根据url自动识别    | 这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName(建议配置下) |
| initialSize                   | 0                  | 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时 |
| maxActive                     | 8                  | 最大连接池数量                                               |
| maxIdle                       | 8                  | 已经不再使用，配置了也没效果                                 |
| minIdle                       |                    | 最小连接池数量                                               |
| maxWait                       |                    | 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |
| poolPreparedStatements        | false              | 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。 |
| maxOpenPreparedStatements     | -1                 | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100 |
| validationQuery               |                    | 用来检测连接是否有效的sql，要求是一个查询语句。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会其作用。 |
| testOnBorrow                  | true               | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testOnReturn                  | false              | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能 |
| testWhileIdle                 | false              | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。 |
| timeBetweenEvictionRunsMillis |                    | 有两个含义：  1) Destroy线程会检测连接的间隔时间 2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明 |
| minEvictableIdleTimeMillis    |                    |                                                              |
| connectionInitSqls            |                    | 物理连接初始化的时候执行的sql                                |
| exceptionSorter               | 根据dbType自动识别 | 当数据库抛出一些不可恢复的异常时，抛弃连接                   |
| filters                       |                    | 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有：  监控统计用的filter:stat日志用的filter:log4j防御sql注入的filter:wall |
| proxyFilters                  |                    | 类型是List<com.alibaba.druid.filter.Filter>，如果同时配置了filters和proxyFilters，是组合关系，并非替换关系 |



### 监控配置

- 通过 yml 配置文件配置

```yml
spring:
  datasource:
    ...
    druid:    
	  ...
      
      #-------------------------------------------------------------------------------
      # 要打开Druid监控页面需要做以下配置
      #-------------------------------------------------------------------------------
      stat-view-servlet:
        enabled: true # 这个一定要加，不然【http://localhost:port/xx/druid/index.html】页面打不开
        url-pattern: '/druid/*'
        allow: 127.0.0.1
        deny: 
        login-username: admin
        login-password: 123456
        reset-enable: false
      web-stat-filter:
        enabled: true # 这个一定要加，不然【http://localhost:port/xx/druid/index.html】页面打不开
        url-pattern: '/*'
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'
        session-stat-enable: true      
```



- 通过代码配置

**需要注入 `DataSource` 的 Bean**

```java
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * druid 数据库连接池的监控配置类
 */
@Configuration
public class DruidConfig {
    /**
     * 监控台的 servlet
     * 返回StatViewServlet
     */
    @Bean
    public ServletRegistrationBean<StatViewServlet> statViewServletRegistrationBean() {
        ServletRegistrationBean<StatViewServlet> registrationBean = new ServletRegistrationBean<>();
        registrationBean.setServlet(new StatViewServlet());
        registrationBean.addUrlMappings("/druid/*");

        // 添加IP白名单
        registrationBean.addInitParameter("allow", "");
        // 添加IP黑名单，当白名单和黑名单重复时，黑名单优先级更高
        registrationBean.addInitParameter("deny", "127.0.0.1");

        // 添加控制台管理用户
        registrationBean.addInitParameter("loginUsername", "admin");
        registrationBean.addInitParameter("loginPassword", "123456");

        // 是否能够重置数据
        registrationBean.addInitParameter("resetEnable", "false");

        return registrationBean;
    }

    /**
     * 配置服务过滤器：监控哪些访问
     * @return 返回过滤器配置对象
     */
    @Bean
    public FilterRegistrationBean<WebStatFilter> webStatFilterRegistrationBean() {
        FilterRegistrationBean<WebStatFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new WebStatFilter());

        // 添加过滤规则
        registrationBean.addUrlPatterns("/*");
        // 忽略过滤格式
        registrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");

        registrationBean.addInitParameter("sessionStatEnable", "true");

        return registrationBean;
    }
}
```



### 访问 Druid Monitor

通过 http://ip:port/xx/druid/index.html 即可访问 Druid 的监控页面

![image-20230104150333611](https://img-note.langyastudio.com/202301041503789.png?x-oss-process=style/watermark)






## 简介

dynamic-datasource-spring-boot-starter 是一个基于 springboot 的快速集成多数据源的启动器。

其支持 **Jdk 1.7+, SpringBoot 1.4.x 1.5.x 2.x.x**。

### 特性

- 支持 **数据源分组** ，适用于多种场景 纯粹多库 读写分离 一主多从 混合模式。
- 支持数据库敏感配置信息 **加密** ENC()。
- 支持每个数据库独立初始化表结构schema和数据库database。
- 支持无数据源启动，支持懒加载数据源（需要的时候再创建连接）。
- 支持 **自定义注解** ，需继承DS(3.2.0+)。
- 提供并简化对Druid，HikariCp，BeeCp，Dbcp2的快速集成。
- 提供对Mybatis-Plus，Quartz，ShardingJdbc，P6sy，Jndi等组件的集成方案。
- 提供 **自定义数据源来源** 方案（如全从数据库加载）。
- 提供项目启动后 **动态增加移除数据源** 方案。
- 提供Mybatis环境下的 **纯读写分离** 方案。
- 提供使用 **spel动态参数** 解析数据源方案。内置spel，session，header，支持自定义。
- 支持 **多层数据源嵌套切换** 。（ServiceA >>> ServiceB >>> ServiceC）。
- 提供 **基于seata的分布式事务方案。**
- 提供 **本地多数据源事务方案。** 附：不能和原生spring事务混用。



### 约定

1. 本框架只做 **切换数据源** 这件核心的事情，并**不限制你的具体操作**，切换了数据源可以做任何CRUD。
2. 配置文件所有以下划线 `_` 分割的数据源 **首部** 即为组的名称，相同组名称的数据源会放在一个组下。
3. 切换数据源可以是组名，也可以是具体数据源名称。组名则切换时采用负载均衡算法切换。
4. 默认的数据源名称为 **master** ，你可以通过 `spring.datasource.dynamic.primary` 修改。
5. 方法上的注解优先于类上注解。
6. DS支持继承抽象类上的DS，暂不支持继承接口上的DS。



### 快速入门

> 示例：https://github.com/dynamic-datasource/dynamic-datasource-samples/tree/master/datasource-samples

- 引入dynamic-datasource-spring-boot-starter

```xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
  <version>${version}</version>
</dependency>
```

- 配置数据源

```yaml
spring:
  datasource:
    dynamic:
      primary: master #设置默认的数据源或者数据源组,默认值即为master
      strict: false #严格匹配数据源,默认false. true未匹配到指定数据源时抛异常,false使用默认数据源
      datasource:
        master:
          url: jdbc:mysql://xx.xx.xx.xx:3306/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver # 3.2.0开始支持SPI可省略此配置
        slave_1:
          url: jdbc:mysql://xx.xx.xx.xx:3307/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver
        slave_2:
          url: ENC(xxxxx) # 内置加密,使用请查看详细文档
          username: ENC(xxxxx)
          password: ENC(xxxxx)
          driver-class-name: com.mysql.jdbc.Driver
          
       #......省略
       #以上会配置一个默认库master，一个组slave下有两个子库slave_1,slave_2
# 多主多从                      纯粹多库（记得设置primary）                   混合配置
spring:                               spring:                               spring:
  datasource:                           datasource:                           datasource:
    dynamic:                              dynamic:                              dynamic:
      datasource:                           datasource:                           datasource:
        master_1:                             mysql:                                master:
        master_2:                             oracle:                               slave_1:
        slave_1:                              sqlserver:                            slave_2:
        slave_2:                              postgresql:                           oracle_1:
        slave_3:                              h2:                                   oracle_2:
```

- 使用 **@DS** 切换数据源

**@DS** 可以注解在方法上或类上，**同时存在就近原则 方法上注解 优先于 类上注解**。

|     注解      |                   结果                   |
| :-----------: | :--------------------------------------: |
|    没有@DS    |                默认数据源                |
| @DS("dsName") | dsName可以为组名也可以为具体某个库的名称 |

```java
@Service
@DS("slave")
public class UserServiceImpl implements UserService {

  @Autowired
  private JdbcTemplate jdbcTemplate;

  public List selectAll() {
    return  jdbcTemplate.queryForList("select * from user");
  }
  
  @Override
  @DS("slave_1")
  public List selectByCondition() {
    return  jdbcTemplate.queryForList("select * from user where age >10");
  }
}
```

> DS 注解是基于 AOP 的原理实现的，aop 的常见失效场景应清楚。 比如内部调用失效，shiro 代理失效。 具体见切换数据源失败。通常建议 DS 放在 serviceImpl 的方法上，如事务注解一样

![img](https://img-note.langyastudio.com/202210201650489.png?x-oss-process=style/watermark)



## 连接池集成

- 每个数据源都有一个 type 来指定连接池

- 每个数据源甚至可以使用不同的连接池，如无特殊需要并不建议

- type **不是必填字段** ，在没有设置 type 的时候系统会自动按以下顺序查找并使用连接池
  Druid > HikariCp > BeeCp > DBCP2 > Spring Basic

```yaml
spring:
  datasource:
    dynamic:
      primary: db1 
      datasource:
        db1:
          url: jdbc:mysql://xx.xx.xx.xx:3306/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver
          type: com.zaxxer.hikari.HikariDataSource #使用Hikaricp
        db2:
          url: jdbc:mysql://xx.xx.xx.xx:3307/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver
          type: com.alibaba.druid.pool.DruidDataSource #使用Druid
        db3:
          url: jdbc:mysql://xx.xx.xx.xx:3308/dynamic
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver
          type: cn.beecp.BeeDataSource #使用beecp
```



### 集成 Druid

Druid Github https://github.com/alibaba/druid

Druid 文档 https://github.com/alibaba/druid/wiki



- 项目引入 `druid-spring-boot-starter` 依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>${version}</version>
</dependency>
```



- 排除原生 Druid 的快速配置类

注意：**v3.3.3 及以上**版本不用排除了

```java
@SpringBootApplication(exclude = DruidDataSourceAutoConfigure.class)
public class Application {

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }
}
```

某些 springBoot 的版本上面可能无法排除可用以下方式排除。

```yaml
spring:
  autoconfigure:
    exclude: com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceAutoConfigure
```



- 参数配置

  - 如果参数都未配置，则保持原组件默认值

  - 如果配置了全局参数，则每一个数据源都会继承对应参数

  - 每一个数据源可以单独设置参数覆盖全局参数

```yaml
spring:
  datasource:
    druid:
      stat-view-servlet:
        enabled: true
        loginUsername: admin
        loginPassword: 123456
    dynamic:
      druid: #以下是支持的全局默认值
        initial-size:
        max-active:
        min-idle:
        max-wait:
        time-between-eviction-runs-millis:
        time-between-log-stats-millis:
        stat-sqlmax-size:
        min-evictable-idle-time-millis:
        max-evictable-idle-time-millis:
        test-while-idle:
        test-on-borrow:
        test-on-return:
        validation-query:
        validation-query-timeout:
        use-global-datasource-stat:
        async-init:
        clear-filters-enable:
        reset-stat-enable:
        not-full-timeout-retry-count:
        max-wait-thread-count:
        fail-fast:
        phyTimeout-millis:
        keep-alive:
        pool-prepared-statements:
        init-variants:
        init-global-variants:
        use-unfair-lock:
        kill-when-socket-read-timeout:
        connection-properties:
        max-pool-prepared-statement-per-connection-size:
        init-connection-sqls:
        share-prepared-statements:
        connection-errorretry-attempts:
        break-after-acquire-failure:
        filters: stat # 注意这个值和druid原生不一致，默认启动了stat
        wall:
            noneBaseStatementAllow:
            callAllow:
            selectAllow:
            selectIntoAllow:
            selectIntoOutfileAllow:
            selectWhereAlwayTrueCheck:
            selectHavingAlwayTrueCheck:
            selectUnionCheck:
            selectMinusCheck:
            selectExceptCheck:
            selectIntersectCheck:
            createTableAllow:
            dropTableAllow:
            alterTableAllow:
            renameTableAllow:
            hintAllow:
            lockTableAllow:
            startTransactionAllow:
            blockAllow:
            conditionAndAlwayTrueAllow:
            conditionAndAlwayFalseAllow:
            conditionDoubleConstAllow:
            conditionLikeTrueAllow:
            selectAllColumnAllow:
            deleteAllow:
            deleteWhereAlwayTrueCheck:
            deleteWhereNoneCheck:
            updateAllow:
            updateWhereAlayTrueCheck:
            updateWhereNoneCheck:
            insertAllow:
            mergeAllow:
            minusAllow:
            intersectAllow:
            replaceAllow:
            setAllow:
            commitAllow:
            rollbackAllow:
            useAllow:
            multiStatementAllow:
            truncateAllow:
            commentAllow:
            strictSyntaxCheck:
            constArithmeticAllow:
            limitZeroAllow:
            describeAllow:
            showAllow:
            schemaCheck:
            tableCheck:
            functionCheck:
            objectCheck:
            variantCheck:
            mustParameterized:
            doPrivilegedAllow:
            dir:
            tenantTablePattern:
            tenantColumn:
            wrapAllow:
            metadataAllow:
            conditionOpXorAllow:
            conditionOpBitwseAllow:
            caseConditionConstAllow:
            completeInsertValuesCheck:
            insertValuesCheckSize:
            selectLimit:
        stat:
          merge-sql:
          log-slow-sql:
          slow-sql-millis: 
      datasource:
        master:
          username: root
          password: 123456
          driver-class-name: com.mysql.jdbc.Driver
          url: jdbc:mysql://xx.xx.xx.xx:3306/dynamic?characterEncoding=utf8&useSSL=false
          druid: # 以下是独立参数，每个库可以重新设置
            initial-size: 20
            validation-query: select 1 FROM DUAL #比如oracle就需要重新设置这个
            public-key: #（非全局参数）设置即表示启用加密,底层会自动帮你配置相关的连接参数和filter，推荐使用本项目自带的加密方法。

# 生成 publickey 和密码，推荐使用本项目自带的加密方法。
# java -cp druid-1.1.10.jar com.alibaba.druid.filter.config.ConfigTools youpassword
```

如上即可配置访问用户和密码，访问 http://localhost:8080/druid/index.html 查看 druid 监控



- 核心源码

  - Druid 数据源创建器

    https://github.com/baomidou/dynamic-datasource-spring-boot-starter/blob/master/src/main/java/com/baomidou/dynamic/datasource/creator/DruidDataSourceCreator.java

  - Druid 参数源码

    https://github.com/baomidou/dynamic-datasource-spring-boot-starter/blob/master/src/main/java/com/baomidou/dynamic/datasource/spring/boot/autoconfigure/druid/DruidConfig.java



## 第三方集成

### 集成 Mybatis-Plus

- MybatisPlus Github https://github.com/baomidou/mybatis-plus
- MybatisPlus 文档 https://mybatis.plus/

> 只要引入了 mybatisPlus 相关 jar 包，项目自动集成，兼容 mybatisPlus 2.x 和 3.x 的版本



- 项目引入 `mybatis-Plus` 依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>${version}</version>
</dependency>
```



- 使用 `DS` 注解进行切换数据源

```java
@Service
@DS("mysql")
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {

    @DS("oracle")
    publid void addUser(User user){
        //do something
        baseMapper.insert(user);
    }
}
```



- 分页配置

```java
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    
    //如果是不同类型的库，请不要指定DbType，其会自动判断。
    interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
    
   // interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
    return interceptor;
}
```



- 注意事项

mp 内置的 ServiceImpl 在新增,更改,删除等一些方法上自带事务导致不能切换数据源

**解决办法:**

1. 复制 ServiceImpl 出来为自己的 MyServiceImpl，并去掉所有事务注解
2. 创建一个新方法，并在此方法上加 DS 注解. 如上面的 `addUser` 方法



- 为什么要单独拿出来说和 mybatisPlus 的集成

因为 mybatisPlus 重写了一些核心类，必须通过解析获得真实的代理对象。

如果自己写多数据源，则很难完成与 mp 的集成。

核心解析源码 https://github.com/baomidou/dynamic-datasource-spring-boot-starter/blob/master/src/main/java/com/baomidou/dynamic/datasource/support/DataSourceClassResolver.java



### 集成 ShardingJdbc

- shardingsphere Github https://github.com/apache/shardingsphere
- shardingsphere 文档 https://shardingsphere.apache.org/
- shardingsphere 示例 https://github.com/apache/shardingsphere/tree/master/examples

------

多数据源与 shardingsphere 集成的场景：部分表比较大需要分表由 shardingsphere 完成，同时又有多库的需求。

可参考文章 https://zhuanlan.zhihu.com/p/166105923



- 项目引入 `shardingsphere` 依赖

```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>${version}</version>
</dependency>
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-namespace</artifactId>
    <version>${version}</version>
</dependency>
```



- 分别配置 shardingjdbc 和多数据源

```yaml
spring:
  # shardingjdbc 配置
  shardingsphere:
    datasource:
      names: shardingmaster,shardingslave0,shardingslave1
      shardingmaster:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: org.h2.Driver
        jdbc-url: jdbc:h2:mem:test
        username: sa
        password: ""
      shardingslave0:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: org.h2.Driver
        jdbc-url: jdbc:h2:mem:test
        username: sa
        password: ""
      shardingslave1:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: org.h2.Driver
        jdbc-url: jdbc:h2:mem:test
        username: sa
        password: ""
    masterslave:
      name: ms
      load-balance-algorithm-type: round_robin
      master-data-source-name: shardingmaster
      slave-data-source-names: shardingslave0,shardingslave1
    sharding:
      tables:
        t_order:
          actualDataNodes: shardingmaster.t_order${0..1}
          tableStrategy:
            inline:
              shardingColumn: order_id
              algorithmExpression: t_order${order_id % 2}
          keyGenerator:
            type: SNOWFLAKE
            column: order_id
            
  # 动态数据源配置
  datasource:
    dynamic:
      datasource:
        master:
          username: sa
          password: ""
          url: jdbc:h2:mem:master
          driver-class-name: org.h2.Driver
          schema: db/schema.sql
        test:
          username: sa
          password: ""
          url: jdbc:h2:mem:test
          driver-class-name: org.h2.Driver
          schema: db/schema.sql
```



- 通过自定义 DynamicDataSourceProvider 完成与 shardingsphere 的集成

```java
@Configuration
public class MyDataSourceConfiguration {

    @Resource
    private DynamicDataSourceProperties properties;

    /**
     * 分片: shardingDataSource
     * 主从: masterSlaveDataSource
     * 根据自己场景修改注入
     */
//    @Lazy 某些springBoot版本不加会报错（暂不清楚原理0 0）
    @Resource
    private MasterSlaveDataSource masterSlaveDataSource;

 //    @Lazy
//    @Resource
//    private ShardingDataSource shardingDataSource;

    @Bean
    public DynamicDataSourceProvider dynamicDataSourceProvider() {
        return new AbstractDataSourceProvider() {

            @Override
            public Map<String, DataSource> loadDataSources() {
                Map<String, DataSource> dataSourceMap = new HashMap<>();
                dataSourceMap.put("sharding", masterSlaveDataSource);
                
//下面的代码可以把 shardingJdbc 内部管理的子数据源也同时添加到动态数据源里 (根据自己需要选择开启+注解了@Lazy被代理了不可以)
                Map<String, DataSource> shardingInnerDataSources = masterSlaveDataSource.getDataSourceMap();
                dataSourceMap.putAll(shardingInnerDataSources);
                return dataSourceMap;
            }
        };
    }

    /**
     * 将动态数据源设置为首选的
     * 当spring存在多个数据源时, 自动注入的是首选的对象
     * 设置为主要的数据源之后，就可以支持shardingjdbc原生的配置方式了
     * 3.4.0版本及以上使用以下方式注入,老版本请阅读文档  进阶-手动注入多数据源
     */
    @Primary
    @Bean
    public DataSource dataSource() { 
        DynamicRoutingDataSource dataSource = new DynamicRoutingDataSource();
        
        dataSource.setPrimary(properties.getPrimary());
        dataSource.setStrict(properties.getStrict());
        dataSource.setStrategy(properties.getStrategy());
        dataSource.setP6spy(properties.getP6spy());
        dataSource.setSeata(properties.getSeata());
        
        return dataSource;
    }
}
```



### 集成 Quartz

- Quartz Github https://github.com/quartz-scheduler/quartz
- Quartz 文档 http://www.quartz-scheduler.org/

------

Quartz是一个定时任务框架，常常用于解决分布式系统下的定时任务协调问题。

Quartz常常需要独立运行在主业务数据库外,在springboot场景中可以以下面方式运行。



- 项目引用 `quartz-starter` 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```



- 配置数据源参数

根据需要可配置独立数据源的参数，或把 quartz 数据源也作为动态数据源的一个数据源。

一般来说二种方式根据需要选择一种即可，如果没有在动态数据源里需要切到 quartz 的的场景，建议久独立配吧。

```yaml
spring:
  datasource: #独立quartz配置
    username: root
    password: 123456
    url: jdbc:mysql://39.108.158.138:3306/quartz
    driver-class-name: com.mysql.cj.jdbc.Driver
    dynamic:
      datasource:
        master:
          username: sa
          password: ""
          url: jdbc:h2:mem:test
          driver-class-name: org.h2.Driver
        quartz:  #把quartz数据源也作为动态数据源的一个数据源
          username: root
          password: 123456
          url: jdbc:mysql://39.108.158.138:3306/quartz
          driver-class-name: com.mysql.cj.jdbc.Driver
  quartz:
    job-store-type: jdbc
    jdbc:
      initialize-schema: always
```



- 创建任务

创建一个每秒打印一次 hello world 的任务

```java
@Slf4j
public class HelloworldJob extends QuartzJobBean {

    private static int time = 0;

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        log.info("Hello world!:" + jobExecutionContext.getJobDetail().getKey() + "-" + (time++));
    }
}
```

对应的自动配置

```java
@Configuration
public class QuartzCommonAutoConfiguration {

    @Bean
    public JobDetail job() {
        return JobBuilder.newJob(HelloworldJob.class)
                .withIdentity("myJob1", "myJobGroup1")
                //JobDataMap可以给任务execute传递参数
                .usingJobData("job_param", "job_param1")
                .storeDurably()
                .build();
    }

    @Bean
    public Trigger myTrigger(JobDetail jobDetail) {
        return TriggerBuilder.newTrigger()
                .forJob(jobDetail)
                .withIdentity("myTrigger1", "myTriggerGroup1")
                .usingJobData("job_trigger_param", "job_trigger_param1")
                .startNow()
                .withSchedule(CronScheduleBuilder.cronSchedule("0/1 * * * * ? *"))
                .build();
    }
}
```



- 配置Quartz实际使用的数据源

具体方式有二种，随意选择一种即可。建议方式一，简单点。

**自定义 SchedulerFactoryBeanCustomizer**

```java
spring:
  datasource: #独立quartz配置
    username: root
    password: 123456
    url: jdbc:mysql://39.108.158.138:3306/quartz
    driver-class-name: com.mysql.cj.jdbc.Driver
        
@Configuration
public class MyQuartzAutoConfigurationMode1 {

    @Autowired
    private DataSourceProperties dataSourceProperties;

    @Order(1)
    @Bean
    public SchedulerFactoryBeanCustomizer schedulerFactoryBeanCustomizer() {
        DataSource dataSource = dataSourceProperties.initializeDataSourceBuilder().build();
        return schedulerFactoryBean -> {
            schedulerFactoryBean.setDataSource(dataSource);
            schedulerFactoryBean.setTransactionManager(new DataSourceTransactionManager(dataSource));
        };
    }

    //如果需要使用动态数据源里的某个数据源则打开以下配置，关闭上面配置。
//    @Order(1)
//    @Bean
//    public SchedulerFactoryBeanCustomizer schedulerFactoryBeanCustomizer(DataSource dataSource) {
//        DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
//        DataSource quartz = ds.getDataSource("quartz");
//        return schedulerFactoryBean -> {
//            schedulerFactoryBean.setDataSource(quartz);
//            schedulerFactoryBean.setTransactionManager(new DataSourceTransactionManager(quartz));
//        };
//    }

}
```

**使用 @QuartzDataSource 来指明 quartz 数据源**

```java
@Configuration
public class MyQuartzAutoConfigurationMode2 {

    @Autowired
    private DataSourceProperties dataSourceProperties;
    @Autowired
    private DynamicDataSourceProperties properties;

    //3.4.0版本及以上使用以下方式注入,老版本请阅读文档  进阶-手动注入多数据源
    @Primary
    @Bean
    public DataSource dataSource() {
        DynamicRoutingDataSource dataSource = new DynamicRoutingDataSource();
        dataSource.setPrimary(properties.getPrimary());
        dataSource.setStrict(properties.getStrict());
        dataSource.setStrategy(properties.getStrategy());
        dataSource.setP6spy(properties.getP6spy());
        dataSource.setSeata(properties.getSeata());
        return dataSource;
    }

    @QuartzDataSource
    @Bean
    public DataSource quartzDataSource() {
        return dataSourceProperties.initializeDataSourceBuilder().build();
    }

    //如果需要使用动态数据源里的某个数据源则打开以下配置，关闭上面配置。
//    @QuartzDataSource
//    @Bean
//    public DataSource quartzDataSource(DataSource dataSource) {
//        DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
//        return ds.getDataSource("quartz");
//    }
}
```



## 进阶使用

### 动态添加移除数据源

主要在多租户场景中，常常新的一个租户进来需要动态的添加一个数据源到库中，使得系统不用重启即可切换数据源。

```java
import com.baomidou.dynamic.datasource.DynamicRoutingDataSource;
import com.baomidou.dynamic.datasource.creator.*;
import com.baomidou.dynamic.datasource.spring.boot.autoconfigure.DataSourceProperty;
import com.baomidou.samples.ds.dto.DataSourceDTO;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import javax.sql.DataSource;
import java.util.Set;

@RestController
@RequestMapping("/datasources")
@Api(tags = "添加删除数据源")
public class DataSourceController {

    @Autowired
    private DataSource dataSource;
   // private final DataSourceCreator dataSourceCreator; //3.3.1及以下版本使用这个通用
    @Autowired
    private DefaultDataSourceCreator dataSourceCreator;
    @Autowired
    private BasicDataSourceCreator basicDataSourceCreator;
    @Autowired
    private JndiDataSourceCreator jndiDataSourceCreator;
    @Autowired
    private DruidDataSourceCreator druidDataSourceCreator;
    @Autowired
    private HikariDataSourceCreator hikariDataSourceCreator;
    @Autowired
    private BeeCpDataSourceCreator beeCpDataSourceCreator;
    @Autowired
    private Dbcp2DataSourceCreator dbcp2DataSourceCreator;

    @GetMapping
    @ApiOperation("获取当前所有数据源")
    public Set<String> now() {
        DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
        return ds.getDataSources().keySet();
    }

    //通用数据源会根据maven中配置的连接池根据顺序依次选择。
    //默认的顺序为druid>hikaricp>beecp>dbcp>spring basic
    @PostMapping("/add")
    @ApiOperation("通用添加数据源（推荐）")
    public Set<String> add(@Validated @RequestBody DataSourceDTO dto) {
        DataSourceProperty dataSourceProperty = new DataSourceProperty();
        BeanUtils.copyProperties(dto, dataSourceProperty);
        DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
        DataSource dataSource = dataSourceCreator.createDataSource(dataSourceProperty);
        ds.addDataSource(dto.getPoolName(), dataSource);
        return ds.getDataSources().keySet();
    }

    @PostMapping("/addBasic(强烈不推荐，除了用了马上移除)")
    @ApiOperation(value = "添加基础数据源", notes = "调用Springboot内置方法创建数据源，兼容1,2")
    public Set<String> addBasic(@Validated @RequestBody DataSourceDTO dto) {
        DataSourceProperty dataSourceProperty = new DataSourceProperty();
        BeanUtils.copyProperties(dto, dataSourceProperty);
        DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
        DataSource dataSource = basicDataSourceCreator.createDataSource(dataSourceProperty);
        ds.addDataSource(dto.getPoolName(), dataSource);
        return ds.getDataSources().keySet();
    }

    @PostMapping("/addJndi")
    @ApiOperation("添加JNDI数据源")
    public Set<String> addJndi(String pollName, String jndiName) {
        DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
        DataSource dataSource = jndiDataSourceCreator.createDataSource(jndiName);
        ds.addDataSource(poolName, dataSource);
        return ds.getDataSources().keySet();
    }

    @PostMapping("/addDruid")
    @ApiOperation("基础Druid数据源")
    public Set<String> addDruid(@Validated @RequestBody DataSourceDTO dto) {
        DataSourceProperty dataSourceProperty = new DataSourceProperty();
        BeanUtils.copyProperties(dto, dataSourceProperty);
        dataSourceProperty.setLazy(true);
        
        DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
        DataSource dataSource = druidDataSourceCreator.createDataSource(dataSourceProperty);
        ds.addDataSource(dto.getPoolName(), dataSource);
        
        return ds.getDataSources().keySet();
    }

    @PostMapping("/addHikariCP")
    @ApiOperation("基础HikariCP数据源")
    public Set<String> addHikariCP(@Validated @RequestBody DataSourceDTO dto) {
        DataSourceProperty dataSourceProperty = new DataSourceProperty();
        BeanUtils.copyProperties(dto, dataSourceProperty);
        dataSourceProperty.setLazy(true);//3.4.0版本以下如果有此属性，需手动设置，不然会空指针。
        DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
        DataSource dataSource = hikariDataSourceCreator.createDataSource(dataSourceProperty);
        ds.addDataSource(dto.getPoolName(), dataSource);
        return ds.getDataSources().keySet();
    }

    @PostMapping("/addBeeCp")
    @ApiOperation("基础BeeCp数据源")
    public Set<String> addBeeCp(@Validated @RequestBody DataSourceDTO dto) {
        DataSourceProperty dataSourceProperty = new DataSourceProperty();
        BeanUtils.copyProperties(dto, dataSourceProperty);
        dataSourceProperty.setLazy(true);//3.4.0版本以下如果有此属性，需手动设置，不然会空指针。
        DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
        DataSource dataSource = beeCpDataSourceCreator.createDataSource(dataSourceProperty);
        ds.addDataSource(dto.getPoolName(), dataSource);
        return ds.getDataSources().keySet();
    }

    @PostMapping("/addDbcp")
    @ApiOperation("基础Dbcp数据源")
    public Set<String> addDbcp(@Validated @RequestBody DataSourceDTO dto) {
        DataSourceProperty dataSourceProperty = new DataSourceProperty();
        BeanUtils.copyProperties(dto, dataSourceProperty);
        dataSourceProperty.setLazy(true);//3.4.0版本以下如果有此属性，需手动设置，不然会空指针。
        DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
        DataSource dataSource = dbcp2DataSourceCreator.createDataSource(dataSourceProperty);
        ds.addDataSource(dto.getPoolName(), dataSource);
        return ds.getDataSources().keySet();
    }

    @DeleteMapping
    @ApiOperation("删除数据源")
    public String remove(String name) {
        DynamicRoutingDataSource ds = (DynamicRoutingDataSource) dataSource;
        ds.removeDataSource(name);
        return "删除成功";
    }
}
```



### 动态解析数据源

- 默认有三个职责链来处理动态参数解析器 header->session->spel。所有以 `#` 开头的参数都会从参数中获取数据源。

```java
@DS("#session.tenantName")//从session获取
public List selectSpelBySession() {
	return userMapper.selectUsers();
}

@DS("#header.tenantName")//从header获取
public List selectSpelByHeader() {
	return userMapper.selectUsers();
}

@DS("#tenantName")//使用spel从参数获取
public List selectSpelByKey(String tenantName) {
	return userMapper.selectUsers();
}

@DS("#user.tenantName")//使用spel从复杂参数获取
public List selecSpelByTenant(User user) {
	return userMapper.selectUsers();
}
```



- 在Mapper接口下面的方法使用

```java
public interface UserMapper{
    // 前缀可以是p0,a0
    @DS("#p0.tenantName")
    public List selecSpelByTenant(User user);
}
```

对于在接口下面的使用, 由于编译器的默认配置没有将接口参数的元数据写入字节码文件中。
所以 spring el 会无法识别参数名称, 只能用默认的参数命名方式

1. 第一个参数: p0,a0,(加入-parameter后,可以使用参数具体的名字,例如这里的#user)
2. 第二个参数: p1,a1
3. 第三个参数: P2,a2

可以通过修改 maven 配置和 java 编译配置将接口参数信息写入字节码文件
maven 配置:

```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <!-- 想启用  <parameters>true</parameters> 的maven编译最低版本为:3.6.2 -->
        <version>3.6.2</version>
        <configuration>
            <source>${java.version}</source>
            <target>${java.version}</target>
            <parameters>true</parameters>
        </configuration>
    </plugin>
```

idea java编译配置: `-parameters`

> java 支持-parameters的最低版本为 1.8



### 数据库加密

支持 `url , username, password`  的加密

简单来说就是生成两把钥匙，私钥加密，公钥解密。 公钥可以发布出去，解密也是用的公钥

- 获得加密字符串

```java
import com.baomidou.dynamic.datasource.toolkit.CryptoUtils;

public class Demo {

    public static void main(String[] args) throws Exception {
        String password = "123456";
        //使用默认的publicKey ，建议还是使用下面的自定义
        String encodePassword = CryptoUtils.encrypt(password);
        System.out.println(encodePassword);
    }

        //自定义publicKey
    public static void main(String[] args) throws Exception {
        String[] arr = CryptoUtils.genKeyPair(512);
        System.out.println("privateKey:  " + arr[0]);
        System.out.println("publicKey:  " + arr[1]);
        System.out.println("url:  " + CryptoUtils.encrypt(arr[0], "jdbc:mysql://127.0.0.1:3306/order"));
        System.out.println("username:  " + CryptoUtils.encrypt(arr[0], "root"));
        System.out.println("password:  " + CryptoUtils.encrypt(arr[0], "123456"));
    }
}
```

![img](https://img.kancloud.cn/2c/ff/2cffd7b2b2b551c51d0554bc49b9c2e8_3034x306.png)



- 配置加密 yml

ENC(xxx)` 中包裹的xxx即为使用加密方法后生成的字符串

```java
spring:
  datasource:
    dynamic:
      public-key: #有默认值，强烈建议更换
      datasource:
        master:
          url: ENC(xxxxx)
          username: ENC(xxxxx)
          password: ENC(xxxxx)
          driver-class-name: com.mysql.jdbc.Driver
          public-key: #每个数据源可以独立设置，没有就继承上面的。
```



- 自定义解密

一些公司要求使用自己的方式加密，解密。从3.5.0版本开始，扩展了一个event。用户自行实现注入即可。

```java
public interface DataSourceInitEvent {

    /**
     * 连接池创建前执行（可用于参数解密）
     *
     * @param dataSourceProperty 数据源基础信息
     */
    void beforeCreate(DataSourceProperty dataSourceProperty);

    /**
     * 连接池创建后执行
     *
     * @param dataSource 连接池
     */
    void afterCreate(DataSource dataSource);
}
```

默认的实现是 EncDataSourceInitEvent，即 ENC 方式的。



### 启动初始化执行脚本

```yaml
spring:
  datasource:
    dynamic:
      primary: order
      datasource:
        master:
          # 基础配置省略...
          init:
              schema: db/order/schema.sql # 配置则生效,自动初始化表结构
              data: db/order/data.sql # 配置则生效,自动初始化数据
              continue-on-error: true # 默认true,初始化失败是否继续
              separator: ";" # sql默认分号分隔符，一般无需更改
```



### 手动切换数据源

- 在某些情况您可能需要手动切换数据源

```java
import com.baomidou.dynamic.datasource.toolkit.DynamicDataSourceContextHolder;

@Service
public class UserServiceImpl implements UserService {

  @Autowired
  private JdbcTemplate jdbcTemplate;

  public List selectAll() {
    DynamicDataSourceContextHolder.push("slave");//手动切换
    return  jdbcTemplate.queryForList("select * from user");
  }
 
}
```



- 需要注意的是手动切换的数据源，最好自己在合适的位置 调用 DynamicDataSourceContextHolder.clear() 清空当前线程的数据源信息。如果你不太清楚什么时候调用，那么可以参考下面写一个拦截器，注册进 spring 里即可。

```java
@Slf4j
public class DynamicDatasourceClearInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        DynamicDataSourceContextHolder.clear();
    }
}
```



### 手动注入多数据源

绝大部分是因为您的系统需要和其他数据源共同存在使用。如 Quartz 和 ShardingJdbc 等都需要使用独立的数据源。

```java
//3.4.0版本及以上
@Primary
@Bean
public DataSource dataSource() {
    DynamicRoutingDataSource dataSource = new DynamicRoutingDataSource();
    
    dataSource.setPrimary(properties.getPrimary());
    dataSource.setStrict(properties.getStrict());
    dataSource.setStrategy(properties.getStrategy());
    dataSource.setP6spy(properties.getP6spy());
    dataSource.setSeata(properties.getSeata());
    
    return dataSource;
}
```

主要变更是因为 3.4.0 支持了多个 provider 同时生效，采取了内部注入。 源码改动参考 https://github.com/baomidou/dynamic-datasource-spring-boot-starter/commit/6e8d2954499f83269302d23b58f8832c31e07ef7	



## 自定义

### 自定义注解

如果你只有几个确定的库，可以尝试自定义注解替换掉`@DS`。

建议从3.2.1版本开始使用自定义注解，另外组件自带了`@Master`和`@Slave`注解。

- 需要自己定义一个注解并继承自 DS

```java
import com.baomidou.dynamic.datasource.annotation.DS;
import java.lang.annotation.*;

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@DS("product")
public @interface Product {
}
```



- 注解在需要切换数据源的方法上或类上

```java
@Service
@Product
public class ProductServiceImpl implements ProductService {

    @Product
    public List selectAll() {
      return  jdbcTemplate.queryForList("select * from products");
    }
}
```



### 自定义数据源来源

- 基础介绍

数据源来源的默认实现是 `YmlDynamicDataSourceProvider`，其从 yaml 或 properties 中读取信息并解析出所有数据源信息

```java
public interface DynamicDataSourceProvider {

    /**
     * 加载所有数据源
     *
     * @return 所有数据源，key为数据源名称
     */
    Map<String, DataSource> loadDataSources();
}
```



- 自定义示例

可以参考 `AbstractJdbcDataSourceProvider` （仅供参考）来实现从 JDBC 数据库中获取数据库连接信息

```java
@Bean
public DynamicDataSourceProvider jdbcDynamicDataSourceProvider() {
    return new AbstractJdbcDataSourceProvider("org.h2.Driver", "jdbc:h2:mem:test", "sa", "") {
        @Override
        protected Map<String, DataSourceProperty> executeStmt(Statement statement) throws SQLException {
            ResultSet rs = statement.executeQuery("select * from DB");
            while (rs.next()) {
                String name = rs.getString("name");
                String username = rs.getString("username");
                String password = rs.getString("password");
                String url = rs.getString("url");
                String driver = rs.getString("driver");
                
                DataSourceProperty property = new DataSourceProperty();
                property.setUsername(username);
                property.setPassword(password);
                property.setUrl(url);
                property.setDriverClassName(driver);
                
                map.put(name, property);
            }
            
            return map;
        }
    };
}
```

>  从3.4.0开始，可以注入多个 DynamicDataSourceProvider 的 Bean 以实现同时从多个不同来源加载数据源，注意同名会被覆盖



### 自定义负载均衡策略

如下图`slave`组下有三个数据源，当用户使用`slave`切换数据源时会使用负载均衡算法。

系统自带了两个负载均衡算法

- `LoadBalanceDynamicDataSourceStrategy` 轮询,是默认的。
- `RandomDynamicDataSourceStrategy` 随机的。

```yaml
spring:
  datasource:
    dynamic:
      datasource:
        master:
          username: sa
          password: ""
          url: jdbc:h2:mem:test
          driver-class-name: org.h2.Driver
          schema: db/schema.sql
        slave_1:
          username: sa
          password: ""
          url: jdbc:h2:mem:test
          driver-class-name: org.h2.Driver
        slave_2:
          username: sa
          password: ""
          url: jdbc:h2:mem:test
          driver-class-name: org.h2.Driver
        slave_3:
          username: sa
          password: ""
          url: jdbc:h2:mem:test
          driver-class-name: org.h2.Driver
      strategy: com.baomidou.dynamic.datasource.strategy.LoadBalanceDynamicDataSourceStrategy
```



- 如果默认的两个都不能满足要求，可以参考源码自定义。 暂时只能全局更改

```java
import java.util.List;
import java.util.concurrent.ThreadLocalRandom;
import javax.sql.DataSource;

public class RandomDynamicDataSourceStrategy implements DynamicDataSourceStrategy {
    public RandomDynamicDataSourceStrategy() {
    }

    public DataSource determineDataSource(List<DataSource> dataSources) {
        return (DataSource)dataSources.get(ThreadLocalRandom.current().nextInt(dataSources.size()));
    }
}
```



## 无注解方案

```java
public class DynamicRoutingDataSource {
@Override
public DataSource determineDataSource() {
    //这里是核心，是从ThreadLocal中获取当前数据源。
    String dsKey = DynamicDataSourceContextHolder.peek();
    return getDataSource(dsKey);
}
```

所以我们就可以根据需求，选择合适的时机调用DynamicDataSourceContextHolder.push("对应数据源")。

默认的`@DS`注解来切换数据源是根据spring AOP的特性，在方法开启前设置数据源KEY，方法执行完成后移除对应数据源KEY。



### filter 切换数据源

拦截以 filter/** 开头的所有请求，如果后面的方法以a开头就切换到db1，以b开头就切换到db2，其余使用默认数据源。

实现如下：

```java
@Slf4j
@WebFilter(filterName = "dsFilter", urlPatterns = {"/filter/*"})
public class DynamicDatasourceFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("loading filter {}", filterConfig.getFilterName());
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;

        String requestURI = request.getRequestURI();
        log.info("经过多数据源filter,当前路径是{}", requestURI);
//        String headerDs = request.getHeader("ds");
//        Object sessionDs = request.getSession().getAttribute("ds");
        String s = requestURI.replaceFirst("/filter/", "");

        String dsKey = "master";
        if (s.startsWith("a")) {
            dsKey = "db1";
        } else if (s.startsWith("b")) {
            dsKey = "db2";
        } else {

        }

        //执行
        try {
            DynamicDataSourceContextHolder.push(dsKey);
            filterChain.doFilter(servletRequest, servletResponse);
        } finally {
            DynamicDataSourceContextHolder.poll();
        }
    }

    @Override
    public void destroy() {

    }
}

@SpringBootApplication
@ServletComponentScan //filter必须使用这个
@MapperScan("com.baomidou.samples.ds.mapper")
public class AllDataSourceApplication {

    public static void main(String[] args) {
        SpringApplication.run(AllDataSourceApplication.class, args);
    }
}
```



### intercepror 切换数据源

拦截以 filter/** 开头的所有请求，如果后面的方法以a开头就切换到db1，以b开头就切换到db2，其余使用默认数据源。

实现如下：

```java
import com.baomidou.dynamic.datasource.toolkit.DynamicDataSourceContextHolder;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Slf4j
public class DynamicDatasourceInterceptor implements HandlerInterceptor {

    /**
     * 在请求处理之前进行调用（Controller方法调用之前）
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String requestURI = request.getRequestURI();
        log.info("经过多数据源Interceptor,当前路径是{}", requestURI);
//        String headerDs = request.getHeader("ds");
//        Object sessionDs = request.getSession().getAttribute("ds");
        String s = requestURI.replaceFirst("/interceptor/", "");

        String dsKey = "master";
        if (s.startsWith("a")) {
            dsKey = "db1";
        } else if (s.startsWith("b")) {
            dsKey = "db2";
        } else {

        }

        DynamicDataSourceContextHolder.push(dsKey);
        return true;
    }

    /**
     * 请求处理之后进行调用，但是在视图被渲染之前（Controller方法调用之后）
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {

    }

    /**
     * 在整个请求结束之后被调用，也就是在DispatcherServlet 渲染了对应的视图之后执行（主要是用于进行资源清理工作）
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        DynamicDataSourceContextHolder.clear();
    }
}

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MyWebAutoConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new DynamicDatasourceInterceptor()).addPathPatterns("/interceptor/**");
    }
}
```



### mybatis 下读写分离

场景：

- 在纯的读写分离环境，写操作全部是 master，读操作全部是 slave

- 不想通过注解配置完成以上功能

答：在 mybatis 环境下可以基于 mybatis 插件结合本数据源完成以上功能。

- 手动注入插件

```java
@Bean
public MasterSlaveAutoRoutingPlugin masterSlaveAutoRoutingPlugin(){
    return new MasterSlaveAutoRoutingPlugin();
}
```

默认主库名称 master, 从库名称 slave

```java
@Intercepts({@Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}),
        @Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class})})
@Slf4j
public class MasterSlaveAutoRoutingPlugin implements Interceptor {

    private static final String MASTER = "master";
    private static final String SLAVE = "slave";

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        Object[] args = invocation.getArgs();
        MappedStatement ms = (MappedStatement) args[0];
        try {
            DynamicDataSourceContextHolder.push(SqlCommandType.SELECT == ms.getSqlCommandType() ? SLAVE : MASTER);
            return invocation.proceed();
        } finally {
            DynamicDataSourceContextHolder.clear();
        }
    }

    @Override
    public Object plugin(Object target) {
        return target instanceof Executor ? Plugin.wrap(target, this) : target;
    }

    @Override
    public void setProperties(Properties properties) {
    }
}
```



## 事务

问：使用了事务如@Transational 无法切换数据源？

答： 是的，本组件是基于 springAop 的方案来进行多数据源的管理和切换的，要想保证**多个库的整体事务则需要分布式事务**。

问：为什么使用了事务如 @Transational 就无法切换数据源？

答：开启了事务后，**spring 事务管理器会保证在事务下整个线程后续拿到的都是同一个 connection**。



### 事务概念

- 原生 JDBC 处理事务

```java
try{
    connection.setAutoCommit( false);  
      //这里用connection对数据库做了一系列操作，CRUD
    connection.commit();//没有异常就提交
}catch(Exception ex){
    connection.rollback();  //异常回滚
}finally{  
    connection.setAutoCommit( true);
}
```

以上是事务的核心，所有操作要一起成功，只要出错就回滚。



- Spring 处理事务

spring 开启事务很简单，在需要事务的方法和类上添加 `@Transactional` 注解。

```java
@Transactional
public void test() {
      Aservice.dosometing();
      Bservice.dosometing();
}
```

使用了 `@Transactional`，**spring会保证整个事务下都复用同一个 connection**



### 本地事务

- 背景

多数据源事务方案一直是一个难题，通常的解决方案有以下二种

1. 利用 atomiks 手动构建多数据源事务，适合数据源较少，配置的参数也不太多,性能要求不高的项目。难点就是手动配置量大，需要耗费一定时间。
2. 用 seata 类似的分布式事务解决方案,难点就是需要搭建维护如 seata-server 的统一管理中心。

每一种方案都有其适用场景，本项目作者常常收到的问题就是

1. 为什么我加了事务注解，切换数据源失败？
2. 我了解涉及了分布式事务了，但我不想用 seata。我场景简单，有没有不依赖第三方的方案？



- 基础介绍

自从3.3.0开始，由 seata 的核心贡献者 https://github.com/a364176773 贡献了基于 connection 代理的方案。
建议从 3.4.0 版本开始使用，其修复了一个功能，老版本不加 `@DS` 只加 `@DSTransactional` 会报错。



- 注意事项

本地事务实现很简单，就是循环提交，发生错误，循环回滚。 我们默认的前提是数据库本身不会异常，比如宕机。
如**数据在回滚的过程突然宕机，本地事务就会有问题**。如果你需要完整分布式方案请使用 seata 方案。

1. **不支持 spring 原生事务，千万不能混用**
2. 只适合简单本地多数据源场景， 如果涉及异步和微服务等完整事务场景，请使用 seata 方案
3. 暂时不支持更多配置，如只读，如 spring 的传播特性。 后续会根据反馈考虑支持



- 使用方法

在最外层的方法添加 `@DSTransactional`，底下调用的各个类该切数据源就正常使用 `DS `切换数据源即可。 就是这么简单。~

```java
//如AService调用BService和CService的方法，A,B,C分别对应不同数据源。

public class AService {
    
    @DS("a")//如果a是默认数据源则不需要DS注解。
    @DSTransactional
    public void dosomething(){
        BService.dosomething();
        CService.dosomething();
    }
}

public class BService {    
    @DS("b")
    public void dosomething(){
        //dosomething
    }
}

public class CService {    
    @DS("c")
    public void dosomething(){
        //dosomething
    }
}
```

只要 `@DSTransactional` 注解下任一环节发生异常，则全局多数据源事务回滚。
如果 B C 上也有 `@DSTransactional` 会有影响吗？答：没有影响的。



- 示例项目

https://github.com/dynamic-datasource/dynamic-datasource-samples/tree/master/tx-samples/tx-local-sample

完整示例项目 数据库都已准备好，可以直接运行测试。http://localhost:8080/doc.html



### seata 事务

- seata Github地址 https://github.com/seata/seata
- seata 文档 https://seata.io/zh-cn/docs/overview/what-is-seata.html
- seata 示例 https://github.com/seata/seata-samples

> 一般需要分布式事务的场景大多数都是微服务化，个人并不建议在单体项目引入多数据源+分布式事务，有能力尽早拆开，可为过度方案。



#### 注意事项

`dynamic-datasource-sring-boot-starter` 组件内部开启 seata 后会自动使用 DataSourceProxy 来包装 DataSource,所以需要以下方式来保持兼容。

1.如果你引入的是 seata-all, 请不要使用 @EnableAutoDataSourceProxy 注解

2.如果你引入的是 seata-spring-boot-starter 请关闭自动代理

```yaml
seata:
  enable-auto-data-source-proxy: false
```



#### 示例项目

https://github.com/dynamic-datasource/dynamic-datasource-samples/tree/master/tx-samples/tx-seata-sample

此工程为 多数据源+druid+seata+mybatisPlus 的版本。

模拟用户下单，扣商品库存，扣用户余额，初步可分为订单服务+商品服务+用户服务。

- 环境准备

为了快速演示相关环境都采用 docker 部署，生产上线请参考 seata 官方文档使用。

1. 准备seata-server。

```bash
docker run --name seata-server -p 8091:8091 -d seataio/seata-server
```

1. 准备mysql数据库，账户root密码123456。

```bash
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

1. 创建相关数据库。

创建 `seata-order``seata-product``seata-account` 模拟连接不同的数据库。

```bash
CREATE DATABASE IF NOT EXIST seata-order;
CREATE DATABASE IF NOT EXIST seata-product;
CREATE DATABASE IF NOT EXIST seata-account;
```

1. 准备相关数据库脚本。

每个数据库下脚本相关的表，seata需要undo_log来监测和回滚。

相关的脚本不用自行准备，本工程已在resources/db下面准备好，另外配合多数据源的自动执行脚本功能，应用启动后会自动执行。



- 工程准备

1. 引入相关依赖，seata+druid+mybatisPlus+dynamic-datasource+mysql+lombok。

```xml
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
    <version>1.4.2</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
    <version>3.4.0</version>
</dependency>
# 省略，查看示例项目
```

1. 编写相关yaml配置

```yaml
spring:
  application:
    name: dynamic
  datasource:
    dynamic:
      primary: order
      strict: true
      seata: true    #开启seata代理，开启后默认每个数据源都代理，如果某个不需要代理可单独关闭
      seata-mode: AT #支持XA及AT模式,默认AT
      datasource:
        order:
          username: root
          password: 123456
          url: jdbc:mysql://39.108.158.138:3306/seata_order?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true&useSSL=false
          driver-class-name: com.mysql.cj.jdbc.Driver
          schema: classpath:db/schema-order.sql
        account:
          username: root
          password: 123456
          url: jdbc:mysql://39.108.158.138:3306/seata_account?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true&useSSL=false
          driver-class-name: com.mysql.cj.jdbc.Driver
          schema: classpath:db/schema-account.sql
        product:
          username: root
          password: 123456
          url: jdbc:mysql://39.108.158.138:3306/seata_product?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true&useSSL=false
          driver-class-name: com.mysql.cj.jdbc.Driver
          schema: classpath:db/schema-product.sql
        test:
          username: sa
          password: ""
          url: jdbc:h2:mem:test
          driver-class-name: org.h2.Driver
          seata: false #这个数据源不需要seata
seata:
  enabled: true
  application-id: applicationName
  tx-service-group: my_test_tx_group
  enable-auto-data-source-proxy: false   #一定要是false
  service:
    vgroup-mapping:
      my_test_tx_group: default  #key与上面的tx-service-group的值对应
    grouplist:
      default: 39.108.158.138:8091 #seata-server地址仅file注册中心需要
  config:
    type: file
  registry:
    type: file
```



- 代码编写

参考工程下面的代码完成 controller,service,maaper,entity,dto 等。

订单服务

```java
@Slf4j
@Service
public class OrderServiceImpl implements OrderService {

  @Resource
  private OrderDao orderDao;
  @Autowired
  private AccountService accountService;
  @Autowired
  private ProductService productService;
  
  @DS("order")//每一层都需要使用多数据源注解切换所选择的数据库
  @Override
  @Transactional
  @GlobalTransactional //重点 第一个开启事务的需要添加seata全局事务注解
  public void placeOrder(PlaceOrderRequest request) {
    log.info("=============ORDER START=================");
    Long userId = request.getUserId();
    Long productId = request.getProductId();
    Integer amount = request.getAmount();
    log.info("收到下单请求,用户:{}, 商品:{},数量:{}", userId, productId, amount);

    log.info("当前 XID: {}", RootContext.getXID());

    Order order = Order.builder()
        .userId(userId)
        .productId(productId)
        .status(OrderStatus.INIT)
        .amount(amount)
        .build();

    orderDao.insert(order);
    log.info("订单一阶段生成，等待扣库存付款中");
    // 扣减库存并计算总价
    Double totalPrice = productService.reduceStock(productId, amount);
    // 扣减余额
    accountService.reduceBalance(userId, totalPrice);

    order.setStatus(OrderStatus.SUCCESS);
    order.setTotalPrice(totalPrice);
    orderDao.updateById(order);
    log.info("订单已成功下单");
    log.info("=============ORDER END=================");
  }
}
```

商品服务

```java
@Slf4j
@Service
public class ProductServiceImpl implements ProductService {

  @Resource
  private ProductDao productDao;

  /**
   * 事务传播特性设置为 REQUIRES_NEW 开启新的事务  重要！！！！一定要使用REQUIRES_NEW
   */
  @DS("product")
  @Transactional(propagation = Propagation.REQUIRES_NEW)
  @Override
  public Double reduceStock(Long productId, Integer amount) {
    log.info("=============PRODUCT START=================");
    log.info("当前 XID: {}", RootContext.getXID());

    // 检查库存
    Product product = productDao.selectById(productId);
    Integer stock = product.getStock();
    log.info("商品编号为 {} 的库存为{},订单商品数量为{}", productId, stock, amount);

    if (stock < amount) {
      log.warn("商品编号为{} 库存不足，当前库存:{}", productId, stock);
      throw new RuntimeException("库存不足");
    }
    log.info("开始扣减商品编号为 {} 库存,单价商品价格为{}", productId, product.getPrice());
    // 扣减库存
    int currentStock = stock - amount;
    product.setStock(currentStock);
    productDao.updateById(product);
    double totalPrice = product.getPrice() * amount;
    log.info("扣减商品编号为 {} 库存成功,扣减后库存为{}, {} 件商品总价为 {} ", productId, currentStock, amount, totalPrice);
    log.info("=============PRODUCT END=================");
    return totalPrice;
  }
}
```

用户服务

```java
@Slf4j
@Service
public class AccountServiceImpl implements AccountService {

  @Resource
  private AccountDao accountDao;

  /**
   * 事务传播特性设置为 REQUIRES_NEW 开启新的事务    重要！！！！一定要使用REQUIRES_NEW
   */
  @DS("account")
  @Override
  @Transactional(propagation = Propagation.REQUIRES_NEW)
  public void reduceBalance(Long userId, Double price) {
    log.info("=============ACCOUNT START=================");
    log.info("当前 XID: {}", RootContext.getXID());

    Account account = accountDao.selectById(userId);
    Double balance = account.getBalance();
    log.info("下单用户{}余额为 {},商品总价为{}", userId, balance, price);

    if (balance < price) {
      log.warn("用户 {} 余额不足，当前余额:{}", userId, balance);
      throw new RuntimeException("余额不足");
    }
    log.info("开始扣减用户 {} 余额", userId);
    double currentBalance = account.getBalance() - price;
    account.setBalance(currentBalance);
    accountDao.updateById(account);
    log.info("扣减用户 {} 余额成功,扣减后用户账户余额为{}", userId, currentBalance);
    log.info("=============ACCOUNT END=================");
  }

}
```



- 测试

在 schema 自动执行的脚本里，默认设置了商品价格为10，商品总数量为20，用户余额为50。

启动项目后通过命令行执行。

1. 模拟正常下单，买一个商品

```bash
curl -X POST \
  http://localhost:8080/order/placeOrder \
  -H 'Content-Type: application/json' \
  -d '{
    "userId": 1,
    "productId": 1,
    "amount": 1
}'
```

1. 模拟库存不足，事务回滚

```bash
curl -X POST \
  http://localhost:8080/order/placeOrder \
  -H 'Content-Type: application/json' \
  -d '{
    "userId": 1,
    "productId": 1,
    "amount": 22
}'
```

1. 模拟用户余额不足，事务回滚

```bash
curl -X POST \
  http://localhost:8080/order/placeOrder \
  -H 'Content-Type: application/json' \
  -d '{
    "userId": 1,
    "productId": 1,
    "amount": 6
}'
```

注意观察运行日志，至此分布式事务集成案例全流程完毕。



## 常见问题

### 切换数据源失败

- 开启了spring的事务

原因： spring开启事务后会维护一个ConnectionHolder，保证在整个事务下，都是用同一个数据库连接。

请检查整个调用链路涉及的类的方法和类本身还有继承的抽象类上是否有`@Transactional`注解。

如强烈需要事务保证多个库同时执行成功或者失败，请查看事务专栏的解决办法。



- 方法内部调用

查看以下示例 回答 外部调用 `userservice.test1()` 能在执行到 `test2()` 切换到second数据源吗？

```java
public UserService {

    @DS("first")
    public void test1() {
        // do something
         test2();
    }

    @DS("second")
    public void test2() {
        // do something
    }
}
```



- PostConstruct 初始化顺序

初始化包括: @PostConstruct 注解, InitializingBean 接口, 自定义 init-method

```java
@Component
public class MyConfiguration {
    @Resource
    private UserMapper userMapper;
    
    @DS("slave")
    @PostConstruct
    public void init(){
        
        // 无法选择正确的数据源
        userMapper.selectById(1);
    }
}
```

解决方法：监听容器启动完成事件, 在容器完成后做初始化

```java
@Component
public class MyConfiguration {

    @DS("slave")
    @EventListener
    public void onApplicationEvent(ContextRefreshedEvent event) {
        // 成功选择正确的数据源
        userMapper.selectById(1);
    }
}
```

> 相关spring源码 : `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean
>
> 在初始化完成后, bean 进入增强阶段, 所以这个阶段的任何AOP都是无效的



- @Async 或者 java8 的 ParallelStream 并行流之类方法。

  这种情况都是新开了线程去处理，不受当前线程管控了。 可以在新开的方法上加对应的 DS 注解解决



### 事务常见问题

- 原生Spring `@Transational` 能和 `@DS` 一起使用吗？

```java
public class AService {
    
    @DS("a")
    @Transational
    public void dosomething(){
    //some code
    }
}
```

可以的，其会先切换使用数据源a再开启事务，整个原生事务内部不管是注解切换还是手动调用代码切换都不能切换，会一直使用a数据源。所以确认整个事务下不再切换其他数据源，用原生 `@Transational` 是建议的，毕竟其更完善。



- 本地事务和Spring原生事务不能混用是什么意思

场景一：先使用的 Spring `@Transational` 内部方法调用了本地 `@DSTransactional`。

```java
public class AService {
    
    @DS("a")
    @Transational
    public void dosomething(){
        Bservice.dosomething(); //B是另外数据源，然后注解了@DS("b")和@DSTransactional
        Cservice.dosomething(); //C是另外数据源，然后注解了@DS("c")和@DSTransactional
    }
}
```

基于核心知识： 整理事务下都是a数据源，内部无论做什么都改变不了使用a数据源。

场景二： 先使用的本地 `@DSTransactional` 内部方法调用了 Spring `@Transational` 。

```java
public class AService {
    
    @DS("a")
    @DSTransactional
    public void dosomething(){
        Amapper.updateSomeThing();
        Bservice.dosomething(); //B是另外数据源，然后注解了@DS("b")和@Transational
        Cservice.dosomething(); //C是另外数据源，然后注解了@DS("b")和@Transational
    }
}
```

B 和 C 都是独立的事务。C 发生错误 B 会回滚吗？不会 B 已经提交了。A 会回滚吗？会。

不建议混用，除非你确实非常理解事务，能随心所欲掌握你代码的执行流程。



- 本地事务标准使用

```java
public class AService {
    
    @DS("a")
    @DSTransactional//最外层开启即可
    public void dosomething(){
        Amapper.updateSomeThing();
        Bservice.dosomething(); //B是另外数据源，然后注解了@DS("b")
        Cservice.dosomething(); //C是另外数据源，然后注解了@DS("b")
    }
}
```



## 调试源码

- 开启动态数据源的debug日志

```yaml
logging:
  level:
    com.baomidou.dynamic: debug
```



- 检查日志输出是否正确

断点调试 DynamicDataSourceAnnotationInterceptor

```java
public class DynamicDataSourceAnnotationInterceptor implements MethodInterceptor {

    private static final String DYNAMIC_PREFIX = "#";

    private final DataSourceClassResolver dataSourceClassResolver;
    private final DsProcessor dsProcessor;

    public DynamicDataSourceAnnotationInterceptor(Boolean allowedPublicOnly, DsProcessor dsProcessor) {
        dataSourceClassResolver = new DataSourceClassResolver(allowedPublicOnly);
        this.dsProcessor = dsProcessor;
    }

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        try {
            //这里把获取到的数据源标识如master存入本地线程
            String dsKey = determineDatasourceKey(invocation);
            
            DynamicDataSourceContextHolder.push(dsKey);
            return invocation.proceed();
        } finally {
            DynamicDataSourceContextHolder.poll();
        }
    }

    private String determineDatasourceKey(MethodInvocation invocation) {
        String key = dataSourceClassResolver.findDSKey(invocation.getMethod(), invocation.getThis());
        return (!key.isEmpty() && key.startsWith(DYNAMIC_PREFIX)) ? dsProcessor.determineDatasource(invocation, key) : key;
    }
}
```



- 断点调试 DynamicRoutingDataSource

```java
public class DynamicRoutingDataSource extends AbstractRoutingDataSource {

    private Map<String, DataSource> dataSourceMap = new LinkedHashMap<>();

    private Map<String, DynamicGroupDataSource> groupDataSources = new ConcurrentHashMap<>();

    @Override
    public DataSource determineDataSource() {
        //从本地线程获取key解析最终真实的数据源
    String dsKey = DynamicDataSourceContextHolder.peek();
        return getDataSource(dsKey);
    }

    private DataSource determinePrimaryDataSource() {
        log.debug("从默认数据源中返回数据");
        return groupDataSources.containsKey(primary) ? groupDataSources.get(primary).determineDataSource() : dataSourceMap.get(primary);
    }

    public DataSource getDataSource(String ds) {
        if (StringUtils.isEmpty(ds)) {
            return determinePrimaryDataSource();
        } else if (!groupDataSources.isEmpty() && groupDataSources.containsKey(ds)) {
            log.debug("从 {} 组数据源中返回数据源", ds);
            return groupDataSources.get(ds).determineDataSource();
        } else if (dataSourceMap.containsKey(ds)) {
            log.debug("从 {} 单数据源中返回数据源", ds);
            return dataSourceMap.get(ds);
        }
        return determinePrimaryDataSource();
    }
}
```
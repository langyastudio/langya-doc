> 源码：[https://github.com/langyastudio/langya-tech/tree/springboot/cache](https://github.com/langyastudio/langya-tech/tree/springboot/cache)



## MySQL 查询缓存

> 来自：[https://mp.weixin.qq.com/s/LZBctWNWi3qehb-dgUCmxQ](https://mp.weixin.qq.com/s/LZBctWNWi3qehb-dgUCmxQ)

MySQL 的 `QueryCache` 缓存的是 SQL 语句文本以及对应的结果集。 

`QueryCache` 版本里程：

- 4.0 推出
- 5.6 默认禁用
- 5.7 deprecated
- 8.0 Removed 



### **QueryCache 介绍**

MySQL 查询缓存存储 SELECT 语句的文本以及发送给客户机的结果集，如果再次执行相同的 SQL，Server 端将从查询缓存中检索结果返回给客户端，而不是再次解析执行 SQL，查询缓存在 session 之间共享，因此一个客户端生成的缓存结果集，可以响应另一个客户端执行同样的SQL。

![图片](https://img-note.langyastudio.com/20210910094349.webp?x-oss-process=style/watermark)



### **QueryCache 弃用原因**

`QueryCache` 比较适合更新不频繁的数据，如果数据每隔几秒钟更新一次或更加频繁，则查询缓存不合适。

- 缓存失效

  表数据频繁更新，导致 QC 失效

- 降低查询QPS

  **查询缓存使用单个互斥体来控制对缓存的访问**，实际上是给服务器 SQL 处理引擎强加了一个单线程网关，在查询 QPS 比较高的情况下，对数据库并发和处理能力都会降低很多

  同时查询缓存碎片化还会导致服务器的负载升高，影响数据库的稳定性

- 硬件发展

  MySQL存储目前都是SSD、ESSD，访问速度已经很快，启动 QC 用逻辑IO替代物理IO收益不是很大

- 数据库缓存

  在 MySQL 中，buffer pool 用来缓存 index page、data page，当 SQL 查询第一次访问的数据所在数据页不在 buffer pool 中，会从磁盘加载 buffer pool 返回给用户，下次执行 SQL 查询只是逻辑读，没有物理读，提高IO性能

- **Redis 缓存**

  对于频繁访问的数据，可以缓存到 Redis 中，来提高查询的 QPS，这是常用可靠方案



## MyBatis 缓存机制

### 一级缓存

MyBatis 内置了一个强大的**事务性查询缓存机制**，它可以非常方便地配置和定制。**默认情况下，只启用了本地的会话缓存**，它仅仅对一个会话 `SqlSession` 中的数据进行缓存，跨 `SqlSession` 是无效的。

一次数据库会话中，执行多次查询条件完全相同的 SQL，MyBatis 提供了一级缓存的方案优化这部分场景，如果是相同的SQL 语句，会优先命中一级缓存，避免直接对数据库进行查询，提高性能。

> 如果你需要关闭一级缓存的话，可以在 Mapper 映射文件中将 flushCache 属性设置为 true，这种做法只会针对单个 SQL 操作生效，如 <select ... flushCache="true" useCache="false"/>



**使用数据库连接池，一级缓存还有效吗？**

```java
@Autowired
UmsUserMapper umsUserMapper;
    
@GetMapping("/users")
public UmsUser users(@RequestParam(value = "user_name") String userName )
{
    umsUserMapper.selectByPrimaryKey(userName);
    return umsUserMapper.selectByPrimaryKey(userName);
}
```

如示例所示，在同一次请求中调用了两次 `selectByPrimaryKey` 操作，那么 SQL 语句执行 1 次还是 2 次呢？

答案是 1 次，因为如果使用了数据库连接池，如 `Druid`。在一次 API 请求中，通过 MyBastis 调用了多次的 SQL 查询，由于每次 SQL 查询都用了连接池中不同的 `SqlSession` 句柄，导致一级缓存失效。



### 二级缓存

MyBatis 一级缓存最大的共享范围就是一个 `SqlSession` 内部，那么如果多个 `SqlSession` 需要共享缓存，则需要开启二级缓存。**二级缓存的作用域是 namespace，也就是作用范围是同一个命名空间。**

**二级缓存是事务性的，需要使用 `@Transactional` 事务注解才能让二级缓存生效**。这意味着当 `SqlSession` 完成并提交时或是完成并回滚，但没有执行 flushCache=true 的 insert/delete/update 语句时，缓存会获得更新。

要启用全局的二级缓存，只需要在 Mapper 映射文件中添加一行：

```xml
<cache/>
```

这个简单语句的效果如下:

- 映射语句文件中的所有 select 语句的结果将会被缓存
- 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存
- 缓存会使用最近最少使用算法（Least Recently Used）算法来清除不需要的缓存
- 缓存不会定时进行刷新（也就是说，没有刷新间隔）
- 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用
- 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改

> 缓存只作用于 cache 标签所在的映射文件中的语句。如果你混合使用 Java API 和 XML 映射文件，在共用接口中的语句将不会被默认缓存。你需要使用 @CacheNamespaceRef 注解指定缓存作用域



这些属性可以通过 cache 元素属性进行修改。比如：

```xml
<cache  eviction="FIFO"  flushInterval="60000"  size="512"  readOnly="true"/>
```

这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。

**清除策略**

- `LRU` – 最近最少使用：移除最长时间不被使用的对象
- `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们
- `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象
- `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象

**刷新间隔**

flushInterval 可以被设置为任意的正整数，设置的值应该是一个以毫秒为单位的合理时间量。 默认情况是不设置，也就是没有刷新间隔，缓存仅仅会在调用语句时刷新。

**引用数目**

size 属性可以被设置为任意正整数，要注意欲缓存对象的大小和运行环境中可用的内存资源。默认值是 1024。

**只读**

readOnly 可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓存对象的相同实例。 因此这些对象不能被修改。这就提供了可观的性能提升。而可读写的缓存会（通过序列化）返回缓存对象的拷贝。速度上会慢一些，但是更安全，因此默认值是 false。



**cache-ref**

回想一下上一节的内容，对某一命名空间的语句，只会使用该命名空间的缓存进行缓存或刷新。 但你可能会想要在多个命名空间中共享相同的缓存配置和实例。要实现这种需求可以使用 cache-ref 元素来引用另一个缓存。

```xml
<cache-ref namespace="com.someone.application.data.SomeMapper"/>
```



### **使用自定义缓存**

除了上述自定义缓存的方式，也可以通过实现自定义的缓存，或为其他第三方缓存方案创建适配器来完全覆盖缓存行为：

```xml
<cache type="com.domain.something.MyCustomCache"/>
```

这个示例展示了如何使用一个自定义的缓存实现。type 属性指定的类必须实现 `org.apache.ibatis.cache.Cache` 接口，且提供一个接受 String 参数作为 id 的构造器。 这个接口是 MyBatis 框架中许多复杂的接口之一，但是行为却非常简单。

```java
public interface Cache {
  String getId();
  int getSize();
  void putObject(Object key, Object value);
  Object getObject(Object key);
  boolean hasKey(Object key);
  Object removeObject(Object key);
  void clear();
}
```

为了对缓存进行配置，只需要简单地在缓存实现中添加公有的 JavaBean 属性，然后通过 cache 元素传递属性值，例如下面的例子将在缓存实现上调用一个名为 `setCacheFile(String file)` 的方法：

```xml
<cache type="com.domain.something.MyCustomCache">
  <property name="cacheFile" value="/tmp/my-custom-cache.tmp"/>
</cache>
```

你可以使用所有简单类型作为 JavaBean 属性的类型，MyBatis 会进行转换。 你也可以使用占位符（如 `${cache.file}`），以便替换成在 [配置文件属性](https://mybatis.org/mybatis-3/zh/configuration.html#properties) 中定义的值。



从版本 3.4.2 开始，MyBatis 已经支持在所有属性设置完毕之后，调用一个初始化方法。如果想要使用这个特性，需要自定义缓存类里实现 `org.apache.ibatis.builder.InitializingObject` 接口。

```java
public interface InitializingObject {
  void initialize() throws Exception;
}
```

> **提示** 上一节中对缓存的配置（如清除策略、可读或可读写等），不能应用于自定义缓存



## Spring Cache

如果想让数据库的查询缓存在 session 之间共享，还不受事务限制，该如何实现呢？这里面提供一个简单的实现方案：

- 每张表的数据记录都有唯一的逻辑Id

  如 id 号、用户名等

- 逻辑Id作为缓存的 key

  获取信息时，缓存数据；更新或删除记录时，删除缓存数据

技术上使用 spring cache 缓存注解。从 Spring 3.1 开始引入了对 Cache 的支持，作用在方法上的。接下来介绍如何正确的使用缓存注解。



### 环境配置

> spring boot + Mybatis 基础环境的搭建可以查看：[https://mp.weixin.qq.com/s/4VZcYT4gktrmuHBeL96H9Q](https://mp.weixin.qq.com/s/4VZcYT4gktrmuHBeL96H9Q)

Maven 依赖

```xml
<!--spring redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!-- 对象池，使用redis时必须引入 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

配置 Redis

```ini
spring:
  redis:
    host: 192.168.123.22
    port: 6379
    password: xxxxxx
    timeout: 6000ms
    lettuce:
      pool:
        # 最大连接数
        max-active: 8
        # 最大阻塞等待时间(负数表示没限制)
        max-wait: 1000ms
        # 最大空闲
        max-idle: 8
        # 最小空闲
        min-idle: 0  
```



### Redis 缓存管理器

需要继承 `CachingConfigurerSupport` 并提供 `CacheManager`  的 JavaBean 实现。

 ```java
 /**
  * redis配置类
  *
  * 配置序列化方式以及缓存管理器
  */
 @Configuration
 @EnableCaching
 @AutoConfigureAfter(RedisAutoConfiguration.class)
 public class RedisConfig extends CachingConfigurerSupport
 {
     public final static String _PREFIX = "langya_";
 
     /**
      * redis template相关配置
      * 使redis支持插入对象
      *
      * @param factory
      *
      * @return 方法缓存 Methods the cache
      */
     @Bean
     public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory)
     {
         RedisTemplate<String, Object> template = new RedisTemplate<>();
         // 配置连接工厂
         template.setConnectionFactory(factory);
 
         //使用StringRedisSerializer来序列化和反序列化redis的key值
         template.setKeySerializer(new StringRedisSerializer());
         // 值采用json序列化
         template.setValueSerializer(this.fastJsonRedisSerializer(false));
 
         // 设置hash key 和value序列化模式
         template.setHashKeySerializer(new StringRedisSerializer());
         template.setHashValueSerializer(this.fastJsonRedisSerializer(false));
 
         template.afterPropertiesSet();
         return template;
     }
 
     private FastJsonRedisSerializer<Object> fastJsonRedisSerializer(boolean WriteClassName)
     {
         //使用FastJsonRedisSerializer来序列化和反序列化redis的value值
         FastJsonRedisSerializer<Object> fastJsonRedisSerializer = new FastJsonRedisSerializer<>(Object.class);
         FastJsonConfig                  fastJsonConfig          = fastJsonRedisSerializer.getFastJsonConfig();
 
         fastJsonConfig.setDateFormat("yyyy-MM-dd HH:mm:ss");
         fastJsonConfig.setSerializerFeatures(
                 SerializerFeature.PrettyFormat,
                 SerializerFeature.WriteMapNullValue,
                 SerializerFeature.WriteNullListAsEmpty,
                 SerializerFeature.WriteNullStringAsEmpty);
         fastJsonConfig.setCharset(StandardCharsets.UTF_8);
 
         fastJsonConfig.setFeatures(Feature.SupportAutoType);
 
         if(WriteClassName)
         {
             fastJsonConfig.setSerializerFeatures(SerializerFeature.WriteClassName);
         }       
 
         return fastJsonRedisSerializer;
     }
 
     /*===============================注解缓存失效时间配置=============================*/
     /**
      * springboot2.x 设置redis缓存失效时间(注解)：
      *
      * @CacheConfig(cacheNames = "db")
      *
      * @Cacheable 表明对应方法的返回结果可以被缓存。首次调用后，下次就从缓存中读取结果，方法不会再被执行了
      * @CachePut 更新缓存，方法每次都会执行
      * @CacheEvict 清除缓存，方法每次都会执行
      * ...等等
      * <p>
      * 因为主要的业务逻辑在服务层实现，一般会把缓存注解加在服务层的方法上
      */
     @Bean
     public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory)
     {
         return new RedisCacheManager(
                 RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory),
 
                 //3600秒 - // 默认策略，未配置的 key 会使用这个
                 this.getRedisCacheConfigurationWithTtl(3600),
                 this.getRedisCacheConfigurationMap()
         );
     }
 
     /**
      * 缓存的key是 包名+方法名+参数列表
      */
     @Override
     @Bean
     public KeyGenerator keyGenerator()
     {
         return (target, method, objects) -> {
             StringBuilder sb = new StringBuilder();
             sb.append(target.getClass().getName());
             sb.append("::" + method.getName() + ":");
             for (Object obj : objects)
             {
                 sb.append(obj.toString());
             }
 
             return sb.toString();
         };
     }
 
     /**
      * 对每个缓存空间应用不同的配置
      *
      * @return
      */
     private Map<String, RedisCacheConfiguration> getRedisCacheConfigurationMap()
     {
         Map<String, RedisCacheConfiguration> redisCacheConfigurationMap = new HashMap<>();
 
         //进行过期时间配置
         //db缓存2小时
         redisCacheConfigurationMap.put("db", this.getRedisCacheConfigurationWithTtl(7200));
 
         return redisCacheConfigurationMap;
     }
 
     /**
      * 生成一个默认配置，通过config对象即可对缓存进行自定义配置
      *
      * @param seconds 设置缓存的默认过期时间
      *
      * @return
      */
     private RedisCacheConfiguration getRedisCacheConfigurationWithTtl(Integer seconds)
     {
         RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
 
                 // 增加缓存前缀
         config = config.prefixCacheNameWith(_PREFIX)
                 // 设置缓存的默认过期时间
                 .entryTtl(Duration.ofSeconds(seconds))
                 // 设置 key为string序列化                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                 // 设置value为json序列化                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(fastJsonRedisSerializer(true)))
                 // 不缓存空值
                 //.disableCachingNullValues()
                 ;
 
         return config;
     }
 }
 ```



### @Cacheable 使用

表示对应方法的返回结果可以被缓存。首次调用后，下次就从缓存中读取结果，方法体不会再被执行了。

- CacheConfig

  `cacheNames` 定义了全局的**缓存空间**，表示该类方法上的缓存都属于该命名空间（`db`）。如果缓存 key 的值为 `langya`，则实际的 key 为 `db:langya` 。

- Cacheable

  `key = "#userName"` 表示使用方法的参数 `userName` 的值作为缓存的 key，可以使用 SpEL 语法。

  `unless = "#result == null"` 排除法，表示只有方法的返回值不为 null 时才会缓存数据。例如传递了 langya 参数调用方法，在数据库未找到该记录，则不会缓存（实际中比较有用）

```java
@CacheConfig(cacheNames = "db")
public interface UmsUserMapper
{
    /**
     * 信息
     *
     * @param userName 用户名
     * @return
     */
    //只有非null数据才会缓存
    @Cacheable(key = "#userName", unless = "#result == null")
    UmsUser selectByPrimaryKey(String userName);
}
```



**属性介绍**

| 属性名               | 解释                                                      |
| -------------------- | --------------------------------------------------------- |
| value                | 缓存的名称，可定义多个（至少需要定义一个）                |
| cacheNames           | 同 value 属性                                             |
| keyGenerator         | key 生成器，字符串为：beanName                            |
| key                  | 缓存的 key，可使用 SpEL。优先级大于 keyGenerator          |
| cacheManager         | 缓存管理器，填写 beanName                                 |
| cacheResolver        | 缓存处理器，填写 beanName                                 |
| condition            | 缓存条件，若填写了，返回 true 才会执行此缓存。可使用 SpEL |
| unless               | 否定缓存，false 就生效。可以写 SpEL                       |
| **sync**             | **true 所有相同 key 的同线程顺序执行。缺省为 false**      |
| allEntries           | 是否清空所有缓存内容，缺省为 false                        |
| **beforeInvocation** | **是否在方法执行前就清空，缺省为 false**                  |



### @CacheEvict  使用 

清除缓存，方法每次都会执行。

- @Caching

   `@Caching` 适合复杂的应用场景，例如调用方法时，可以缓存多个 key，如用户名、手机号；可以删除多个 key，例如 用户名、软删除的用户名等。简单场景使用类似 ` @CacheEvict(key = "#userName", beforeInvocation=true)`  即可

- @CacheEvict

  beforeInvocation 表示方法执行前就清空。缺省时表示方法执行后再清空缓存

```java
/**
 * 软删除数据
 *
 * @param userName 用户名
 * @return
 */
//1.0
// @CacheEvict(key = "#userName", beforeInvocation=true)
//or
//2.0
//多个 CacheEvict CachePut or Cacheable
@Caching(evict={
        @CacheEvict(key = "#userName", beforeInvocation=true),
        @CacheEvict(key = "#userName + '_ex'", beforeInvocation=true)
})
int deleteByPrimaryKey(String userName);

/**
 * 更新数据
 *
 * @param record UmsUser
 * @return
*/
@CacheEvict(key = "#record.userName", beforeInvocation=true)
int updateByPrimaryKeySelective(UmsUser record);
```



## 扩展阅读

关于缓存的数据应该在方法调用前删除还是调用后删除，可以查看下列文章

[缓存如何确保数据的一致性](https://mp.weixin.qq.com/s/bPTvZzOJpZ4Uq3mWWkpoQA)

[当@Transactional遇到@CacheEvict，你的代码还运行正常吗？](https://juejin.cn/post/6844904004514742279)

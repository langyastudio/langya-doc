> http://www.javaboy.org/2019/0603/springboot-redis.html
>
> http://www.javaboy.org/2019/0416/springboot-redis.html

Spring Boot 默认使用 Lettuce 作为 Redis 客户端，同步使用时，应通过连接池提高效率。

在 Spring Boot 中，要访问 Redis，可以直接引入 `spring-boot-starter-data-redis` 依赖，它实际上是 Spring Data的一个子项目—— Spring Data Redis，主要用到了这几个组件：

- Lettuce：一个基于 **Netty** 的高性能 Redis 客户端
- RedisTemplate：一个类似于 **JdbcTemplate** 的接口，用于简化 Redis 的操作

> 因为 Spring Data Redis 引入的依赖项很多，如果只是为了使用 Redis，完全可以只引入 Lettuce，剩下的操作都自己来完成



### 依赖库

```xml
<!--spring redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!--spring session redis-->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
<!-- 对象池，使用redis时必须引入 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```



```yml
spring:  
  session:
      store-type: redis
      redis:
        flush-mode: immediate
        namespace: "a:"

    redis:
      host: 192.168.123.22
      port: 6379
      password: Hacfin_Redis8
      timeout: 6000ms
      lettuce:
        pool:
          #最大连接数
          max-active: 10
          #最大阻塞等待时间(负数表示没限制)
          max-wait: 10000ms
          #最大空闲
          max-idle: 10
          #最小空闲
          min-idle: 1
```



### @EnableCaching

一旦开启了缓存支持，Spring Boot 会按照下面的顺序查找缓存提供者。在使用缓存技术前，需要配置相关的CacheManager 的 Bean

- Generic：SimpleCacheManager

- JCache（JSR-107）：JCacheCacheManager

- EhCache 2.x：EhCacheCacheManager

- Hazelcast：HazelcastCacheManager

- Infinispan：SpringEmbeddedCacheManager

- CouchBase：CouchbaseCacheManager

- Redis：RedisCacheManager

- Caffeine：CaffeineCacheManager

- Simple：ConcurrentMapCacheManager



### 使用

```java
/**
 * redis配置类
 */
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport
{
    static final String _PREFIX = "ars:";

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

        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值（默认使用JDK的序列化方式）
        Jackson2JsonRedisSerializer<Object> jsonSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);

        ObjectMapper om = new ObjectMapper();
        // 指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // 指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer等会跑出异常
        om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL,
                                 JsonTypeInfo.As.WRAPPER_ARRAY);
        jsonSerializer.setObjectMapper(om);

        //使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());
        // 值采用json序列化
        template.setValueSerializer(jsonSerializer);

        // 设置hash key 和value序列化模式
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(jsonSerializer);

        template.afterPropertiesSet();
        return template;
    }
}
```



### 使用缓存技术

缓存服务数据能极大地提升应用的性能。在使用 Spring 缓存前，需开启 @EnableCaching，说明如下。

- @Cacheable

  注解方法可以被缓存。对于特定缓存 key，方法只执行一次，后续的请求将不执行方法，而是从缓存中获取数据

- @CachePut

  注解方法触发缓存添加操作。每次请求都会执行@CachePut注解的方法

- @CacheEvict

  注解方法触发从缓存中移除旧数据的操作

- @Caching

  支持@Cacheable、@CachePut和@CacheEvict组合注解在一个方法上

- @CacheConfig

  类级别的共享配置



可以通过使用 CacheProperties 提供的 spring.cache.*  来定制一些配置

```yml
spring:
	cache:
    	type: redis
```

![image-20210819141307564](https://img-note.langyastudio.com/20210819141310.png?x-oss-process=style/watermark)

![image-20210819141331210](https://img-note.langyastudio.com/20210819141332.png?x-oss-process=style/watermark)

- @CacheConfig 设置缓存名称作为整个类级别的共享配置，类中的方法注解可以不指定缓存名称

- 缓存注解默认使用参数作为缓存的 key，可以用 **SPEL** 从参数对象中获取值，如获得 student 的 id 可以使用 key = "#student.id"。
- @CachePut 将 key 为 student 的 id、值为返回值的数据存入缓存

- @CacheEvict 可以删除方法中参数（即id）为 key 的值

- 当 key（方法参数的id）第一次请求到此方法时，查询数据库，并将返回值放入缓存中。后续相同 key 的请求将直接从缓存中获取
- unless 可做否决，当表达式为 true 时，不存入缓存；当表达式为 false 时，存入缓存。unless 在方法执行完成后进行评估，可以获得方法的返回值 result。当返回值为 Optional 时，result 仍然是其包装的 student。unless 中的表达式的含义是：**如果返回值为空，则不放入缓存**。


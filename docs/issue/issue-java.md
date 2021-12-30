## java

#### `Collection` 转 List

```java
new ArrayList<V>( Collection<V> )
```



#### 多线程HttpServletRequest失效

异步调用时，不能使用 `HttpServletRequest` 等，因为它有生命周期



#### 无法加载类

- class path 有问题
- Module 有问题，例如父模块的目录包含的子目录为子模块，导致依赖异常



## spring 

> spring boot 在 `SpringApplication.run` 启动后才会执行组件、bean 的扫码组装操作

#### Unable to start LiveReload server

在使用 SpringBoot 热部署插件 `devtools` ，同时启动多个 Application 时，控制台会报这个警告

```yaml
spring:
    devtools:
       livereload:
          port: 35739
```



#### Error creating bean with name xxx

创建系统内置的 bean 引发异常

```java
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.context.annotation.internalAsyncAnnotationProcessor' 
```

**解决方案：**

指定项目对应的包 or 包放到多目录下（com.abc.edu）

```java
@ComponentScan(value = {"control", "entity", "service"})
```



#### org.springframework.data.redis.serializer.SerializationException: Cannot deserialize; 

发现之前给该类增加过字段，然后 `redis` 的数据也忘记做清除，导致 `redis`中的数据结构还是旧的，返回的序列号也是之前的，所以 `spring` 在拿到该数据后不能正确的给予反序列化，导致该报错

**解决方案：**

> 找到问题原因，解决就很简单了，直接将相关的 `redis` 做清除

```
redis-cli keys "cpt_*"  |  xargs redis-cli del
```



#### Controller 执行2次

google 浏览器的前端一次 API Get 请求，Controller 层的函数被执行 2 次，而 Firefox 没有该现象。



**解决方案：**

无解。后端编码时需注意，例如涉及缓存的处理，注意2次调用导致意想不到的 bug



#### Validator 不生效

- **单个参数校验**需要在参数上增加校验注解，并在类上标注 `@Validated`

- SpringBoot 2.3 版本默认移除了校验功能，如果想要开启的话需要添加如下依赖：

```xml
#不需要引入 jakarta.validation-jakarta.validation-api 库
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

> - `@Valid` 是标准 JSR-303 规范的标记型注解，用来标记验证属性和方法返回值，进行级联和递归校验
> - `@Validated` 是 Spring 提供的注解，是标准 `JSR-303` 的一个变种（补充），提供了一个分组功能，可以在入参验证时，根据不同的分组采用不同的验证机制
> - `@Validated` 只能用在类、方法和参数上，而 `@Valid` 可用于方法、参数和**字段、构造器**



#### Log4j 不生效

基于 spring 框架下的日志不生效

**解决方案：**

除了基本依赖库外

```xml
<!--common log + log4j-->
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.14.1</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.14.1</version>
    <scope>runtime</scope>
</dependency>
```

还需要 `log4j-jcl` 桥接

```xml
<!--If existing components use Apache Commons Logging 1.x
and you want to have this logging routed to Log4j2,
then add the following but do not remove any Commons Logging 1.x dependencies.-->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-jcl</artifactId>
    <version>2.14.1</version>
    <scope>runtime</scope>
</dependency>
```



#### maven 镜像仓库

在用户主目录下进入`.m2`目录，创建一个`settings.xml`配置文件，内容如下：

```xml
<settings>
    <mirrors>
        <mirror>
            <id>aliyun</id>
            <name>aliyun</name>
            <mirrorOf>central</mirrorOf>
            <!-- 国内推荐阿里云的Maven镜像 -->
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        </mirror>
    </mirrors>
</settings>
```



#### org.apache.maven.plugin.MojoExecutionException: Input length = 1

在于 SpringBoot 2.4.0 升级了 maven-resources-plugin 到 3.2.0 版本，且工程中的 `application.properties `不是`UTF-8 `编码，在编译时出现异常。

**解决方案：**

修改 `application.properties` 文件的编码方式，选用 UTF-8 编码方式，idea中设置如下

![在这里插入图片描述](https://img-note.langyastudio.com/20210406160232.png?x-oss-process=style/watermark)



#### POST 请求字段值null

由于错误使用注解，导致 RequestBody 实体类的字段 null，无法赋值

**解决方案：**

在 dto 的实体类中，移除 `org.jetbrains.annotations.NotNull` 等错误的注解



##  Mybatis

#### There is neither 'privateLookupIn(Class, Lookup)' nor 'Lookup(Class, int)' method in java.lang.invoke.MethodHandles

出现在 MyBatisMapperxy 文件中

**解决方案：**

Mapper 类型映射错误导致，例如将 `java.lang.String` 映射为 `resultMap`



####  Error getting generated key or setting result to parameter object. No setter found for the keyProperty 'sg_id' in xxx

因为 xml 文件中定义的主键名称与 Model 层实体类定义不一致导致

**解决方案：**

keyProperty 定义的主键名称与 Model 层定义的主键字段名称严格一致



#### com.mysql.jdbc.MysqlDataTruncation: Data truncation: Data too long for column 'column_name'

数据过长导致存储失败

**解决方案：**

自动截取

```ini
spring.datasource.url=jdbc:mysql://localhost:3306/conweb?sessionVariables=sql_mode='NO_ENGINE_SUBSTITUTION'&jdbcCompliantTruncation=false
```



#### mybatis Invalid bound statement (not found)

出错的原因是没有找到相对应的XML文件

> https://github.com/mybatis/spring-boot-starter/issues/141

解决方案：

**方案1：**增加 xml 扫描

查看生成的目录，增加 `mybatis.mapper-locations=classpath*:mapper/*.xml`

**方案2：**严格对应 mapper 与 xml 的对应关系

`src/main/java/**/dao/**` -> `src/main/resources/**/dao/**.xml`

![img](https://img-note.langyastudio.com/20210513233326.png?x-oss-process=style/watermark)



#### Mybatis Plus 分页获取 total 有误

自动分页插件根据  `select xx from table1 where xxx`  的 `table1`  进行数目 `count` 的统计，当实际查询表位于 `left join` 后面时就会产生查询统计有误的问题

**解决方案：**

将数据表放入首位



## 网络

#### “Refused to execute script from '...' because its MIME type ('') is not executable, and strict MIME type checking is enabled.”

![image-20210428201959115](https://img-note.langyastudio.com/20210428202001.png?x-oss-process=style/watermark)

> HttpHeaderSecurityFilter 设置的默认安全行为

`X-Content-Type-Options` 响应首部相当于一个提示标志，被服务器用来提示客户端一定要遵循在  `Content-Type` 首部对 MIME 类型的设定，而不能进行修改。遇到了格式不正确的 `Content-Type` 格式，就会造成请求的阻塞

**解决方案：**

- 服务器返回正确的 contentType
- 禁用 X-Content-Type-Options

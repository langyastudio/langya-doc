> 本文来自JavaGuide，郎涯进行简单排版与补充



###  `@SpringBootApplication`

```java
@SpringBootApplication
public class SpringSecurityJwtGuideApplication {
      public static void main(java.lang.String[] args) {
        SpringApplication.run(SpringSecurityJwtGuideApplication.class, args);
    }
}
```

我们可以把  `@SpringBootApplication` 看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。

```java
package org.springframework.boot.autoconfigure;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
   ......
}

package org.springframework.boot;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {

}
```

根据 SpringBoot 官网，这三个注解的作用分别是：

- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
- `@ComponentScan`： 扫描被 `@Component` (`@Service`,`@Controller`) 注解的 bean，注解默认会扫描该类所在的包下所有的类
- `@Configuration`：允许在 Spring 上下文中注册额外的 bean 或导入其他配置类



### Spring Bean 相关

#### `@Autowired`

**自动导入对象到类**中，被注入进的类同样要被 Spring 容器管理比如：Service 类注入到 Controller 类中。

```java
@Service
public class UserService {
  ......
}

@RestController
@RequestMapping("/users")
public class UserController {
   @Autowired
   private UserService userService;
   ......
}
```



#### `@Component` `@Repository` `@Service` `@Controller`

我们一般使用 `@Autowired` 注解让 Spring 容器帮我们自动装配 bean。要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类，可以采用以下注解实现：

- `@Component` 

  通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用 `@Component` 注解标注

- `@Repository` 

  对应持久层即 **Dao 层**，主要用于数据库相关操作

- `@Service` 

  对应**服务层**，主要涉及一些复杂的逻辑，需要用到 Dao 层

- `@Controller` 

  对应 Spring MVC **控制层**，主要用户接受用户请求并调用 Service 层返回数据给前端页面



#### `@RestController`

`@RestController` 注解是 `@Controller` 和 @`ResponseBody` 的合集, 表示这是个控制器 bean, 并且是将函数的返回值直接填入 HTTP 响应体中, 是 **REST** 风格的控制器。

单独使用 `@Controller` 不加 `@ResponseBody `的话一般使用在要返回一个视图的情况，这种情况属于比较传统的 Spring MVC 的应用，对应于前后端不分离的情况。`@Controller` + `@ResponseBody` 返回 JSON 或 XML 形式数据。



#### `@Scope`

声明 Spring Bean 的作用域，使用方法:

```java
@Bean
@Scope("singleton")
public Person personSingleton() {
    return new Person();
}
```

**四种常见的 Spring Bean 的作用域：**

- singleton 

  唯一 bean 实例，Spring 中的 bean 默认都是单例的

- prototype 

  每次请求都会创建一个新的 bean 实例

- request 

  每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效

- session 

  每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效



#### `Configuration`

一般用来声明配置类，可以使用 `@Component` 注解替代，不过使用 `Configuration` 注解声明配置类更加语义化。

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```



### 处理常见的 HTTP 请求类型

**常见的请求类型:**

- **GET** 

  请求从服务器获取特定资源。举个例子：`GET /users`（获取所有学生）

- **POST** 

  在服务器上创建一个新的资源。举个例子：`POST /users`（创建学生）

- **PUT** 

  更新服务器上的资源（客户端提供更新后的整个资源）。举个例子：`PUT /users/12`（更新编号为 12 的学生）

- **DELETE** 

  从服务器删除特定的资源。举个例子：`DELETE /users/12`（删除编号为 12 的学生）

- **PATCH** 

  更新服务器上的资源（客户端提供更改的属性，可以看做作是部分更新），使用的比较少，这里就不举例子了。



#### GET 请求

```java
@GetMapping("users")` 
//等价于`@RequestMapping(value="/users", method=RequestMethod.GET)
public ResponseEntity<List<User>> getAllUsers() {
 return userRepository.findAll();
}
```



#### POST 请求

关于 `@RequestBody` 注解的使用，在下面的 “前后端传值” 这块会讲到

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody UserCreateRequest userCreateRequest) {
 return userRespository.save(user);
}
```



#### PUT 请求

```java
@PutMapping("/users/{userId}")`
public ResponseEntity<User> updateUser(@PathVariable(value = "userId") Long userId,
  @Valid @RequestBody UserUpdateRequest userUpdateRequest) {
  ......
}
```



#### **DELETE 请求**

```java
@DeleteMapping("/users/{userId}")`
public ResponseEntity deleteUser(@PathVariable(value = "userId") Long userId){
  ......
}
```



#### **PATCH 请求**

一般实际项目中，我们都是 PUT 不够用了之后才用 PATCH 请求去更新数据

```java
  @PatchMapping("/profile")
  public ResponseEntity updateStudent(@RequestBody StudentUpdateRequest studentUpdateRequest) {
        studentRepository.updateDetail(studentUpdateRequest);
        return ResponseEntity.ok().build();
    }
```



### 前后端传值

#### `@PathVariable` 和 `@RequestParam`

- `@PathVariable` 用于获取路径参数
- `@RequestParam` 用于获取查询参数

举个简单的例子：

```java
@GetMapping("/klasses/{klassId}/teachers")
public List<Teacher> getKlassRelatedTeachers(
         @PathVariable("klassId") Long klassId,
         @RequestParam(value = "type", required = false) String type ) {
...
}
```

如果我们请求的 url 是：`/klasses/{123456}/teachers?type=web`

那么我们服务获取到的数据就是：`klassId=123456,type=web`。



#### `@RequestBody`

用于读取 Request 请求（可能是 POST, PUT, DELETE, GET 请求）的 **body** 部分并且 **Content-Type 为 application/json** 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去。系统会使用`HttpMessageConverter` 或者自定义的 `HttpMessageConverter` 将请求的 body 中的 json 字符串转换为 java 对象。

有一个注册的接口：

```java
@PostMapping("/sign-up")
public ResponseEntity signUp(@RequestBody @Valid UserRegisterRequest userRegisterRequest) {
  userService.save(userRegisterRequest);
  return ResponseEntity.ok().build();
}
```

`UserRegisterRequest`对象：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UserRegisterRequest {
    @NotBlank
    private String userName;
    @NotBlank
    private String password;
    @FullName
    @NotBlank
    private String fullName;
}
```

我们发送 post 请求到这个接口，并且 body 携带 JSON 数据：

```
{"userName":"coder","fullName":"shuangkou","password":"123456"}
```

这样我们的后端就可以直接把 json 格式的数据映射到我们的 `UserRegisterRequest` 类上。![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

> 👉 需要注意的是：**一个请求方法只可以有一个 `@RequestBody`，但是可以有多个 `@RequestParam`和`@PathVariable`**。 



### 读取配置信息

很多时候我们需要将一些常用的配置信息比如阿里云 oss、发送短信、微信认证的相关配置信息等等放到配置文件中。

下面我们来看一下 Spring 为我们提供了哪些方式帮助我们从配置文件中读取这些配置信息。



数据源 `application.yml` 内容如下：

```yml
wuhan2020: 2020年初武汉爆发了新型冠状病毒，疫情严重，但是，我相信一切都会过去！武汉加油！中国加油！

my-profile:
  name: Guide哥
  email: koushuangbwcx@163.com

library:
  location: 湖北武汉加油中国加油
  books:
    - name: 天才基本法
      description: 二十二岁的林朝夕在父亲确诊阿尔茨海默病这天，得知自己暗恋多年的校园男神裴之即将出国深造的消息——对方考取的学校，恰是父亲当年为她放弃的那所。
    - name: 时间的秩序
      description: 为什么我们记得过去，而非未来？时间“流逝”意味着什么？是我们存在于时间之内，还是时间存在于我们之中？卡洛·罗韦利用诗意的文字，邀请我们思考这一亘古难题——时间的本质。
    - name: 了不起的我
      description: 如何养成一个新习惯？如何让心智变得更成熟？如何拥有高质量的关系？ 如何走出人生的艰难时刻？
```



#### `@value`(常用)

使用 `@Value("${property}")` 读取比较简单的配置信息：

```java
@Value("${wuhan2020}")
String wuhan2020;
```



#### `@ConfigurationProperties`(常用)

通过 `@ConfigurationProperties` 读取配置信息并与 bean 绑定

```java
@Component
@ConfigurationProperties(prefix = "library")
class LibraryProperties {
    @NotEmpty
    private String location;
    private List<Book> books;

    @Setter
    @Getter
    @ToString
    static class Book {
        String name;
        String description;
    }
  省略getter/setter
  ......
}
```

你可以像使用普通的 Spring bean 一样，将其注入到类中使用。



#### `PropertySource`（不常用）

`@PropertySource` 读取指定 properties 文件

```java
@Component
@PropertySource("classpath:website.properties")
class WebSite {
    @Value("${url}")
    private String url;

  省略getter/setter
  ......
}
```



### 参数校验

#### 字段验证的注解

| 注解             | 校验功能                           |
| ---------------- | ---------------------------------- |
| @AssertFalse     | 必须是false                        |
| @AssertTrue      | 必须是true                         |
| @DecimalMax      | 小于等于给定的值                   |
| @DecimalMin      | 大于等于给定的值                   |
| @Digits          | 可设定最大整数位数和最大小数位数   |
| @Email           | 校验是否符合Email格式              |
| @Future          | 必须是将来的时间                   |
| @FutureOrPresent | 当前或将来时间                     |
| @Max             | 最大值                             |
| @Min             | 最小值                             |
| @Negative        | 负数（不包括0）                    |
| @NegativeOrZero  | 负数或0                            |
| @NotBlank        | 不为null并且包含至少一个非空白字符 |
| @NotEmpty        | 不为null并且不为空                 |
| @NotNull         | 不为null                           |
| @Null            | 为null                             |
| @Past            | 必须是过去的时间                   |
| @PastOrPresent   | 必须是过去的时间，包含现在         |
| @Pattern         | 必须满足正则表达式                 |
| @PositiveOrZero  | 正数或0                            |
| @Size            | 校验容器的元素个数                 |



#### 验证请求体(RequestBody)

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {
    @NotNull(message = "classId 不能为空")
    private String classId;

    @Size(max = 33)
    @NotNull(message = "name 不能为空")
    private String name;

    @Pattern(regexp = "((^Man$|^Woman$|^UGM$))", message = "sex 值不在可选范围")
    @NotNull(message = "sex 不能为空")
    private String sex;

    @Email(message = "email 格式不正确")
    @NotNull(message = "email 不能为空")
    private String email;

}
```



需要验证的参数上加上了 `@Valid` 注解，如果验证失败，它将抛出 `MethodArgumentNotValidException`。

```java
@RestController
@RequestMapping("/api")
public class PersonController {

    @PostMapping("/person")
    public ResponseEntity<Person> getPerson(@RequestBody @Valid Person person) {
        return ResponseEntity.ok().body(person);
    }
}
```



#### 验证请求参数(Path Variables 和 Request Parameters)

> 一定一定不要忘记在类上加上 `Validated` 注解了，这个参数可以告诉 Spring 去校验方法参数

```java
@RestController
@RequestMapping("/api")
@Validated
public class PersonController {
    @GetMapping("/person/{id}")
    public ResponseEntity<Integer> getPersonByID(@Valid @PathVariable("id") @Max(value = 5,message = "超过 id 的范围了") Integer id) {
        return ResponseEntity.ok().body(id);
    }
}
```



### 全局处理 Controller 层异常

**相关注解：**

- `@ControllerAdvice` ：注解定义全局异常处理类

- `@ExceptionHandler` ：注解声明异常处理方法

例如如果方法参数不对的话就会抛出 `MethodArgumentNotValidException`，我们来处理这个异常

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    /**
     * 请求参数异常处理
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex, HttpServletRequest request) {
       ......
    }
}
```



### 其他

- `@ActiveProfiles` 一般作用于测试类上， 用于声明生效的 Spring 配置文件

```java
@SpringBootTest(webEnvironment = RANDOM_PORT)
@ActiveProfiles("test")
@Slf4j
public abstract class TestBase {
  ......
}
```

- `@Test` 声明一个方法为测试方法

- `@Transactional`被声明的测试方法的数据会回滚，避免污染测试数据

- `@WithMockUser` Spring Security 提供的，用来模拟一个真实用户，并且可以赋予权限

```java
    @Test
    @Transactional
    @WithMockUser(username = "user-id-18163138155", authorities = "ROLE_TEACHER")
    void should_import_student_success() throws Exception {
        ......
    }
```
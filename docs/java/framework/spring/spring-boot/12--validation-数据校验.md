> 源码：[https://github.com/langyastudio/langya-tech/tree/springboot/validation](https://github.com/langyastudio/langya-tech/tree/springboot/validation)

JSR（Java Specification Requests，**Java 规范请求**）是对 Java 新功能的请求，是 JCP 组织的一部分。Java 社区的参与者们通过 JCP 组织，利用自己的创意来影响 Java 语言的发展。在 JSR 中，jar 的包名一般以 javax 开头，如javax.validation。JSR 只提供功能规范定义，不提供实现。

Spring Boot 支持基于 JSR-303/349/380 等规范的 Bean 校验API。Spring Boot 的 Web 依赖添加了 spring-boot-starter-validation，它添加了规范包 jakarta.validation-api-x.x.x.jar 和实现包 hibernate-validator-x.x.x.Final.jar，BindingResult 可直接作为参数注入，从而获得校验的错误。



## 参数校验

在日常的接口开发中，为了防止非法参数对业务造成影响，经常需要对接口的参数做校验，例如登录的时候需要校验用户名密码是否为空、创建用户的时候需要校验邮件、手机号码格式是否准确。

### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

>  SpringBoot 2.3.1 版本默认移除了校验功能，如果想要开启的话需要添加以上依赖

- `@Valid` 是标准 **JSR-303 规范**的标记型注解，用来标记验证属性和方法返回值，进行级联和递归校验
- `@Validated` 是 **Spring 提供的注解**，是标准 `JSR-303` 的一个变种（补充），提供了一个**分组功能**，可以在入参验证时，根据不同的分组采用不同的验证机制
- `@Validated` 只能用在类、方法和参数上，而 `@Valid` 可用于方法、参数和**字段、构造器**



### 内置注解

[https://beanvalidation.org/2.0/spec/](https://beanvalidation.org/2.0/spec/)

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
| **@Pattern**     | **必须满足正则表达式**             |
| @PositiveOrZero  | 正数或0                            |
| **@Size**        | **校验容器的元素个数**             |



### RequestBody 请求体校验

**定义传入参数**

```java
@Data
public class UserParam
{
    /**
     * 不能为null
     * 长度2-20
     */
    @NotNull
    @Size(min=2, max = 20)
    private String userName;

    /**
     * 长度2-20
     */
    @Size(min=2, max = 20)
    private String nickName;

    /**
     * 长度2-20
     * 邮箱格式
     */
    @NotNull
    @Email(message = "邮箱格式有误")
    private String email;

    /**
     * 手机号正则匹配
     */
    @Pattern(regexp = "^1[3,4,5,6,7,8,9]{1}[0-9]{9}$")
    private String telephone;
}
```



**接口层**

函数参数增加 `@Valid` 注解

```java
@RestController
@RequestMapping("/api")
public class ApiController
{
    @PostMapping("/user/add")
    public UserParam addUser(@Valid @RequestBody UserParam userParam)
    {
        return userParam;
    }
}
```



**接口测试**

![image-20210908154756248](https://img-note.langyastudio.com/20210908154756.png?x-oss-process=style/watermark)



**捕获异常**

参数校验失败后，会抛出 `MethodArgumentNotValidException` 异常，可以被 `ExceptionHandler` 捕获

> 详细请看：[spring boot  exception全局异常处理](https://mp.weixin.qq.com/s/NwFvsN55NjivB3_gLx2hIA)

```java
@ControllerAdvice
@Log4j2
public class ExceptionHandle
{
    // 处理 json 请求体调用接口校验失败抛出的异常
    @ResponseBody
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResultInfo methodArgumentNotValidExceptionHandler(MethodArgumentNotValidException e)
    {
        List<FieldError> fieldErrors = e.getBindingResult().getFieldErrors();
        List<String> collect = new ArrayList<>();
        for(FieldError fieldError :fieldErrors)
        {
            collect.add(fieldError.getField() + fieldError.getDefaultMessage());
        }
        return ResultInfo.data(EC.ERROR_PARAM_EXCEPTION, collect);
    }

    // 处理单个参数校验失败抛出的异常
    @ResponseBody
    @ExceptionHandler(ConstraintViolationException.class)
    public ResultInfo constraintViolationExceptionHandler(ConstraintViolationException e)
    {
        Set<ConstraintViolation<?>> constraintViolations = e.getConstraintViolations();
        List<String> collect = constraintViolations.stream()
                .map(ConstraintViolation::getMessage)
                .collect(Collectors.toList());
        return ResultInfo.data(EC.ERROR_PARAM_EXCEPTION, collect);
    }
}
```



### RequestParam & RequestPath 请求参数校验

> 参数校验失败后，会抛出 `ConstraintViolationException` 异常

**接口层增加校验注解**

- 类增加 `@Validated` 注解
- 函数参数增加 `@Valid` 注解

> 只要有 `@Validated`+ `@Valid`，在 service 层也可以使用参数校验

增加了 `RequestParam` 类型的 pwd 测试参数

```java
@RestController
@RequestMapping("/api")
@Validated
public class ApiController
{
    @PostMapping("/user/add")
    public UserParam addUser(@Valid @RequestBody UserParam userParam,
                             @Valid @RequestParam(value = "pwd") @Size(min = 6) String pwd)
    {
        return userParam;
    }
}
```



**接口测试**

![image-20210908160136018](https://img-note.langyastudio.com/20210908160136.png?x-oss-process=style/watermark)



## 自定义校验

虽然 Spring boot 提供的注解基本上够用，但是面对复杂的定义还是需要自定义相关注解来实现自动校验。例如传值只能是 "F"、"L" 这两种枚举类型

**自定义注解** 

```java
/**
 * 值是否在指定枚举范围的校验器
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD,ElementType.PARAMETER})
@Constraint(validatedBy = InValidator.class)
public @interface InValue
{
    String[] value() default {"1", "2"};

    String message() default "参数值异常";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```



**定义Validator实现**

继承 `ConstraintValidator`。InValue 通过该类进行具体的逻辑校验

```java
public class InValidator implements ConstraintValidator<InValue, Object>
{
    private String[] values;

    @Override
    public void initialize(InValue flagValidator)
    {
        this.values = flagValidator.value();
    }

    @Override
    public boolean isValid(Object value, ConstraintValidatorContext constraintValidatorContext)
    {
        boolean isValid = false;
        if (value == null)
        {
            //当状态为空时使用默认值
            return true;
        }

        for (int i = 0; i < values.length; i++)
        {
            if (values[i].equals(String.valueOf(value)))
            {
                isValid = true;
                break;
            }
        }

        return isValid;
    }
}
```



**测试**

当传入的参数值不为 "F", "L" 时，将抛出异常

```java
    @PostMapping("/user/add")
    public String addUser(@Valid @RequestParam(value = "type") @InValue({"F", "L"}) String type)
    {
        return type;
    }
```



## 分组校验

一个传入的参数在新增的时候某些字段为必填，在更新的时候又非必填。面对这种场景你会怎么处理呢？

这时候就需要用到分组校验了。比如：新注册用户还没起 id 号，我们允许 `id` 字段为空，但是不允许将 id 更新为空。



**分组校验有三个步骤：**

- 定义一个分组类（或接口）
- 在校验注解上添加 `groups` 属性指定分组

- `Controller` 方法的 `@Validated` 注解添加分组类

```java
public interface InsertV  extends Default
{
}
public interface UpdateV extends Default
{
}

@Data
public class UserParam
{
    @Null(message = "新增时id必须为空", groups = {InsertV.class})
    @NotNull(message = "更新时id不能为空", groups = {UpdateV.class})
    @Positive
    private Integer id;
    
    ...
}
```



**测试**

```java
@RestController
@RequestMapping("/api")
@Validated
public class ApiController
{
    @PostMapping("/user/add")
    public UserParam addUser(@Validated({UpdateV.class}) @RequestBody UserParam userParam) String type)
    {
        return userParam;
    }
}
```

![image-20210908173630617](https://img-note.langyastudio.com/20210908173630.png?x-oss-process=style/watermark)



**扩展**

自定义的 `UpdateV` 分组接口继承了 `Default` 接口。校验注解(如：`@NotBlank`)和 `@validated` 默认都属于`Default.class` 分组，这一点在 `javax.validation.groups.Default` 注释中有说明

```java
/**
 * Default Jakarta Bean Validation group.
 * <p>
 * Unless a list of groups is explicitly defined:
 * <ul>
 *     <li>constraints belong to the {@code Default} group</li>
 *     <li>validation applies to the {@code Default} group</li>
 * </ul>
 * Most structural constraints should belong to the default group.
 *
 * @author Emmanuel Bernard
 */
public interface Default {
}
```

在编写 `UpdateV` 分组接口时，如果继承了 `Default`，下面两个写法就是等效的：

```java
@Validated({UpdateV.class})
@Validated({UpdateV.class, Default.class})
```

> 参考廖雪峰等文章


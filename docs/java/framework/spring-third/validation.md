### 依赖库

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>	
```

> 不需要引入 jakarta.validation-jakarta.validation-api 库



### 使用

https://beanvalidation.org/2.0/spec/



内置注解

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



### 分组

如果同一个参数，需要在不同场景下应用不同的校验规则，就需要用到分组校验了。比如：新注册用户还没起名字，我们允许`name`字段为空，但是不允许将名字更新为空字符。

分组校验有三个步骤：

1. 定义一个分组类（或接口）
2. 在校验注解上添加`groups`属性指定分组
3. `Controller`方法的`@Validated`注解添加分组类

```java
public interface Update extends Default{
}
public class UserVO {
    @NotBlank(message = "name 不能为空",groups = Update.class)
    private String name;
    // 省略其他代码...
}
@PostMapping("update")
public ResultInfo update(@Validated({Update.class}) UserVO userVO) {
    return new ResultInfo().success(userVO);
}
```



细心的同学可能已经注意到，自定义的`Update`分组接口继承了`Default`接口。校验注解(如：`@NotBlank`)和`@validated`默认都属于`Default.class`分组，这一点在`javax.validation.groups.Default`注释中有说明

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



在编写`Update`分组接口时，如果继承了`Default`，下面两个写法就是等效的：

```java
@Validated({Update.class})
@Validated({Update.class,Default.class})
```



请求一下`/update`接口可以看到，不仅校验了`name`字段，也校验了其他默认属于`Default.class`分组的字段

```json
{
    "status": 400,
    "message": "客户端请求参数错误",
    "response": [
        "name 不能为空",
        "age 不能为空",
        "email 不能为空"
    ]
}
```



如果`Update`不继承`Default`，`@Validated({Update.class})`就只会校验属于`Update.class`分组的参数字段，修改后再次请求该接口得到如下结果，可以看到， 其他字段没有参与校验：

```json
{
    "status": 400,
    "message": "客户端请求参数错误",
    "response": [
        "name 不能为空"
    ]
}
```


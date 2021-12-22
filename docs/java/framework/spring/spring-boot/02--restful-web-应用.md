> 先看 [spring boot 2.x 零基础快速入门](https://mp.weixin.qq.com/s/Rali52EZKjAzhgXv7PWgvg)
>
> 基于上述代码修改

使用 `@RestController` 可以方便地编写 REST 服务，Spring 默认使用 `JSON` 作为输入和输出。



## 编写 RestController

> 编写 Rest Controller 只需要遵循以下要点：
>
> 总是标记 `@RestController` 而不是 `@Controller`、`@Component`

使用 `@RestController` 替代 `@Controller` 后，**每个方法自动变成 API 接口方法**。输入和输出只要能被 Jackson 序列化或反序列化为 JSON 就没有问题。

编写 `ApiController` 如下：

```java
@RestController
public class ApiController
{
    @GetMapping("/users")
    public List<String> users(@RequestParam(value = "user_name") String userName )
    {
        return List.of("Apple", "Orange", "Banana", userName);
    }

    @GetMapping("/user/{user_name}")
    public String user(@PathVariable("user_name") String userName)
    {
        return userName;
    }
}
```

`@GetMapping` 表示 GET 类型的请求，括号里面的值 `"/users"` 表示请求地址

`@RequestParam`  用于获取查询参数

`@PathVariable`  用于获取路径参数

> 如果方法参数需要传入`HttpServletRequest`、`HttpServletResponse` 或者 `HttpSession`，直接添加这个类型的参数即可，Spring MVC 会自动按类型传入



**测试如下：**

`192.168.123.100:8080/users?user_name=langya`

即传入请求参数 user_name

![image-20210730103538332](https://img-note.langyastudio.com/20210730103538.png?x-oss-process=style/watermark)

`192.168.123.100:8080/user/langya`

即传入path路径传入参数 langya

![image-20210730103829293](https://img-note.langyastudio.com/20210730103829.png?x-oss-process=style/watermark)



## 请求地址分组

对 URL 进行分组，每组对应一个 Controller 是一种很好的组织形式，有效避免重复的URL映射。例如：

```java
@RestController
@RequestMapping("/api")
public class ApiController{
  
}
```

此时上述的API请求路径变成 **前缀（api）+ 路径**，如 `/api/usesrs`



## 请求与传参

**常见的请求类型:**

- **GET** ：请求从服务器获取特定资源
- **POST** ：在服务器上创建一个新的资源
- **PUT** ：更新服务器上的资源（客户端提供更新后的整个资源）
- **DELETE** ：从服务器删除特定的资源
- **PATCH** ：更新服务器上的资源（客户端提供更改的属性，可以看做作是部分更新）



### POST

- ReuqestBody 主要是处理 json 串格式的请求参数，要求使用方指定 header `content-type:application/json`
- RequestBody 通常要求调用方使用 post 请求（**也可以使用其他类型**）
- RequestBody 接收到数据之后会自动将数据绑定到 Java 对象。系统会使用 `HttpMessageConverter` 或者自定义的`HttpMessageConverter` 将请求的 body 中的 json 字符串转换为 java 对象



定义传入参数 `UserParam`

这里使用了 `lombok` 依赖库 —— 为了精简代码，不用写一堆SET、GET属性函数

```java
package com.langyastudio.springboot.bean.dto;

import lombok.Data;

/**
 * user 传入参数
 */
@Data
public class UserParam
{
    private String userName;
    private String nickName;
    private String telephone;
}
```

定义接口

```java
@RestController
@RequestMapping("/api")
public class ApiController
{
    @PostMapping("/user/add")
    public UserParam addUser(@RequestBody UserParam userParam)
    {
        return userParam;
    }

    @PostMapping("/user/addex")
    public Map<String, Object> addUserEx(@RequestBody Map<String, Object> params)
    {
        return params;
    }
}
```

`@RequestBody` 既可以使用 bean 实体类接收传入参数（**建议使用**），也可以使用 Map 接收（可读性、可维护性差）



**测试如下：**

`192.168.123.100:8080/api/user/add`

使用 Postman 测试时需要注意，使用 `Body -> raw -> json`

![image-20210730110351122](https://img-note.langyastudio.com/20210730110351.png?x-oss-process=style/watermark)



>  一个请求方法只可以有一个 `@RequestBody`，但是可以有多个  `@RequestParam`  和  `@PathVariable`
>
>  完整代码：https://github.com/langyastudio/langya-tech/tree/springboot/restful/spring-boot



## 常用注解

- @RequestHeader

  header 为元数据信息

- @RequestBody

  请求体数据

- @RequestParam

  请求参数

- @PathVariable

  路径变量

- @ResponseStatus

  status 状态信息

- @ResponseBody

  body 为返回数据

- @CookieValue

  客户端cookie的值

- MultipartFile

  接收上传的文件



## Advice

**ControllerAdvice**

@ControllerAdvice 注解是一个特殊的 @Component，顾名思义它负责所有**控制器共享**的功能，如异常处理（配合@ExceptionHandler）、数据绑定（配合 @InitBinder ）等。@ControllerAdvice 通过属性指定控制器的生效范围。



**RestControllerAdvice**

@RestControllerAdvice 是组合注解，它组合了 @ControllerAdvice 和 @ResponseBody，它的功能和@ControllerAdvice 一样，主要用于对 RESTful 的请求体和返回体进行定制处理。

- 对请求体进行定制处理用 RequestBodyAdvice 接口，它会在请求体进入控制器方法之前对请求体进行处理

- 对返回体进行定制处理用 ResponseBodyAdvice 接口，它会在控制器方法返回值确定之后再对返回值进行处理。它们需要和 @RestControllerAdvice 一起使用



## RestTemplate

可以使用 RestTemplate 作为客户端来访问其他应用提供的 RESTful 服务。在前面的演示中，使用的是 Postman 或者浏览器，这里使用 RestTemplate 作为客户端完成同样的功能。

Spring Boot 自动配置了 RestTemplateBuilder，我们可以通过它来获得 RestTemplate。

![image-20210819065853645](https://img-note.langyastudio.com/20210819065856.png?x-oss-process=style/watermark)

![image-20210819065911585](https://img-note.langyastudio.com/20210819065916.png?x-oss-process=style/watermark)

使用 postForEntity() 来保存 Person，方法参数 Person.class 用来指定返回值的类型。这里服务端的代码针对的是下面的内容。

使用 getForObject() 获取某个 Person，路径变量通过 params 来传递。

同样，通过 getForObject() 按请求参数查询 Person，查询参数通过 UriComponentsBuilder 的 queryParam() 方法来构造。

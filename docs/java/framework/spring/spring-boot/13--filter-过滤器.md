### Filter vs Interceptor vs Listener

- 过滤器（Filter）

  当你有一堆东西的时候，你只希望 **选择** 符合你要求的某一些东西。定义这些要求的工具，就是过滤器

- 拦截器（Interceptor）

  在一个流程正在进行的时候，你希望 **干预** 它的进展，甚至终止它，这是拦截器做的事情

- 监听器（Listener）

  当一个事件发生的时候，你希望获得这个事件发生的详细 **信息**，而并不想干预这个事件本身的进程，这就要用到监听器



### FilterRegistrationBean

在 Spring Boot 中添加 `Filter` 更加方便，并且支持对多个 `Filter` 进行排序。本质上就是通过代理，自动扫描所有的 **`FilterRegistrationBean`** 类型的 Bean，然后将它们返回的 `Filter` 自动注册到 Servlet 容器中，无需任何配置。



以 `AuthFilter` 为例，首先编写一个 `AuthFilterRegistrationBean`，它继承自 `FilterRegistrationBean`：

```java
@Order(10)
@Component
public class AuthFilterRegistrationBean extends FilterRegistrationBean<Filter> {
    @Autowired
    UserService userService;

    @Override
    public Filter getFilter() {
        return new AuthFilter();
    }

    class AuthFilter implements Filter {
        ...
    }
}
```

`FilterRegistrationBean` 本身不是 `Filter`，它实际上是 `Filter` 的工厂。Spring Boot 会调用 `getFilter()`，把返回的 `Filter` 注册到 Servlet 容器中。因为我们可以在 `FilterRegistrationBean` 中注入需要的资源，然后在返回的`AuthFilter` 中，这个内部类可以引用外部类的所有字段，自然也包括注入的 `UserService`，所以整个过程完全基于Spring 的 IoC 容器完成。

再注意到 `AuthFilterRegistrationBean` 标记了一个 `@Order(10)`，因为 Spring Boot 支持给多个 `Filter` 排序，数字小的在前面，所以多个 `Filter` 的顺序是可以固定的。



### setUrlPatterns

再编写一个 `ApiFilter`，专门过滤 `/api/*`  这样的URL。首先编写一个 `ApiFilterRegistrationBean`

```java
@Order(20)
@Component
public class ApiFilterRegistrationBean extends FilterRegistrationBean<Filter> {
    @PostConstruct
    public void init() {
        setFilter(new ApiFilter());
        setUrlPatterns(List.of("/api/*"));
    }

    class ApiFilter implements Filter {
        ...
    }
}
```

这个 `ApiFilterRegistrationBean` 和 `AuthFilterRegistrationBean` 又有所不同。因为我们要过滤URL，而不是针对所有 URL 生效，因此在 `@PostConstruct` 方法中，通过 `setFilter()` 设置一个 `Filter` 实例后，再调用**`setUrlPatterns()`** 传入要过滤的 URL 列表。

# 基础

## 入门

> Spring Security 是强大的且容易定制的，基于 Spring 开发的实现**认证登录与资源授权**的应用安全框架
>

### 依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

SpringSecurity 的核心功能：

- **Authentication**：身份认证，用户登陆的验证（解决你是谁的问题）

- **Authorization**：访问授权，授权系统资源的访问权限（解决你能干什么的问题）

- 安全防护，防止跨站请求，session 攻击等

  

### 与Shiro对比

Shiro 的使用量一直高于 spring security 。但是从趋势上来看（2020年10月的百度指数），Spring Security 是在一直上升的，shiro 的使用量同比、环比都进入了下滑期。

![img](https://img-note.langyastudio.com/20210419105909.png?x-oss-process=style/watermark)

> 如果你只是想实现一个简单的 web 应用，shiro 更加的轻量级，学习成本也更低。
>
> 如果您正在开发一个分布式的、微服务的或者与 Spring Cloud 系列框架深度集成的项目，笔者还是建议您使用Spring Security



**功能丰富性**

Spring Security 因为它的复杂，所以从功能的丰富性的角度更胜一筹。其中比较典型的如：

- Spring Security 默认含有对 OAuth2.0 的支持，与 Spring Social 一起使用完成社交媒体登录也比较方便。shiro 在这方面只能靠自己写代码实现
- Spring Security 在网络安全的方面下的功夫更多，但是笔者并未有非常直接的感受，有可能出现安全问题的时候才会感到不够安全的痛



### SpringSecurity 的配置类

- configure(HttpSecurity httpSecurity)

    用于配置需要拦截的 url 路径、jwt 过滤器及出异常后的处理器

- configure(AuthenticationManagerBuilder auth)

    用于配置 UserDetailsService 及 PasswordEncoder

- RestfulAccessDeniedHandler

    当用户没有访问权限时的处理器，用于返回 JSON 格式的处理结果

- RestAuthenticationEntryPoint

    当未登录或 token 失效时，返回 JSON 格式的结果

- UserDetailsService

    SpringSecurity 定义的核心接口，用于根据用户名获取用户信息，需要自行实现

- UserDetails

    SpringSecurity 定义用于封装用户信息的类（主要是用户信息和权限），需要自行实现

- PasswordEncoder

    SpringSecurity 定义的用于对密码进行编码及比对的接口，目前使用的是 BCryptPasswordEncoder

- JwtAuthenticationTokenFilter

    在用户名和密码校验前添加的过滤器，如果有 jwt 的 token，会自行根据 token 信息进行登录



## PasswordEncoder

如何处理密码的加密与校验的接口及实现类

BCryptPasswordEncoder 是 Spring Security 推荐使用的 `PasswordEncoder` 接口实现类

```java
public class PasswordEncoderTest {
  
  @Test
  void bCryptPasswordTest(){
    PasswordEncoder passwordEncoder =  new BCryptPasswordEncoder();
    String rawPassword = "123456";  //原始密码
    String encodedPassword = passwordEncoder.encode(rawPassword); //加密后的密码

    System.out.println("原始密码" + rawPassword);
    System.out.println("加密之后的hash密码:" + encodedPassword);

    System.out.println(rawPassword + "是否匹配" + encodedPassword + ":"   //密码校验：true
            + passwordEncoder.matches(rawPassword, encodedPassword));

    System.out.println("654321是否匹配" + encodedPassword + ":"   //定义一个错误的密码进行校验:false
            + passwordEncoder.matches("654321", encodedPassword));
  }
}
```

> 对于同一个原始密码，每次加密之后的hash密码都是不一样的，这正是BCryptPasswordEncoder的强大之处



**加密结果：**

```java
$2a$10$zt6dUMTjNSyzINTGyiAgluna3mPm7qdgl26vj4tFpsFO6WlK5lXNm
```

BCrypt 加密后的密码有三个部分，由 $分隔：

- "2a" 表示 BCrypt 算法版本

- "10" 表示算法的强度

- "zt6dUMTjNSyzINTGyiAglu" 部分实际上是随机生成的盐。通常来说前 22 个字符是盐，剩余部分是纯文本的实际哈希值。

> BCrypt* 算法生成长度为 60 的字符串，因此我们需要确保密码将存储在可以容纳密码的数据库列中



## 登录认证

> Spring Security 的登录认证并不需要我们自己去写登录认证的 Controller 方法，而是使用 **过滤器**

![img](https://img-note.langyastudio.com/20210419150413.png?x-oss-process=style/watermark)

- 贯穿于整个过滤器链始终有一个上下文对象 `SecurityContext` 和一个 `Authentication` 对象（登录认证的主体）
- 一旦某一个该主体通过其中某一个过滤器的认证，Authentication 对象信息被填充，比如：isAuthenticated=true 表示该主体通过验证
- 如果该主体通过了所有的过滤器，仍然没有被认证，在整个过滤器链的最后方有一个` FilterSecurityInterceptor` 过滤器（虽然叫 Interceptor ，但它是名副其实的过滤器不是拦截器）。判断 Authentication 对象的认证状态，**如果没有通过认证则抛出异常，通过认证则访问后端API**

- 之后进入响应阶段，FilterSecurityInterceptor 抛出的异常被 `ExceptionTransactionFilter` 对异常进行相应的处理。比如：用户名密码登录异常，会被引导到登录页重新登陆。
- 如果是登陆成功且没有任何异常，在请求响应中最后一个过滤器 `SecurityContextPersistenceFilter` 中将SecurityContext 放入 session。下次再进行请求的时候，直接从 SecurityContextPersistenceFilter 的 session 中取出认证信息



### formLogin登录认证及资源访问权限的控制

首先，我们要继承 WebSecurityConfigurerAdapter ，重写 configure(HttpSecurity http) 方法，该方法用来配置登录验证逻辑。请注意看下文代码中的注释信息。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
   http.csrf().disable() //禁用跨站csrf攻击防御，后面的章节会专门讲解
       .formLogin()
           .loginPage("/login.html")//一旦用户的请求没有权限就跳转到这个页面
           .loginProcessingUrl("/login")//登录表单form中action的地址，也就是处理认证请求的路径
           .usernameParameter("username")///登录表单form中用户名输入框input的name名，不修改的话默认是username
           .passwordParameter("password")//form中密码输入框input的name名，不修改的话默认是password
           .defaultSuccessUrl("/")//登录认证成功后默认转跳的路径
       .and()
           .authorizeRequests()
           .antMatchers("/login.html","/login")
       			.permitAll()//不需要通过登录验证就可以被访问的资源路径
           .antMatchers("/","/biz1","/biz2") //资源路径匹配
               .hasAnyAuthority("ROLE_user","ROLE_admin")  //user角色和admin角色都可以访问
           .antMatchers("/syslog","/sysuser")  //资源路径匹配
               .hasAnyRole("admin")  //admin角色可以访问
           //.antMatchers("/syslog").hasAuthority("sys:log")
           //.antMatchers("/sysuser").hasAuthority("sys:user")
           .anyRequest().authenticated();
}
```

上面的代码分为两部分：

- 第一部分是formLogin配置段，用于配置登录验证逻辑相关的信息。如：登录页面、登录成功页面、登录请求处理路径等。和login.html页面的元素配置要一一对应。
- "/"在spring boot应用里面作为资源访问的时候比较特殊，它就是“/index.html”.所以defaultSuccessUrl登录成功之后就跳转到index.html

![img](https://img.kancloud.cn/19/16/191693fe6ec816947d22a3098c2ae9fc_1721x360.png)

- 第二部分是authorizeRequests配置段，用于配置资源的访问控制规则。如：开发登录页面的permitAll开放访问，“/biz1”（业务一页面资源）需要有角色为user或admin的用户才可以访问。
- `hasAnyAuthority("ROLE_user","ROLE_admin")`等价于`hasAnyRole("user","admin")`,角色是一种特殊的权限。
- "sys:log"或"sys:user"是我们自定义的权限ID，有这个ID的用户可以访问对应的资源

这时候我们通过浏览器访问，随便测试一个用户没有访问权限的资源，都会跳转到login.html页面。




### 用户及角色信息配置

在上文中，我们配置了登录验证及资源访问的权限规则，我们还没有具体的用户，下面我们就来配置具体的用户。重写WebSecurityConfigurerAdapter的 configure(AuthenticationManagerBuilder auth)方法

```java
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
                .withUser("user")
                .password(passwordEncoder().encode("123456"))
                .roles("user")
            .and()
                .withUser("admin")
                .password(passwordEncoder().encode("123456"))
                //.authorities("sys:log","sys:user")
                .roles("admin")
            .and()
                .passwordEncoder(passwordEncoder());//配置BCrypt加密
}

@Bean
public PasswordEncoder passwordEncoder(){
    return new BCryptPasswordEncoder();
}
```

- `inMemoryAuthentication`指的是在内存里面存储用户的身份认证和授权信息。

- `withUser("user")`用户名是user

- `password(passwordEncoder().encode("123456"))`密码是加密之后的123456

- `authorities("sys:log","sys:user")`指的是admin用户拥有资源ID对应的资源访问的的权限："/syslog"和"/sysuser"

- `roles()`方法用于指定用户的角色，一个用户可以有多个角色

  

### 静态资源访问

在我们的实际开发中，登录页面 login.html 和控制层 Controller 登录验证 '/login' 都必须无条件的开放。除此之外，一些静态资源如 css、js 文件通常也都不需要验证权限，我们需要将它们的访问权限也开放出来。下面就是实现的方法：重写WebSecurityConfigurerAdapter 类的 configure(WebSecurity web)  方法

```java
   @Override
    public void configure(WebSecurity web) {
        //将项目中静态资源路径开放出来
        web.ignoring().antMatchers( "/css/**", "/fonts/**", "/img/**", "/js/**");
    }
```

那么这些静态资源的开放，和 Controller 服务资源的开放为什么要分开配置？有什么区别呢？

- Controller 服务资源要经过一系列的过滤器的验证，我们配置的是验证的放行规则
- 这里配置的是静态资源的开放，不经过任何的过滤器链验证，直接访问资源。



## 登录验证结果

在我之前的文章中，做过登录验证流程的源码解析。其中比较重要的就是

```java
http.formLogin()
    .loginPage("/login.html")//一旦用户的请求没有权限就跳转到这个页面
    .defaultSuccessUrl("/")//登录认证成功后默认转跳的路径
    .failureUrl("/login.html")//登陆失败跳转路径
```

- 当我们登录成功的时候，是由 AuthenticationSuccessHandler 进行登录结果处理，默认跳转到 defaultSuccessUrl 配置的路径对应的资源页面（一般是首页index.html）。
- 当我们登录失败的时候，是由 AuthenticationfailureHandler 进行登录结果处理，默认跳转到failureUrl配置的路径对应的资源页面（一般也是跳转登录页login.html，重新登录）。

但是在 web 应用开发过程中需求是千变万化的，有时需要我们针对登录结果做个性化处理，比如：

- 我们希望不同的人登陆之后，看到不同的首页（及向不同的路径跳转）
- 我们应用是前后端分离的，验证响应结果是JSON格式数据，而不是页面跳转

以上的这些情况，使用 Spring Security 作为安全框架的时候，都需要我们使用本节学到的知识进行自定义的登录验证结果处理。



### 自定义登陆成功

AuthenticationSuccessHandler 接口是 Security 提供的认证成功处理器接口，我们只需要去实现它即可。但是通常来说，我们不会直接去实现 AuthenticationSuccessHandler 接口，而是继承SavedRequestAwareAuthenticationSuccessHandler  类，这个类会记住用户上一次请求的资源路径，比如：用户请求books.html，没有登陆所以被拦截到了登录页，当你完成登陆之后会自动跳转到books.html，而不是主页面。

```java
@Component
public class MyAuthenticationSuccessHandler 
                        extends SavedRequestAwareAuthenticationSuccessHandler {

    //在application配置文件中配置登陆的类型是JSON数据响应还是做页面响应
    @Value("${spring.security.logintype}")
    private String loginType;
    
    //Jackson JSON数据处理类
    private  static ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, 
                                        HttpServletResponse response, 
                                        Authentication authentication) 
                                        throws ServletException, IOException {

        if (loginType.equalsIgnoreCase("JSON")) {
            response.setContentType("application/json;charset=UTF-8");
            response.getWriter().write(objectMapper.writeValueAsString(AjaxResponse.success()));
        } else {
            // 会帮我们跳转到上一次请求的页面上
            super.onAuthenticationSuccess(request, response, authentication);
        }
    }
}
```

- 在上面的自定义登陆成功处理中，既适应JSON前后端分离的应用登录结果处理，也适用于模板页面跳转应用的登录结果处理
- ObjectMapper 是Spring Boot默认集成的JSON数据处理类库Jackson中的类。
- AjaxResponse是一个自定义的通用的JSON数据接口响应类。



### 自定义登录失败

这里我们同样没有直接实现AuthenticationFailureHandler接口，而是继承SimpleUrlAuthenticationFailureHandler 类。该类中默认实现了登录验证失败的跳转逻辑，即登陆失败之后回到登录页面。我们可以利用这一点简化我们的代码。

```java
@Component
public class MyAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    //在application配置文件中配置登陆的类型是JSON数据响应还是做页面响应
    @Value("${spring.security.logintype}")
    private String loginType;

    private  static ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public void onAuthenticationFailure(HttpServletRequest request,
                                        HttpServletResponse response, 
                                        AuthenticationException exception) 
                                        throws IOException, ServletException {

        if (loginType.equalsIgnoreCase("JSON")) {
            response.setContentType("application/json;charset=UTF-8");
            response.getWriter().write(
                    objectMapper.writeValueAsString(
                            AjaxResponse.error(
                                    new CustomException(
                                        CustomExceptionType.USER_INPUT_ERROR,
                                        "用户名或密码存在错误，请检查后再次登录"))));
        } else {
            response.setContentType("text/html;charset=UTF-8");
            super.onAuthenticationFailure(request, response, exception);
        }

    }
}
```

- 在上面的自定义登陆失败处理中，既适应JSON前后端分离的应用登录失败结果处理，也适用于模板页面跳转应用的登录失败结果处理
- 登陆失败之后，将默认跳转到默认的failureUrl，即登录界面。



### 配置SecurityConfig

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Resource
    private MyAuthenticationSuccessHandler myAuthenticationSuccessHandler;

    @Resource
    private MyAuthenticationFailureHandler myAuthenticationFailureHandler;

   @Override
   protected void configure(HttpSecurity http) throws Exception {
       http.csrf().disable() //禁用跨站csrf攻击防御，后面的章节会专门讲解
           .formLogin()
           .successHandler(myAuthenticationSuccessHandler)
           .failureHandler(myAuthenticationFailureHandler)
           //.defaultSuccessUrl("/index")//登录认证成功后默认转跳的路径
           //.failureUrl("/login.html") //登录认证是被跳转页面
}
```

- 将自定义的AuthenticationSuccessHandler和AuthenticationFailureHandler注入到Spring Security配置类中
- 使用fromlogin模式，配置successHandler和failureHandler。
- 不要配置defaultSuccessUrl和failureUrl，否则自定义handler将失效。handler配置与URL配置只能二选一



### 自定义权限访问异常

除了登陆成功、登陆失败的结果处理，Spring Security还为我们提供了其他的结果处理类。比如用户未登录就访问系统资源，可以实现 AuthenticationEntryPoint 接口进行响应处理，提示用户应该先去登录

```java
public class MyAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest httpServletRequest, 
                        HttpServletResponse httpServletResponse, 
                        AuthenticationException e) throws IOException, ServletException{
         //仿造上文使用response将响应信息写回
    }
}
```

比如用户登录后访问没有权限访问的资源，可以实现 AccessDeniedHandler 接口进行相应处理，提示用户没有访问权限

```java
public class MyAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest httpServletRequest, 
                       HttpServletResponse httpServletResponse, 
                        AccessDeniedException e) throws IOException, ServletException {
        //仿造上文使用response将响应信息写回
    }
}
```

通过下面的方法进行注册生效

```java
protected void configure(HttpSecurity http) throws Exception {
    //省略其他配置内容
    http.exceptionHandling()
            .accessDeniedHandler(accessDeniedHandler)
            .authenticationEntryPoint(authenticationEntryPoint);
     //省略其他配置内容
}
```



# 认证授权鉴权

## RBAC

### 页面访问权限与操作权限

- 页面访问权限

  所有系统都是由一个个的页面组成，页面再组成模块，用户是否能看到这个页面的菜单、是否能进入这个页面就称为页面访问权限。

- 操作权限

  用户在操作系统中的任何动作、交互都需要有操作权限，如增删改查等。比如：某个按钮，某个超链接用户是否可以点击，是否应该看见的权限。

![img](https://img-note.langyastudio.com/20210419163950.png?x-oss-process=style/watermark)

### 数据权限

数据权限比较好理解，就是某个用户能够访问和操作哪些数据。

- 通常来说，数据权限由用户所属的组织来确定。比如：生产一部只能看自己部门的生产数据，生产二部只能看自己部门的生产数据；销售部门只能看销售数据，不能看财务部门的数据。而公司的总经理可以看所有的数据。
- 在实际的业务系统中，数据权限往往更加复杂。非常有可能销售部门可以看生产部门的数据，以确定销售策略、安排计划等。

所以为了面对复杂的需求，**数据权限的控制通常是由程序员书写个性化的SQL来限制数据范围的**，而不是交给权限模型或者 Spring Security 或 shiro 来控制。当然也可以从权限模型或者权限框架的角度去解决这个问题，但适用性有限。



## 动态加载权限

![img](https://img-note.langyastudio.com/20210419180707.png?x-oss-process=style/watermark)



### UserDetails 与 UserDetailsService接口

UserDetailsService 接口表达的是如何动态加载 UserDetails 数据。

- UserDetailsService 接口有一个方法叫做 loadUserByUsername，我们实现动态加载用户、角色、权限信息就是通过实现该方法。函数见名知义：通过用户名加载用户。该方法的返回值就是 UserDetails。
- UserDetails 就是用户信息，即：用户名、密码、该用户所具有的权限。

下面我们来看一下 UserDetails 接口都有哪些方法：

```java
public interface UserDetails extends Serializable {
    //获取用户的权限集合
    Collection<? extends GrantedAuthority> getAuthorities();

    //获取密码
    String getPassword();

    //获取用户名
    String getUsername();

    //账号是否没过期
    boolean isAccountNonExpired();

    //账号是否没被锁定
    boolean isAccountNonLocked();

    //密码是否没过期
    boolean isCredentialsNonExpired();

    //账户是否可用
    boolean isEnabled();
}
```

现在我们明白了，只要我们把这些信息提供给 Spring Security，Spring Security 就知道怎么做登录验证了，根本不需要我们自己写 Controller 实现登录验证逻辑。那我们怎么把这些信息提供给 Spring Security，用的就是下面的接口方法。

```java
public interface UserDetailsService {
   UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```



### 实现UserDetails 接口

```java
public class MyUserDetails implements UserDetails {

  String password;  //密码
  String username;  //用户名
  boolean accountNonExpired;   //是否没过期
  boolean accountNonLocked;   //是否没被锁定
  boolean credentialsNonExpired;  //密码是否没过期
  boolean enabled;  //账号是否可用
  Collection<? extends GrantedAuthority> authorities;  //用户的权限集合

  public void setPassword(String password) {
    this.password = password;
  }

  public void setUsername(String username) {
    this.username = username;
  }

  public void setAccountNonExpired(boolean accountNonExpired) {
    this.accountNonExpired = accountNonExpired;
  }

  public void setAccountNonLocked(boolean accountNonLocked) {
    this.accountNonLocked = accountNonLocked;
  }

  public void setCredentialsNonExpired(boolean credentialsNonExpired) {
    this.credentialsNonExpired = credentialsNonExpired;
  }

  public void setEnabled(boolean enabled) {
    this.enabled = enabled;
  }

  public void setAuthorities(Collection<? extends GrantedAuthority> authorities) {
    this.authorities = authorities;
  }

  @Override
  public Collection<? extends GrantedAuthority> getAuthorities() {
    return authorities;
  }

  @Override
  public String getPassword() {
    return password;
  }

  @Override
  public String getUsername() {
    return username;
  }

  @Override
  public boolean isAccountNonExpired() {
    return true;   //暂时未用到，直接返回true，表示账户未过期
  }

  @Override
  public boolean isAccountNonLocked() {
    return true;   //暂时未用到，直接返回true，表示账户未被锁定
  }

  @Override
  public boolean isCredentialsNonExpired() {
    return true;   //暂时未用到，直接返回true，表示账户密码未过期
  }

  @Override
  public boolean isEnabled() {
    return enabled;
  }
}
```

我们就是写了一个适应于 UserDetails 的 java POJO 类，所谓的 UserDetails 接口实现就是一些get方法。

- get 方法由 Spring Security 调用，获取认证及鉴权的数据
- 我们通过 set 方法或构造函数为 Spring Security 提供 UserDetails 数据（从数据库查询）
- **当 enabled 的值为 false 的时候，Spring Security 会自动的禁用该用户，禁止该用户进行系统登录。**
- 通常数据库表 sys_user 字段要和 MyUserDetails 属性一一对应，比如 username、password、enabled。

> 目前数据库表里面没有定义accountNonExpired、accountNonLocked、credentialsNonExpired这三个字段，我一般不喜欢搞这么多字段控制用户的登录认证行为，笔者觉得简单点好，一个enabled字段就够了。所以这三个成员变量对应的 get 方法，直接返回 true 即可。
>
> （后续章节实现《多次登陆失败账户锁定功能》的时候，我们用到了 accountNonLocked，到时候我们再到数据库里面添加字段）



### 实现UserDetailsService接口

```java
@Component
public class MyUserDetailsService implements UserDetailsService {

    @Resource
    private MyUserDetailsServiceMapper myUserDetailsServiceMapper;

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {

        //获得用户信息
        MyUserDetails myUserDetails =
                myUserDetailsServiceMapper.findByUserName(username);

        if(myUserDetails == null){
            throw new UsernameNotFoundException("用户名不存在");
        }

        //获得用户角色列表
        List<String> roleCodes =
                myUserDetailsServiceMapper.findRoleByUserName(username);

        //通过角色列表获取权限列表
        List<String> authorities =
                myUserDetailsServiceMapper.findAuthorityByRoleCodes(roleCodes);

        //为角色标识加上ROLE_前缀（Spring Security规范）
        roleCodes = roleCodes.stream()
                .map(rc -> "ROLE_" + rc )
                .collect(Collectors.toList());

        //角色是一种特殊的权限，所以合并
        authorities.addAll(roleCodes);
        
        //转成用逗号分隔的字符串，为用户设置权限标识
        myUserDetails.setAuthorities(
                AuthorityUtils.commaSeparatedStringToAuthorityList(
                    String.join(",",authorities)
                )
        );

        return myUserDetails;
    }
}
```

- 角色是一种特殊的权限，在 Spring Security 我们可以使用 **hasRole**(角色标识)表达式判断用户是否具有某个角色，决定他是否可以做某个操作;通过 **hasAuthority**(权限标识)表达式判断是否具有某个操作权限。
- 上述实现中用到的 MyUserDetailsServiceMapper 是 Mybatis 操作数据库的接口实现



### 注册UserDetailsService

重写 WebSecurityConfigurerAdapter 的 configure(AuthenticationManagerBuilder auth)方法

```java
@Bean("passwordEncoder")
public PasswordEncoder passwordEncoder(){
    return new BCryptPasswordEncoder();
}

@Resource
private MyUserDetailsService userDetailsService;

@Override
protected void configure(AuthenticationManagerBuilder builder) throws Exception {
    builder.userDetailsService(userDetailsService)
           .passwordEncoder(passwordEncoder());
}
```

使用 BCryptPasswordEncoder，表示存储中（数据库）取出的密码必须是经过 BCrypt 加密算法加密的。
这里需要注意的是，因为我们使用了 BCryptPasswordEncoder 加密解密，所以数据库表里面存的密码应该是加密之后的密码（造数据的过程），可以使用如下代码加密（如密码是：123456）。将打印结果保存到密码字段。

```java
@Resource
PasswordEncoder passwordEncoder;

@Test
public void contextLoads() {
    System.out.println(passwordEncoder.encode("123456"));
}
```

![登陆验证使用的载体也是一种Authentication](https://img-note.langyastudio.com/20210428104833.webp?x-oss-process=style/watermark)

### 最后说明

至此，我们将系统里面的所有的用户、角色、权限信息都通过UserDetailsService和UserDetails告知了Spring Security。但是多数朋友可能仍然不知道该怎样实现登录的功能，其实剩下的事情很简单了：

- 写一个登录界面，写一个登录表单，表单使用post方法提交到默认的/login路径
- 表单的用户名、密码字段名称默认是username、password
- 写一个登录成功之后的跳转页面，比如index.html

然后把这些信息通过配置方式告知Spring Security ，以上的配置信息名称都可以灵活修改。如果您不知道如何配置请参考本号之前的文章《formLogin登录认证模式》。



## 权限表达式

### SPEL表达式权限控制

从`spring security 3.0`开始已经可以使用`spring Expression`表达式来控制授权，允许在表达式中使用复杂的布尔逻辑来控制访问的权限。Spring Security可用表达式对象的基类是SecurityExpressionRoot。

| 表达式函数                       | 描述                                                         |
| :------------------------------- | :----------------------------------------------------------- |
| `hasRole([role]`)                | 用户拥有指定的角色时返回true （`Spring security`默认会带有`ROLE_`前缀）,去除前缀参考[Remove the ROLE_](https://github.com/spring-projects/spring-security/issues/4134) |
| `hasAnyRole([role1,role2])`      | 用户拥有任意一个指定的角色时返回true                         |
| `hasAuthority([authority])`      | 拥有某资源的访问权限时返回true                               |
| `hasAnyAuthority([auth1,auth2])` | 拥有某些资源其中部分资源的访问权限时返回true                 |
| `permitAll`                      | 永远返回true                                                 |
| `denyAll`                        | 永远返回false                                                |
| `anonymous`                      | 当前用户是`anonymous`时返回true                              |
| `rememberMe`                     | 当前用户是`rememberMe`用户返回true                           |
| `authentication`                 | 当前登录用户的`authentication`对象                           |
| `fullAuthenticated`              | 当前用户既不是`anonymous`也不是`rememberMe`用户时返回true    |
| `hasIpAddress('192.168.1.0/24')` | 请求发送的IP匹配时返回true                                   |

> 部分朋友可能会对Authority和Role有些混淆。Authority作为资源访问权限可大可小，可以是某按钮的访问权限（如资源ID：biz1），也可以是某类用户角色的访问权限（如资源ID：ADMIN）。当Authority作为角色资源权限时，hasAuthority（'ROLE_ADMIN'）与hasRole（'ADMIN'）是一样的效果。



### SPEL在全局配置中的使用

**URL安全表达式**

```java
config.antMatchers("/system/*")
      .access("hasRole('admin') or hasAuthority('ROLE_admin')")
      .anyRequest()
      .authenticated();
```

这里我们定义了应用`/person/*`URL的范围，只有拥有`ADMIN`或者`USER`权限的用户才能访问这些person资源



### 安全表达式中引用bean

这种方式，比较适合有复杂权限验证逻辑的情况

```java
@Component("rbacService")
@Slf4j
public class RbacService {
    //返回true表示验证通过
    public boolean hasPermission(HttpServletRequest request, Authentication authentication) {
        //验证逻辑代码
        return true;
    }
    public boolean checkUserId(Authentication authentication, int id) {
        //验证逻辑代码
        return true;
    }
}
```

对于"/person/{id}"对应的资源的访问，调用rbacService的bean的方法checkUserId进行权限验证，传递参数为authentication对象和person的id。该id为PathVariable，以#开头表示。

```java
config.antMatchers("/person/{id}")
      .access("@rbacService.checkUserId(authentication,#id)")
      .anyRequest()
      .access("@rbacService.hasPermission(request,authentication)");
```



### Method表达式安全控制

如果我们想实现方法级别的安全配置，`Spring Security`提供了四种注解，分别是`@PreAuthorize` , `@PreFilter` , `@PostAuthorize` 和 `@PostFilter`

**开启方法级别注解的配置**

在Spring安全配置代码中，加上EnableGlobalMethodSecurity注解，开启方法级别安全配置功能

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
```



**使用PreAuthorize注解**

@PreAuthorize 注解适合**进入方法前**的权限验证。只有拥有ADMIN角色才能访问findAll方法

```java
@PreAuthorize("hasRole('admin')")
public List<PersonDemo> findAll(){
    return null;
}
```

> 如果当前登录用户没有PreAuthorize需要的权限，将抛出org.springframework.security.access.AccessDeniedException异常！



**使用PostAuthorize注解**

@PostAuthorize 在**方法执行后**再进行权限验证,适合根据返回值结果进行权限验证。`Spring EL` 提供返回对象能够在表达式语言中获取返回的对象`returnObject`。下文代码只有返回值的name等于authentication对象的name（当前登录用户名）才能正确返回，否则抛出异常

```java
@PostAuthorize("returnObject.name == authentication.name")
public PersonDemo findOne(){
    String authName =
            SecurityContextHolder.getContext().getAuthentication().getName();
    System.out.println(authName);
    return new PersonDemo("admin");
}
```



**使用PreFilter注解**

PreFilter 针对参数进行**过滤**,下文代码表示针对ids参数进行过滤，只有id为偶数的元素才被作为参数传入函数

```java
//当有多个对象是使用filterTarget进行标注
@PreFilter(filterTarget="ids", value="filterObject%2==0")
public void delete(List<Integer> ids, List<String> usernames) {

}
```



**使用PostFilter 注解**

PostFilter 针对返回**结果进行过滤**，特别适用于集合类返回值，过滤集合中不符合表达式的对象

```java
@PostFilter("filterObject.name == authentication.name")
public List<PersonDemo> findAllPD(){

    List<PersonDemo> list = new ArrayList<>();
    list.add(new PersonDemo("kobe"));
    list.add(new PersonDemo("admin"));

    return list;
}
```

如果使用admin登录系统，上面的函数返回值list中kobe将被过滤掉，只剩下admin。



## 单点登录

### 基于session共享

> 特别适合一份应用代码启动多个实例

- 同一IP(域名)，不同端口，在同一个浏览器 cookies 是共享的。

  不同IP(域名)的 Cookies，在同一个浏览器 Cookies 肯定不共享的。对于这种情况需要在集群应用的前面加上负载均衡器逆向代理，如：nginx，haproxy。让客户端看上去访问的是同一个IP(代理IP)，从而浏览器认为基于这个IP的Cookies是共享的。

- SESSION 正常是由 Servlet 容器来维护的（内存里面，每个服务器内存是不共享的），这样集群节点之间 SESSION 就无法共享。如果希望 Session 共享，就需要把 sessionID 的存储放到一个统一的地方，如：redis。SessionID 的维护交给 Spring session 则更加方便。

- 除了 Cookies 可以维持 Sessionid，Spring Session 还提供了了另外一种方式，就是使用 header 传递 SESSIONID。目的是为了方便类似于手机这种没有 cookies 的客户端进行 session 共享。

![img](https://img-note.langyastudio.com/20210420072010.png?x-oss-process=style/watermark)



### 基于CAS单点登录

> 不同的代码启动的应用实例之间实现单点登录
>
> 比如说：chrome、youtube、gmail都是谷歌公司的，但是不同的产品，我们在chrome浏览器登陆了，就直接可以访问YouTube和gmail

![img](https://img-note.langyastudio.com/20210420072409.png?x-oss-process=style/watermark)

需要在应用服务之外单独搭建一个CAS Server，也就是登录认证服务器

1. 用户访问应用1的资源
2. 应用1发现这个用户没有登录，响应302，告知浏览器将该用户的请求重定向到CAS Server进行登录
3. 浏览器重定向到CAS Server
4. CAS Server收到请求，同样确认该用户未登录，响应login页面让其登录
5. 用户输入正确用户名和密码认证后，CAS Server为该用户颁发TGC和ST，并在自己的内存里保存TGT。并告知浏览器重定向回到应用1
   - TGC与TGT的关系就好像sessionid与session的关系。TGT中保存了该登陆用户的信息，TGC是用于匹配这些信息的钥匙
   - TGC保存在cookies里面，在浏览器请求与响应中自动携带。
   - ST比较单纯，Service Ticket，用于访问服务资源，并且只能使用一次
6. 浏览器重定向回到应用1的资源，并携带(TGC、ST)
7. 应用1收到重定向请求后，向CAS Server发请求，根据TGC找到TGT，并验证ST是否有效
8. CAS Server验证该ST是有效的，应用1已知该用户合法。在自己的应用里面记录用户的session信息，并将用户请求的资源响应回浏览器



### 基于JWT

上面的这个CAS Server集中验证的方式虽然能够实现单点登录，但是请求认证过程过于复杂，不够灵活，需要独立维护一个CAS Server。

有状态应用用户的数据是存储在session里面，或者CAS Server提供认证状态，或者redis session状态集中存储。那么无状态应用，用户的登录状态、角色、权限这些数据放在哪？答案就是JWT令牌，JWT实现也有两种方式

- 方式一

  JWT中只含用户的唯一标志，用户每一次访问资源都使用该唯一标志重新去数据库加载用户状态信息数据

- 方式二

  JWT中包含所有的用户状态信息数据，这样做的坏处是增加了网络带宽的负担，但是因为完全不需要session，从而降低了对于内存的需求；也降低了数据库用户状态数据查询操作的需求

用户在应用A上登录认证，应用A会颁发给他一个JWT令牌（一个包含若干用户状态信息的字符串）。当用户访问应用B接口的时候，将这个字符串交给应用B，应用B根据Token中的内容进行鉴权。不同的应用之间按照统一标准发放JWT令牌，统一标准验证JWT令牌。从而你在应用A上获得的令牌，在应用B上也被认可，当然这样这些应用之间底层数据库必须是同一套用户、角色、权限数据。

![img](https://img.kancloud.cn/f6/de/f6de020bb5879c0a3ee53371b0738ee4_878x525.png)

- 认证Controller代码统一
- 鉴权Filter代码统一、校验规则是一样的
- 使用同一套授权数据
- 同一个用于签名和解签的secret



### 基于OAuth2（也可以加JWT）

以上的所有的单点集群登陆方案，都是有一个前提就是：应用A、应用B、应用1、应用2、应用3都是你们公司的，你们公司内部应用之间进行单点登陆验证。

但是我想大家都见过这样一个场景：我们登录某一个网站，然后使用的是我们在QQ、微信上保存的用户数据。也就是说第三方应用想使用某个权威平台的用户数据做登录认证，那么这个权威平台该如何对第三方应用提供认证服务？目前比较通用的做法就是OAuth2（现代化的社交媒体网站登录基本都使用OAuth2）



# 前后端分离

## JWT实现

### 无状态登录

**什么是有状态？**

有状态服务，即服务端需要记录每次会话的客户端信息，从而识别客户端身份，根据用户身份进行请求的处理，典型的设计如Tomcat中的Session。例如登录：用户登录后，我们把用户的信息保存在服务端session中，并且给用户一个cookie值，记录对应的session，然后下次请求，用户携带cookie值来（这一步有浏览器自动完成），我们就能识别到对应session，从而找到用户的信息。这种方式目前来看最方便，但是也有一些缺陷，如下：

- 服务端保存大量数据，增加服务端压力
- 服务端保存用户状态，不支持集群化部署



**什么是无状态**

微服务集群中的每个服务，对外提供的都使用RESTful风格的接口。而RESTful风格的一个最重要的规范就是：服务的无状态性，即：

- 服务端不保存任何客户端请求者信息
- 客户端的每次请求必须具备自描述信息，通过这些信息识别客户端身份

那么这种无状态性有哪些好处呢？

- 客户端请求不依赖服务端的信息，多次请求不需要必须访问到同一台服务器
- 服务端的集群和状态对客户端透明
- 服务端可以任意的迁移和伸缩（可以方便的进行集群化部署）
- 减小服务端存储压力



**如何实现无状态**

无状态登录的流程：

- 首先客户端发送账户名/密码到服务端进行认证
- 认证通过后，服务端将用户信息加密并且编码成一个token，返回给客户端
- 以后客户端每次发送请求，都需要携带认证的token
- 服务端对客户端发送来的token进行解密，判断是否有效，并且获取用户登录信息



JWT 是一个加密后的接口访问密码，并且该密码里面包含用户名信息

- JWT 就像是一把钥匙，用来开你家里的锁。用户把钥匙一旦丢了，家自然是不安全的。其实和使用session管理状态是一样的，一旦网络或浏览器被劫持了，肯定不安全

- JWT 服务端保存了一把钥匙，就是暗号 secret。用来数据的签名和解签，secret 一旦丢失，所有用户都是不安全的

  ![img](https://img-note.langyastudio.com/20210426210202.png?x-oss-process=style/watermark)

- 认证：使用可信用户信息（用户名密码、短信登录）换取带有签名的JWT令牌

- 鉴权：解签JWT令牌，校验用户权限

    

### JWT结构分析

![img](https://img.kancloud.cn/15/b7/15b76e670c51539f5aecaa1e847502d1_692x257.png)

从图中，我们可以看到JWT分为三个部分：

- Header：头部，通常头部有两部分信息：

  - 声明类型，这里是JWT
  - 加密算法，自定义

  我们会对头部进行Base64Url编码（可解码），得到第一部分数据。

- Payload：载荷，就是有效数据，在官方文档中(RFC7519)，这里给了7个示例信息：

  - iss (issuer)：表示签发人
  - exp (expiration time)：表示token过期时间
  - sub (subject)：主题
  - aud (audience)：受众
  - nbf (Not Before)：生效时间
  - iat (Issued At)：签发时间
  - jti (JWT ID)：编号

  这部分也会采用Base64Url编码，得到第二部分数据。

- Signature：签名，是整个数据的认证信息。一般根据前两步的数据，再加上服务的的密钥secret（密钥保存在服务端，不能泄露给客户端），通过Header中配置的加密算法生成。用于验证整个数据完整和可靠性。



### 结合Spring Security认证说明

#### 认证流程细节

![img](https://img-note.langyastudio.com/20210427134324.png?x-oss-process=style/watermark)

- JwtAuthenticationController

  用户登录功能的实现

  如果登录成功，生成JWT令牌

- 用户登录

  需要结合Spring Security实现。通过向Spring Security提供的AuthenticationManager的authenticate()方法传递用户名密码，由spring Security帮我们实现用户登录认证功能

- 登陆成功

  为该用户生成JWT令牌了。通常需要使用UserDetailsService的loadUserByUsername方法加载用户信息，然后根据信息生成JWT令牌，JWT令牌生成之后返回给客户端

- JwtTokenUtil

  该工具类的主要功能就是根据用户信息生成JWT，解签JWT获取用户信息，校验令牌是否过期，刷新令牌等。



#### 接口鉴权细节

当客户端获取到JWT之后，他就可以使用JWT请求接口资源服务了。在到达Controller之前通过Filter过滤器进行JWT解签和权限校验。

![img](https://img-note.langyastudio.com/20210427134922.png?x-oss-process=style/watermark)

假如我们有一个接口资源“/hello”定义在HelloWorldcontroller中，鉴权流程如下：

- 当客户端请求“/hello”资源的时候，需要在HTTP请求的Header带上JWT字符串
- 服务端需要自定义JwtRequestFilter，拦截HTTP请求，并判断请求Header中是否有JWT令牌。如果没有，就执行后续的过滤器。因为Spring Security是有完整的鉴权体系的，你没赋权该请求就是非法的，后续的过滤器链会将该请求拦截，最终返回无权限访问的结果
- 如果在HTTP中解析到JWT令牌，就调用JwtTokenUtil对令牌的有效期及合法性进行判定。如果是伪造的或者过期的，同样返回无权限访问的结果
- 如果JWT令牌在有效期内并且校验通过，仍然要通过UserDetailsService加载该用户的权限信息，并将这些信息交给Spring Security。只有这样，该请求才能顺利通过Spring Security一系列过滤器的关卡，顺利到达HelloWorldcontroller并访问“/hello”接口



#### 其他的细节问题

- 一旦发现用户的JWT令牌被劫持，或者被个人泄露该怎么办？JWT令牌有一个缺点就是一旦发放，在有效期内都是可用的，那怎么回收令牌？我们可以通过设置黑名单ip、用户，或者为每一个用户JWT令牌使用一个secret密钥，可以通过修改secret密钥让该用户的JWT令牌失效
- 如何刷新令牌？为了提高安全性，我们的令牌有效期通常时间不会太长。那么，我们不希望用户正在使用app的时候令牌过期了，用户必须重新登陆，很影响用户体验。这怎么办？这就需要在客户端根据业务选择合适的时机或者定时的刷新JWT令牌。所谓的刷新令牌就是用有效期内，用旧的合法的JWT换取新的JWT。



#### JWT 存在的问题

说了这么多，JWT 也不是天衣无缝，由客户端维护登录状态带来的一些问题在这里依然存在，举例如下：

1. 续签问题，这是被很多人诟病的问题之一，传统的cookie+session的方案天然的支持续签，但是jwt由于服务端不保存用户状态，因此很难完美解决续签问题，如果引入redis，虽然可以解决问题，但是jwt也变得不伦不类了。
2. 注销问题，由于服务端不再保存用户信息，所以一般可以通过修改secret来实现注销，服务端secret修改后，已经颁发的未过期的token就会认证失败，进而实现注销，不过毕竟没有传统的注销方便。
3. 密码重置，密码重置后，原本的token依然可以访问系统，这时候也需要强制修改secret。
4. 基于第2点和第3点，一般建议不同用户取不同secret。



## Spring-CORS 规则基础配置

想在Spring或Spring Boot的web环境下实现跨域资源共享，主要有三种实现方式：

- @CrossOrigin注解，这个注解是作用于Controller类或者请求方法上的，实现局部接口的跨域资源共享
- 实现WebMvcConfigurer接口addCorsMappings方法，实现全局配置的跨域资源共享
- 注入CorsFilter过滤器，实现全局配置的跨域资源共享。推荐使用



#### Spring Security 中的配置CORS

当我们的应用使用了Spring Security之后，我们会发现上面的配置方法全部失效。此时需要在spring security的WebSecurityConfigurerAdapter中的configure(HttpSecurity http)配置方法，加上`http.cors()`配置，第二小节中的配置才会生效。

```java
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors().and()
        ...
    }
}
```

另外Spring Security为我们提供了一种新的CORS规则的配置方法：CorsConfigurationSource 。使用这种方法实现的效果等同于注入一个CorsFilter过滤器。

```java
@Bean
CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration configuration = new CorsConfiguration();
    configuration.setAllowedOrigins(Arrays.asList("http://localhost:8888"));
    configuration.setAllowedMethods(Arrays.asList("GET","POST"));
    configuration.applyPermitDefaultValues();
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", configuration);
    return source;
}
```


















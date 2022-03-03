通过认证服务进行统一认证，然后通过网关来统一校验认证和鉴权。

> 将采用 Nacos 作为注册中心，Gateway 作为网关，使用 `nimbus-jose-jwt` JWT 库操作 JWT 令牌



## 理论介绍

> Spring Security 是强大的且容易定制的，基于 Spring 开发的实现**认证登录与资源授权**的应用安全框架

SpringSecurity 的核心功能：

- **Authentication**：身份认证，用户登陆的验证（解决你是谁的问题）
- **Authorization**：访问授权，授权系统资源的访问权限（解决你能干什么的问题）
- 安全防护，防止跨站请求，session 攻击等



### SpringSecurity 配置类

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



### JWT 单点登录

用户在应用 A 上登录认证，应用 A 会颁发给他一个 JWT 令牌（一个包含若干用户状态信息的字符串）。当用户访问应用 B 接口的时候，将这个字符串交给应用 B，应用 B 根据 Token 中的内容进行鉴权。不同的应用之间按照统一标准发放 JWT令牌，统一标准验证 JWT 令牌。从而你在应用 A 上获得的令牌，在应用 B 上也被认可，当然这样这些应用之间底层数据库必须是同一套用户、角色、权限数据。

![img](https://img.kancloud.cn/f6/de/f6de020bb5879c0a3ee53371b0738ee4_878x525.png)

- 认证 Controller 代码统一
- 鉴权 Filter 代码统一、校验规则是一样的
- 使用同一套授权数据
- 同一个用于签名和解签的 secret



**JWT 存在的问题**

说了这么多，JWT 也不是天衣无缝，由客户端维护登录状态带来的一些问题在这里依然存在，举例如下：

- 续签问题，这是被很多人诟病的问题之一，传统的cookie+session的方案天然的支持续签，但是jwt由于服务端不保存用户状态，因此很难完美解决续签问题，如果引入redis，虽然可以解决问题，但是jwt也变得不伦不类了

- 注销问题，由于服务端不再保存用户信息，所以一般可以通过修改secret来实现注销，服务端secret修改后，已经颁发的未过期的token就会认证失败，进而实现注销，不过毕竟没有传统的注销方便

- 密码重置，密码重置后，原本的token依然可以访问系统，这时候也需要强制修改secret

- 基于第2点和第3点，一般**建议不同用户取不同secret**



### OAuth2

以上的所有的单点集群登陆方案，都是有一个前提就是：应用A、应用B、应用1、应用2、应用3都是你们公司的，你们公司内部应用之间进行单点登陆验证。

但是大家都见过这样一个场景：登录某一个网站，然后使用的是在QQ、微信上保存的用户数据。也就是说第三方应用想使用某个权威平台的用户数据做登录认证，那么这个权威平台该如何对第三方应用提供认证服务？目前比较通用的做法就是OAuth2（现代化的社交媒体网站登录基本都使用OAuth2）

- Spring Security OAuth 项目进入维护状态，不再做新特性的开发。只做功能维护和次要特性开发
- 未来所有的基于 Spring 的 OAuth2.0 的支持都基于 Spring Security 5.2 版本开发。即：Spring Security 5.2 以后的版本是 OAuth2.0 支持库，用来**替换 Spring Security OAuth**



#### OAuth2 需求场景

在说明OAuth2需求及使用场景之前，需要先介绍一下OAuth2授权流程中的各种角色：

- 资源拥有者（User） - 指应用的用户，通常指的是系统的登录用户
- 认证服务器 （Authorization Server）- 提供登录认证接口的服务器，比如：github登录、QQ登录、微信登录等
- 资源服务器 （Resources Server） - 提供资源接口及服务的服务器，比如：用户信息接口等。通常和认证服务器是同一个应用。
- 第三方客户端（Client） - 第三方应用，希望使用资源服务器提供的资源
- 服务提供商(Provider): 认证服务和资源服务归属于一个机构，该机构就是服务提供商

![img](https://img-note.langyastudio.com/202201181218908.png?x-oss-process=style/watermark)



#### OAuth2 四种授权模式

- 授权码模式（authorization code）
- 简化模式（implicit）
- 密码模式（resource owner password credentials）
- 客户端模式（client credentials）

密码模式与授权码模式最大的区别在于：

- 授权码模式申请授权码的过程是用户直接与认证服务器进行交互，然后授权结果由认证服务器告知第三方客户端，也就是不会向第三方客户端暴露服务提供商的用户密码信息
- 密码模式，是用户将用户密码信息交给第三方客户端，然后由第三方向服务提供商进行认证和资源请求。绝大多数的服务提供商都会选择使用授权码模式，避免自己的用户密码暴漏给第三方。所以密码模式只适用于服务提供商对第三方厂商（第三方应用）高度信任的情况下才能使用，或者这个“第三方应用”实际就是服务提供商自己的应用



#### 整合 JWT

因为 Spring Security OAuth “认证服务器”支持多种认证模式，所以不想抛弃它。但是想把最后的"资源访问令牌"，由 AccessToken 换成 JWT 令牌。因为 AccessToken 不带有任何的附加信息，就是一个字符串，JWT 是可以携带附加信息的。

![img](https://img.kancloud.cn/79/6b/796bb4344db0e061c4e5d7084d2f8614_787x428.png)



## 应用架构

> 理想的解决方案应该是这样的，认证服务负责认证，网关负责校验认证和鉴权，其他 API 服务负责处理自己的业务逻辑。安全相关的逻辑只存在于认证服务和网关服务中，其他服务只是单纯地提供服务而没有任何安全相关逻辑

相关服务划分：

- security-oauth2-gateway：网关服务，负责请求转发和鉴权功能，整合 Spring Security+Oauth2
- security-oauth2-auth：Oauth2认证服务，负责对登录用户进行认证，整合 Spring Security+Oauth2
- security-oauth2-api：受保护的 API 服务，用户鉴权通过后可以访问该服务，不整合 Spring Security+Oauth2

![图片](https://img-note.langyastudio.com/202201180952033.webp?x-oss-process=style/watermark)



## 方案实现

> 下面介绍下这套解决方案的具体实现，依次搭建认证服务、网关服务和 API 服务

### security-oauth2-auth 认证

> 首先来搭建认证服务，它将作为 Oauth2 的认证服务使用，并且网关服务的鉴权功能也需要依赖它

#### pom 依赖

- 在 `pom.xml ` 中添加相关依赖，主要是 Spring Security、Oauth2、JWT、Redis 相关依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security.oauth.boot</groupId>
        <artifactId>spring-security-oauth2-autoconfigure</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-rsa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.nimbusds</groupId>
        <artifactId>nimbus-jose-jwt</artifactId>
    </dependency>    
</dependencies>
```

- 在 `application.yml` 中添加相关配置，主要是 Nacos 和 Redis 相关配置

```yaml
server:
  port: 9401

spring:
  profiles:
    active: dev
  
  application:
    name: security-oauth2-auth
  
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.123.22:8848
        username: nacos
        password: nacos       

  redis:
    port: 6379
    host: localhost
    password: xxx
    
management:
  endpoints:
    web:
      exposure:
        include: "*"
```



#### 认证服务配置

##### 生成密钥库

- 使用 `keytool` 生成 RSA 证书 `jwt.jks`，复制到 `resource` 目录下，在 JDK 的 `bin` 目录下使用如下命令即可

```bash
keytool -genkey -alias jwt -keyalg RSA -keystore jwt.jks
```



##### 加载用户信息

- 创建 `UserServiceImpl` 类实现 Spring Security 的 `UserDetailsService` 接口，用于加载用户信息

```java
/**
 * 用户管理业务类
 */
@Service
public class UserDetailsServiceImpl implements UserDetailsService
{
    @Autowired
    private UmsAdminService    adminService;
    @Autowired
    private HttpServletRequest request;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException
    {
        String  clientId = request.getParameter("client_id");
        UserDto userDto  = null;
        if (AuthConstant.ADMIN_CLIENT_ID.equals(clientId))
        {
            userDto = adminService.loadUserByUsername(username);
        }
        if (null == userDto)
        {
            throw new UsernameNotFoundException(EC.ERROR_USER_PASSWORD_INCORRECT.getMsg());
        }

        SecurityUserDetails securityUser = new SecurityUserDetails(userDto);
        if (!securityUser.isEnabled())
        {
            throw new DisabledException(EC.ERROR_USER_ENABLED.getMsg());
        }
        else if (!securityUser.isAccountNonLocked())
        {
            throw new LockedException(EC.ERROR_USER_LOCKED.getMsg());
        }
        else if (!securityUser.isAccountNonExpired())
        {
            throw new AccountExpiredException(EC.ERROR_USER_EXPIRE.getMsg());
        }
        else if (!securityUser.isCredentialsNonExpired())
        {
            throw new CredentialsExpiredException(EC.ERROR_USER_UNAUTHORIZED.getMsg());
        }
        return securityUser;
    }

}

@Component
public class UmsAdminService
{
    @Autowired
    private PasswordEncoder passwordEncoder;

    public UserDto loadUserByUsername(String username)
    {
        String  password = passwordEncoder.encode("123456a");
        if("admin".equals(username))
        {
            return new UserDto("admin", password, 1, "", CollUtil.toList("ADMIN"));
        }
        else if("langya".equals(username))
        {
            return new UserDto("langya", password, 1, "", CollUtil.toList("ADMIN", "TEST"));
        }

        return null;
    }
}
```



##### 认证服务配置

- 添加认证服务相关配置 `Oauth2ServerConfig`，需要配置加载用户信息的服务 `UserServiceImpl` 及 RSA 的钥匙对`KeyPair`

```java
/**
 * 认证服务器配置
 */
@AllArgsConstructor
@Configuration
@EnableAuthorizationServer
public class Oauth2ServerConfig extends AuthorizationServerConfigurerAdapter
{
    private final PasswordEncoder        passwordEncoder;
    private final UserDetailsServiceImpl userDetailsService;
    private final AuthenticationManager  authenticationManager;
    private final JwtTokenEnhancer       jwtTokenEnhancer;

    /**
     * 客户端信息配置
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception
    {
        clients.inMemory()
                .withClient(AuthConstant.ADMIN_CLIENT_ID)
                .secret(passwordEncoder.encode("123456"))
                .scopes("all")
                .authorizedGrantTypes("password", "refresh_token")
                .accessTokenValiditySeconds(3600 * 24)
                .refreshTokenValiditySeconds(3600 * 24 * 7)
                .and()
                .withClient(AuthConstant.PORTAL_CLIENT_ID)
                .secret(passwordEncoder.encode("123456"))
                .scopes("all")
                .authorizedGrantTypes("password", "refresh_token")
                .accessTokenValiditySeconds(3600 * 24)
                .refreshTokenValiditySeconds(3600 * 24 * 7);
    }

    /**
     * 配置授权（authorization）以及令牌（token）
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception
    {
        TokenEnhancerChain  enhancerChain = new TokenEnhancerChain();
        List<TokenEnhancer> delegates     = new ArrayList<>();
        delegates.add(jwtTokenEnhancer);
        delegates.add(accessTokenConverter());

        //配置JWT的内容增强器
        enhancerChain.setTokenEnhancers(delegates);

        endpoints.authenticationManager(authenticationManager)
                //配置加载用户信息的服务
                .userDetailsService(userDetailsService)
                .accessTokenConverter(accessTokenConverter())
                .tokenEnhancer(enhancerChain);
    }

    /**
     * 允许表单认证
     */
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception
    {
        security.allowFormAuthenticationForClients();
    }

    /**
     * 使用非对称加密算法对token签名
     */
    @Bean
    public JwtAccessTokenConverter accessTokenConverter()
    {
        JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
        //or 设置对称签名
        //jwtAccessTokenConverter.setSigningKey("2430B31859314947BC84697E70B3D31F");
        jwtAccessTokenConverter.setKeyPair(keyPair());
        return jwtAccessTokenConverter;
    }

    /**
     * 从classpath下的密钥库中获取密钥对(公钥+私钥)
     */
    @Bean
    public KeyPair keyPair()
    {
        //从classpath下的证书中获取秘钥对
        KeyStoreKeyFactory keyStoreKeyFactory = new KeyStoreKeyFactory(new ClassPathResource("jwt.jks"),
                                                                       "123456".toCharArray());
        return keyStoreKeyFactory.getKeyPair("jwt", "123456".toCharArray());
    }
}

```



##### jwt 增强

- 如果你想往 JWT 中添加自定义信息的话，比如说 `登录用户的ID`，可以自己实现 `TokenEnhancer` 接口

```java
/**
 * JWT 内容增强器
 */
@Component
public class JwtTokenEnhancer implements TokenEnhancer
{
    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication)
    {
        SecurityUserDetails securityUser = (SecurityUserDetails) authentication.getPrincipal();

        //把用户名设置到JWT中
        Map<String, Object> info = new HashMap<>();
        info.put("user_name", securityUser.getUsername());
        info.put("client_id", securityUser.getClientId());
        ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(info);
        return accessToken;
    }
}
```



##### 公钥获取接口

- 由于的网关服务需要 RSA 的公钥来验证签名是否合法，所以认证服务需要有个接口把公钥暴露出来

```java
/**
 * 获取RSA公钥接口
 */
@RestController
public class KeyPairController
{
    @Autowired
    private KeyPair keyPair;

    @GetMapping("/rsa/publicKey")
    public Map<String, Object> getKey()
    {
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        RSAKey       key       = new RSAKey.Builder(publicKey).build();
        return new JWKSet(key).toJSONObject();
    }
}
```



#### 安全配置

- 不要忘了还需要配置 Spring Security，允许获取公钥接口的访问

```java
/**
 * SpringSecurity 安全配置
 */
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter
{
    @Override
    protected void configure(HttpSecurity http) throws Exception
    {
        http.authorizeRequests()
                .requestMatchers(EndpointRequest.toAnyEndpoint()).permitAll()
                .antMatchers("/rsa/publicKey").permitAll()
                .anyRequest().authenticated();
    }

    /**
     *  如果不配置 SpringBoot 会自动配置一个 AuthenticationManager 覆盖掉内存中的用户
     */
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception
    {
        return super.authenticationManagerBean();
    }

    @Bean
    public PasswordEncoder passwordEncoder()
    {
        return new BCryptPasswordEncoder();
    }
}
```



#### 资源角色映射缓存

- 创建一个资源服务 `ResourceServiceImpl`，初始化的时候把资源与角色匹配关系缓存到 Redis 中，方便网关服务进行鉴权的时候获取

```java
/**
 * 资源与角色匹配关系管理业务类
 * <p>
 * 初始化的时候把资源与角色匹配关系缓存到Redis中，方便网关服务进行鉴权的时候获取
 */
@Service
public class ResourceServiceImpl
{
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    private Map<String, List<String>> resourceRolesMap;

    @PostConstruct
    public void initData()
    {
        resourceRolesMap = new TreeMap<>();
        resourceRolesMap.put("/admin/hello", CollUtil.toList("ADMIN"));
        resourceRolesMap.put("/admin/user/currentUser", CollUtil.toList("ADMIN", "TEST"));
        redisTemplate.opsForHash().putAll(AuthConstant.RESOURCE_ROLES_MAP_KEY, resourceRolesMap);
    }
}
```



如果资源权限存储到数据库，也可以直接使用 SQL 语句形成结果集，如：

![img](https://img-note.langyastudio.com/202201182324673.png?x-oss-process=style/watermark)



### security-oauth2-gateway 鉴权

> 接下来就可以搭建网关服务了，它将作为 Oauth2 的资源服务、客户端服务使用，对访问微服务的请求进行统一的校验认证和鉴权操作

#### pom 依赖

- 在 `pom.xml` 中添加相关依赖，主要是 Gateway、Oauth2 和 JWT 相关依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    <!--lb:// need-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-resource-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-jose</artifactId>
    </dependency>
    <dependency>
        <groupId>com.nimbusds</groupId>
        <artifactId>nimbus-jose-jwt</artifactId>
    </dependency>
</dependencies>
```

- 在 `application.yml` 中添加相关配置，主要是路由规则的配置、Oauth2中RSA公钥的配置及路由白名单的配置

```yaml
server:
  port: 9201

spring:
  main:
    #springcloudgateway 的内部是通过 netty+webflux 实现的
    #webflux 实现和 spring-boot-starter-web 依赖冲突
    web-application-type: reactive

  profiles:
    active: dev

  application:
    name: security-oauth2-gateway

  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        username: nacos
        password: nacos

    gateway:
      routes: #配置路由路径
        - id: oauth2-api-route
          uri: lb://security-oauth2-api
          predicates:
            - Path=/admin/**
          filters:
            - StripPrefix=1
        - id: oauth2-auth-route
          uri: lb://security-oauth2-auth
          predicates:
            - Path=/auth/**
          filters:
            - StripPrefix=1
      discovery:
        locator:
          #开启从注册中心动态创建路由的功能
          enabled: true
          #使用小写服务名，默认是大写
          lower-case-service-id: true

  security:
    oauth2:
      resourceserver:
        jwt:
          #配置RSA的公钥访问地址
          jwk-set-uri: 'http://localhost:9401/rsa/publicKey'

  redis:
    host: 192.168.123.22
    port: 6379
    password: Hacfin_Redis8
    timeout: 6000ms

secure:
  ignore:
    #配置白名单路径
    urls:
      - "/actuator/**"
      - "/auth/oauth/token"
```



#### 资源服务器配置

- 对网关服务进行配置安全配置，由于 Gateway 使用的是 `WebFlux`，所以需要使用 `@EnableWebFluxSecurity`注解开启

```java
/**
 * 资源服务器配置
 */
@AllArgsConstructor
@Configuration
@EnableWebFluxSecurity
public class ResourceServerConfig
{
    private final AuthorizationManager         authorizationManager;
    private final IgnoreUrlsConfig             ignoreUrlsConfig;
    private final RestfulAccessDeniedHandler   restfulAccessDeniedHandler;
    private final RestAuthenticationEntryPoint restAuthenticationEntryPoint;
    private final IgnoreUrlsRemoveJwtFilter    ignoreUrlsRemoveJwtFilter;

    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http)
    {
        http.oauth2ResourceServer().jwt()
                .jwtAuthenticationConverter(jwtAuthenticationConverter());

        //自定义处理JWT请求头过期或签名错误的结果
        http.oauth2ResourceServer().authenticationEntryPoint(restAuthenticationEntryPoint);

        //对白名单路径，直接移除JWT请求头
        http.addFilterBefore(ignoreUrlsRemoveJwtFilter, SecurityWebFiltersOrder.AUTHENTICATION);

        http.authorizeExchange()
                //白名单配置
                .pathMatchers(ArrayUtil.toArray(ignoreUrlsConfig.getUrls(), String.class)).permitAll()
                //鉴权管理器配置
                .anyExchange().access(authorizationManager)
                .and()
                .exceptionHandling()
                //处理未授权
                .accessDeniedHandler(restfulAccessDeniedHandler)
                //处理未认证
                .authenticationEntryPoint(restAuthenticationEntryPoint)
                .and()
                .csrf().disable();

        return http.build();
    }

    /**
     * @linkhttps://blog.csdn.net/qq_24230139/article/details/105091273
     * ServerHttpSecurity 没有将 jwt 中 authorities 的负载部分当做 Authentication
     * 需要把 jwt 的 Claim 中的 authorities 加入
     * 方案：重新定义 ReactiveAuthenticationManager 权限管理器，默认转换器 JwtGrantedAuthoritiesConverter
     */
    @Bean
    public Converter<Jwt, ? extends Mono<? extends AbstractAuthenticationToken>> jwtAuthenticationConverter()
    {
        JwtGrantedAuthoritiesConverter jwtGrantedAuthoritiesConverter = new JwtGrantedAuthoritiesConverter();
        jwtGrantedAuthoritiesConverter.setAuthorityPrefix(AuthConstant.AUTHORITY_PREFIX);
        jwtGrantedAuthoritiesConverter.setAuthoritiesClaimName(AuthConstant.AUTHORITY_CLAIM_NAME);

        JwtAuthenticationConverter jwtAuthenticationConverter = new JwtAuthenticationConverter();
        jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(jwtGrantedAuthoritiesConverter);
        return new ReactiveJwtAuthenticationConverterAdapter(jwtAuthenticationConverter);
    }
}

```



#### 鉴权管理器

- 在 `WebFluxSecurity` 中自定义鉴权操作需要实现 `ReactiveAuthorizationManager` 接口

```java
@Component
public class AuthorizationManager implements ReactiveAuthorizationManager<AuthorizationContext> {
    @Autowired
    private RedisTemplate<String,Object> redisTemplate;

    @Override
    public Mono<AuthorizationDecision> check(Mono<Authentication> mono, AuthorizationContext authorizationContext) {
        //从Redis中获取当前路径可访问角色列表
        URI uri = authorizationContext.getExchange().getRequest().getURI();
        Object obj = redisTemplate.opsForHash().get(RedisConstant.RESOURCE_ROLES_MAP, uri.getPath());
        List<String> authorities = Convert.toList(String.class,obj);
        authorities = authorities.stream().map(i -> i = AuthConstant.AUTHORITY_PREFIX + i).collect(Collectors.toList());
        //认证通过且角色匹配的用户可访问当前路径
        return mono
                .filter(Authentication::isAuthenticated)
                .flatMapIterable(Authentication::getAuthorities)
                .map(GrantedAuthority::getAuthority)
                .any(authorities::contains)
                .map(AuthorizationDecision::new)
                .defaultIfEmpty(new AuthorizationDecision(false));
    }
}
```



#### 过滤器

- 这里还需要实现一个全局过滤器 `AuthGlobalFilter`，当鉴权通过后将 JWT 令牌中的用户信息解析出来，然后存入请求的 Header 中，这样后续服务就不需要解析 JWT 令牌了，可以直接从请求的 Header 中获取到用户信息

```java
/**
 * 将登录用户的JWT转化成用户信息的全局过滤器
 */
@Component
public class AuthGlobalFilter implements GlobalFilter, Ordered
{
    @Autowired
    private RedisTemplate redisTemplate;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain)
    {
        //认证信息从Header 或 请求参数 中获取
        ServerHttpRequest serverHttpRequest = exchange.getRequest();
        String token = serverHttpRequest.getHeaders().getFirst(AuthConstant.JWT_TOKEN_HEADER);
        if (Objects.isNull(token))
        {
            token = serverHttpRequest.getQueryParams().getFirst(AuthConstant.JWT_TOKEN_HEADER);
        }

        if (StrUtil.isEmpty(token))
        {
            return chain.filter(exchange);
        }
        try
        {
            //从token中解析用户信息并设置到Header中去
            String    realToken = token.replace(AuthConstant.JWT_TOKEN_PREFIX, "");
            JWSObject jwsObject = JWSObject.parse(realToken);
            String    userStr   = jwsObject.getPayload().toString();

            // 黑名单token(登出、修改密码)校验
            JSONObject jsonObject = JSONUtil.parseObj(userStr);
            String     jti        = jsonObject.getStr("jti");

            Boolean isBlack = redisTemplate.hasKey(AuthConstant.TOKEN_BLACKLIST_PREFIX + jti);
            if (isBlack)
            {

            }

            // 存在token且不是黑名单，request写入JWT的载体信息
            ServerHttpRequest request = serverHttpRequest.mutate().header(AuthConstant.USER_TOKEN_HEADER, userStr).build();
            exchange = exchange.mutate().request(request).build();
        }
        catch (ParseException e)
        {
            e.printStackTrace();
        }

        return chain.filter(exchange);
    }

    @Override
    public int getOrder()
    {
        return 0;
    }
}
```



### security-oauth2-api 

> 最后搭建一个API服务，它不会集成和实现任何安全相关逻辑，全靠网关来保护它

#### pom 依赖

- 在`pom.xml`中添加相关依赖，就添加了一个web依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```



#### 登录信息接口

- 创建一个 `LoginUserHolder` 组件，用于从请求的 Header 中直接获取登录用户信息

```java
/**
 * 获取登录用户信息
 */
@Component
public class LoginUserHolder
{
    public UserDto getCurrentUser(HttpServletRequest request)
    {
        String userStr = request.getHeader(AuthConstant.USER_TOKEN_HEADER);

        JSONObject userJsonObject = new JSONObject(userStr);

        UserDto userDTO = new UserDto();
        userDTO.setUserName(userJsonObject.getStr("user_name"));
        userDTO.setClientId(userJsonObject.getStr("client_id"));
        userDTO.setRoles(Convert.toList(String.class, userJsonObject.get(AuthConstant.AUTHORITY_CLAIM_NAME)));
        return userDTO;
    }
}
```

- 创建一个获取当前用户信息的接口

```java
@RestController
@RequestMapping("/user")
public class UserController{
    @Autowired
    private LoginUserHolder loginUserHolder;

    @GetMapping("/currentUser")
    public UserDTO currentUser() {
        return loginUserHolder.getCurrentUser();
    }
}
```



## 功能演示

> 接下来来演示下微服务系统中的统一认证鉴权功能，所有请求均通过网关访问

- 启动 Nacos 和 Redis 服务

- 启动 `security-oauth2-auth`、`security-oauth2-gateway` 及 `security-oauth2-api` 服务

![image-20220118174027689](https://img-note.langyastudio.com/202201181740897.png?x-oss-process=style/watermark)



- 使用密码模式获取 JWT 令牌，**POST** 访问地址：http://localhost:9201/auth/oauth/token

![image-20220118174115511](https://img-note.langyastudio.com/202201181741628.png?x-oss-process=style/watermark)

- 使用获取到的JWT令牌访问获取当前登录用户信息的接口，访问地址：http://localhost:9201/api/user/currentUser

![image-20220118174148483](https://img-note.langyastudio.com/202201181741589.png?x-oss-process=style/watermark)

- 当 JWT 令牌过期时，使用 refresh_token 获取新的 JWT令牌，访问地址：http://localhost:9201/auth/oauth/token

![image-20220118174719053](https://img-note.langyastudio.com/202201181747148.png?x-oss-process=style/watermark)



## 扩展





## 参考

[Spring Cloud Gateway + Oauth2 实现统一认证和鉴权](http://www.macrozheng.com/#/cloud/gateway_oauth2)

[Spring Cloud Gateway + Spring Security OAuth2 + JWT 实现微服务统一认证授权鉴权](https://www.cnblogs.com/haoxianrui/p/13719356.html)

[Oauth2 自定义处理结果的最佳方案](http://www.macrozheng.com/#/cloud/oauth2_custom)

[听说你的JWT库用起来特别扭，推荐这款贼好用的！](https://mp.weixin.qq.com/s/Jo3PZoa7nL99c8UCxPiTTA)
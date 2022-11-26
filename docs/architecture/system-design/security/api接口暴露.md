> 本文来自不才陈某，郎涯进行简单排版与补充

在业务开发的时候，经常会遇到某一个接口不能对外暴露，只能内网服务间调用的实际需求。面对这样的情况，我们该如何实现呢？今天，我们就来理一理这个问题，从几个可行的方案中，挑选一个来实现。



## 内外网接口微服务隔离

将对外暴露的接口和对内暴露的接口分别放到两个微服务上，一个服务里所有的接口均对外暴露，另一个服务的接口只能内网服务间调用。

该方案需要额外编写一个只对内部暴露接口的微服务，将所有只能对内暴露的业务接口聚合到这个微服务里，通过这个聚合的微服务，分别去各个业务侧获取资源。

该方案，新增一个微服务做请求转发，增加了系统的复杂性，增大了调用耗时以及后期的维护成本。



## 网关 + redis 实现白名单机制

在 redis 里维护一套接口白名单列表，外部请求到达网关时，从 redis 获取接口白名单，在白名单内的接口放行，反之拒绝掉。

该方案的好处是，对业务代码零侵入，只需要维护好白名单列表即可；

不足之处在于，白名单的维护是一个持续性投入的工作，在很多公司，业务开发无法直接触及到 redis，只能提工单申请，增加了开发成本；另外，每次请求进来，都需要判断白名单，增加了系统响应耗时，考虑到正常情况下外部进来的请求大部分都是在白名单内的，只有极少数恶意请求才会被白名单机制所拦截，所以该方案的性价比很低。



## 方案三 网关 + AOP

相比于方案二对接口进行白名单判断而言，方案三是对请求来源进行判断，并将该判断下沉到业务侧。避免了网关侧的逻辑判断，从而提升系统响应速度。

我们知道，外部进来的请求一定会经过网关再被分发到具体的业务侧，内部服务间的调用是不用走外部网关的（走 k8s 的 service）。

**根据这个特点，我们可以对所有经过网关的请求的header里添加一个字段，业务侧接口收到请求后，判断header里是否有该字段，如果有，则说明该请求来自外部，没有，则属于内部服务的调用，再根据该接口是否属于内部接口来决定是否放行该请求。**

该方案将内外网访问权限的处理分布到各个业务侧进行，消除了由网关来处理的系统性瓶颈；同时，开发者可以在业务侧直接确定接口的内外网访问权限，提升开发效率的同时，增加了代码的可读性。

当然该方案会对业务代码有一定的侵入性，不过可以通过注解的形式，最大限度的降低这种侵入性。

![图片](https://img-note.langyastudio.com/202210111411042.jpeg?x-oss-process=style/watermark)



## 具体实操

下面就方案三，进行具体的代码演示。

首先在网关侧，需要对进来的请求header添加外网标识符: from=public

```java
@Component
public class AuthFilter implements GlobalFilter, Ordered {
    @Override
    public Mono < Void > filter ( ServerWebExchange exchange, GatewayFilterChain chain ) {
         return chain.filter(
         exchange.mutate().request(
         exchange.getRequest().mutate().header("id", "").header("from", "public").build())
         .build()
         )；
    }

    @Override
    public int getOrder () {
        return 0;
    }
 }
```

接着，编写内外网访问权限判断的AOP和注解

```java
@Aspect
@Component
@Slf4j
public class OnlyIntranetAccessAspect {
 @Pointcut ( "@within(org.openmmlab.platform.common.annotation.OnlyIntranetAccess)" )
 public void onlyIntranetAccessOnClass () {}
 @Pointcut ( "@annotation(org.openmmlab.platform.common.annotation.OnlyIntranetAccess)" )
 public void onlyIntranetAccessOnMethed () {
 }

 @Before ( value = "onlyIntranetAccessOnMethed() || onlyIntranetAccessOnClass()" )
 public void before () {
     HttpServletRequest hsr = (( ServletRequestAttributes ) RequestContextHolder.getRequestAttributes()) .getRequest ();
     String from = hsr.getHeader ( "from" );
     if ( !StringUtils.isEmpty( from ) && "public".equals ( from )) {
        log.error ( "This api is only allowed invoked by intranet source" );
        throw new MMException ( ReturnEnum.C_NETWORK_INTERNET_ACCESS_NOT_ALLOWED_ERROR);
            }
     }
 }

@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface OnlyIntranetAccess {
}
```

最后，在只能内网访问的接口上加上@OnlyIntranetAccess注解即可

```java
@GetMapping ( "/role/add" )
@OnlyIntranetAccess
public String onlyIntranetAccess() {
    return "该接口只允许内部服务调用";
}
```
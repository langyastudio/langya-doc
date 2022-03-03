Spring Cloud Ribbon 是Spring Cloud Netflix 子项目的核心组件之一，主要给服务间调用及 API 网关转发提供负载均衡的功能。

> 从 `SpringCloud 2020` 版本开始被 **移除**，替代品为 spring-cloud-loadbalancer



### 全局配置

```yaml
ribbon:
  ConnectTimeout: 1000 #服务请求连接超时时间（毫秒）
  ReadTimeout: 3000 #服务请求处理超时时间（毫秒）
  OkToRetryOnAllOperations: true #对超时请求启用重试机制
  MaxAutoRetriesNextServer: 1 #切换重试实例的最大个数
  MaxAutoRetries: 1 # 切换实例后重试最大次数
  NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #修改负载均衡算法
```



### 指定服务进行配置

与全局配置的区别就是 ribbon 节点挂在服务名称下面，如下是对调用 user-service 时的单独配置

```yaml
user-service:
  ribbon:
    ConnectTimeout: 1000 #服务请求连接超时时间（毫秒）
    ReadTimeout: 3000 #服务请求处理超时时间（毫秒）
    OkToRetryOnAllOperations: true #对超时请求启用重试机制
    MaxAutoRetriesNextServer: 1 #切换重试实例的最大个数
    MaxAutoRetries: 1 # 切换实例后重试最大次数
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #修改负载均衡算法
```



### Ribbon的负载均衡策略

所谓的负载均衡策略，就是当 A 服务调用 B 服务时，此时 B 服务有多个实例，这时 A 服务以何种方式来选择调用的 B 实例，ribbon 可以选择以下几种负载均衡策略

- com.netflix.loadbalancer.RandomRule：从提供服务的实例中以随机的方式
- com.netflix.loadbalancer.RoundRobinRule：以线性轮询的方式，就是维护一个计数器，从提供服务的实例中按顺序选取，第一次选第一个，第二次选第二个，以此类推，到最后一个以后再从头来过
- com.netflix.loadbalancer.RetryRule：在RoundRobinRule的基础上添加重试机制，即在指定的重试时间内，反复使用线性轮询策略来选择可用实例
- com.netflix.loadbalancer.WeightedResponseTimeRule：对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
- com.netflix.loadbalancer.BestAvailableRule：选择并发较小的实例
- com.netflix.loadbalancer.AvailabilityFilteringRule：先过滤掉故障实例，再选择并发较小的实例
- com.netflix.loadbalancer.ZoneAwareLoadBalancer：采用双重过滤，同时过滤不是同一区域的实例和故障实例，选择并发较小的实例



### 参考

[Spring Cloud OpenFeign：基于Ribbon和Hystrix的声明式服务调用](http://www.macrozheng.com/#/cloud/ribbon)
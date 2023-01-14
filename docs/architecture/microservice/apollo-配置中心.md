## 简介

> https://github.com/apolloconfig/apollo/wiki

Apollo（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。

服务端基于 Spring Boot 和 Spring Cloud 开发，打包后可以直接运行，不需要额外安装 Tomcat 等应用容器。

Java 客户端不依赖任何框架，能够运行于所有 Java 运行时环境，同时对 Spring/Spring Boot 环境也有较好的支持。



### 核心功能

```
统一管理不同环境、不同集群配置：
	1:Apollo 提供了一个统一界面集中式管理不同环境（environment）、不同集群（cluster）、不同命名空间（namespace）的配置。
	2:同一份代码部署在不同的集群，可以有不同的配置，比如 zk 的地址等
	3:通过命名空间（namespace）可以很方便的支持多个不同应用共享同一份配置，同时还允许应用对共享的配置进行覆盖
	4:配置界面支持多语言（中文，English）
	
热发布:
	用户在 Apollo 修改完配置并发布后，客户端能实时（1秒）接收到最新的配置，并通知到应用程序。

版本发布管理:
	所有的配置发布都有版本概念，从而可以方便的支持配置的回滚。

灰度发布:
	支持配置的灰度发布，比如点了发布后，只对部分应用实例生效，等观察一段时间没问题后再推给所有应用实例。

权限管理、发布审核、操作审计:
	1:应用和配置的管理都有完善的权限管理机制，对配置的管理还分为了编辑和发布两个环节，从而减少人为的错误。
	2:所有的操作都有审计日志，可以方便的追踪问题。
	
客户端配置信息监控:
	可以方便的看到配置在被哪些实例使用
	
提供 Java 和 .Net 原生客户端:
	1:提供了 Java 和 .Net 的原生客户端，方便应用集成
	2:支持 Spring Placeholder，Annotation 和 Spring Boot 的 ConfigurationProperties，方便应用使用（需要 Spring 3.1.1+）
	3:同时提供了 Http 接口，非 Java 和 .Net 应用也可以方便的使用
	
提供开放平台 API:
	1:Apollo 自身提供了比较完善的统一配置管理界面，支持多环境、多数据中心配置管理、权限、流程治理等特性。
	2:不过 Apollo 出于通用性考虑，对配置的修改不会做过多限制，只要符合基本的格式就能够保存。
	3:在我们的调研中发现，对于有些使用方，它们的配置可能会有比较复杂的格式，如 xml, json，需要对格式做校验。
	4:还有一些使用方如 DAL，不仅有特定的格式，而且对输入的值也需要进行校验后方可保存，如检查数据库、用户名和密码是否匹配。
	5:对于这类应用，Apollo 支持应用方通过开放接口在 Apollo 进行配置的修改和发布，并且具备完善的授权和权限控制
	
部署简单:
	1:配置中心作为基础服务，可用性要求非常高，这就要求 Apollo 对外部依赖尽可能地少
	2:目前唯一的外部依赖是 MySQL，所以部署非常简单，只要安装好 Java 和 MySQL 就可以让 Apollo 跑起来
	3:Apollo 还提供了打包脚本，一键就可以生成所有需要的安装包，并且支持自定义运行时参数
```



相关阅读：
[Apollo 在有赞的实践](https://mp.weixin.qq.com/s/Ge14UeY9Gm2Hrk--E47eJQ)
[分布式配置中心选型，为什么选择 Apollo？—微观技术-2021-04-23](https://mp.weixin.qq.com/s?__biz=Mzg2NzYyNjQzNg==&mid=2247484920&idx=1&sn=76d91ce217bf508aa2ee7156e1ba0994&source=41#wechat_redirect)



### 谁在用它

![file](https://img-note.langyastudio.com/202212280930767.png?x-oss-process=style/watermark)



## 单机部署

![file](https://img-note.langyastudio.com/202212280933702.png?x-oss-process=style/watermark)



- Apollo Config Service

  提供配置的读取、推送等功能，服务对象是 Apollo 客户端

- Apollo Admin Service

  提供配置的修改、发布等功能，服务对象是 Apollo Portal

- Apollo Portal

  Apollo 的管理界面，进行不同项目的配置（项目配置、权限配置等），服务对象是开发者和开放平台 API

- （Software） Load Balancer

  为了实现 MetaServer 的高可用，MetaServer 通常以集群的形式部署。

  Client/Portal 直接访问 （Software） Load Balancer ，然后，再由其进行负载均衡和流量转发。

- Meta Server

  为了实现跨语言使用，通常的做法就是暴露 HTTP 接口。为此，Apollo 引入了 MetaServer。

  Meta Server 其实就是 Eureka 的 Proxy，作用就是将 Eureka 的服务发现接口以 HTTP 接口的形式暴露出来。 这样的话，我们通过 HTTP 请求就可以访问到 Config Service 和 AdminService。

  通常情况下，我们都是建议基于 Meta Server 机制来实现 Config Service 的服务发现，这样可以实现 Config Service 的高可用。不过， 你也可以选择跳过 MetaServer，直接指定 Config Service 地址（apollo-client 0.11.0 及以上版本）。

  

![img](https://img-note.langyastudio.com/202212262240193.png?x-oss-process=style/watermark)



## 基本使用

### 应用接入

官方给出的架构图如下：

![5a92344b-1b99-4e84-ad83-dce41e3ee48c](https://img-note.langyastudio.com/202212091029303.png?x-oss-process=style/watermark)

- Client 端（客户端，用于应用获取配置）流程 ：Client 通过域名走 slb（软件负载均衡）访问 Meta Server，Meta Server 访问 Eureka 服务注册中心获取 Config Service 服务列表（IP+Port）。有了 IP+Port，我们就能访问 Config Service 暴露的服务比如通过 GET 请求获取配置的接口（/configs/{appId}/{clusterName}/{namespace:.+}）即可获取配置

- Portal 端（UI 界面，用于可视化配置管理）流程 ：Portal 端通过域名走 slb（软件负载均衡）访问 Meta Server，Meta Server 访问 Eureka 服务注册中心获取 Admin Service 服务列表（IP+Port）。有了 IP+Port，我们就能访问 Admin Service 暴露的服务比如通过 POST 请求访问发布配置的接口（/apps/{appId}/envs/{env}/clusters/{clusterName}/namespaces/{namespaceName}/releases）即可发布配置




另外，杨波老师的[微服务架构~携程 Apollo 配置中心架构剖析](https://mp.weixin.qq.com/s/-hUaQPzfsl9Lm3IqQW3VDQ)这篇文章对 Apollo 的架构做了简化，值得一看。

我会从上到下依次介绍架构图中涉及到的所有角色的作用。



#### Client 

Apollo 官方提供的客户端，目前有 Java 和 .Net 版本。非 Java 和.Net 应用可以通过调用 HTTP 接口来使用 Apollo。

Client 的作用主要就是提供一些开箱即用的方法方便应用获取以及实时更新配置。

比如你通过下面的几行代码就能获取到 someKey 对应的实时最新的配置值：

```java
Config config = ConfigService.getAppConfig();
String someKey = "someKeyFromDefaultNamespace";
String someDefaultValue = "someDefaultValueForTheKey";
String value = config.getProperty(someKey, someDefaultValue);
```



再比如你通过下面的代码就能监听配置变化：

```java
Config config = ConfigService.getAppConfig();
config.addChangeListener(new ConfigChangeListener() {
    @Override
    public void onChange(ConfigChangeEvent changeEvent) {
       //......
    }
});
```

#### 

### [网络策略](https://www.apolloconfig.com/#/zh/deployment/distributed-deployment-guide?id=_14-网络策略)

分布式部署的时候，`apollo-configservice` 和 `apollo-adminservice` 需要把自己的 IP 和端口注册到 Meta Server（apollo-configservice 本身）。

Apollo 客户端和 Portal 会从 Meta Server 获取服务的地址（IP+端口），然后通过服务地址直接访问。

需要注意的是，`apollo-configservice` 和 `apollo-adminservice` 是基于内网可信网络设计的，所以出于安全考虑，**请不要将 `apollo-configservice` 和 `apollo-adminservice` 直接暴露在公网**。

所以如果实际部署的机器有多块网卡（如 docker），或者存在某些网卡的 IP 是 Apollo 客户端和 Portal 无法访问的（如网络安全限制），那么我们就需要在 `apollo-configservice` 和 `apollo-adminservice` 中做相关配置来解决连通性问题。



#### [忽略某些网卡](https://www.apolloconfig.com/#/zh/deployment/distributed-deployment-guide?id=_141-忽略某些网卡)

可以分别修改 `apollo-configservice` 和 `apollo-adminservice` 的 startup.sh，通过 JVM System Property 传入 -D 参数，也可以通过 OS Environment Variable 传入，下面的例子会把 `docker0` 和 `veth` 开头的网卡在注册到 Eureka 时忽略掉。

JVM System Property 示例：

```properties
-Dspring.cloud.inetutils.ignoredInterfaces[0]=docker0
-Dspring.cloud.inetutils.ignoredInterfaces[1]=veth.*
```

OS Environment Variable 示例：

```properties
SPRING_CLOUD_INETUTILS_IGNORED_INTERFACES[0]=docker0
SPRING_CLOUD_INETUTILS_IGNORED_INTERFACES[1]=veth.*
```



#### [指定要注册的 IP](https://www.apolloconfig.com/#/zh/deployment/distributed-deployment-guide?id=_142-指定要注册的ip)

可以分别修改 `apollo-configservice` 和 `apollo-adminservice` 的 startup.sh，通过 JVM System Property 传入 -D参数，也可以通过 OS Environment Variable 传入，下面的例子会指定注册的 IP 为 `1.2.3.4`。

JVM System Property 示例：

```properties
-Deureka.instance.ip-address=1.2.3.4
```

OS Environment Variable 示例：

```properties
EUREKA_INSTANCE_IP_ADDRESS=1.2.3.4
```



#### [指定要注册的 URL](https://www.apolloconfig.com/#/zh/deployment/distributed-deployment-guide?id=_143-指定要注册的url)

可以分别修改 `apollo-configservice` 和 `apollo-adminservice` 的 startup.sh，通过 JVM System Property 传入 -D 参数，也可以通过 OS Environment Variable 传入，下面的例子会指定注册的 URL 为 `http://1.2.3.4:8080`。

JVM System Property 示例：

```properties
-Deureka.instance.homePageUrl=http://1.2.3.4:8080
-Deureka.instance.preferIpAddress=false
```

OS Environment Variable 示例：

```properties
EUREKA_INSTANCE_HOME_PAGE_URL=http://1.2.3.4:8080
EUREKA_INSTANCE_PREFER_IP_ADDRESS=false
```



## 参考

[Apollo教程从入门到精通](https://juejin.cn/post/7111569085473194021#heading-0)
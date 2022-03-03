本项目很多内容来自廖雪峰、JavaGuide、advanced-java、CS-Notes、srs 等开源库，进行简单排版与补充整理，内容涵盖 Java、PHP、C++、JVM、CS、Redis、MySQL、高并发、高可用、分布式、微服务、海量数据处理等领域知识。

本项目基于 [Docsify](https://docsify.js.org/#/zh-cn/) 进行构建，目前支持以下站点访问：

GitHub Pages：https://langyastudio.github.io/langya-doc

> 下载到本地查看，推荐 Markdown 编辑器 [Typora](https://typora.io/)    



##  Java

[java 后端开发最全知识点](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484666&idx=1&sn=af2994b8b33ec4712464e3d5199c56b5&chksm=c03a565bf74ddf4d379ba675628aa323605c04806fec8edfe8ea7858126dc9f85b52f41e6a22&scene=21#wechat_redirect)

### JDK

> 依据 jvm 规范实现的一套 API，反射、泛型、IO、函数编程、异常、注解等

- [java 基础知识](./docs/java/jdk/java-基础知识.md)
- [java 面向对象](./docs/java/jdk/java-面向对象.md)
- [java IO 流](./docs/java/jdk/java-io流.md)
- [java reflection 反射详解](./docs/java/jdk/reflection.md)
- [java generics 泛型详解](./docs/java/jdk/generics.md)
- [java enum 枚举详解](./docs/java/jdk/enum.md)



**扩展**

- [java 常见关键字](./docs/java/jdk/java-常见关键字.md)
- [java IO 模型](./docs/java/jdk/io-模型.md)
- [java BIO NIO AIO 你了解吗](./docs/java/jdk/bio-nio-aio.md)
- [java BigDecimal 如何解决精度损失的问题](./docs/java/jdk/bigdecimal.md)
- [java 代理模式](./docs/java/jdk/代理模式.md)
- [java security 加密解密详解](./docs/java/jdk/security.md)
- [java junit 测试框架](./docs/java/jdk/junit.md)



### 新特性

- [java8 Date&Time 详解](./docs/java/new-features/date&time.md)
- [foreach 指南](./docs/java/new-features/foreach-指南.md)
- [java8 新特性指南](./docs/java/new-features/java8-指南.md)
- [java9-N 新特性总结](./docs/java/new-features/java-新特性总结.md)
- [java8 用法优雅的函数式编程与stream](./docs/java/new-features/函数式编程与stream.md)



### 集合

- [java 集合基础知识](./docs/java/collection/java-集合基础知识.md)
- [java 集合使用注意事项](./docs/java/collection/java-集合使用注意事项.md)

 

**源码解读**

- [源码分析 Arraylist](./docs/java/collection/源码分析-arraylist.md)
- [源码分析 ConcurrentHashMap](./docs/java/collection/源码分析-concurrent-hashmap.md)
- [源码分析 HashMap](./docs/java/collection/源码分析-hashmap.md)
- [源码分析 LinkedList](./docs/java/collection/源码分析-linkedlist.md)



### 并发

> 多线程、线程安全、AQS、锁、并发容器、原子类、ABA 问题、伪共享等

- [java 并发基础知识](./docs/java/concurrent/java-并发基础知识.md)
- [java 线程池详解](./docs/java/concurrent/线程池.md)
- [java 线程池最佳实践](./docs/java/concurrent/线程池最佳实践.md)



**扩展**

- [java synchronized 锁 你了解吗](./docs/java/concurrent/java-synchronized.md)
- [java AQS 同步器是什么](./docs/java/concurrent/aqs-同步器.md)
- [java Atomic 原子类有什么用](./docs/java/concurrent/atomic-原子类.md)
- [java Concurrent 并发集合有哪些](./docs/java/concurrent/concurrent-集合.md)



### JVM

> 存储级别 or 执行级别，jdk 提供了一系列工具来窥探这些信息。包含jstat、jmap、jstack、jvisualvm 等都是最常用的。
> 垃圾收集器、Class文件结构、类加载机制、参数调优、字节码、锁升级、JMM、JVM并发、JIT等

- [大白话带你认识 jvm](./docs/java/jvm/jvm-intro.md)
- [jvm 垃圾回收详解](./docs/java/jvm/jvm-垃圾回收.md)
- [jvm 内存区域详解](./docs/java/jvm/jvm-内存区域.md)

**扩展**

- [jvm 类加载过程](./docs/java/jvm/类加载过程.md)
- [jvm 类加载器](./docs/java/jvm/类加载器.md)
- [jvm 类文件结构](./docs/java/jvm/类文件结构.md)
- [jvm 参数如何调优](./docs/java/jvm/jvm-参数总结.md)



### Framework

- [J2EE 基础知识](./docs/java/framework/j2ee/j2ee-intro.md)
- [JSP 与 Servlet 区别](./docs/java/framework/j2ee/jsp-servlet区别.md)



**mybatis**

- [Mybatis 常见问题总结](./docs/java/framework/mybatis/mybatis-interview.md)



**spring**

- [SSH 框架已经过时了吗](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484277&idx=1&sn=11c270bbcfa7b6f9e23ef0b0db0b1ae6&chksm=c03a51d4f74dd8c2dd44bde08b6e59ebc22ad9d416fd226ec23f921f29e5d8a2ea075d46bc79&scene=21#wechat_redirect)
- [spring 常见问题总结](./docs/java/framework/spring/spring-常见问题总结.md)
- [spring 注解详解](./docs/java/framework/spring/spring-注解详解.md)
- [spring5 新特性与增强的功能](./docs/java/framework/spring/spring/spring5-新特性与增强.md)
- [spring IOC 容器详解](./docs/java/framework/spring/spring/spring-ioc.md)
- [spring AOP 代理模式详解](./docs/java/framework/spring/spring/spring-aop.md)
- [spring 如何操作数据库](./docs/java/framework/spring/spring/spring-db.md)
- [spring MVC 框架详解](./docs/java/framework/spring/spring/spring-mvc.md)
- [spring Scheduler 如何执行任务](./docs/java/framework/spring/spring/spring-scheduler.md)
- [spring Aware 是什么](./docs/java/framework/spring/spring/spring-aware.md)



**spring boot**

- [spring boot 常见知识点](./docs/java/framework/spring/spring-boot/springboot-常见知识点&面试题汇总.md)
- [spring boot 零基础快速入门](./docs/java/framework/spring/spring-boot/01--springboot-快速入门.md)
- [spring boot restful web 应用](./docs/java/framework/spring/spring-boot/02--restful-web-应用.md)
- [spring boot properties 配置文件详解](./docs/java/framework/spring/spring-boot/03--springboot-配置文件.md)
- [yml 配置文件详解](./docs/java/framework/spring/spring-boot/03--yml文件详解.md)
- [spring boot package 打包与 devtools](./docs/java/framework/spring/spring-boot/04--package-devtools.md)
- [spring boot 日志 log4j2](./docs/java/framework/spring/spring-boot/05--log.md)
- [spring boot actuator 监控/健康检查/审计/统计](./docs/java/framework/spring/spring-boot/06--actuator-应用监控.md)
- [spring boot 深度理解定时任务 schedule](./docs/java/framework/spring/spring-boot/07--scheduled-计划任务.md)
- [spring boot 多线程异步调用 Async](./docs/java/framework/spring/spring-boot/08--async-异步任务.md)
- [spring boot 整合 mybaits 数据库开发框架](./docs/java/framework/spring/spring-boot/09--mybaits-数据库开发框架.md)
- [spring boot mybatis 缓存机制](./docs/java/framework/spring/spring-boot/09--mybatis-缓存机制.md)
- [spring boot  exception 全局异常处理](./docs/java/framework/spring/spring-boot/10--exception-异常处理.md)
- [spring boot 如何解决跨域问题](./docs/java/framework/spring/spring-boot/11--跨域.md)
- [spring boot validation 数据校验](./docs/java/framework/spring/spring-boot/12--validation-数据校验.md)



**打包**

- [spring boot 利用 docker 自动化打包](https://mp.weixin.qq.com/s/3X6vVdWmjmWCyiLm35jpVw)



**工具集**

> fastjson、Hutool 等



## 数据库

- [字符编码有哪些种类](./docs/database/字符编码.md)
- [数据库基础知识](./docs/database/database-intro.md)
- [SQLite 数据类型和如何添加注释](./docs/database/sqlite/sqlite-数据类型和注释.md)

### MySQL

> 数据库范式、字符集、索引（聚集索引、非聚集索引、复合索引、自适应哈希索引）、事务（ACID、隔离级别、MVCC）、锁（锁与同步锁、公开锁、非公平锁、悲观锁、乐观锁、互斥锁、共享锁、死锁）等

- [mysql 基础知识](./docs/database/mysql/mysql-intro.md)
- [mysql 常见必知必会的 SQL 语句操作](./docs/database/mysql/mysql-sql语句操作.md)
- [mysql 索引详解](./docs/database/mysql/mysql-索引详解.md)
- [mysql 事务隔离级别](./docs/database/mysql/mysql-事务隔离级别.md)
- [mysql 高性能优化指南](./docs/database/mysql/mysql-高性能优化规范.md)
- [mysql 日志详解](./docs/database/mysql/mysql-日志详解.md)
- [mysql 如何执行 SQL 语句](./docs/database/mysql/mysql-如何执行sql语句.md)
- [mysql innodb-存储引擎对mvcc的实现](./docs/database/mysql/innodb-存储引擎对mvcc的实现.md)

  

**设计**

- [SQL 如何高效存储目录树结构的数据](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484285&idx=1&sn=b0a3e10e4ff806bd6d257d93dad860c5&chksm=c03a51dcf74dd8caa24df5df90cc649f40556a806e5cfe47c2effc1bfd6ba495447d978d7a86&scene=21#wechat_redirect)
- [mysql 如何存储时间](./docs/database/mysql/设计之存储时间.md)
- [MySQL 目录树实现批量条件循环查询](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484275&idx=1&sn=3ac12fd96bcdfa27bd125304fd2aa340&chksm=c03a51d2f74dd8c440775a410648b79f8881fc3bb0adc5ac0eb6820016ff3bb101d5d13d046e&scene=21#wechat_redirect)



### Druid

> 数据库连接池方面，国内使用 druid 最多

- [Druid 实战](./docs/database/druid/druid-intro.md)



### Mongodb

[mongodb 实战入门](./docs/database/mongodb/mongodb-实战入门.md)



### 数据仓库

> 现在的企业，数据量都非常大，数据仓库是必须的。
> 搜索方面，solr 比较成熟，稳定性更好一些，但实时搜索方面不如 es。
> 列式存储方面，基于 Hadoop 的 hbase，使用最是广泛；基于 LSM 的 leveldb 写入性能优越，但目前主要是作为嵌入式引擎使用多一些。
> 时序数据库方面，opentsdb 用在超大型监控系统多一些



### 数据同步

> 实时数据同步工具，都是把自己模拟成一个从库，进行数据拉取和解析。 mysql 通过 binlog 进行同步；canal、maxwell 等工具，都支持将要同步的数据写入到 mq 中进行后续处理。对于ETL（抽取、清洗、转换）来说，datax、logstash、sqoop 等，都是这样的工具。



## 中间件

### 缓存

> 分布式缓存来说，优先选择的就是 `redis`。由于 redis 是单线程的，并不适合高耗时操作。所以对于一些数据量比较大的缓存，比如图片、视频等，使用老牌的 memcached 效果会好的多。
> redis、caffeine、vernemq等

**Redis**

- [redis 基础知识](./docs/middleware/cache/redis/redis-intro.md)
- [redis 缓存读写策略](./docs/middleware/cache/redis/redis-缓存读写策略.md)
- [redis 除了做缓存，还可以怎么用？](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484620&idx=1&sn=ac71f68713aa2a1bec3bf7382f2198b8&chksm=c03a566df74ddf7b5572b92756746e5758ab17fe28062a015272c150c8e1387ddab3b68c3897&scene=21#wechat_redirect)
- [缓存如何确保数据的一致性](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484329&idx=1&sn=032f2ffb00ee9dd24de51d958fed789a&chksm=c03a5108f74dd81e4ac9712b4e89c4fb3c331bba3bb8adf39a79b904a342e52028f0835b8231&scene=21#wechat_redirect)
- [redis 内存碎片是什么](./docs/middleware/cache/redis/redis-内存碎片.md)
- [redis停止服务报错 NOAUTH Authentication required 导致关机异常慢](https://langyastudio.blog.csdn.net/article/details/88555828)



### 消息队列

> 一个大型的分布式系统，通常都会异步化，走消息总线。
> kafka 有着极高的吞吐量
> rocketmq 和 rabbitmq 都是电信级别的消息队列，在业务上用的比较多
> mqtt 具体来说是一种协议，主要用在物联网方面

- [消息队列基础知识](./docs/middleware/msg-queue/msg-queue-intro.md)

**RocketMQ**

- [rocketmq 基础知识](./docs/middleware/msg-queue/rocketmq/rocketmq-intro.md)
- [rocketmq 常见问题](./docs/middleware/msg-queue/rocketmq/rocketmq-常见问题.md)


**websocket**

- [java websocket 入门](./docs/middleware/msg-queue/websocket/websocket-intro.md)



### 任务调度

> quartz 是 java 中比较古老的调度方案，分布式调度采用数据库锁的方式，管理界面需要自行开发。相对来说 xxl-job 更加轻量好用

- [定时任务 你了解吗](./docs/middleware/task-scheduling/定时任务.md)



### RPC框架

> thrift、dubbo、gRPC 默认都是二进制序列化方式的 socket 通讯框架；feign、hessian 都是 onhttp 的远程调用框架。

- [rpc 基础知识](./docs/middleware/rpc/rpc-intro.md)

**Dubbo**

- [dubbo 基础知识](./docs/middleware/rpc/dubbo/dubbo-知识点总结.md)



### 通讯框架

> Java 中，netty 已经成为当之无愧的网络开发框架，包括其上的 socketio。服务的响应时间主要耗费在业务逻辑以及数据库上，**通讯层耗时在其中的占比很小**。



### 服务器

> Tomcat、Nginx、Apache等

**Apache**

- [Apache 如何配置多端口与域名访问](https://langyastudio.blog.csdn.net/article/details/49429563)
- [Apache 修改最大连接数/并发数](https://langyastudio.blog.csdn.net/article/details/91987214)
- [Apache ab 并发负载压力测试](https://langyastudio.blog.csdn.net/article/details/78061061)
- [Apache mod_xsendfile 为php提供更快的文件下载](https://langyastudio.blog.csdn.net/article/details/50300257)
- [Apache 支持mp4与flv拖动播放的功能模块](https://langyastudio.blog.csdn.net/article/details/50300765)
- [Apache 防盗链模块mod_auth_token的安装配置](https://langyastudio.blog.csdn.net/article/details/50301505)
- [Apache 启用GZIP压缩功能 mod_deflate的安装配置](https://langyastudio.blog.csdn.net/article/details/52291423)
- [xampp 安装与配置](./docs/middleware/server/apache/xampp-安装与配置.md)



**Tomcat**

- [Tomcat 对应 Servlet、JSP、JDK 版本问题](https://langyastudio.blog.csdn.net/article/details/85265030)
- [Tomcat 重要参数优化](./docs/middleware/server/tomcat/tomcat-重要参数调优.md)



**nginx**

- [nginx 入门](./docs/middleware/server/nginx/nginx-intro.md)
- [nginx 如何配置反向代理](./docs/middleware/server/nginx/nginx-proxy.md)



## 架构

### 分布式

> 分布式系统 zookeeper 能用在很多场景，与其类似的还有基于 raft 协议的 etcd 和 consul。
> CAP/BASE、Paxos/Raft、分布式锁、API网关、CC、分布式文件系统、分布式Id、分布式事务等

- [cap&base 理论](./docs/architecture/distributed-system/theory/cap&base-理论.md)
- [paxos&raft 算法](./docs/architecture/distributed-system/theory/cap&base-理论.md)
- [分布式 id](./docs/architecture/distributed-system/分布式-id.md)
- [分布式事务](./docs/architecture/distributed-system/分布式-transactions.md)
- [网关](./docs/architecture/distributed-system/网关.md)



### 微服务

> 注册中心默认的 eureka 不再维护，consul 已经成为首选，**nacos** 带有后台，比较适合国人使用习惯
> 熔断组件官方的 hystrix 不再维护，推荐阿里的 **sentinel** or  resilience4j
> 调用链推荐 jaeger or **skywalking**
> 配置中心推荐 **apollo**
> 对于 spring cloud 来说，zuul 系列推荐使用 zuul2，zuul1 是多线程阻塞的有硬伤。spring-cloud-gateway 是 spring cloud 亲生的

- [spring cloud 入门总结](https://juejin.cn/post/6844904007975043079)



### 高并发

> 读写分离、负载均衡、分库分表（推荐使用驱动层的`sharding-jdbc`，或者代理层的`mycat`，**方案一旦确定，几乎无法回退**）进行垂直拆分、水平拆分、不停机切换、HA&FailOver等

- [读写分离&分库分表](./docs/architecture/high-performance/读写分离&分库分表.md)
- [负载均衡](./docs/architecture/high-performance/负载均衡.md)



### 性能优化

> 内核参数优化、jvm优化、网络参数优化、事务优化、数据库优化、池化等

- [5秒到1秒 记一次效果“非常”显著的性能优化](https://mp.weixin.qq.com/s/7Pr5iX3KE08h32AytBuNqQ)
- [php-性能压榨](./docs/architecture/performance-optimization/php-性能压榨.md)



### 高可用

> 限流、熔断（sentinel）、降级、排队、超时与重试、容灾、应用层容灾、跨机房容灾等

- [高可用基础知识](./docs/architecture/high-availability/high-availability-intro.md)
- [超时&重试机制](./docs/architecture/high-availability/超时&重试机制.md)
- [集群](./docs/architecture/high-availability/集群.md)
- [降级&熔断](./docs/architecture/high-availability/降级&熔断.md)
- [排队](./docs/architecture/high-availability/排队.md)
- [限流](./docs/architecture/high-availability/限流.md)
- [灾备设计&异地多活](./docs/architecture/high-availability/灾备设计&异地多活.md)



### 系统设计

> DDD、Aotor模式、响应式设计、RESTful、Service Mesh等

- [resultful api](./docs/architecture/system-design/basic/resultful-api.md)

#### [设计模式](./docs/architecture/system-design/design-patterns/README.md)

#### 安全

> web 安全（SQL注入、XSS、CSRF、DDOS、脚本注入、漏洞、验证码）、隐私信息保护、加密解密、证书体系、网络隔离、内外网隔离、跳板机、授权认证（OAuth、SSO、JWT）等

- [认证&授权基础知识](./docs/architecture/system-design/security/认证授权.md)
- [隐私信息保护](./docs/architecture/system-design/security/隐私信息.md)
- [敏感词过滤](./docs/architecture/system-design/security/敏感词过滤.md)
- [jwt 身份认证](./docs/architecture/system-design/security/jwt-身份认证.md)
- [web 常见安全漏洞](./docs/architecture/system-design/security/web-常见安全漏洞.md)



## 计算机基础

### 数据结构

> 基本的数据结构非常重要，无论接触什么编程语言，基本数据结构都是首先要掌握的。
> 队列、栈、链表、数组、字典、图、堆、树（红黑树、B、B+、B*树、LSM 树、二叉树、平衡二叉树、平衡二叉树、BST 二叉查找树）等

- [布隆过滤器](./docs/cs-basic/data-structure/布隆过滤器.md)
- [堆](./docs/cs-basic/data-structure/堆.md)
- [红黑树](./docs/cs-basic/data-structure/红黑树.md)
- [树](./docs/cs-basic/data-structure/树.md)
- [图](./docs/cs-basic/data-structure/图.md)
- [线性数据结构](./docs/cs-basic/data-structure/线性数据结构.md)



### 算法

> 算法是某些大厂的门槛，能够培养逻辑思维能力和动手能力，最快的进阶途径就是刷 leetcode。
> 排序算法、贪心算法、动态规划、回溯算法、剪枝算法、图算法等

- [剑指 offer 编程题](./docs/cs-basic/algorithm/剑指offer编程题.md)
- [链表算法题](./docs/cs-basic/algorithm/链表算法题.md)
- [字符串算法题](./docs/cs-basic/algorithm/字符串算法题.md)



### 计算机网络

> 熟悉 Netty 开发是入门网络开发的捷径。
> 网络基础、网络模型、Epoll、Kqueue、长连接、爬虫等

- [网络基础知识](./docs/cs-basic/network/network-intro.md)
- [HTTPS tls详解](./docs/cs-basic/network/https-tls.md)
- [HTTP 缓存](./docs/cs-basic/network/http-缓存.md)
- [chrome80 cookie 跨域 samesite 错误](./docs/cs-basic/network/issue/chrome80-cookie跨域samesite错误.md)

  

### 操作系统

> 对于计算密集型应用，就需要关注程序执行的效率；对于I/O密集型，要关注进程（线程）之间的切换以及I/O设备的优化以及调度。
> 计算机原理、CPU、内存、IO、进程线程等

- [操作系统基础知识](./docs/cs-basic/operating-system/操作系统基础知识.md)
- [linux 入门](./docs/cs-basic/operating-system/linux-入门.md)
- [shell 入门](./docs/cs-basic/operating-system/shell-入门.md)



## 视音频

### 原理

- [PCM 音频编码格式详解](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484274&idx=1&sn=5108fd37730a5605cc58bd63f6f2c2c0&chksm=c03a51d3f74dd8c5733796f9836bc2ead537d509f45845329dd1b26ce99321cc1bc820245cb0&scene=21#wechat_redirect)
- [音频属性详解](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484009&idx=2&sn=bcf5fa077b7600c231680a3b2ae60368&chksm=c03a50c8f74dd9dec1a9a014ca0f771ba178862b3aebd812c1c0e7d95dbbd4dc7dc32219532e&scene=21#wechat_redirect)
- [MPEG I B P 帧](https://langyastudio.blog.csdn.net/article/details/40399181)
- [颜色空间详解](./docs/video/theory/颜色空间详解.md)
- [FFMPEG H264/H265 编码延迟问题](https://langyastudio.blog.csdn.net/article/details/40397199)
- [常用的流媒体网络协议](https://langyastudio.blog.csdn.net/article/details/109726200)



### 流媒体

- [浏览器没有 Flash 如何进行 RTMP 直播](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484282&idx=1&sn=057d324c5459b888214b06e60cc3fb82&chksm=c03a51dbf74dd8cd86bbcf995389b5bd312767774ea35355d1a27402526c1be714efe068f1f7&scene=21#wechat_redirect)
- [HLS 直播协议 m3u8 详解](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484009&idx=1&sn=107040a55095c62fb00b2c8b266ec8ba&chksm=c03a50c8f74dd9de5e9440f7490f5b975b817e3dd5ec62bf9ad03a2645250b257c343459af9b&scene=21#wechat_redirect)

  

**srs**

- [srs 与主流流媒体服务器的对比](./docs/video/live/srs/srs-intro.md)
- [srs 部署分发 HLS 与 FLV 服务](./docs/video/live/srs/srs-部署分发hls与flv服务.md)
- [srs RTMP/HLS 低延时模式与 reload](./docs/video/live/srs/srs-低延时模式与reload.md)
- [srs 回调授权与管理](./docs/video/live/srs/srs-回调授权与管理.md)
- [srs 使用阿里云cdn提高并发](./docs/video/live/srs/srs-使用阿里云cdn提高并发.md)
- [srs 利用集群提高并发量 支持更多的推流与播放](./docs/video/live/srs/srs-高并发.md)



## 运维

### 故障排查

> 内存溢出排查、堆内外层排查、网络排查、IO排查、高负载排查等

- [java 性能问题排查](./docs/devops/troubleshoot/java-性能问题排查.md)
- [jadx 反编译工具](./docs/devops/troubleshoot/jadx.md)



### 运维

> 服务器一般采用稳定性较好的 centos，并配备 ansible工具进行支持
> haproxy、lvs、keepalived、APM、Docker、CI/CD、jenkins、自动化（ansible）、监控（zabbix、prometheus + grafana + telegraf、es/logstash/kibana）等

**Docker**

- [docker 安装/离线安装](./docs/devops/docker/docker-安装与离线安装.md)
- [docker 镜像与容器操作](./docs/devops/docker/docker-镜像与容器.md)
- [docker 镜像仓库](./docs/devops/docker/docker-镜像仓库.md)
- [docker 同主机/host 容器间网络互联互通](./docs/devops/docker/docker-网络.md)
- [docker 配置与调试](./docs/devops/docker/docker-配置与调试.md)

  

**硬件**

- [交换机的堆叠与级联](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484276&idx=1&sn=4635e18705667b6a03ab086d7dd5c228&chksm=c03a51d5f74dd8c36eda4a2f0530f342f45c66fb9d9e3203aa831e9cd3dc8c5528930972791f&scene=21#wechat_redirect)
- [RAID 几张图让你完全理解独立磁盘冗余阵列](https://langyastudio.blog.csdn.net/article/details/119538972)



**Linux**

- [ECS 安装图形化桌面](./docs/devops/centos/ecs-安装图形化桌面.md)
- [Linux-CentOS 搭建ssr服务端](./docs/devops/centos/linux-centos-搭建ssr服务端.md)
- [U盘安装CentOS系统 提示No Caching mode page found /dev/root does not exist错误的解决方法](https://langyastudio.blog.csdn.net/article/details/50436603)
- [CentOS  安装FFmpeg](https://langyastudio.blog.csdn.net/article/details/50134539)
- [CentOS 磁盘挂载与NFS共享](https://langyastudio.blog.csdn.net/article/details/78518936)
- [/bin/bash^M: 坏的解释器: 没有那个文件或目录](https://langyastudio.blog.csdn.net/article/details/81195982)
- [Linux 时间如何同步](./docs/devops/centos/linux-centos-时间同步.md)
- [Linux 最大打开文件数和进程数](./docs/devops/centos/linux-centos-最大打开文件数和进程数.md)



**Windows**

- [制作 xp+win7+pe 多合一安装盘](./docs/devops/windows/制作xp-win7-pe多合一安装盘.md)
- [Windows 同时开启核心显卡与独立显卡](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484149&idx=1&sn=b893294ea89a8862f1ebcad3eca53ed7&chksm=c03a5054f74dd94226450b41d08459b8b886796aea99662c8da2f0aa38c7634cc2f393db36f1&scene=21#wechat_redirect)



**监控**

[监控入门](./docs/devops/monitor/monitor-intro.md)



**Platform**

- [aliyun 钉钉开发平台配置](./docs/devops/platform/aliyun-钉钉开发平台配置.md)
- [aliyun 短信配置](./docs/devops/platform/aliyun-短信配置.md)
- [aliyun 直播配置](./docs/devops/platform/aliyun-直播.md)
- [weixin 公众号配置](./docs/devops/platform/weixin-公众号配置.md)



## 工具

> IDE、代码管理、项目构建等

### IDEA

- [IDEA 常用插件推荐](./docs/tool/idea/idea-插件推荐.md)
- [IDEA 如何快速的进行代码重构](./docs/tool/idea/idea-代码重构.md)
- [IDEA 源码阅读技巧](./docs/tool/idea/idea-源码阅读技巧.md)
- [IDEA Tomcat 配置详解](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484228&idx=1&sn=0b5b5856f645dcd99c6289a5953c306b&chksm=c03a51e5f74dd8f3687ef6398764a55b60f96bd8bfba6c93aff24a9b497d7a2b98ddd53c41ea&scene=21#wechat_redirect)
- [IDEA 配置XDebug 远程调试 PHP 代码 详细教程](https://langyastudio.blog.csdn.net/article/details/47005477)



### Database

- [chiner 数据库建模工具](./docs/tool/database/chiner-数据库建模工具.md)
- [screw 一键生成数据库文档](./docs/tool/database/screw-一键生成数据库文档.md)
- [datagrip idea官方数据库管理](./docs/tool/database/datagrip-idea官方数据库管理.md)

  

### Git

- [Git 基础知识](./docs/tool/git/git-intro.md)
- [Github 使用技巧](./docs/tool/git/github-使用技巧.md)
- [Git .gitignore 添加后无效的解决办法](https://langyastudio.blog.csdn.net/article/details/72457109)
- [Git 工作流](https://langyastudio.blog.csdn.net/article/details/78788428)
- [Git LFS 支持大文件存储](https://langyastudio.blog.csdn.net/article/details/71248896)



### 其他

- [Maven 使用详解](./docs/tool/maven/maven-intro.md)
- [TreeNMS 好用的 Redis Memcached 可视化管理及监控工具](https://langyastudio.blog.csdn.net/article/details/80221975)
- [最强抓包神器 Fiddler 手机抓包详解](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484011&idx=1&sn=e0f4bf9c5a033de39b5bc04d105273bc&chksm=c03a50caf74dd9dc8044a9b306c4efe0b0f4610c00cdd31b7d74deca995957ae28a1724d756d&scene=21#wechat_redirect)



## 管理

- [个人简历](./docs/manage/team/interview/langyastudio.md)



### 开源

- [免费课程整理](./docs/manage/open/course.md)
- [优质开源项目整理](./docs/manage/open/source.md)

  

### 项目

> 架构评审、重构、代码规范、代码评审、RUP、看板管理、SCRUM、敏捷开发、结对编程、PDCA、FMEA 管理模式等

- [命名之道](./docs/manage/project/命名之道.md)
- [软件版本号的命名格式和规则](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484280&idx=1&sn=fb162047fd1ada7560b17a4197c02e4f&chksm=c03a51d9f74dd8cfb30dd8f5d72bcff34306e35b8350821308c7b77131907eab13857141c3cd&scene=21#wechat_redirect)



### 团队

-  [程序员职业发展三重门](./docs/manage/team/articles/程序员职业发展三重门.md)
-  [如何面试程序员](./docs/manage/team/interview/如何面试程序员.md)
-  [新入职一家公司如何快速进入工作状态](./docs/manage/team/articles/新入职一家公司如何快速进入工作状态.md)
-  [高级别开发同学的七条建议](./docs/manage/team/articles/高级别开发同学的七条建议.md)
-  [大龄程序员所经历的面试的历炼和思考](./docs/manage/team/articles/大龄程序员所经历的面试的历炼和思考.md)
-  [java 如何高薪](./docs/manage/team/interview/java如何高薪.md)
-  [35 岁出路](./docs/manage/team/articles/35岁出路.md)



### 测试

> TDD、单元测试、压力测试、全链路压测

- [性能测试介绍](./docs/manage/test/性能测试.md)



### 运营



### 资讯

> 行业、技术趋势、行业数据分析等

- [tiobe 最受欢迎的编程语言](https://www.tiobe.com/tiobe-index/)
- [PYPL 最受欢迎的编程语言、IDE 和数据库](https://pypl.github.io/PYPL.html)
- [2019 年度最受欢迎的中国开源软件](https://www.oschina.net/project/top_cn_2019?sort=1)
- [2019 Web开发技术指南和趋势](https://blog.csdn.net/weixin_44811417/article/details/93078574)



## Web

### 前端

- [Android IOS浏览器不自动播放视频](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484017&idx=1&sn=b4e030a618cb0bd7184a452ef75d1440&chksm=c03a50d0f74dd9c6bc2603e66ad62db7597b1d0061234958c61bc59ef9a040d755bc8aea6d6a&scene=21#wechat_redirect)
- [微信分享链接如何定制缩略图和标题](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484012&idx=1&sn=8ed50e1f525339e68963ff4627384bba&chksm=c03a50cdf74dd9dbd2820bd2c8de8b83b74f3e7aa9ba3dde6fe04d97c88510529bd2c865118a&scene=21#wechat_redirect)

  

### PHP

- [PHP Supported Versions 支持的版本](https://langyastudio.blog.csdn.net/article/details/88724663)
- [PHP8 新特性](./docs/web/php/php8-新特性.md)
- [PHP this parent static self 关键字](https://langyastudio.blog.csdn.net/article/details/100573161)
- [PHP 密码哈希password_hash的使用方法](https://langyastudio.blog.csdn.net/article/details/85079659)
- [分分钟解决 PHP 上传文件大小限制的问题](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484279&idx=1&sn=1f44579249f12f09f04f00881d6070a0&chksm=c03a51d6f74dd8c00b794ed9db932c42bc0368c2d51d7153102e7f595cd9a374efb25ff85a5a&scene=21#wechat_redirect)
- [PHP 获取 IP 地址所在的地理位置信息/城市](https://langyastudio.blog.csdn.net/article/details/80245487)
- [PHP 获取内存/CPU/负载/网络带宽数据包/磁盘IO读写等监控指标](https://langyastudio.blog.csdn.net/article/details/112325035)
- [PHP 实现敏感词过滤（附敏感词库）](https://langyastudio.blog.csdn.net/article/details/85072625)

  

**扩展**

- [PHP 性能优化简述](https://langyastudio.blog.csdn.net/article/details/78332767)
- [PHP SSO 原来单点登录这么简单](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484278&idx=1&sn=0562afdd3a55ab2673d6c3a3e29827be&chksm=c03a51d7f74dd8c12e25687df268b56dd28973ce27454b87cccbc14308e3400b77349313fa4b&scene=21#wechat_redirect)
- [PHP List数据集/数组转换成树状结构Tree](https://langyastudio.blog.csdn.net/article/details/83862452)
- [PHP ajax请求返回数据后，如何在后台继续执行代码](http://mp.weixin.qq.com/s?__biz=Mzg5MjYzNTA4Ng==&mid=2247484281&idx=1&sn=889c43871e10fa1ea2b13153d2e4df20&chksm=c03a51d8f74dd8cef230842dfcaeb8ec4d1f63ffa5e9130ba3e15c7be5f5a87cf4fada063ae3&scene=21#wechat_redirect)
- [PHP扩展 opcache 操作码优化加速组件的配置](https://langyastudio.blog.csdn.net/article/details/53219266)
- [PHP 源码加密解密工具php-beast](https://langyastudio.blog.csdn.net/article/details/53403862)
- [soar-php SQL语句优化与重写的自动化工具](https://langyastudio.blog.csdn.net/article/details/109851376)
- [PHP Yaconf 一个高性能的配置管理扩展（PHP7）](https://langyastudio.blog.csdn.net/article/details/79641936)
- [thinkphp 如何有效提高应用性能](./docs/web/php/thinkphp/thinkphp-如何有效提高应用性能.md)
- [workerman 支持多少并发量](./docs/web/php/workerman/workerman-支持多少并发.md)



### C++

- [struct 结构体的变量声明加冒号](https://langyastudio.blog.csdn.net/article/details/37819173)
- [C++ 回调函数详解](https://langyastudio.blog.csdn.net/article/details/38543157)
- [C++ 硬件信息 获取网卡MAC地址](https://langyastudio.blog.csdn.net/article/details/40708683)
- [C++ 硬件信息 获取硬盘序列号](https://langyastudio.blog.csdn.net/article/details/40708825)
- [C++ 硬件信息 获取CPU序列号](https://langyastudio.blog.csdn.net/article/details/44958907)
- [C++ 硬件信息 获取主板序列号](https://langyastudio.blog.csdn.net/article/details/44958985)



### C#

- [**C#调用C++ 基于P/Invoke实现**](https://github.com/langyastudio/pinvoke)
- [**C#调用C++ 基于CLI实现**](https://github.com/langyastudio/cli)
- [U 盘插拔监控](https://langyastudio.blog.csdn.net/article/details/43203739)
- [垃圾回收机制 GC 详解](https://langyastudio.blog.csdn.net/article/details/38581101)
- [C# WinForm 用户自定义控件闪烁的问题](https://langyastudio.blog.csdn.net/article/details/45251711)
- [C# 注册自定义文件类型](https://langyastudio.blog.csdn.net/article/details/44815751)
- [C# UrlEncode 与 Java、PHP 不一致](https://langyastudio.blog.csdn.net/article/details/86376803)

## 简介

### 了解 [Consul](https://www.consul.io/)

[Consul](https://www.consul.io/) 是一个支持多数据中心分布式高可用的 `服务发现` 和 `配置共享` 的服务软件，由 `HashiCorp` 公司用 `Go` 语言开发，基于 `Mozilla Public License 2.0` 的协议进行开源。 Consul 支持 `健康检查`，并允许 `HTTP` 、`GRPC` 和 `DNS` 协议调用 API 存储键值对。
命令行超级好用的虚拟机管理软件 vgrant 也是 HashiCorp 公司开发的产品。
一致性协议采用 Raft 算法，用来保证服务的高可用。使用 GOSSIP 协议管理成员和广播消息，并且支持 ACL 访问控制。



### Consul 使用场景

- `Docker` 实例的注册与配置共享
- `Coreos` 实例的注册与配置共享
- `SaaS` 应用的配置共享、服务发现和健康检查
- `vitess` 集群
- 与 confd 服务集成，动态生成 nginx 和 haproxy 配置文件



### Consul 优势

市面现在有很多类似的软件比如：`zookeeper` 、`Etcd`、`doozerd`、`eureka`，Consul 相比这些软件有什么优势呢？
官方出了相比较这些软件区别的一篇 [Consul vs. ZooKeeper，doozerd，etcd](https://www.consul.io/intro/vs/zookeeper.html) 文章。

下面总结一下 Consul 的优势有那几点：

- 使用 [`Raft`](https://www.jdon.com/artichect/raft.html) 算法来保证一致性, 比复杂的 `Paxos` 算法更直接。相比较而言, zookeeper 采用的是 Paxos, 而 etcd 使用的则是 Raft
- 支持 `多数据中心`，内外网的服务采用不同的端口进行监听。 多数据中心集群可以避免单数据中心的单点故障，而其部署则需要考虑网络延迟，分片等情况等。zookeeper 和 etcd 均不提供多数据中心功能的支持
- 支持 `健康检查`。 etcd 不提供此功能
- 支持 `HTTP`、`DNS` 和 `GPRS` 协议接口。 zookeeper 的集成较为复杂，etcd 只支持 http 协议
- 官方提供 `WEB管理界面`，etcd 无此功能
- 综合比较, Consul 作为服务注册和配置管理的新星，比较值得关注和研究



### Consul 中的概念
![image_1d78u4elv18glqtipc18f1aiv9.png-474.2kB](https://img-note.langyastudio.com/202212221342725.png?x-oss-process=style/watermark)

- **Client**：表示 Consul 客户端模式，是 Consul 节点的一种模式，所有注册到 Client 节点的服务会被转发到 Server 。本身无状态不持久化数据。Client 通过 HTTP、DNS、GRPC 接口请求转发给局域网内的服务端集群
- **Server**：表示 Consul 的服务端模式， Server 功能和 Client 都一样，不同的是 **Server 持久化数据到本地**。在局域网内与本地 Client 通讯，通过广域网与其他数据中心通讯。每个数据中心的 Server 数量推荐为 3 个或是 5 个
- **Server-Leader** ：表示这个 Server 是它们的老大，它和其它 Server 不一样的一点是，它需要负责**同步注册的信息**给其它的 Server 节点，同时也要负责各个节点的健康监测。如果 Leader 宕机了，数据中心的所有 Server 内部会使用 [`Raft`](https://www.jdon.com/artichect/raft.html) 算法来在其中选取一个 Leader 出来
- **Agent** ：Agent 是 Consul 的核心进程，Agent 的工作是维护成员关系信息、注册服务、健康检查、响应查询等。Consul 集群的每一个节点都必须运行 agent 进程
- **其它**
  需要了解 Consul 原理、的通信方式、协议信息、算法、帮助文档等。有兴趣可以前往官方查看 [官方文档](https://www.consul.io/docs/agent/basics.html)。

> 文档：https://www.consul.io/docs/agent/basics.html
> 官网：[https://www.consul.io](https://www.consul.io/)



## 安装

Consul 镜像提供了几个个常用环境变量

- `CONSUL_CLIENT_INTERFACE` ：配置 Consul 的 `-client=<interface ip>` 命令参数
- `CONSUL_BIND_INTERFACE` ：配置 Consul 的 `-bind=<interface ip>` 命令参数
- `CONSUL_DATA_DIR` ：配置 Consul 的数据持久化目录
- `CONSUL_CONFIG_DIR`：配置 Consul 的配置文件目录

Consul 镜像的详细说明请前往[官方使用文档](https://github.com/docker-library/docs/tree/master/consul)。



### Consul 命令简单介绍

- `agent` : 表示启动 Agent 进程
- `-server`：表示启动 Consul Server 模式
- `-client`：表示启动 Consul Cilent 模式
- `-bootstrap`：表示这个节点是 `Server-Leader` ，**每个数据中心只能运行一台服务器**。技术角度上讲 Leader 是通过 Raft 算法选举的，但是集群第一次启动时需要一个引导 Leader，在引导群集后，建议不要使用此标志
- `-ui`：表示启动 Web UI 管理器，默认开放端口 `8500`，所以上面使用 Docker 命令把 8500 端口对外开放
- `-node`：节点的名称，**集群中必须是唯一的**
- `-client`：表示 Consul 将绑定客户端接口的地址，`0.0.0.0` 表示所有地址都可以访问
- `-join`：表示加入到某一个集群中去。 如：`-join=192.168.1.2`

```
-client
作用：指定节点为client，指定客户端接口的绑定地址，包括：HTTP、DNS、RPC
默认是127.0.0.1，只允许回环接口访问
-config-dir
放置到consul agent通过参数-config-dir指定的目录下面，缺省目录是：/consul/config/
-data-dir
作用：指定agent储存状态的数据目录
这是所有agent都必须的
对于server尤其重要，因为他们必须持久化集群的状态
-config-dir
作用：指定service的配置文件和检查定义所在的位置
通常会指定为"某一个路径/consul.d"（通常情况下，.d表示一系列配置文件存放的目录）
-config-file
作用：指定一个要装载的配置文件
该选项可以配置多次，进而配置多个配置文件（后边的会合并前边的，相同的值覆盖）
-dev
作用：创建一个开发环境下的server节点
该参数配置下，不会有任何持久化操作，即不会有任何数据写入到磁盘
这种模式不能用于生产环境（因为第二条）
-bootstrap-expect
作用：该命令通知consul server我们现在准备加入的server节点个数，该参数是为了延迟日志复制的启动直到我们指定数量的server节点成功的加入后启动。
-node
作用：指定节点在集群中的名称
该名称在集群中必须是唯一的（默认采用机器的host）
推荐：直接采用机器的IP
-bind
作用：指明节点的IP地址
-server
作用：指定节点为server
每个数据中心（DC）的server数推荐为3或5（理想的是，最多不要超过5）
所有的server都采用raft一致性算法来确保事务的一致性和线性化，事务修改了集群的状态，且集群的状态保存在每一台server上保证可用性
server也是与其他DC交互的门面（gateway）
-join
作用：将节点加入到集群
-dc
作用：指定机器加入到哪一个dc中
```



## 应用接入



## 参考

[Docker - 容器部署 Consul 集群](https://www.cnblogs.com/lfzm/p/10633595.html)

[consul 集群部署](https://www.cnblogs.com/wangguishe/p/15599233.html)
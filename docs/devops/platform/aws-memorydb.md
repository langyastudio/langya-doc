## 前言

说到云厂商，大家可能第一反应是阿里云、华为云、腾讯云等，国内阿里云是市场份额占用最多的，但全球范围的云计算市场份额，亚马逊才是老大哥：

![img](https://img-note.langyastudio.com/202206041205083.jpeg?x-oss-process=style/watermark)



目前亚马逊云科技提供了 100 余种产品的**免费套餐**。其中，计算资源 Amazon EC2 首年 12 个月免费，750 小时/月；存储资源 Amazon S3 首年 12 个月免费，5GB 标准存储容量；数据库资源 Amazon RDS 首年12个月免费，750 小时；Amazon Dynamo DB 25GB 存储容量永久免费等等。

它有**三种不同类型的免费优惠**可供选择：

- 免费试用

  [短期免费试用优惠从您激活特定的服务之日开始计算](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free Tier Types=tier%23trial&awsf.Free Tier Categories=*all#Free_Tier_details)

- 12个月免费

  [自初次注册 AWS 之日起 12 个月内免费使用这些产品](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free Tier Types=tier%2312monthsfree&awsf.Free Tier Categories=*all#Free_Tier_details)

- 永久免费

  [这些免费套餐产品不会过期，且适用于所有 AWS 客户](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free Tier Types=tier%23always-free&awsf.Free Tier Categories=*all&all-free-tier.sort-order=asc&awsf.Free Tier Types=*all&awsf.Free Tier Categories=*all#Free_Tier_details)



没有任何套路，纯纯是白嫖，[AWS 免费套餐，白嫖入口](https://aws.amazon.com/cn/free/?nc2=h_ql_pr_ft&all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all&trk=e0213267-9c8c-4534-bf9b-ecb1c06e4ac6&sc_channel=el)

![image-20220604114758482](https://img-note.langyastudio.com/202206041147574.png?x-oss-process=style/watermark)



本文将以 **Amazon MemoryDB** 为例，介绍如何从零开始并构建你专属的 Redis 内存数据库。

## MemoryDB 优势

搭建传统的自建 Redis，需要考虑性能、数据持久性、扩展性、防火墙等一系列搭建环境，步骤很繁琐，费时费力。通过云数据库搭建 Redis，从**效率、性能、数据安全、扩展性**等方面来说，解决了传统方式的痛点，节省大量时间与运维成本。如：

### 通过 Redis 快速构建

通过连续五年被 Stack Overflow 评为“最受欢迎”的数据库 Redis 来快速构建应用程序。使用灵活的 Redis 数据结构和 API（例如，流式传输、列表和集合）。



### 实现超快性能

以微秒级的读取和个位数毫秒级的写入延迟和高吞吐量来访问数据。MemoryDB 每天可以处理超过 13 万亿个请求，并支持每秒超过 1.6 亿个请求的峰值。



### 以持久性和高可用性存储数据

MemoryDB 将数据存储在内存中，并使用多可用区事务日志实现快速数据库恢复和重启，而不会丢失数据。由于 MemoryDB 可以持久地存储数据，因此可以将其用作主数据库。

​	

### 轻松扩展

MemoryDB 可将每个集群的存储空间从几 GB 无缝扩展到 100 TB 以上，以满足您的应用程序需求。



## 注册账号

没有 AWS 帐号的小伙伴需要先进行帐号注册：**[注册及上手试用地址](https://aws.amazon.com/cn/getting-started/databases/get-started/?nc=sn&loc=4&trk=fab55528-7c2e-4517-b90e-65b760ecfc1c&sc_channel=el)** （在账单登记页可以使用**国内的信用卡**）

![image-20220604153731187](https://img-note.langyastudio.com/202206041537294.png?x-oss-process=style/watermark)



## 登录控制台

[控制台登录入口](https://us-east-1.console.aws.amazon.com/memorydb/home?region=us-east-1)

输入邮箱地址与密码即可

![image-20220604154228343](https://img-note.langyastudio.com/202206041542448.png?x-oss-process=style/watermark)



进来以后 Service 内容超级多，为了方便，我们可以像下图一样在这里搜 MemoryDB，选择搜索结果的第一个选项

![image-20220604154445529](https://img-note.langyastudio.com/202206041544593.png?x-oss-process=style/watermark)

可以看到整体的界面，点击开始使用即可：

![image-20220604160924065](https://img-note.langyastudio.com/202206041609260.png?x-oss-process=style/watermark)



## MemoryDB 上手教程

Amazon MemoryDB for Redis 是一项**与 Redis 兼容**、极具持久性的内存数据库服务，可实现超快性能。为具有微服务架构的现代化应用程序提供亚毫秒级延迟、高吞吐量和多可用区持久性。

官方文档：https://docs.aws.amazon.com/zh_cn/memorydb/latest/devguide/getting-started.html



### 创建集群

在左侧导航窗格中选择集群，然后点击**创建集群**

![image-20220604161832964](https://img-note.langyastudio.com/202206041618033.png?x-oss-process=style/watermark)

- 集群信息

  输入集群的**名称**、描述信息

  ![image-20220604163456916](https://img-note.langyastudio.com/202206041634956.png?x-oss-process=style/watermark)

- 子网组

  创建一个新的**子网**组，或从可用列表中选择要应用于此集群的现有子网组。子网类似于局域网
  
  ![image-20220605181514795](https://img-note.langyastudio.com/202206051815865.png?x-oss-process=style/watermark)

- 集群设置

  使用 **t4g.small** 实例，2个月内免费（每月免费提供实例和 20GB 数据）

  - 适用于**Redis 版本兼容性**中，接受默认值 `6.2`
  - 适用于**端口**，接受默认 Redis 端口 6379
  - 适用于**参数组**中，接受 `default.memorydb-redis6` 参数组
  
  ![image-20220605181156173](https://img-note.langyastudio.com/202206051811234.png?x-oss-process=style/watermark)

- 安全性

  **安全组** - 充当**防火墙**来控制对集群的网络访问，**很重要！！！**

  **静态加密** – 对磁盘上存储的数据启用加密

  设置后，最后点击创建即可

  ![image-20220605182441915](https://img-note.langyastudio.com/202206051824966.png?x-oss-process=style/watermark)



当您的集群状态为 **available** 时，便可向其授予 EC2 访问权限，连接到集群并开始使用它。

![image-20220605184105508](https://img-note.langyastudio.com/202206051841576.png?x-oss-process=style/watermark)



### 授予访问权限

- 创建 EC2 实例

  此部分假设你已经熟悉 Amazon EC2 实例的启动和连接。有关更多信息，请参阅 [Amazon EC2 入门指南](https://docs.aws.amazon.com/AWSEC2/latest/GettingStartedGuide/)。

  > **密钥对(登录)** 信息，千万不要选择【在没有密钥对的情况下继续】的选项，否则会导致 EC2 实例创建后无法直接连接访问

- 授权访问权限

  通过**安全组**配置授权访问。

  所有 MemoryDB 集群旨在通过 Amazon EC2 实例进行访问。最常见的情况是从同一 Amazon Virtual Private Cloud (Amazon VPC) 中的 Amazon EC2 实例访问 MemoryDB 集群。必须先授权 EC2 实例访问集群，然后您才能从 EC2 实例连接到集群。

  这里为了演示，直接配置完整入站访问，**0.0.0.0/0 即所有设备都可以访问**

  ![image-20220605203834451](https://img-note.langyastudio.com/202206052038576.png?x-oss-process=style/watermark)



### 连接集群

要从 MemoryDB 节点中访问数据，可以使用利用安全套接字层 (SSL) 的客户端，也可以在 Amazon Linux 2 上使用具有 TLS/SSL 的 redis-cli。

若要使用 redis-cli 连接到 Amazon Linux 2 上的 MemoryDB 集群，步骤如下：

- 登录 EC2 命令行控制台

  选择 EC2 Instance Connect 连接类型

  ![image-20220605204242117](https://img-note.langyastudio.com/202206052042221.png?x-oss-proces=style/watermark)


- 下载并编译 redis-cli 实用工具

  在 EC2 实例的命令提示符处，键入以下命令

  ```bash
  #Amazon Linux 2
  $ sudo yum -y install openssl-devel gcc
  $ wget http://download.redis.io/redis-stable.tar.gz
  $ tar xvzf redis-stable.tar.gz
  $ cd redis-stable
  $ make distclean
  $ make redis-cli BUILD_TLS=yes
  $ sudo install -m 755 src/redis-cli /usr/local/bin/
  ```

-  在 EC2 实例的命令提示符处，键入以下命令，并使用你的集群和端口的终端节点替换此示例中显示的相应内容

  ```bash
  # 示例
  # src/redis-cli -c -h Cluster Endpoint --tls -p 6379
  $ src/redis-cli -c -h clustercfg.redis-free.uyejvs.memorydb.ap-southeast-1.amazonaws.com --tls -p 6379
  ```
  
  其中 **Cluster Endpoint** 位于 MemoryDB 集群信息下 **集群端点**
  
  ![image-20220605211153340](https://img-note.langyastudio.com/202206052111465.png?x-oss-process=style/watermark)
  



实操结果如下：
![image-20220605211612948](https://img-note.langyastudio.com/202206052116051.png?x-oss-process=style/watermark)

​      

### 受限 Redis 命令

为了提供托管服务体验，MemoryDB 限制了对某些需要高级特权的命令的访问。以下命令不可用：

- `acl deluser`
- `acl load`
- `acl save`
- `acl setuser`
- `bgrewriteaof`
- `bgsave`
- `cluster addslot`
- `cluster delslot`
- `cluster setslot`
- `config`
- `debug`
- `migrate`
- `module`
- `psync`
- `replicaof`
- `save`
- `shutdown`
- `slaveof`
- `sync`



## 总结

MemoryDB 与 Redis 兼容，是一个很受欢迎的开源数据存储，使您能够使用他们目前已经使用的同样灵活友好的 Redis 数据结构、API 和命令快速构建应用程序。使用 MemoryDB，您的所有数据都存储在内存中，这使您能够实现微秒读取和单位数毫秒的写入延迟和高吞吐量。MemoryDB 还使用多可用区事务日志跨多个可用区 (AZ) 持久存储数据，以实现快速故障切换、数据库恢复和节点重启。

Memory DB 既具有内存中的性能和多可用区持久性，可用作微服务应用程序的高性能主数据库，从而无需分别管理缓存和持久数据库。

**亚马逊云科技还专为开发者们打造了多种学习平台：**

- 入门资源中心：从0到1 轻松上手云服务，内容涵盖：成本管理，上手训练，开发资源：**[点我访问](https://aws.amazon.com/cn/getting-started/?nc1=h_ls&trk=32540c74-46f0-46dc-940d-621a1efeedd0&sc_channel=el)**

- 架构中心：亚马逊云科技架构中心提供了云平台参考架构图表、经过审查的架构解决方案、Well-Architected 最佳实践、模式、图标等：**[点我访问](https://aws.amazon.com/cn/architecture/?intClick=dev-center-2021_main&trk=3fa608de-d954-4355-a20a-324daa58bbeb&sc_channel=el)**

- 构建者库：了解亚马逊云科技如何构建和运营软件：**[点我访问](https://aws.amazon.com/cn/builders-library/?cards-body.sort-by=item.additionalFields.sortDate&cards-body.sort-order=desc&awsf.filter-content-category=\*all&awsf.filter-content-type=\*all&awsf.filter-content-level=\*all&trk=835e6894-d909-4691-aee1-3831428c04bd&sc_channel=el)**

- 用于在亚马逊云科技平台上开发和管理应用程序的工具包：**[点我访问](https://aws.amazon.com/cn/tools/?intClick=dev-center-2021_main&trk=972c69e1-55ec-43af-a503-d458708bb645&sc_channel=el)**
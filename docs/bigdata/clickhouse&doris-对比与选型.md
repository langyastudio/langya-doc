## 背景介绍

ClickHouse 是俄罗斯的搜索公司 Yandex 开源的 MPP 架构的分析引擎，号称比事务数据库快 100-1000 倍，团队有计算机体系结构的大牛，最大的特色是高性能的**向量化执行引擎**，而且功能丰富、可靠性高。

Apache Doris 是由百度贡献的开源 MPP 分析型数据库产品，分布式架构简洁，易于运维。



## 差异和选择建议

### ClickHouse 更优的方面

- github **26.2 k**

- **性能更佳**，导入性能和单表查询性能更好，同时**数据压缩更高、稳定性更好**

- **扩展性更好**，功能丰富，非常多的表引擎，更多复杂结构和分析函数支持，更好的聚合函数以及庞大的优化参数选项

- 集群管理工具更多，更好**多租户和配额管理**，灵活的集群管理，方便的集群间迁移工具



### Doris 更优的方面

- github **6.5k**

- **使用更简单**， **Join 大表性能更好**，查询速度是亚秒级的，导数功能更强大

- **运维更简单**，如灵活的扩缩容能力，故障节点自动恢复，社区提供的支持更好

- 分布式更强，支持事务和幂等性导数



### 那么两者之间如何选择呢

> 都支持 10PB+ 级别的数据，如京东采用 ClickHouse 为主 Doris 为辅的策略，形成高低搭配

- 业务场景复杂、数据规模巨大，选择 ClickHouse

- 业务场景简单、希望运维简单，选择 Doris



## 架构分析

从部署运维、分布式能力、数据导入、查询、存储和使用成本等方面进行对比，这部分会涉及到内核中的设计原理、方案和实现，了解这些原理会有助于理解上文的结论。

### 部署运维

#### 部署和日常运维

部署指部署集群，安装相关依赖和核心组件，修改配置文件，让集群正常运行起来；运维指日常集群版本更新，配置文件更改、扩缩容等相关事项。集群所需组件如下：

- 左侧是 Doris 的部署架构图，JDBC 指接入协议，DNS 是域名和请求分发系统。Managerment Panel 是管控面。Frontend 指前端模块简称 FE，包含了 SQL 解析、查询计划、计划调度、元数据管理等功能，Backend 指后端模块简称 BE 负责存储层、数据读取和写入，另外还有一个 BrokerLoad 导数组件最好是单独部署。所以，Doris 一般只需要FE 和 BE 两个组件

- 右侧是 ClickHouse 的部署架构图，ClickHouse 本身只有一个模块，就是 ClickHouse Server，周边有两个模块，如ClickHouseProxy 主要是转发请求、配额限制和容灾等，ZooKeeper 这块负责分布式 DDL 和副本间数据同步，ClickHouseCopier 负责集群和数据迁移，ClickHouse 一般需要 Server、ZooKeeper 和 CHProxy 三个组件

- 日常运维如更新版本、更改配置文件两者都需要依赖 Ansible 或者 SaltStack 来进行批量更新。**两者都有部分配置文件可以热更新，不用重启节点**，而且有 Session 相关参数可以设置可以覆盖配置文件。Doris 有较多的 SQL 命令协助运维，比如增加节点，Doris 中 Add Backend 即可，ClickHouse 中需要更改配置文件并下发到各个节点上

![developer_7986cd1](https://img-note.langyastudio.com/202211241334820.jpg?x-oss-process=style/watermark)



#### 多租户管理

**ClickHouse 的权限和 Quota 的粒度更细**，可以很方便的**支持多租户使用共享集群**。比如可以设置查询内存、查询线程数量、查询超时等，以便来限制查询的大小；同时结合查询并发和一定时间窗口内的查询数量，以便来控制查询数量。多租户的方案，对发展中的业务非常友好，因为使用共享集群资源，可以快速动态调整配额，如果是独占集群资源利用率不高、扩容相对麻烦。

![developer_7264381](https://img-note.langyastudio.com/202211251214580.jpg?x-oss-process=style/watermark)



#### 集群迁移

ClickHouse 有几个方法实现数据迁移，数据量大通过自带的 Clickhouse-copier 工具进行集群间的数据拷贝，实现数据的跨集群迁移，需要手工配置很多信息；数据量小通过 SQL 命令 remote 关键字实现跨集群的数据迁移。而官方对实现其他存储介质的备份和恢复的推荐是采用文件系统的 snapshot 实现，或者可以通过三方工具（https://github.com/AlexAkulov/clickhouse-backup）来实现。

Doris 通过内置的 backup/restore 命令将数据和元数据备份到三方对象存储或者 HDFS 上，backup 可以通过快照的方式完整导出一致性的数据和元数据，并且可以按照分区来实现增量备份，降低备份的成本。在 Doris 中，有一种变通的迁移集群的方法，把新机器分批加入到已有的集群，然后再把旧机器逐步下线，集群能够自动均衡，这个过程视集群数据量可能会持续数天。



#### 扩容/缩容

**ClickHouse 的扩容缩容复杂**，目前做不到自动在线操作。扩容时需要部署新的节点，添加新分片和副本到配置文件中，并在新节点上创建元数据，如果是扩副本数据会自动均衡，如果是扩分片，需要手工去做均衡，或自研相关工具，让均衡自动进行。

**Doris 支持集群的在线动态扩缩容**，通过内置的 SQL 命令 alter system add/decomission backends 即可进行节点的扩缩容，数据均衡的粒度是 tablet ，每个 tablet 大概是数百兆，扩容后表的 tablet 会自动拷贝到新的 BE 节点，如果在线扩容，应该小批量去增加 BE，避免过于剧烈导致集群不稳定。



#### 操作系统

ClickHouse 支持 FreeBSD、Linux、macOS。而 Doris 只支持 Linux。



### 分布式能力

#### 分布式协议和高可用

ClickHouse 目前版本是基于 ZooKeeper 来存储元数据，包含分布式的 DDL、表和数据 Part 信息。ClickHouse 依赖 Zookeeper 来实现数据的高可用，Zookeeper 带来额外的运维复杂性的同时也有性能问题。

ClickHouse 没有集中的元数据管理，每个节点分别管理，高可用一般依赖业务方来实现。ClickHouse 中某个副本节点宕机，对查询和分布式表的导入没有影响，本地表导入要在导数程序中做灾备方案比如选择健康的副本，对 DDL 操作是有影响的，需要及时处理。

![developer_52bdfce](https://img-note.langyastudio.com/202211241345229.jpg?x-oss-process=style/watermark)

Doris 的 FrontEnd 可以部署 3 个 Follwer + n 个 Oberserver（n>=0）的方式来实现元数据和访问连接的高可用，Follower 参与选主，在有 Follwer 宕机时，会自动的选举出新节点保证读写高可用，Observer 是只读的扩展节点，可以水平扩展实现读的扩展。BE 通过多副本来实现高可用，一般来说也采取默认的三副本，写入的时候采用 Quroum 协议保证数据一致性。

Doris 的元数据和数据多副本存储的，能自动复制具有自动灾备的能力，服务挂了可以自动重启，坏一块盘数据自动均衡，小范围的节点宕机不会影响集群对外的服务，但宕机后数据均衡过程会消耗集群资源，引发短时间的负载过高。架构如下图：

![developer_75063ad](https://img-note.langyastudio.com/202211241344259.jpg?x-oss-process=style/watermark)



### 数据导入

ClickHouse 中并没有后台导数任务这一概念，它更多的是通过各种引擎去连接到各种存储系统中。导数在 1048576 条以内是原子的，要么都生效，要么都失败，但是没有类似 Doris 中事务 ID 的概念，在 Doris 中相同的事务 ID 插入数据是无效的，这也避免了重复的导数，在 CH 中如果导数重复，只能删除重新导入。**CH 中比较有特色的是既可以写分布式表又可以写本地表**。

导入性能因为 ClickHouse 可以导入本地表，而且没有事务的限制，所以导入性能差不多是节点磁盘写入的性能，而 **Doris 的导数受限于只能分布式表的导入，导入性能差一些**。

如果数据量少可以使用 OLAP 中的导数，数据量大逻辑复杂，一般使用 Spark/Flink 等外部计算引擎来做 ETL 和导数功能，主要是导数消耗集群资源。

Doris 中 有 RoutineLoad、BrokerLoad 和 StreamLoad 等丰富内置的导数方式，这些功能非常好用，虽然无法处理复杂的 ETL 逻辑，但是支持简单过滤和转换函数，也能容忍少量的数据异常，同时支持 ACID 和导数幂等性。

![202211251214290](https://img-note.langyastudio.com/202211251419016.jpg?x-oss-process=style/watermark)



### 存储架构

#### MVCC 模型

ClickHouse 有两个操作，一种是 Merge 合并小的 Part 文件到一个大的 Part，提升查询性能避免扫描多个小文件。另外一种是 Mutation 就是在已有的 Part 中实现数据的变更或元数据的变更，如下图的SQL：

```sql
ALTER TABLE [db.]table DELETE WHERE filter_expr;
ALTER TABLE [db.]table UPDATE column1 = expr1 [, …] WHERE filter_expr;
```

![developer_8395a09](https://img-note.langyastudio.com/202211241351122.jpg?x-oss-process=style/watermark)



Doris 的存储部分参考 GoogleMesa，采用的 MVCC 模式，MVCC 指 Multi-version concurrency control 多版本控制，通过版本可以实现事务的两段提交，可以通过版本进行小文件合并，也可以在明细表和物化视图之间实现强一致性。

![developer_4723871](https://img-note.langyastudio.com/202211241350959.jpg?x-oss-process=style/watermark)



#### 存储结构

两者都是列存，列存的好处就是：

- 分析场景中需要读大量行但是少数几个列，只需要读取参与计算的列即可，极大的减低了 IO，加速了查询

- 同一列中的数据属于同一类型，压缩效果显著，节省存储空间，降低了存储成本

- 高压缩比，意味着同等大小的内存能够存放更多数据，系统 Cache 效果更好

![developer_748f3c7](https://img-note.langyastudio.com/202211251215699.jpg?x-oss-process=style/watermark)

Doris 的数据划分方式是 Table、Partition、Bucket/Tablet、Segment 几个部分，其中 Partition 代表数据的纵向划分分区一般是日期列，Bucket/Tablet 一般指数据的横向切割分桶规则一般为某主键， Segment 是具体的存储文件。Segment 中包含数据和索引，数据部分包含多个列的数据按列存放，有三种索引：物理索引、稀疏索引和 ZoneMap 索引。

ClickHouse 中分为 DistributeTable、LocalTable、Partition、Shard、Part、Column 几个部分，差不多能和 Doris 对应起来，区别就是 CH 中每个 Column 都对应一组数据文件和索引文件，**好处就是命中系统 Cache 性能更高，不好的地方就是 IO 较高且文件数量较多**，另外 CH 有 Count 索引，所以 Count 时命中索引会比较快。

通过分区分桶的方式可以让用户自定义数据在集群中的数据分布方式，降低数据查询的扫描量，方便集群的管理。分区作为数据管理的手段， Doris 支持按照 range 分区，ClickHouse 可以表达式来自定义。Doris 可以通过动态分区的配置来按照时间自动创建新的分区，也可以做冷热数据的分级存储。ClickHouse 通过 distrubute 引擎来进行多节点的数据分布，但是因为缺少 bucket 这一层，会导致集群的迁移扩容比较麻烦， Doris 通过分桶的配置可以进一步对数据划分，方便数据的均衡和迁移。

![developer_22619db](https://img-note.langyastudio.com/202211241355665.jpg?x-oss-process=style/watermark)



#### 表引擎/模型

两者都有典型的表类型（引擎类型）的支持

- ClickHouse : 主要是 MergeTree 表引擎家族，主要是 ReplicatedMergeTree 带副本的、ReplacingMergeTree 可以更新数据的和 AggregatingMergeTree 可以聚合数据，另外还有内存字典表可以加载数据字典、内存表可以加速查询或获得更好写入性能。CH 比较特殊地方是分布式表和每个节点的本地表都要单独创建，物化视图无法自动路由
- Doris：可重复的 Duplicated Key 就是明细表，按维度聚合的 Aggregate Key，唯一主键 Unique Key，UniqueKey这个可以视为 AggregateKey 的特例，另外在这三种基础上可以建立 Rollup（上卷），可以理解为物化视图

![developer_1e3fee7](https://img-note.langyastudio.com/202211241355806.jpg?x-oss-process=style/watermark)



#### 数据类型

ClickHouse 中存在**较多的复杂类型的支持**如 Array/Nested/Map/Tuple/Enum 等，这些类型能够满足一些特性场景，还是比较好用的。

![developer_d3dcc9d](https://img-note.langyastudio.com/202211241356517.jpg?x-oss-process=style/watermark)



### 数据查询

#### 查询架构

![developer_008fc48](https://img-note.langyastudio.com/202211242118146.jpg?x-oss-process=style/watermark)

分布式查询指查询分布在多台服务器上的数据，就如同使用一张表一样，分布式 Join 比较繁琐，**Doris 的分布式 Join 有Local join，Broadcast join，Shuffle join，Hash join 等方式。ClickHouse 只有 Local 和 Broadcast 两种 Join**，这种架构比较简单，也限制了 Join SQL 的自由度，变通的方式是通过子查询和查询嵌套来实现多级的 Join。



#### 并发能力

**OLAP 因为 MPP 架构，每一个 SQL 所有节点都会参与计算**，以此来加速海量计算，因此一个集群的并发能力和单台没有太大的区别，所以，OLAP 和数据库类似，并不是能够承担极高并发的系统。但是也并非毫无办法，比如通过增加副本数来达到承载较大并发的能力。比如 4 个分片 1 个副本，能承担 100 QPS，那么如果要承担 500 的 QPS，则只需要把副本数扩展到 5 个副本即可。另外一点很重要的是查询能否利用到 Cache，包括 ResultCache，Page Cache 和 CPU Cache，这样并发还能提升一个很大的台阶。在查询时间优化到几十毫秒以内，**增加副本数可以让 QPS 达到数千甚至上万**。

Doris 有两点比较优势，一是**副本数的设置是在表级别的**，只需要把并发大的表设置副本数多一些即可，当然副本数不能超过集群的节点数，而 ClickHouse 的副本数设置是集群级别的。



#### SQL 支持

Doris 与 MySQL 语法兼容，支持 SQL 99 规范以及部分 SQL 2003 新标准（比如窗口函数，Grouping sets）。

ClickHouse 部分支持 SQL-2011 标准（https://clickhouse.tech/docs/en/sql-reference/ansi/），但是由于 Planner 的一些限制，ClickHouse 的多表关联需要对 SQL 做大量改写工作，比如需要手动下推条件到子查询中，所以复杂查询使用不太方便。

ClickHouse 支持支持 ODBC、JDBC、HTTP 接口，Doris 支持 JDBC 和 ODBC 接口。



#### 联邦查询

![developer_fabe538](https://img-note.langyastudio.com/202211251215429.jpg?x-oss-process=style/watermark)



#### 函数支持

![developer_73d9fb2](https://img-note.langyastudio.com/202211251215447.jpg?x-oss-process=style/watermark)

![developer_d90d4e2](https://img-note.langyastudio.com/202211251216751.jpg?x-oss-process=style/watermark)



### 使用成本

#### 使用成本

Doris 使用成本低，是一个强一致性元数据的系统，导数功能较为完备，查询 SQL 的标准兼容好无需额外的工作，弹性伸缩能力要好，而 ClickHouse 则需要做较多工作。



#### 代码框架

ClickHouse 包含 ClickHouse Client/Copier/Server 这几个比较主要的模块，其中 Client 是日常使用的命令行客户端，Copier 是数据迁移工具，Server 是集群核心服务。Server 部分包含 Parser、Interpreter、Storage、Database、Function 等模块。代码整体上是 C++11 以上的风格，大量使用 Poco 库，大量使用较新的语言特性。

因此 **ClickHouse 对二次开发更加友好**，技术栈单一，且测试框架完善，模块间互相依赖关系相对较小。

![developer_4c7bd56](https://img-note.langyastudio.com/202211251216417.jpg?x-oss-process=style/watermark)

Doris 整体架构分为 FrontEnd 和 BackEnd，FE 由 Java 编写，BE 是 C/C++，通信部分是 BRPC。FE 中包含了元数据、SQL Parser、Optimizer、Planner 和 Coodinator 几个部分，BE 中包含写入、存储、索引和查询执行部分。Doris 的代码风格整体质量是比较高的，风格统一，有较为完善的单测用例，如果要在 Doris 上做二次开发，则需要熟悉 Java 或C++。

![img](https://img-note.langyastudio.com/202211251246284.png?x-oss-process=style/watermark)



#### 接口对接

左侧是 Doris，右侧是 Clickhouse。可以看出 **Clickhouse 的兼容性更好**。

![image-20221125171515981](https://img-note.langyastudio.com/202211251715091.png?x-oss-process=style/watermark)



## 性能测试

**TPC-DS 测试**是大数据领域比较常用的一个测试，24 张表、99 个 SQL，可以生成不同容量的数据，京东内部常用来做不同引擎的对比测试。

- 为了简化测试过程，挑选了 9 个关联查询的 SQL，然后自己构造了 9 个单表查询的 SQL，共 18 个 SQL 来测试性能

- 这两个引擎都无法全部支持 99 个SQL，不支持的部分根据各个引擎不同特点，进行手工 SQL 改写让其能正确执行，Doris 改动量较小，ClickHouse 的多表关联几乎都要改写

如下是一个典型的多表关联例子，在 Doris 中不需要做改动，但是在 ClickHouse 中，需要改为多个 Global Inner Join 来执行， ClickHouse 的多表关联查询一般都需要改写。

```sql
--Doris/DorisDB SQL 1
select i_item_id, avg(cs_quantity) agg1, avg(cs_list_price) agg2, avg(cs_coupon_amt) agg3, avg(cs_sales_price) agg4
from catalog_sales, customer_demographics, date_dim, item, promotion
where cs_sold_date_sk = d_date_sk and 
    cs_item_sk = i_item_sk and
    cs_bill_cdemo_sk = cd_demo_sk and
    cs_promo_sk = p_promo_sk and
    cd_gender = 'M' and 
    cd_marital_status = 'D' and
    cd_education_status = 'Advanced Degree' and
    (p_channel_email = 'N' or p_channel_event = 'N') and d_year = 1998 
group by i_item_id order by i_item_id limit 10;
--ClickHouse SQL 1
select i_item_id, avg(cs_quantity) agg1, avg(cs_list_price) agg2, avg(cs_coupon_amt) agg3, avg(cs_sales_price) agg4 from catalog_sales_dist
global inner join (select cd_demo_sk from customer_demographics_dist where cd_gender = 'M' and cd_marital_status = 'D' and cd_education_status = 'Advanced Degree' ) 
on cs_bill_cdemo_sk = cd_demo_sk
global inner join (select d_date_sk from date_dim_dist where d_year = 1998 ) on  cs_sold_date_sk = d_date_sk
global inner join ( select i_item_sk, i_item_id from item_dist ) on cs_item_sk = i_item_sk
global inner join ( select p_promo_sk from promotion_dist where p_channel_email = 'N' or p_channel_event = 'N') on cs_promo_sk = p_promo_sk
group by i_item_id order by i_item_id limit 10;
```



### 测试环境

- 硬件：3 台 32 核/ 128G 内存/ HDD 磁盘的服务器

- 软件：Doris 0.13.1、ClickHouse 21.3.13.1

- 配置：3 个分片 1 副本，都是默认配置



### 测试总结

- **单表性能 ClickHouse 更好**，无论是查询延时还是并发能力

- **多表性能 Doris 优势更明显**，特别是复杂 Join 和大表 Join 大表的场景，另外 ClickHouse 需要改写 SQL 有一些工作量（单表 join 小维表的性能很好）



### 单表延时和并发

单表的 SQL 都比较简单，大部分是全表分组 group by 之后 avg/sum/count/count distinct 的各种聚合，单表查询时间如下，可以看出整体上 ClickHouse 要明显好一些，同时，为了压测方便我们找了 2 个延时低的 SQL Single4 和 Single5 测试了一下不同并发下的 QPS，发现也是 ClickHouse 更优一些。

ClickHouse 的单表性能好，**得益于向量化执行引擎**，在数据密集情况下，利用内存的 PageCache 和 CPU 的 L2 Cache 可以大大加速查询过程。

![developer_c41d642](https://img-note.langyastudio.com/202211251217378.jpg?x-oss-process=style/watermark)

![developer_a1c56f5](https://img-note.langyastudio.com/202211251220034.jpg?x-oss-process=style/watermark)



### Join 的延时和并发

从多表测试来看，Doris 在 Join6、Join7、Join8、Join9 要好一些，Join3 两者差不多，其他情况 ClickHouse 好一些。同样，我们挑选了 2 个延时低的 SQL Join8 和 Join9 做了一下并发测试，并发的测试结果和延时表现比较匹配，延时低的SQL 测试并发时 QPS 同样高。

Doris 多表关联有四种 Join 方式，BroadCast Join，Shuffle Join/Bucket, Shuffle Join和 Colocation Join，ClickHouse 只有 Global Join（就是 BroadCast Join）和 Local Join（对应 Colocation Join），因此在大表 Join 大表时，要把右表广播到所有节点，性能可想而知。

**Doris 的执行计划对 SQL 进行了较多的优化**，因此多表关联中的大部分情况，能找到最优的执行方式，因此多表关联性能较好一些，但是也**并不是所有的关联 SQL 都要好**。

![developer_ffb7aba](https://img-note.langyastudio.com/202211251217072.jpg?x-oss-process=style/watermark)

![developer_0aa0806](https://img-note.langyastudio.com/202211251217397.jpg?x-oss-process=style/watermark)



### ClickHouse 小表不同数据量下延时

不是说 ClickHouse 的 Join 性能不行么，为什么表现并不差呢？因此，贴一个去年做的一组 ClickHouse 大小表的测试供大家参考，就是用一个大表关联查询不同数据规模的小表，看 Join 性能情况怎么样。横轴是指小表的不同数据量，纵轴是执行时间。可以看出，因为 Join 机制不一样，ClickHouse 的延时随小表数据量加大梯度更大，**ClickHouse 小表数据量 1000 万以内尚可，超过 1000 万性能比就比较差了**。

![developer_3e01df3](https://img-note.langyastudio.com/202211251218135.jpg?x-oss-process=style/watermark)



## 现有客户

### ClickHouse

字节、阿里、腾讯、京东、快手都在使用。字节属于极其重度用户，节点超过一万五千个



### Doris

百度系公司用的多，美团、小米、京东、快手等



## 附录参考

![img](https://img-note.langyastudio.com/202211171436220.png?x-oss-process=style/watermark)

![img](https://img-note.langyastudio.com/202211171436160.png?x-oss-process=style/watermark)



## 参考资料

- [Google mesa](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/42851.pdf)
- [Oracle Berkeley DB High Availability](https://www.oracle.com/technetwork/database/database-technologies/berkeleydb/overview/berkeleydb-je-ha-whitepaper-132079.pdf)
- [ClickHouse 内核分析-ZooKeeper在分布式集群中的作用](https://developer.aliyun.com/article/762917)
- [ClickHouse Polymorphic Parts特性浅析](https://www.jianshu.com/p/f3881fae4b9e)
- [Everything You Always Wanted to Know About Compiled and Vectorized Queries But Were Afraid to Ask](http://www.vldb.org/pvldb/vol11/p2209-kersten.pdf) 
- [Doris 与 ClickHouse 的深度对比及选型建议](https://developer.baidu.com/article/detail.html?id=294354)
- [System Properties Comparison Apache Doris vs. ClickHouse vs. Google BigQuery](https://db-engines.com/en/system/Apache+Doris%3BClickHouse%3BGoogle+BigQuery)
- [ClickHouse和Doris打配合，京东万亿级OLAP稳得一批](https://www.sohu.com/a/496321317_411876)
- [Apache Doris vs Clickhouse vs Greenplum](https://www.jianshu.com/p/baec20f2a465)
- [Apache Doris产品调研报告](https://www.gbase8.cn/5113)
- [DorisDB vs ClickHouse SSB对比测试](https://fuxkdb.com/2021/02/12/2021-02-12-DorisDB-vs-ClickHouse-SSB%E5%AF%B9%E6%AF%94%E6%B5%8B%E8%AF%95/#DorisDB-vs-ClickHouse-SSB%E5%AF%B9%E6%AF%94%E6%B5%8B%E8%AF%95)
- [聊聊 Apache Doris 的性能优化实战技巧](https://my.oschina.net/u/4855753/blog/5420300)
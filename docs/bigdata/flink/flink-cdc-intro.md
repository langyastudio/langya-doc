### CDC

CDC 的全称是 Change Data Capture ，在广义的概念上，只要是能捕获数据变更的技术，我们都可以称之为 CDC 。目前通常描述的 CDC 技术主要面向数据库的变更，是一种用于捕获数据库中数据变更的技术。CDC 技术的应用场景非常广泛：

- 数据同步：用于备份，容灾
- 数据分发：一个数据源分发给多个下游系统
- 数据采集：面向数据仓库 / 数据湖的 ETL 数据集成，是非常重要的数据源

![null](https://img-note.langyastudio.com/202210271050129.png?x-oss-process=style/watermark)



### 传统 CDC ETL 分析

![null](https://img-note.langyastudio.com/202210271056197.jpeg?x-oss-process=style/watermark)

传统的基于 CDC 的 ETL 分析中，**数据采集工具是必须的，国外用户常用 Debezium**，国内用户常用阿里开源的 Canal，采集工具负责采集数据库的增量数据，一些采集工具也支持同步全量数据。采集到的数据一般**输出到消息中间件**如 Kafka，然后 Flink 计算引擎再去消费这一部分数据写入到目的端，目的端可以是各种 DB，数据湖，实时数仓和离线数仓。

注意，Flink 提供了 changelog-json format，可以将 changelog 数据写入离线数仓如 Hive / HDFS；对于实时数仓，Flink 支持将 changelog 通过 upsert-kafka connector 直接写入 Kafka。



![null](https://img-note.langyastudio.com/202210271056906.jpeg?x-oss-process=style/watermark)

我们一直在思考是否可以使用 Flink CDC 去替换上图中虚线框内的**采集组件和消息队列**，从而简化分析链路，降低维护成本。同时更少的组件也意味着数据时效性能够进一步提高。答案是可以的，于是就有了我们基于 Flink CDC 的 ETL 分析流程。



### 基于 Flink CDC 的 ETL 分析

在使用了 Flink CDC 之后，除了组件更少，维护更方便外，另一个优势是通过 **Flink SQL 极大地降低了用户使用门槛**，可以看下面的例子：

![null](https://img-note.langyastudio.com/202210271058453.png?x-oss-process=style/watermark)

该例子是通过 Flink CDC 去同步数据库数据并写入到 TiDB，用户直接使用 Flink SQL 创建了产品和订单的 MySQL-CDC 表，然后对数据流进行 JOIN 加工，加工后直接写入到下游数据库。通过一个 Flink SQL 作业就完成了 CDC 的数据分析，加工和同步。

大家会发现这是一个纯 SQL 作业，这意味着只要会 SQL 的 BI，业务线同学都可以完成此类工作。与此同时，用户也可以利用 **Flink SQL 提供的丰富语法进行数据清洗、分析、聚合**。



### Flink CDC

Flink CDC (CDC Connectors for Apache Flink®)[1] 是 Apache Flink® 的一组 **Source 连接器**，支持从 MySQL，MariaDB, RDS MySQL，Aurora MySQL，PolarDB MySQL，PostgreSQL，Oracle，MongoDB，SqlServer，OceanBase，PolarDB-X，TiDB 等数据库中实时地读取存量历史数据和增量变更数据，用户既可以选择**用户友好的 SQL API**，也可以使用**功能更为强大的 DataStream API**。

作为新一代的数据集成框架， Flink CDC 不仅可以替代传统的 DataX 和 Canal 工具做实时数据同步，将数据库的全量和增量数据一体化地同步到消息队列和数据仓库中；也可以用于实时数据集成，将数据库数据实时入湖入仓；同时还支持强大的数据加工能力，可以**通过 SQL 对数据库数据做实时关联、打宽、聚合，并将物化结果写入到各种存储中**。

相对于其他数据集成框架，Flink CDC 具有全增量一体化、无锁读取、并发读取、表结构变更自动同步、分布式架构等技术优势。

![img](https://img-note.langyastudio.com/202210270955153.png?x-oss-process=style/watermark)
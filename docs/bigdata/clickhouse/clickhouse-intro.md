## 综述

ClickHouse 是一个用于联机分析(OLAP)的列式数据库管理系统(DBMS)。

- 在传统的行式数据库系统中，数据按如下顺序存储：

| Row  | WatchID     | JavaEnable | Title              | GoodEvent | EventTime           |
| ---- | ----------- | ---------- | ------------------ | --------- | ------------------- |
| #0   | 89354350662 | 1          | Investor Relations | 1         | 2016-05-18 05:19:20 |
| #1   | 90329509958 | 0          | Contact us         | 1         | 2016-05-18 08:10:20 |
| #2   | 89953706054 | 1          | Mission            | 1         | 2016-05-18 07:38:00 |
| #N   | …           | …          | …                  | …         | …                   |

处于同一行中的数据总是被物理的存储在一起。常见的行式数据库系统有：`MySQL`、`Postgres`和`MS SQL Server`。

- 在列式数据库系统中，数据按如下的顺序存储：

| Row:        | #0                  | #1                  | #2                  | #N   |
| ----------- | ------------------- | ------------------- | ------------------- | ---- |
| WatchID:    | 89354350662         | 90329509958         | 89953706054         | …    |
| JavaEnable: | 1                   | 0                   | 1                   | …    |
| Title:      | Investor Relations  | Contact us          | Mission             | …    |
| GoodEvent:  | 1                   | 1                   | 1                   | …    |
| EventTime:  | 2016-05-18 05:19:20 | 2016-05-18 08:10:20 | 2016-05-18 07:38:00 | …    |

这些示例只显示了数据的排列顺序。来自不同列的值被单独存储，来自同一列的数据被存储在一起。



### 关键特征

- 绝大多数是读请求
- 数据以相当大的批次(> 1000行)更新，而不是单行更新；或者根本没有更新
- 已添加到数据库的数据不能修改
- 对于读取，从数据库中提取相当多的行，但**只提取列的一小部分**
- **宽表**，即每个表包含着大量的列
- 每个查询有一个大表。除了他以外，其他的都很小
- **查询相对较少**(QPS 100 以内)
- 对于简单查询，允许延迟大约50毫秒
- 列中的数据相对较小：数字和短字符串(例如，每个URL 60个字节)
- **处理单个查询时需要高吞吐量**(每台服务器每秒可达数十亿行)
- 事务不是必须的
- 对数据一致性要求低
- 查询结果明显小于源数据。换句话说，数据经过过滤或聚合，因此结果适合于单个服务器的 RAM 中

> CK 的优化引擎能力比较差，有关联运算时甚至跑不过 MySQL



### 限制

- 没有完整的事务支持
- 缺少高频率，低延迟的修改或删除已存在数据的能力。仅能用于批量删除或修改数据，但这符合 [GDPR](https://gdpr-info.eu/)。

- 稀疏索引使得 ClickHouse 不适合通过其键检索单行的点查询



### 更适合 OLAP 场景的原因

对于大多数查询而言，处理速度至少提高了100倍

**行式**

![Row oriented](https://clickhouse.com/docs/assets/images/row-oriented-d515facb5bffb48cbd09dc7d064c8816.gif)



**列式**

![Column oriented](https://clickhouse.com/docs/assets/images/column-oriented-b992c529fa4085b63b57452fbbeb27ba.gif)



### 特性

除了 OLAP 通用的特性外，还有如下特性：

**真正的列式数据库管理系统**

在一个真正的列式数据库管理系统中，除了数据本身外不应该存在其他额外的数据。这意味着为了避免在值旁边存储它们的长度«number»，你必须支持**固定长度**数值类型。



**数据的磁盘存储**

许多的列式数据库(如 SAP HANA, Google PowerDrill) 只能在内存中工作，这种方式会造成比实际更多的设备预算。

ClickHouse 被设计用于工作在传统磁盘上的系统，它提供每GB更低的存储成本，但如果可以使用 SSD 和内存，它也会合理的利用这些资源。



**多服务器分布式处理**

数据可以保存在不同的 shard 上，每一个 shard 都由一组用于容错的 replica 组成，查询可以并行地在所有 shard 上进行处理。这些对用户来说是透明的



**支持 SQL**

- ClickHouse 支持一种[基于SQL的声明式查询语言](https://clickhouse.com/docs/zh/sql-reference/)，它在许多情况下与 [ANSI SQL标准](https://clickhouse.com/docs/zh/sql-reference/ansi) 相同

- 支持的查询 [GROUP BY](https://clickhouse.com/docs/zh/sql-reference/statements/select/group-by),  [ORDER BY](https://clickhouse.com/docs/zh/sql-reference/statements/select/order-by),  [FROM](https://clickhouse.com/docs/zh/sql-reference/statements/select/from),  [JOIN](https://clickhouse.com/docs/zh/sql-reference/statements/select/join),  [IN ](https://clickhouse.com/docs/zh/sql-reference/operators/in)以及非相关子查询

- 相关(依赖性)子查询和窗口函数暂不受支持，但将来会被实现



**向量引擎**

为了高效的使用 CPU，数据不仅仅按列存储，同时还按向量(列的一部分)进行处理，这样可以更加高效地使用 CPU。为了提高 CPU 效率，查询语言必须是声明型的( SQL 或 MDX )， 或者至少一个向量(J，K)。 查询应该只包含隐式循环，允许进行优化



**索引**

按照主键对数据进行排序，这将帮助 ClickHouse 在几十毫秒以内完成对数据特定值或范围的查找



### 核心概念

**表引擎 Engine**

表引擎决定了数据在文件系统中的存储方式，常用的也是官方推荐的存储引擎是 MergeTree 系列，如果需要数据副本的话可以使用 ReplicatedMergeTree 系列，相当于 MergeTree 的副本版本，读取集群数据需要使用分布式表引擎Distribute。



**表分区 Partition**

表中的数据可以按照**指定的字段分区存储**，每个分区在文件系统中都是都以目录的形式存在。常用**时间字段作为分区字段**，数据量大的表可以按照小时分区，数据量小的表可以在按照天分区或者月分区，查询时，使用分区字段作为 Where 条件，可以有效的过滤掉大量非结果集数据。



**分片 Shard**

一个分片本身就是 ClickHouse 一个**实例节点**，分片的本质就是为了提高查询效率，将一份全量的数据分成多份（片），从而降低单节点的数据扫描数量，提高查询性能。



**复制集 Replication**

简单理解就是相同的数据备份，在 ClickHouse 中通过复制集，我们实现了保障数据可靠性外，也通过多副本的方式，增加了 ClickHouse 查询的并发能力。这里一般有 2 种方式：1. 基于 ZooKeeper 的表复制方式；2. 基于 Cluster 的复制方式。



**集群 Cluster**

可以使用多个 ClickHouse 实例组成一个集群，并统一对外提供服务。



## 性能

- 单个大查询的吞吐量

吞吐量可以使用每秒处理的行数或每秒处理的字节数来衡量。如果数据被放置在 **page cache** 中，则一个不太复杂的查询在单个服务器上大约能够以 2-10GB／s（未压缩）的速度进行处理（对于简单的查询，速度可以达到 30 GB/s）。如果数据没有在 page cache 中的话，那么速度将取决于你的**磁盘系统**和**数据的压缩率**。例如，如果一个磁盘允许以 400MB/s 的速度读取数据，并且数据压缩率是 3，则数据的处理速度为 1.2GB/s。这意味着，如果你是在提取一个10 字节的列，那么它的处理速度大约是 1-2 亿行每秒

对于分布式处理，处理速度几乎是线性扩展的，但这受限于聚合或排序的结果不是那么大的情况下。

- 处理短查询的延迟时间

如果一个查询使用主键并且没有太多行(几十万)进行处理，并且没有查询太多的列，那么在数据被 page cache 缓存的情况下，它的延迟应该小于 50 毫秒(在最佳的情况下应该小于 10 毫秒)。 否则，延迟取决于数据的查找次数。如果你当前使用的是 HDD，在数据没有加载的情况下，查询所需要的延迟可以通过以下公式计算得知： **查找时间（10 ms） * 查询的列的数量 * 查询的数据块的数量**

- 处理大量短查询的吞吐量

在相同的情况下，ClickHouse 可以在单个服务器上每秒处理数**百个查询**（在最佳的情况下最多可以处理数千个）。但是由于这不适用于分析型场景。因此我们建议每秒最多查询 100 次

- 数据的写入性能

我们建议**每次写入不少于 1000 行的批量写入或每秒不超过一个写入请求**。当使用 tab-separated 格式将一份数据写入到MergeTree 表中时，写入速度大约为 50 到 200MB/s。如果您写入的数据每行为 1Kb，那么写入的速度为 50，000 到200，000 行每秒。如果您的行更小，那么写入速度将更高。



## 安装部署

### 系统要求

ClickHouse 可以在任何具有 x86_64，AArch64 或 PowerPC64LE CPU 架构的 Linux，FreeBSD 或 Mac OS X 上运行。

官方预构建的二进制文件通常针对 x86_64 进行编译，并利用 `SSE 4.2` 指令集，因此除非另有说明，支持它的 CPU 使用将成为额外的系统需求。下面是检查当前 CPU 是否支持 SSE 4.2 的命令:

```bash
$ grep -q sse4_2 /proc/cpuinfo && echo "SSE 4.2 supported" || echo "SSE 4.2 not supported"
```

要在不支持 `SSE 4.2` 或 `AArch64`，`PowerPC64LE` 架构的处理器上运行 ClickHouse，您应该通过适当的配置调整从[源代码构建 ClickHouse](https://clickhouse.com/docs/zh/getting-started/install#from-sources)。



### Docker 安装

https://hub.docker.com/r/bitnami/clickhouse/

https://hub.docker.com/r/clickhouse/clickhouse-server/



## 快速上手

### 创建数据库

ClickHouse 在逻辑上将表分组为数据库。包含一个 `default` 数据库，但我们将创建一个新的数据库 `fi_osp65`:

```sql
CREATE DATABASE IF NOT EXISTS fi_osp65 [ ENGINE = Atomic]
```



### 创建表

与创建数据库相比，创建表的语法要复杂得多（请参阅[参考资料](https://clickhouse.com/docs/zh/sql-reference/statements/create). 一般 `CREATE TABLE` 声明必须指定三个关键的事情:

- 要创建的表的名称
- 表结构，例如：列名和对应的 [数据类型](https://clickhouse.com/docs/zh/sql-reference/data-types/)。
- [表引擎](https://clickhouse.com/docs/zh/engines/table-engines/)及其设置，这决定了对此表的查询操作是如何在物理层面执行的所有细节



## 参考

[Clickhouse 官方文档](https://clickhouse.com/docs/zh/)

[ClickHouse 挺快，esProc SPL 更快](https://mp.weixin.qq.com/s/DyRhLjJzYuCJqXSSeriPOg)


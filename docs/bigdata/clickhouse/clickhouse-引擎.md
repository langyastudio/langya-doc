## 数据库引擎

数据库引擎允许处理数据表。默认情况下，ClickHouse 使用 [Atomic](https://clickhouse.com/docs/zh/engines/database-engines/atomic) 数据库引擎。

数据库 `Atomic` 中的所有表都有唯一的 [UUID](https://clickhouse.com/docs/zh/sql-reference/data-types/uuid)，并将数据存储在目录 `/clickhouse_path/store/xxx/xxxyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy/`，其中 `xxxyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy` 是该表的 UUID。



还可以使用以下数据库引擎：

### [Lazy](https://clickhouse.com/docs/zh/engines/database-engines/lazy)

在最后一次访问之后，只在 RAM 中保存 `expiration_time_in_seconds` 秒。只能用于 Log 日志表



### 远程数据库

ClickHouse 和 **远程数据库** 之间交换数据

#### [MySQL](https://clickhouse.com/docs/zh/engines/database-engines/mysql)

支持的数据类型

**其他的 MySQL 数据类型将全部都转换为 [String](https://clickhouse.com/docs/zh/sql-reference/data-types/string).**

| MySQL                            | ClickHouse                                                   |
| -------------------------------- | ------------------------------------------------------------ |
| UNSIGNED TINYINT                 | [UInt8](https://clickhouse.com/docs/zh/sql-reference/data-types/int-uint) |
| TINYINT                          | [Int8](https://clickhouse.com/docs/zh/sql-reference/data-types/int-uint) |
| UNSIGNED SMALLINT                | [UInt16](https://clickhouse.com/docs/zh/sql-reference/data-types/int-uint) |
| SMALLINT                         | [Int16](https://clickhouse.com/docs/zh/sql-reference/data-types/int-uint) |
| UNSIGNED INT, UNSIGNED MEDIUMINT | [UInt32](https://clickhouse.com/docs/zh/sql-reference/data-types/int-uint) |
| INT, MEDIUMINT                   | [Int32](https://clickhouse.com/docs/zh/sql-reference/data-types/int-uint) |
| UNSIGNED BIGINT                  | [UInt64](https://clickhouse.com/docs/zh/sql-reference/data-types/int-uint) |
| BIGINT                           | [Int64](https://clickhouse.com/docs/zh/sql-reference/data-types/int-uint) |
| FLOAT                            | [Float32](https://clickhouse.com/docs/zh/sql-reference/data-types/float) |
| DOUBLE                           | [Float64](https://clickhouse.com/docs/zh/sql-reference/data-types/float) |
| DATE                             | [Date](https://clickhouse.com/docs/zh/sql-reference/data-types/date) |
| DATETIME, TIMESTAMP              | [DateTime](https://clickhouse.com/docs/zh/sql-reference/data-types/datetime) |
| BINARY                           | [FixedString](https://clickhouse.com/docs/zh/sql-reference/data-types/fixedstring) |

[Nullable ](https://clickhouse.com/docs/zh/sql-reference/data-types/nullable) 已经支持



#### [PostgreSQL](https://clickhouse.com/docs/zh/engines/database-engines/postgresql)

#### [SQLite](https://clickhouse.com/docs/zh/engines/database-engines/sqlite)



## 表引擎

表引擎（即表的类型）决定了：

- 数据的存储方式和位置，写到哪里以及从哪里读取数据
- 支持哪些查询以及如何支持
- 并发数据访问
- 索引的使用（如果存在）
- 是否可以执行多线程请求
- 数据复制参数

![image.png](https://img-note.langyastudio.com/202211081548726.png?x-oss-process=style/watermark)

### [MergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#mergetree) 合并家族树

适用于**高负载**任务的最通用和**功能最强大**的表引擎。

`MergeTree` 系列的引擎被设计用于**插入极大量的数据**到一张表当中。数据可以以**数据片段**的形式一个接着一个的快速写入，数据片段在**后台按照一定的规则进行合并**。相比在插入时不断修改（重写）已存储的数据，这种策略会高效很多。



#### 主要特点

- 存储的数据按主键排序

  这使得能够创建一个小型的稀疏索引来加快数据检索

- 如果指定了 [分区键](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/custom-partitioning-key) 的话，可以使用分区

  在相同数据集和相同结果集的情况下 ClickHouse 中某些带分区的操作会比普通操作更快。查询中**指定了分区键时 ClickHouse 会自动截取分区数据**。这也有效增加了查询性能

- 支持数据副本

  `ReplicatedMergeTree` 系列的表提供了数据副本功能。更多信息，请参阅 [数据副本](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/replication) 一节。

- 支持数据采样

  需要的话，可以给表设置一个采样方法



#### 建表

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = MergeTree()
ORDER BY expr
[PARTITION BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'], ...]
[SETTINGS name=value, ...]
```

对于以上参数的描述，可参考 [CREATE 语句 的描述](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree) 。



**子句**

- `ENGINE` - 引擎名和参数。 `ENGINE = MergeTree()`. `MergeTree` 引擎没有参数。

- `ORDER BY` — 排序键。

  可以是一组列的元组或任意的表达式。 例如: `ORDER BY (CounterID, EventDate)` 。

  如果没有使用 `PRIMARY KEY` 显式指定的主键，ClickHouse 会使用排序键作为主键。

  如果不需要排序，可以使用 `ORDER BY tuple()`. 参考 [选择主键](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/mergetree/#selecting-the-primary-key)

- `PARTITION BY` — [分区键](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/custom-partitioning-key) ，可选项。

  大多数情况下，不需要使用分区键。即使需要使用，也不需要使用比月更细粒度的分区键。分区不会加快查询（这与 ORDER BY 表达式不同）。**永远别使用过细粒度的分区键**。不要使用客户端指定分区标识符或分区字段名称来对数据进行分区（而是将**分区字段标识或名称作为 ORDER BY 表达式的第一列来指定分区**）。

  要按月分区，可以使用表达式 `toYYYYMM(date_column)` ，这里的 `date_column` 是一个 [Date](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree) 类型的列。分区名的格式会是 `"YYYYMM"` 。

- `PRIMARY KEY` - 如果要 [选择与排序键不同的主键](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#choosing-a-primary-key-that-differs-from-the-sorting-key)，在这里指定，可选项。

  默认情况下主键跟排序键（由 `ORDER BY` 子句指定）相同。 因此，大部分情况下不需要再专门指定一个 `PRIMARY KEY` 子句。

- `SAMPLE BY` - 用于抽样的表达式，可选项。

  如果要用抽样表达式，主键中必须包含这个表达式。例如： `SAMPLE BY intHash32(UserID) ORDER BY (CounterID, EventDate, intHash32(UserID))` 。

- `TTL` - 指定行存储的持续时间并定义数据片段在硬盘和卷上的移动逻辑的规则列表，可选项。

  表达式中必须存在至少一个 `Date` 或 `DateTime` 类型的列，比如：

  `TTL date + INTERVAl 1 DAY`

  规则的类型 `DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'`指定了当满足条件（到达指定时间）时所要执行的动作：移除过期的行，还是将数据片段（如果数据片段中的所有行都满足表达式的话）移动到指定的磁盘（`TO DISK 'xxx'`) 或 卷（`TO VOLUME 'xxx'`）。默认的规则是移除（`DELETE`）。可以在列表中指定多个规则，但最多只能有一个`DELETE`的规则。

  更多细节，请查看 [表和列的 TTL](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#table_engine-mergetree-ttl)

- `SETTINGS` — 控制 `MergeTree` 行为的额外参数，可选项：

  - `index_granularity` — 索引粒度。索引中相邻的『标记』间的数据行数。默认值8192 。参考[数据存储](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#mergetree-data-storage)。

  - `index_granularity_bytes` — 索引粒度，以字节为单位，默认值: 10Mb。如果想要仅按数据行数限制索引粒度, 请设置为0(不建议)。

  - `min_index_granularity_bytes` - 允许的最小数据粒度，默认值：1024b。该选项用于防止误操作，添加了一个非常低索引粒度的表。参考[数据存储](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#mergetree-data-storage)

  - `enable_mixed_granularity_parts` — 是否启用通过 `index_granularity_bytes` 控制索引粒度的大小。在19.11版本之前, 只有 `index_granularity` 配置能够用于限制索引粒度的大小。当从具有很大的行（几十上百兆字节）的表中查询数据时候，`index_granularity_bytes` 配置能够提升ClickHouse的性能。如果您的表里有很大的行，可以开启这项配置来提升`SELECT` 查询的性能。

  - `use_minimalistic_part_header_in_zookeeper` — ZooKeeper中数据片段存储方式 。如果`use_minimalistic_part_header_in_zookeeper=1` ，ZooKeeper 会存储更少的数据。更多信息参考[服务配置参数]([Server Settings | ClickHouse Documentation](https://clickhouse.com/docs/zh/operations/server-configuration-parameters/settings/))这章中的 [设置描述](https://clickhouse.com/docs/zh/operations/server-configuration-parameters/settings#server-settings-use_minimalistic_part_header_in_zookeeper) 。

  - `min_merge_bytes_to_use_direct_io` — 使用直接 I/O 来操作磁盘的合并操作时要求的最小数据量。合并数据片段时，ClickHouse 会计算要被合并的所有数据的总存储空间。如果大小超过了 `min_merge_bytes_to_use_direct_io` 设置的字节数，则 ClickHouse 将使用直接 I/O 接口（`O_DIRECT` 选项）对磁盘读写。如果设置 `min_merge_bytes_to_use_direct_io = 0` ，则会禁用直接 I/O。默认值：`10 * 1024 * 1024 * 1024` 字节。

    ```
    <a name="mergetree_setting-merge_with_ttl_timeout"></a>
    ```

  - `merge_with_ttl_timeout` — TTL合并频率的最小间隔时间，单位：秒。默认值: 86400 (1 天)。

  - `write_final_mark` — 是否启用在数据片段尾部写入最终索引标记。默认值: 1（不要关闭）。

  - `merge_max_block_size` — 在块中进行合并操作时的最大行数限制。默认值：8192

  - `storage_policy` — 存储策略。 参见 [使用具有多个块的设备进行数据存储](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#table_engine-mergetree-multiple-volumes).

  - `min_bytes_for_wide_part`,`min_rows_for_wide_part` 在数据片段中可以使用`Wide`格式进行存储的最小字节数/行数。您可以不设置、只设置一个，或全都设置。参考：[数据存储](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#mergetree-data-storage)

  - `max_parts_in_total` - 所有分区中最大块的数量(意义不明)

  - `max_compress_block_size` - 在数据压缩写入表前，未压缩数据块的最大大小。您可以在全局设置中设置该值(参见[max_compress_block_size](https://clickhouse.com/docs/zh/operations/settings/settings/#max-compress-block-size))。建表时指定该值会覆盖全局设置。

  - `min_compress_block_size` - 在数据压缩写入表前，未压缩数据块的最小大小。您可以在全局设置中设置该值(参见[min_compress_block_size](https://clickhouse.com/docs/zh/operations/settings/settings/#min-compress-block-size))。建表时指定该值会覆盖全局设置。

  - `max_partitions_to_read` - 一次查询中可访问的分区最大数。您可以在全局设置中设置该值(参见[max_partitions_to_read](https://clickhouse.com/docs/zh/operations/settings/settings/#max_partitions_to_read))。

**示例配置**

```sql
ENGINE MergeTree() PARTITION BY toYYYYMM(EventDate) ORDER BY (CounterID, EventDate, intHash32(UserID)) SAMPLE BY intHash32(UserID) SETTINGS index_granularity=8192
```

在这个例子中，我们设置了按月进行分区。

同时我们设置了一个按用户 ID 哈希的抽样表达式。这使得您可以对该表中每个 `CounterID` 和 `EventDate` 的数据伪随机分布。如果您在查询时指定了 [SAMPLE](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#select-sample-clause) 子句。 ClickHouse会返回对于用户子集的一个均匀的伪随机数据采样。

`index_granularity` 可省略因为 8192 是默认设置 。



#### 数据存储

表由按主键排序的数据片段（DATA PART）组成。

当数据被插入到表中时，会创建**多个数据片段并按主键的字典序排序**。例如，主键是 `(CounterID, Date)` 时，片段中数据首先按 `CounterID` 排序，具有相同 `CounterID` 的部分按 `Date` 排序。

不同分区的数据会被分成不同的片段，ClickHouse 在后台合并数据片段以便更高效存储。不同分区的数据片段不会进行合并。合并机制并不保证具有相同主键的行全都合并到同一个数据片段中。

数据片段可以以 `Wide` 或 `Compact` 格式存储。在 `Wide` 格式下，每一列都会在文件系统中存储为单独的文件，在 `Compact` 格式下所有列都存储在一个文件中。`Compact` 格式可以提高插入量少插入频率频繁时的性能。



#### 主键和索引在查询中的表现

使用索引通常会比全表描述要高效。ClickHouse **不要求主键唯一**，所以可以插入多条具有相同主键的行。

- 主键的选择	

  主键中列的数量并没有明确的限制。依据数据结构，可以在主键包含多些或少些列。长的主键会对插入性能和内存消耗有负面影响，但主键中额外的列并不影响 `SELECT` 查询的性能

- 选择与排序键不同的主键

  排序键用于在数据片段中进行排序，主键用于在索引文件中进行标记的写入。主键表达式元组必须是排序键表达式元组的**前缀**(即主键为(a,b)，排序列必须为(a,b,***\***))。通常只保留少量的列在主键当中用于提升扫描效率，将维度列添加到排序键中

- 索引和分区的应用

  对于 `SELECT` 查询，ClickHouse 分析是否可以使用索引。如果 `WHERE/PREWHERE` 子句具有下面这些表达式（作为完整WHERE条件的一部分或全部）则可以使用索引：进行相等/不相等的比较；对主键列或分区列进行`IN`运算、有固定前缀的 `LIKE` 运算(如name like 'test%')、函数运算(部分函数适用)，还有对上述表达式进行逻辑运算。要检查 ClickHouse 执行一个查询时能否使用索引，可设置 [force_index_by_date](https://clickhouse.com/docs/zh/operations/settings/settings#settings-force_index_by_date) 和 [force_primary_key](https://clickhouse.com/docs/zh/operations/settings/settings) 。

- [跳数索引](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#tiao-shu-suo-yin-fen-duan-hui-zong-suo-yin-shi-yan-xing-de)

  

#### 并发数据访问

当一张表同时被读和更新时，数据从当前查询到的一组片段中读取。没有冗长的的锁。**插入不会阻碍读取**。对表的读操作是自动并行的。



#### [列和表的 TTL](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#table_engine-mergetree-ttl)

TTL 用于设置**值的生命周期**，它既可以为整张表设置，也可以为每个列字段单独设置。

- 列 TTL

`TTL` 子句不能被用于主键字段。当列中的值过期时, ClickHouse 会将它们替换成该列数据类型的**默认值**。

```sql
TTL date_time + INTERVAL 1 MONTH
TTL date_time + INTERVAL 15 HOUR
```

- 表 TTL

如果在两次合并的时间间隔中执行 `SELECT` 查询, 则可能会得到过期的数据。为了避免这种情况，可以在 `SELECT` 之前使用 [OPTIMIZE](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#misc_operations-optimize) 。

> `OPTIMIZE` 语句会引发对数据的大量读写



#### [使用多个块设备进行数据存储](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#table_engine-mergetree-multiple-volumes)

MergeTree 系列表引擎可以将数据存储在多个块设备上。这对某些可以潜在被划分为**“冷”“热”**的表来说是很有用的。



#### 虚拟列

- `_part` - 分区名称
- `_part_index` - 作为请求的结果，按顺序排列的分区数
- `_partition_id` — 分区名称
- `_part_uuid` - 唯一部分标识符（如果 MergeTree 设置`assign_part_uuids` 已启用）
- `_partition_value` — `partition by` 表达式的值（元组）
- `_sample_factor` - 采样因子（来自请求）



#### [自定义分区键](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/custom-partitioning-key)

[MergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree) 系列的表（包括 [可复制表](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/replication) ）可以使用分区。基于 MergeTree 表的 [物化视图](https://clickhouse.com/docs/zh/engines/table-engines/special/materializedview#materializedview) 也支持分区。

分区是在一个表中通过指定的规则划分而成的逻辑数据集。可以按任意标准进行分区，如按月，按日或按事件类型。



#### [数据副本](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/replication)

**只有 MergeTree 系列里的表可支持副本**。副本是表级别的，不是整个服务器级的。所以，服务器里可以同时有复制表和非复制表。副本不依赖分片。每个分片有它自己的独立副本。

要使用副本，需在 [Zookeeper](https://clickhouse.com/docs/zh/operations/server-configuration-parameters/settings#server-settings_zookeeper) 服务器的配置部分设置相应参数。



#### 其他引擎

- [ReplacingMergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/replacingmergetree#replacingmergetree)

  该引擎和 [MergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree) 的不同之处在于它会**删除排序键值相同的重复项**。适用于在后台**清除重复的数据**以节省空间，但是它**不保证没有重复的数据**出现

- [SummingMergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/summingmergetree#summingmergetree)

  当合并 `SummingMergeTree` 表的数据片段时，ClickHouse 会把所有具有相同主键的行合并为一行，该行包含了被合并的行中具有**数值数据类型的列的汇总值**。如果主键的组合方式使得**单个键值对应于大量的**行，则可以显著的减少存储空间并加快数据查询的速度。

  - 如果用于汇总的所有列中的值均为 0，则该行会被删除

  - 如果列不在主键中且无法被汇总，则会在现有的值中任选一个

  - 主键所在的列中的值不会被汇总

- [AggregatingMergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/aggregatingmergetree#aggregatingmergetree)

  可以使用 `AggregatingMergeTree` 表来做**增量数据的聚合统计**，包括物化视图的数据聚合。

- [CollapsingMergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/collapsingmergetree#table_engine-collapsingmergetree)

  会**异步的删除（折叠）**这些除了特定列 `Sign` 有 `1` 和 `-1` 的值以外，其余所有字段的值都相等的成对的行。**没有成对的行会被保留**。如果 `Sign = 1` 则表示这一行是对象的状态，我们称之为«状态»行。如果 `Sign = -1` 则表示是对具有相同属性的状态行的取消，我们称之为«取消»行。该引擎可以显著的降低存储量并提高 `SELECT` 查询效率。为了确保数据的正确性，需要**严格顺序地更新一个对象的变化**。

  ClickHouse 在一个我们无法预料的未知时刻合并数据片段。每组具有相同主键的连续行被减少到不超过两行

  ```
  1. 第一个«取消»和最后一个«状态»行，如果«状态»和«取消»行的数量匹配和最后一个行是«状态»行
  2. 最后一个«状态»行，如果«状态»行比«取消»行多一个或一个以上。
  3. 第一个«取消»行，如果«取消»行比«状态»行多一个或一个以上。
  4. 没有行，在其他所有情况下。
  
  合并会继续，但是 ClickHouse 会把此情况视为逻辑错误并将其记录在服务日志中。这个错误会在相同的数据被插入超过一次时出现。
  ```
  
- [VersionedCollapsingMergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/versionedcollapsingmergetree#versionedcollapsingmergetree)

  用于相同的目的 [折叠树](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/collapsingmergetree) 但使用不同的折叠算法，**允许以多个线程的任何顺序插入数据**。 特别是， `Version` 列有助于正确折叠行，即使它们以错误的顺序插入。 相比之下, `CollapsingMergeTree` 只允许严格连续插入。

- [GraphiteMergeTree](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/graphitemergetree#graphitemergetree)

  用来对 [Graphite](http://graphite.readthedocs.io/en/latest/index.html) 数据进行瘦身及汇总。对于想使用 CH 来存储 Graphite 数据的开发者来说可能有用。



### Log 日志

具有最小功能的[轻量级引擎](https://clickhouse.com/docs/zh/engines/table-engines/log-family/)。当需要快速写入许多**小表（最多约 100 万行）**并在以后整体读取它们时，该类型的引擎是最有效的。

- [TinyLog](https://clickhouse.com/docs/zh/engines/table-engines/log-family/tinylog#tinylog)

- [StripeLog](https://clickhouse.com/docs/zh/engines/table-engines/log-family/stripelog#stripelog)

- [Log](https://clickhouse.com/docs/zh/engines/table-engines/log-family/log#log)

  

### 集成引擎

ClickHouse 提供了多种方式来**与外部系统集成**，包括表引擎。像所有其他的表引擎一样，使用 `CREATE TABLE` 或`ALTER TABLE` 查询语句来完成配置。**对它的查询是代理给外部系统的**。这种透明的查询是这种方法相对于其他集成方法的主要优势之一，比如外部字典或表函数，它们需要在每次使用时使用自定义查询方法。

以下是支持的集成方式:

- [ODBC](https://clickhouse.com/docs/zh/engines/table-engines/integrations/odbc)
- [JDBC](https://clickhouse.com/docs/zh/engines/table-engines/integrations/jdbc)
- [MySQL](https://clickhouse.com/docs/zh/engines/table-engines/integrations/mysql)
- [MongoDB](https://clickhouse.com/docs/zh/engines/table-engines/integrations/mongodb)
- [HDFS](https://clickhouse.com/docs/zh/engines/table-engines/integrations/hdfs)
- [S3](https://clickhouse.com/docs/zh/engines/table-engines/integrations/s3)
- [Kafka](https://clickhouse.com/docs/zh/engines/table-engines/integrations/kafka)
- [EmbeddedRocksDB](https://clickhouse.com/docs/zh/engines/table-engines/integrations/embedded-rocksdb)
- [RabbitMQ](https://clickhouse.com/docs/zh/engines/table-engines/integrations/rabbitmq)
- [PostgreSQL](https://clickhouse.com/docs/zh/engines/table-engines/integrations/postgresql)
- [SQLite](https://clickhouse.com/docs/zh/engines/table-engines/integrations/sqlite)
- [Hive](https://clickhouse.com/docs/zh/engines/table-engines/integrations/hive)
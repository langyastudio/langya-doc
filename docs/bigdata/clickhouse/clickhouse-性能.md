ClickHouse 瓶颈主要在并发上不去，那么对于重要业务，可以选择**多副本和多活集群**，多副本集群多个节点挂会导致业务不可用，可以启动了多活集群的方案，一份数据推多个集群，统一服务查询时，根据集群配置分配不同比例下发查询到各个集群，大小查询分开，把查点的和查列表的分开，查一天和查一个月的分开，提升服务整体 QPS。



## 如何测试

测试查询效率时，需要**先清除查询缓存**，再进行测试，具体脚本如下：

```sql
# Linux
# reset the OS PageCache
sync; echo 1 > /proc/sys/vm/drop_caches

# SQL
-- 重置mark缓存
SYSTEM DROP MARK CACHE;
-- 重置未压缩数据的缓存
SYSTEM DROP UNCOMPRESSED CACHE;
-- 重置已编译的表达式缓存
SYSTEM DROP COMPILED EXPRESSION CACHE;
```



## 编码与压缩

Clickhouse 是支持完全不压缩的，带来的好处就是读取不消耗 CPU，但是相对存储量会大很多，IO 也会有巨大的开销，一般不建议使用这种模式。CK 性能差，很多时候瓶颈在 IO 上。



> 特殊编码与通用的压缩算法相比，区别在于，通用的 LZ4 和 ZSTD 压缩算法是普适行的，不关心数据的分布特点，而特殊编码类型对于特定场景下的数据会有更好的压缩效果

### 通用编码

| 压缩方式       | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| None           | 无压缩                                                       |
| LZ4            | **默认的压缩算法**，非常高效的压缩算法，解压性能可达到单核 4GB/s |
| LZ4HC[(level)] | z4 高压缩率压缩算法版本，level 默认值为 9，支持[1~12]，推荐选用[4~9] |
| ZSTD[(level)]  | zstd 压缩算法，level 默认值为1，支持[1~22]。**高压缩级别**对于非对称场景很有用，例如压缩一次，重复解压缩。更高的级别意味着更好的压缩和更高的 CPU 使用率 |



### 特殊编码

这些编解码器旨在通过使用**数据的特定功能使压缩更有效**。其中一些编解码器本身不压缩数据。相反他们为通用编解码器准备数据，该编解码器比没有这种准备的情况下压缩得更好。

| 编码方式       | 使用场景                                                     |
| -------------- | ------------------------------------------------------------ |
| LowCardinality | **小于 1 万个值**的字符串(低基数的)，且字符串长度越长，效果提升越好 |
| Delta          | 时间序列，不会对数据进行压缩                                 |
| DoubleDelta    | 适用缓慢变化的序列，比如时间序列，对于递增序列效果很好       |
| Gorilla        | 使用缓慢变化的数值类型                                       |
| T64            | 除去随机哈希的整型，比较适合 **Int 类型数据** (including `(U)IntX`、`EnumX`、`Data(Time)`、`DecimalX`) |



#### LowCardinality 低基数类型

把其它数据类型转变为字典编码类型，对很多应用来说，处理字典编码的数据可以显著的增加 [SELECT](https://clickhouse.com/docs/zh/sql-reference/statements/select/) 查询速度。如果一个字典包含**少于 10000 个不同的值**，那么 ClickHouse 可以进行更高效的数据存储和处理。反之如果字典多于 10000，效率会表现的更差。

```sql
CREATE TABLE lc_t
(
    `id` UInt16,
    `strings` LowCardinality(String)
)
ENGINE = MergeTree()
ORDER BY id
```

- `data_type` — [String](https://clickhouse.com/docs/zh/sql-reference/data-types/string), [FixedString](https://clickhouse.com/docs/zh/sql-reference/data-types/fixedstring), [Date](https://clickhouse.com/docs/zh/sql-reference/data-types/date), [DateTime](https://clickhouse.com/docs/zh/sql-reference/data-types/datetime)，包括数字类型，但是 [Decimal](https://clickhouse.com/docs/zh/sql-reference/data-types/decimal) 除外。对一些数据类型来说，`LowCardinality` 并不高效，详查 [allow_suspicious_low_cardinality_types](https://clickhouse.com/docs/zh/operations/settings/settings#allow_suspicious_low_cardinality_types) 设置描述



#### Delta(delta_bytes)

原始值被**两个相邻值的差值替换**，但第一个值保持不变。Up to `delta_bytes` 用于存储增量值，`delta_bytes` 原始值的最大大小也是如此。可能的值：1、2、4、8。



#### DoubleDelta

计算 delta 的 delta 并以紧凑的二进制形式写入。对于具有**恒定步幅的单调序列，例如时间序列数据**，可以获得最佳压缩率。可以与任何固定宽度类型一起使用。实现 Gorilla TSDB 中使用的算法，将其扩展为支持 64 位类型。



#### Gorilla

计算当前值和先前值之间的异或，并以紧凑的二进制形式写入。在存储**一系列变化缓慢的浮点值**时很有效，因为当相邻值是二进制相等时可以获得最佳压缩率。实现 Gorilla TSDB 中使用的算法，将其扩展为支持 64 位类型。



#### FPC

使用两个预测器中的更好者重复预测序列中的下一个**浮点值**，然后将实际值与预测值进行异或，然后前导零压缩结果。与 Gorilla 类似，这在存储**一系列变化缓慢的浮点值**时非常有效。对于 **64 位值（双精度），FPC 比 Gorilla 更快**，对于 32 位值，结果可能会有所不同。



#### T64

在整数数据类型（包括和）`Date` 中**裁剪未使用的高位值**的压缩方法。`DateTime` 在其算法的每一步，编解码器采用 64 个值的块，将它们放入 64x64 位矩阵，转置它，裁剪未使用的值位并将其余的位作为序列返回。未使用的位是在使用压缩的整个数据部分中的最大值和最小值之间没有差异的位。



#### 注意

> Codec is only applicable for data types of size 1, 2, 4, 8 bytes

##### Decimal

- Decimal32(S) 

  -1 * 10^(**9** - S), 1 * 10^(**9** - S) 

- Decimal64(S) 

  -1 * 10^(**18** - S), 1 * 10^(**18** - S) 

- Decimal128(S) 

   -1 * 10^(**38** - S), 1 * 10^(**38** - S) 

兼容的 `Decimal(D, S)` 类型，如 Decimal(22, 6)，本质上使用的是 Decimal128，即 **16 bytes，无法使用特殊编码**，可以根据实际应用场景改为 Decimal(18, 6)/Decimal(18, 3) ，这样使用的是 Decimal64 类型。

Decimail64/32 类型可以配合 **CODEC(T64,  ZSTD(1))**、CODEC(T64, LZ4)、CODEC(DoubleDelta,  ZSTD(1)) 等编码带来 1.5 倍左右的压缩率。

> 由于现代 CPU 不支持 128 位数字，因此 Decimal128 上的操作由软件模拟。所以 Decimal128 的运算速度明显慢于 Decimal32/Decimal64，而且存储空间成倍上升



### 样例

- 每个单独列定义压缩方法

```sql
CREATE TABLE codec_example
(
    dt Date CODEC(ZSTD),
    ts DateTime CODEC(LZ4HC),
    float_value Float32 CODEC(NONE),
    double_value Float64 CODEC(LZ4HC(9)),
    value Float32 CODEC(Delta, ZSTD)
)
ENGINE = <Engine>
...
```



Also you can remove current CODEC from the column and use default compression from config.xml

```sql
ALTER TABLE codec_example MODIFY COLUMN float_value CODEC(Default);
```

 

## Projection 投影

> 因为物化的数据保存在原表的分区，所以数据的更新、合并都是同源的，也就不会出现不一致的情况了
>
> PROJECTION 本质也是在用空间换时间，还是还很划算的

- **part-level 存储** 

  相比普通物化视图是一张独立的表，Projection 物化的数据就保存在原表的分区目录中，支持明细数据的普通Projection 和 预聚合 Projection

- **无感使用，自动命中** 

  可以对一张 MergeTree 创建多个 Projection ，当执行 Select 语句的时候，能根据查询范围，自动匹配最优的 Projection 提供查询加速。如果没有命中 Projection , 就直接查询底表。

- **数据同源、同生共死**



###  PROJECTION 系统表

现在 ClickHouse 也提供了 PROJECTION 的系统表，可以看到相关的存储信息：

```sql
SELECT
    name,
    partition,
    formatReadableSize(bytes_on_disk) AS bytes,
    formatReadableSize(parent_bytes_on_disk) AS parent_bytes,
    parent_rows,
    rows / parent_rows AS ratio
FROM system.projection_parts

Query id: 2887b0e1-b984-4274-862c-0b59c68693c5

┌─name───┬─partition─┬─bytes──────┬─parent_bytes─┬─parent_rows─┬──────ratio─┐
│ agg_p2 │ 201307    │ 490.40 MiB │ 14.06 GiB    │   100000000 │ 0.24070565 │
│ p1     │ 201307    │ 4.95 GiB   │ 18.53 GiB    │   100000000 │     1      │
└────────┴───────────┴────────────┴──────────────┴─────────────┴────────────┘
```



###  Projection 删除

PROJECTION 也支持删除的 DDL:

```sql
ALTER TABLE hits_100m_obfuscated DROP PROJECTION agg_p2
```

除了通过 ALTER 创建，也能在 CREATE TABLE 的时候创建，例如：

```sql
CREATE TABLE xxx 
( 
    `event_key` String, 
    `user` UInt32, 
    `dim1` String, 
    PROJECTION p1 
    ( 
        SELECT 
            groupBitmap(user), 
            count(1) 
        GROUP BY dim1 
    ) 
) 
ENGINE = MergeTree() 
ORDER BY (event_key, user) 
```



### 匹配规则

- 设置了 SET allow_experimental_projection_optimization = 1

- 返回的数据行小于基表总数

- 查询覆盖的分区 part 超过一半

- Where 必须是 PROJECTION 定义中 GROUP BY 的子集

- GROUP BY 必须是 PROJECTION 定义中 GROUP BY 的子集

- SELECT 必须是 PROJECTION 定义中 SELECT 的子集

- 匹配多个 PROJECTION 的时候，选取读取 part 最少的



如果你不知道查询是否匹配了 PROJECTION ，有两种方式可以校验：

```sql
EXPLAIN
SELECT WatchID
FROM hits_100m_obfuscated
WHERE WatchID = 5814563137538961516

Query id: bf008e69-fd68-4928-83f6-a57a2d84e286

┌─explain───────────────────────────────────────────────────────────────────┐
│ Expression ((Projection + Before ORDER BY))                               │
│   SettingQuotaAndLimits (Set limits and quota after reading from storage) │
│     ReadFromStorage (MergeTree(with 0 projection p1))                     │
└───────────────────────────────────────────────────────────────────────────┘
```

MergeTree(with 0 projection p1) 就代表这条 SQL 查询会命中 PROJECTION



利用 PROJECTION ，我们只需面对一张底表查询就行了，既拥有原来物化视图的性能，又免去了维护成本和数据一致性的问题，简直无敌啊。

![img](https://img-note.langyastudio.com/202211031627236.png?x-oss-process=style/watermark)



### 样例

在没有优化的情况下，下面的查询也会全表扫描:

```sql
SELECT
    UserID,
    SearchPhrase,
    count()
FROM hits_100m_obfuscated
GROUP BY
    UserID,
    SearchPhrase
LIMIT 10

Query id: 42c941e0-c15a-4206-9c1b-7350a5a67984

10 rows in set. Elapsed: 2.190 sec. Processed 100.00 million rows, 2.44 GB (45.66 million rows/s., 1.11 GB/s.)
```

创建另外一个聚合 PROJECTION:

```sql
 ALTER TABLE hits_100m_obfuscated ADD PROJECTION agg_p2
    ( 
      SELECT
          UserID, 
          SearchPhrase, 
          count()
        GROUP BY UserID, SearchPhrase
    )
```

由于历史数据已经存在，也要手动触发一下物化：

```sql
alter table hits_100m_obfuscated MATERIALIZE PROJECTION agg_p2
```

物化好了之后，再次执行相同的查询: 数据扫描范围减少了四分之三

```sql
SELECT
    UserID,
    SearchPhrase,
    count()
FROM hits_100m_obfuscated
GROUP BY
    UserID,
    SearchPhrase
LIMIT 10

Query id: 258e556e-ea5b-43f0-980a-997c02abc233

10 rows in set. Elapsed: 1.847 sec. Processed 24.07 million rows, 1.58 GB (13.04 million rows/s., 856.09 MB/s.)
```



参考文档：

https://cloud.tencent.com/developer/article/1878609



## 分区

分区是在一个表中通过指定的规则划分而成的**逻辑数据集**。可以按任意标准进行分区，如**按月，按日或按事件类型**。为了减少需要操作的数据，每个分区都是分开存储的。访问数据时，ClickHouse 尽量使用这些分区的最小子集。允许查询在指定了分区键的条件下，尽可能的**少读取数据**。

![img](https://img-note.langyastudio.com/202211081531519.webp?x-oss-process=style/watermark)

### 如何使用

分区是在 [建表](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#table_engine-mergetree-creating-a-table) 时通过 `PARTITION BY expr` 子句指定的。分区键可以是表中列的任意表达式。例如，指定按月分区，表达式为 `toYYYYMM(date_column)`：

```sql
CREATE TABLE visits
(
    VisitDate Date,
    Hour UInt8,
    ClientID UUID
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(VisitDate)
ORDER BY Hour;
```

分区键也可以是表达式元组（类似 [主键](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#primary-keys-and-indexes-in-queries) ）。例如：设置按一周内的事件类型分区。

```sql
ENGINE = ReplicatedCollapsingMergeTree('/clickhouse/tables/name', 'replica1', Sign)
PARTITION BY (toMonday(StartDate), EventType)
ORDER BY (CounterID, StartDate, intHash32(UserID));
```



ClickHouse 会定期的对插入的数据片段进行合并，大约是在插入后 15 分钟左右。此外，你也可以使用 [OPTIMIZE](https://clickhouse.com/docs/zh/sql-reference/statements/misc#misc_operations-optimize) 语句发起一个计划外的合并。例如：

```sql
#OPTIMIZE TABLE visits PARTITION 存储分区的名称;
OPTIMIZE TABLE visits PARTITION 201902;
```



### 查看分区

可以通过 [system.parts](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/custom-partitioning-key#system_tables-parts) 表查看表**片段和分区**信息。例如，假设我们有一个 `visits` 表，按月分区。对 `system.parts` 表执行 `SELECT`：

```sql
SELECT
    partition,
    name,
    active
FROM system.parts
WHERE table = 'visits'
```

```
┌─partition─┬─name───────────┬─active─┐
│ 201901    │ 201901_1_3_1   │      0 │
│ 201901    │ 201901_1_9_2   │      1 │
│ 201901    │ 201901_8_8_0   │      0 │
│ 201901    │ 201901_9_9_0   │      0 │
│ 201902    │ 201902_4_6_1   │      1 │
│ 201902    │ 201902_10_10_0 │      1 │
│ 201902    │ 201902_11_11_0 │      1 │
└───────────┴────────────────┴────────┘
```

`partition` 列存储分区的名称。此示例中有两个分区：`201901` 和 `201902`。在 [ALTER … PARTITION](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/custom-partitioning-key#alter_manipulations-with-partitions) 语句中你可以使用该列值来指定分区名称。

`name` 列为分区中数据片段的名称。在 [ALTER ATTACH PART](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/custom-partitioning-key#alter_attach-partition) 语句中你可以使用此列值中来指定片段名称。

这里我们拆解下第一个数据片段的名称：`201901_1_3_1`：

- `201901` 是分区名称
- `1` 是数据块的最小编号
- `3` 是数据块的最大编号
- `1` 是块级别（即在由块组成的合并树中，该块在树中的深度）



### 查询流程

![img](https://img-note.langyastudio.com/202211081539250.png?x-oss-process=style/watermark)



### 最佳实践

- 那些有相同分区表达式值的数据片段才会合并。**不应该用太精细的分区方案**（超过 **1000** 分区）。否则会因为文件系统中的文件数量过多和需要打开的文件描述符过多，导致 `SELECT` 查询效率不佳。

- 分区粒度根据业务特点决定，不宜过粗或过细。一般选择按天分区，也可以指定为 Tuple()，以单表一亿数据为例，分区大小控制在 10-30 个为最佳
- 一般都是使用的是日期作为分区键，同一分区内有序，不同分区不能保证有序



## [跳数索引](https://clickhouse.com/docs/zh/guides/improving-query-performance/skipping-indexes)

在传统的关系数据库中，将一个或多个“二级”索引附加到表上。这是一个 b- 树结构，允许数据库在 O(log(n)) 时间内找到磁盘上所有匹配的行，而不是 O(n) 时间(一次表扫描)，其中 n 是行数。但是，这种类型的二级索引不适用于 ClickHouse (或其他面向列的数据库)，因为磁盘上**没有单独的行可以添加到索引**中。

ClickHouse 提供了一种不同类型的索引，在特定情况下可以显著提高查询速度。这些结构被标记为跳数索引，因为它们使 ClickHouse 能够**跳过保证没有匹配值的数据块**。



### 基本操作

用户只能在 **MergeTree** 表引擎上使用数据跳数索引。每个跳数索引都有四个主要参数：

- 索引名称。索引名用于在每个分区中创建索引文件。此外，在删除或具体化索引时需要将其作为参数。
- 索引的表达式。索引表达式用于计算存储在索引中的值集。它可以是列、简单操作符、函数的子集的组合。
- 类型。索引的类型控制计算，该计算决定是否可以跳过读取和计算每个索引块。
- GRANULARITY。**每个索引块由颗粒（granule）组成**。例如如果主表索引粒度为 8192 行，GRANULARITY 为 4，则每个索引“块”将为 32768 行。

当用户创建数据跳数索引时，表的每个数据部分目录中将有两个额外的文件。

- skp*idx*{index_name}.idx：包含排序的表达式值
- skp*idx*{index_name}.mrk2：包含关联数据列文件中的相应偏移量



如果在执行查询并读取相关列文件时，WHERE 子句过滤条件的某些部分与跳数索引表达式匹配，ClickHouse 将使用索引文件数据来确定每个相关的数据块是必须被处理还是可以被绕过(假设块还没有通过应用主键索引被排除)。这里用一个非常简单的示例：考虑以下加载了可预测数据的表。

```sql
CREATE TABLE skip_table
(
  my_key UInt64,
  my_value UInt64
)
ENGINE MergeTree primary key my_key
SETTINGS index_granularity=8192;

INSERT INTO skip_table SELECT number, intDiv(number,4096) FROM numbers(100000000);
```

当执行一个不使用主键的简单查询时，将扫描 my_value 列所有的一亿条记录：

```sql
SELECT * FROM skip_table WHERE my_value IN (125, 700)

┌─my_key─┬─my_value─┐
│ 512000 │      125 │
│ 512001 │      125 │
│    ... |      ... |
└────────┴──────────┘

8192 rows in set. Elapsed: 0.079 sec. Processed 100.00 million rows, 800.10 MB (1.26 billion rows/s., 10.10 GB/s.
```

增加一个基本的跳数索引：

```sql
ALTER TABLE skip_table ADD INDEX vix my_value TYPE set(100) GRANULARITY 2;
```



通常，跳数索引只应用于新插入的数据，所以仅仅添加索引不会影响上述查询。

要使已经存在的数据生效，那执行：

```sql
ALTER TABLE skip_table MATERIALIZE INDEX vix;
```

重跑SQL：

```sql
SELECT * FROM skip_table WHERE my_value IN (125, 700)

┌─my_key─┬─my_value─┐
│ 512000 │      125 │
│ 512001 │      125 │
│    ... |      ... |
└────────┴──────────┘

8192 rows in set. Elapsed: 0.051 sec. Processed 32.77 thousand rows, 360.45 KB (643.75 thousand rows/s., 7.08 MB/s.)
```



这次没有再去处理 1 亿行 800MB 的数据，ClickHouse 只读取和分析 32768 行 360KB 的数据 — 4 个 granule 的数据。

下图是更直观的展示，这就是如何读取和选择 my_value 为 125 的 4096 行，以及如何跳过以下行而不从磁盘读取:

![simple_skip-52c0988d126255fdb7aa8ad94036b1f7](https://img-note.langyastudio.com/202211021526259.svg)



### 跳数索引类型

> 当字段位于主键时，则不能提高 > = < 等**比较符**的查询性能，但 like 等有效

#### minmax

这种轻量级索引类型不需要参数。它存储**每个块的索引表达式的最小值和最大值**(如果表达式是一个元组，它分别存储元组元素的每个成员的值)。对于倾向于按值松散排序的列，这种类型非常理想。在查询处理期间，这种索引类型的开销通常是最小的。

这种类型的索引只适用于标量或元组表达式——索引永远不适用于返回数组或 map 数据类型的表达式。



#### set

这种轻量级索引类型**接受单个参数 max_size**，即**每个块的值集**(0 允许无限数量的离散值)。这个集合包含块中的所有值(如果值的数量超过 max_size 则为空)。这种索引类型适用于每组颗粒中基数较低(本质上是“聚集在一起”)但总体基数较高的列。

该索引的成本、性能和有效性取决于块中的基数。如果每个块包含大量惟一值，那么针对大型索引集计算查询条件将非常昂贵，或者由于索引超过 max_size 而为空，因此索引将不应用。



#### Bloom Filter Types

Bloom filter 是一种数据结构，它允许对集合成员进行高效的是否存在测试，但代价是有轻微的误报。在跳数索引的使用场景，假阳性不是一个大问题，因为惟一的问题只是读取一些不必要的块。潜在的假阳性意味着索引表达式应该为真，否则有效的数据可能会被跳过。

因为 Bloom filter 可以**更有效地处理大量离散值的测试**，所以它们可以适用于大量条件表达式判断的场景。特别的是Bloom filter 索引可以应用于数组，数组中的每个值都被测试，也可以应用于 map，通过使用 mapKeys 或 mapValues 函数将键或值转换为数组。

有三种基于 Bloom 过滤器的数据跳数索引类型：

- 基本的 **bloom_filter** 接受一个可选参数，该参数表示在 0 到 1 之间允许的“假阳性”率(如果未指定，则使用 .025 )。
- 更专业的 **tokenbf_v1**。需要三个参数，用来优化布隆过滤器：（1）过滤器的大小字节(大过滤器有更少的假阳性，有更高的存储成本)，（2）哈希函数的个数(更多的散列函数可以减少假阳性)。（3）布隆过滤器哈希函数的种子。有关这些参数如何影响布隆过滤器功能的更多细节，请参阅 [这里](https://hur.st/bloomfilter/) 。此索引仅适用于 String、FixedString 和 Map 类型的数据。输入表达式被分割为由非字母数字字符分隔的字符序列。例如，列值 `This is a candidate for a "full text" search` 将被分割为 `This` `is` `a` `candidate` `for` `full` `text` `search`。它用于 LIKE、EQUALS、in、hasToken() 和类似的长字符串中单词和其他值的搜索。例如，一种可能的用途是在非结构的应用程序日志行列中搜索少量的类名或行号。
- 更专业的 **ngrambf_v1**。该索引的功能与 tokenbf_v1 相同。在 Bloom filter 设置之前需要一个额外的参数，即要索引的 ngram 的大小。一个 ngram 是长度为 n 的任何字符串，比如如果 n 是 4，`A short string `会被分割为 `A sh`` sho`, `shor`, `hort`, `ort s`, `or st`, `r str`, `stri`, `trin`, `ring`。这个索引对于文本搜索也很有用，特别是没有单词间断的语言，比如中文。



### 跳数索引函数

跳数索引核心目的是限制流行查询分析的数据量。鉴于 ClickHouse 数据的分析特性，这些查询的模式在大多数情况下都包含函数表达式。因此，跳数索引必须与常用函数正确交互才能提高效率。这种情况可能发生在:

- 插入数据并将索引定义为一个函数表达式(表达式的结果存储在索引文件中)或者
- 处理查询，并将表达式应用于存储的索引值，以确定是否排除数据块。

每种类型的跳数索引支持的函数列表可以查看 [这里](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/mergetree/#functions-support) 。通常，集合索引和基于 Bloom filter 的索引(另一种类型的集合索引)都是无序的，因此不能用于范围。相反，最大最小值索引在范围中工作得特别好，因为确定范围是否相交非常快。部分匹配函数 LIKE、startsWith、endsWith 和 hasToken 的有效性取决于使用的索引类型、索引表达式和数据的特定形状。

![image-20221103172845790](https://img-note.langyastudio.com/202211031728893.png?x-oss-process=style/watermark)



### 跳数索引的配置

有两个可用的设置可应用于跳数索引。

- **use_skip_indexes** (0或1，默认为1)。不是所有查询都可以有效地使用跳过索引。如果一个特定的过滤条件可能包含很多颗粒，那么应用数据跳过索引将导致不必要的、有时甚至是非常大的成本。对于不太可能从任何跳过索引中获益的查询，将该值设置为 0。
- **force_data_skipping_indexes** (以逗号分隔的索引名列表)。此设置可用于防止某些类型的低效查询。在某些情况下，除非使用跳过索引，否则查询表的开销太大，如果将此设置与一个或多个索引名一起使用，则对于任何没有使用所列索引的查询将返回一个异常。这将防止编写糟糕的查询消耗服务器资源。



### 最佳实践

跳数索引并不直观，特别是对于来自 RDMS 领域并且习惯二级行索引或来自文档存储的反向索引的用户来说。要获得任何优化，应用 ClickHouse 数据**跳数索引必须避免足够多的颗粒读取**，以抵消计算索引的成本。关键是，如果一个值在一个索引块中只出现一次，就意味着整个块必须读入内存并计算，而索引开销是不必要的。

考虑以下数据分布：

![bad_skip_1-43ca5929727f9e32bf386d7687525d1b](https://img-note.langyastudio.com/202211021525855.svg)



假设主键/顺序是时间戳，并且在 visitor_id 上有一个索引。考虑下面的查询:

```sql
SELECT timestamp, url FROM table WHERE visitor_id = 1001
```

对于这种数据分布，传统的二级索引非常有利。不是读取所有的 32678 行来查找具有请求的 visitor_id 的5行，而是二级索引只包含 5 行位置，并且只从磁盘读取这 5 行。ClickHouse 数据跳过索引的情况正好相反。无论跳转索引的类型是什么，visitor_id 列中的所有 32678 值都将被测试。

因此，试图通过简单地向键列添加索引来加速 ClickHouse 查询的冲动通常是不正确的。只有在研究了其他替代方法之后，才应该使用此高级功能，**例如修改主键(查看 [如何选择主键](https://clickhouse.com/docs/zh/guides/improving-query-performance/sparse-primary-indexes))、使用投影或使用实体化视图**。即使跳数索引是合适的，也经常需要对索引和表进行仔细的调优。

在大多数情况下，一个有用的跳数索引需要主键和目标的非主列/表达式之间具有很强的相关性。如果没有相关性(如上图所示)，那么在包含数千个值的块中，至少有一行满足过滤条件的可能性很高，并且只有几个块会被跳过。相反，如果主键的值范围(如一天中的时间)与潜在索引列中的值强相关(如电视观众年龄)，则最小值类型的索引可能是有益的。注意，在插入数据时，可以增加这种相关性，方法是在 sort /ORDER by 键中包含额外的列，或者以在插入时对与主键关联的值进行分组的方式对插入进行批处理。例如，即使主键是一个包含大量站点事件的时间戳，特定 site_id 的所有事件也都可以被分组并由写入进程插入到一起，这将导致许多只包含少量站点 id 的颗粒，因此当根据特定的 site_id 值搜索时，可以跳过许多块。

跳数索引的另一个候选者是高基数表达式，其中任何一个值在数据中都相对稀疏。一个可能的例子是跟踪 API 请求中的错误代码的可观察性平台。某些错误代码虽然在数据中很少出现，但对搜索来说可能特别重要。error_code 列上的 set skip索引将允许绕过绝大多数不包含错误的块，从而显著改善针对错误的查询。

最后，关键的最佳实践是测试、测试、再测试。同样，与用于搜索文档的b-树二级索引或倒排索引不同，**跳数索引行为是不容易预测的**。将它们添加到表中会在数据摄取和查询方面产生很大的成本，这些查询由于各种原因不能从索引中受益。它们应该总是在真实世界的数据类型上进行测试，**测试应该包括类型、粒度大小和其他参数的变化**。测试通常会暴露仅仅通过思考不能发现的陷阱。



## [主键索引最佳实践](https://clickhouse.com/docs/zh/guides/improving-query-performance/sparse-primary-indexes)



## 物化视图

异步物化视图，在原有的分布式表和本地表中把需要去重的字段类型更改为 AggregateFunction(uniq, String)，uniq 还可以设置为 uniqCombined, uniqCombined64, uniqHLL12, uniqExact。根据业务需要选择不同的精度，聚合数据后 insert的时候 uniqState(A)，查询时使用 uniqMerge(A)，从明细到聚合物化视图，大约可以减少 70% 的数据量，提高查询性能，进一步提高聚合程度的话，可以设置单指标表，查询时 join。



## [使用建议](https://clickhouse.com/docs/zh/operations/tips)

### CPU 频率调节器

始终使用 `performance` 频率调节器。 `on-demand` 频率调节器在持续高需求的情况下，效果更差

```bash
echo 'performance' | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```



### CPU 限制

处理器可能会过热。 使用 `dmesg` 查看 CPU 的时钟速率是否由于过热而受到限制。 该限制也可以在数据中心级别外部设置。 您可以使用  `turbostat`  在负载下对其进行监控



### RAM

对于少量数据（压缩后约 200 GB），最好使用与数据量一样多的内存。 对于大量数据，以及在处理交互式（在线）查询时，应使用合理数量的 RAM（128 GB或更多），以便热数据子集适合页面缓存。 即使对于每台服务器约 50TB 的数据量，与 64GB 相比，使用 128GB 的RAM也可以显着提高查询性能。

不要禁用 overcommit。`cat /proc/sys/vm/overcommit_memory` 的值应该为 0 或 1。运行

```bash
$ echo 0 | sudo tee /proc/sys/vm/overcommit_memory
```



### 大页

始终禁用透明大页(transparent huge pages)。 它会干扰内存分配器，从而导致显着的性能下降

```bash
echo 'never' | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
```

使用 `perf top` 来查看内核在内存管理上花费的时间。 永久大页(permanent huge pages)也不需要被分配。



### 存储子系统

如果预算允许使用 SSD，请使用 SSD。 如果没有，请使用硬盘。 SATA 硬盘 7200 转就行了。

优先选择许多带有本地硬盘驱动器的服务器，而不是少量带有附加磁盘架的服务器。 但是对于存储极少查询的档案，架子可以使用。



### RAID

当使用硬盘，可以结合他们的 RAID-10，RAID-5，RAID-6 或 RAID-50。 对于 Linux，软件 RAID 更好（使用  `mdadm` ). 不建议使用 LVM。 当创建 RAID-10，选择 `far` 布局。 如果预算允许，请选择 RAID-10。

如果有 4 个以上的磁盘，请使用 RAID-6（首选）或 RAID-50，而不是 RAID-5。 当使用 RAID-5、RAID-6 或 RAID-50 时，始终增加 stripe_cache_size，因为默认值通常不是最佳选择。

```bash
echo 4096 | sudo tee /sys/block/md2/md/stripe_cache_size
```



使用以下公式从设备数量和块大小中计算出确切的数量: `2 * num_devices * chunk_size_in_bytes / 4096`。

1024KB 的块大小足以满足所有 RAID 配置。 切勿将块大小设置得太小或太大。

可以在 SSD 上使用 RAID-0。 无论使用哪种 RAID，始终使用复制来保证数据安全。

启用有长队列的 NCQ。 对于 HDD，选择 CFQ 调度程序，对于 SSD，选择 noop。 不要减少 ‘readahead’ 设置。 对于HDD，启用写入缓存。



### 文件系统

Ext4 是最可靠的选择。 设置挂载选项 `noatime`. XFS 也是合适的，但它还没有经过 ClickHouse 的全面测试。 大多数其他文件系统也应该可以正常工作。 具有延迟分配的文件系统工作得更好。



### Linux内核

不要使用过时的 Linux 内核。



### 网络

如果使用的是 IPv6，请增加路由缓存的大小。 3.2 之前的 Linux 内核在 IPv6 实现方面存在许多问题。

如果可能的话，至少使用 10GB 的网络。1GB 也可以工作，但对于使用数十 TB 的数据修补副本或处理具有大量中间数据的分布式查询，情况会更糟。



### 虚拟机监视器(Hypervisor)配置

如果使用的是 OpenStack，请在 nova.conf 中设置

```text
cpu_mode=host-passthrough
```



如果使用的是 libvirt，请在 XML 配置中设置

```text
<cpu mode='host-passthrough'/>
```

这对于 ClickHouse 能够通过 `cpuid` 指令获取正确的信息非常重要。 否则，当在旧的 CPU 型号上运行虚拟机监视器时，可能会导致 `Illegal instruction` 崩溃。
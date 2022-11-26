## SQL 语句

语句表示可以使用 SQL 查询执行的各种操作：

- [SELECT](https://clickhouse.com/docs/zh/sql-reference/statements/select/)
- [INSERT INTO](https://clickhouse.com/docs/zh/sql-reference/statements/insert-into)
- [CREATE](https://clickhouse.com/docs/zh/sql-reference/statements/create/)
- [ALTER](https://clickhouse.com/docs/zh/sql-reference/statements/alter/)
- [SYSTEM](https://clickhouse.com/docs/zh/sql-reference/statements/system)
- [SHOW](https://clickhouse.com/docs/zh/sql-reference/statements/show)
- [GRANT](https://clickhouse.com/docs/zh/sql-reference/statements/grant)
- [REVOKE](https://clickhouse.com/docs/zh/sql-reference/statements/revoke)
- [ATTACH](https://clickhouse.com/docs/zh/sql-reference/statements/attach)
- [CHECK TABLE](https://clickhouse.com/docs/zh/sql-reference/statements/check-table)
- [DESCRIBE TABLE](https://clickhouse.com/docs/zh/sql-reference/statements/describe-table)
- [DETACH](https://clickhouse.com/docs/zh/sql-reference/statements/detach)
- [DROP](https://clickhouse.com/docs/zh/sql-reference/statements/drop)
- [EXISTS](https://clickhouse.com/docs/zh/sql-reference/statements/exists)
- [KILL](https://clickhouse.com/docs/zh/sql-reference/statements/kill)
- [OPTIMIZE](https://clickhouse.com/docs/zh/sql-reference/statements/optimize)
- [RENAME](https://clickhouse.com/docs/zh/sql-reference/statements/rename)
- [SET](https://clickhouse.com/docs/zh/sql-reference/statements/set)
- [SET ROLE](https://clickhouse.com/docs/zh/sql-reference/statements/set-role)
- [TRUNCATE](https://clickhouse.com/docs/zh/sql-reference/statements/truncate)
- [USE](https://clickhouse.com/docs/zh/sql-reference/statements/use)
- [EXPLAIN](https://clickhouse.com/docs/zh/sql-reference/statements/explain)



## [SQL 语法](https://clickhouse.com/docs/zh/sql-reference/syntax)

ClickHouse有2类解析器：完整SQL解析器（递归式解析器），以及数据格式解析器（快速流式解析器） 除了 `INSERT` 查询，其它情况下仅使用完整SQL解析器。 `INSERT`查询会同时使用2种解析器：

```sql
INSERT INTO t VALUES (1, 'Hello, world'), (2, 'abc'), (3, 'def')
```

含 `INSERT INTO t VALUES`  的部分由完整SQL解析器处理，包含数据的部分 `(1, 'Hello, world'), (2, 'abc'), (3, 'def')` 交给快速流式解析器解析。通过设置参数 [input_format_values_interpret_expressions](https://clickhouse.com/docs/zh/operations/settings/settings#settings-input_format_values_interpret_expressions)，你也可以对数据部分开启完整SQL解析器。当 `input_format_values_interpret_expressions = 1` 时，ClickHouse优先采用快速流式解析器来解析数据。如果失败，ClickHouse再尝试用完整SQL解析器来处理，就像处理SQL [expression](https://clickhouse.com/docs/zh/sql-reference/syntax/#syntax-expressions) 一样。

数据可以采用任何格式。当 CH 接收到请求时，服务端先在内存中计算不超过 [max_query_size](https://clickhouse.com/docs/zh/operations/settings/settings#settings-max_query_size) 字节的请求数据（默认1 mb），然后剩下部分交给快速流式解析器。

当 `INSERT` 语句中使用 `Values` 格式时，看起来数据部分的解析和解析 `SELECT` 中的表达式相同，但并不是这样的。 `Values` 格式有非常多的限制。

本文的剩余部分涵盖了完整SQL解析器。关于格式解析的更多信息，参见 [Formats](https://clickhouse.com/docs/zh/interfaces/formats) 章节。



### [关键字](https://clickhouse.com/docs/zh/sql-reference/syntax/#syntax-keywords)

以下场景的关键字是大小写不敏感的：

- 标准 SQL。例如，`SELECT`, `select` 和 `SeLeCt` 都是允许的
- 在某些流行的 RDBMS 中被实现的关键字，例如，`DateTime` 和 `datetime `是一样的

你可以在系统表 [system.data_type_families](https://clickhouse.com/docs/zh/operations/system-tables/data_type_families#system_tables-data_type_families) 中检查某个数据类型的名称是否是大小写敏感型。

和标准SQL相反，所有其它的关键字都是 **大小写敏感的**，包括函数名称。

关键字不是保留的；它们仅在相应的上下文中才会被认为是关键字。如果你使用和关键字同名的 [标识符](https://clickhouse.com/docs/zh/sql-reference/syntax/#syntax-identifiers) ，需要使用双引号或反引号将它们包含起来。例如：如果表 `table_name` 包含列 `"FROM"`，那么 `SELECT "FROM" FROM table_name` 是合法的



### 字符

- 数字

数字类型字符会被做如下解析：

- 首先，当做64位的有符号整数，使用函数 [strtoull](https://en.cppreference.com/w/cpp/string/byte/strtoul)
- 如果失败，解析成64位无符号整数，同样使用函数 [strtoull](https://en.cppreference.com/w/cpp/string/byte/strtoul)
- 如果还失败了，试图解析成浮点型数值，使用函数 [strtod](https://en.cppreference.com/w/cpp/string/byte/strtof)
- 最后，以上情形都不符合时，返回异常

数字类型的值类型为能容纳该值的最小数据类型。 例如：1 解析成 `UInt8`型，256 则解析成 `UInt16`。更多信息，参见 [数据类型](https://clickhouse.com/docs/zh/sql-reference/data-types/)

例如: `1`, `18446744073709551615`, `0xDEADBEEF`, `01`, `0.1`, `1e100`, `-1e-100`, `inf`, `nan`.



- 字符串

ClickHouse 只支持用**单引号包含的字符串**。特殊字符可通过反斜杠进行转义。下列转义字符都有相应的实际值： `\b`, `\f`, `\r`, `\n`, `\t`, `\0`, `\a`, `\v`, `\xHH`。其它情况下，以 `\c`形式出现的转义字符，当`c`表示任意字符时，转义字符会转换成`c`。这意味着你可以使用 `\'`和`\\`。该值将拥有[String](https://clickhouse.com/docs/zh/sql-reference/data-types/string)类型。

在字符串中，你至少需要对 `'` 和 `\` 进行转义。单引号可以使用单引号转义，例如 `'It\'s'` 和 `'It''s'` 是相同的。



- NULL 值

代表不存在的值

为了能在表字段中存储NULL值，该字段必须声明为 [空值](https://clickhouse.com/docs/zh/sql-reference/data-types/nullable) 类型。 根据数据的格式（输入或输出），NULL值有不同的表现形式。更多信息参见文档 [数据格式](https://clickhouse.com/docs/zh/interfaces/formats#formats)

在处理 `NULL`时存在很多细微差别。例如，比较运算的至少一个参数为 `NULL` ，则该结果也是 `NULL` 。与之类似的还有乘法运算, 加法运算,以及其它运算。更多信息，请参阅每种运算的文档部分。

在语句中，可以通过 [IS NULL](https://clickhouse.com/docs/zh/sql-reference/operators/#operator-is-null) 以及 [IS NOT NULL](https://clickhouse.com/docs/zh/sql-reference/operators/) 运算符，以及 `isNull` 、 `isNotNull` 函数来检查 `NULL` 值



### [表达式别名](https://clickhouse.com/docs/zh/sql-reference/syntax/#syntax-expression_aliases)

别名是用户对表达式的自定义名称

```sql
expr AS alias
```

- `AS` — 用于定义别名的关键字。可以对表或 select 语句中的列定义别名(`AS` 可以省略） 例如, `SELECT table_name_alias.column_name FROM table_name table_name_alias`.

  在 [CAST函数](https://clickhouse.com/docs/zh/sql-reference/functions/type-conversion-functions#type_conversion_function-cast) 中，`AS` 有其它含义。请参见该函数的说明部分。

- `expr` — 任意 CH 支持的表达式

  例如, `SELECT column_name * 2 AS double FROM some_table`.

- `alias` — `expr` 的名称。别名必须符合 [标识符](https://clickhouse.com/docs/zh/sql-reference/syntax/#syntax-identifiers) 语法.

  例如, `SELECT "table t".column_name FROM table_name AS "table t"`.



别名在当前查询或子查询中是全局可见的，你可以在查询语句的任何位置对表达式定义别名

- 别名在当前查询的子查询及不同子查询中是不可见的。例如，执行如下查询SQL: `SELECT (SELECT sum(b.a) + num FROM b) - a.a AS num FROM a` ,ClickHouse会提示异常 `Unknown identifier: num`.

- 如果给select子查询语句的结果列定义其别名，那么在外层可以使用该别名。例如, `SELECT n + m FROM (SELECT 1 AS n, 2 AS m)`.



## Distributed DDL

默认情况下，`CREATE`、`DROP`、`ALTER` 和 `RENAME` 查询仅影响执行它们的当前服务器。 在集群设置中，可以使用 `ON CLUSTER` 子句以**分布式方式运行此类查询**。

**分布式引擎本身不存储数据**, 但可以在多个服务器上进行分布式查询，读是自动并行的。读取时，远程服务器表的索引（如果有的话）会被使用。

为了正确运行这些查询，**每个主机必须具有相同的集群定义**（为了简化同步配置，您可以使用 ZooKeeper 替换）。 他们还必须连接到 ZooKeeper 服务器。

当 `Distributed` 表指向当前服务器上的一个表时，你可以采用以下语句:

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster] AS [db2.]name2 ENGINE = Distributed(cluster, database, table[, sharding_key[, policy_name]]) [SETTINGS name=value, ...]
```



### sharding_key 分片key

> 相同数据的分配到相同的分片，另一方面是为了防止表引擎自动删除同主键数据失败等系列问题

分片表达式可以是由常量和表列组成的任何返回整数表达式。例如，可以使用表达式 `rand()` 来随机分配数据，或者使用 `UserID` 来按用户 ID 的余数分布（**相同用户的数据将分配到单个分片上**，这可降低带有用户信息的 IN 和 JOIN 的语句运行的复杂度）。

如果该列数据分布不够均匀，可以将其包装在散列函数中：`intHash64(UserID)`。

如果是字符串类型的字段，想让相同的数据分配到单个分片上，可以使用 `xxHash64`，如下图所示：

```sql
ENGINE = Distributed('sht_ck_cluster_1',
 'ZZCXDB',
 'LSHSPZ2022_DATA',
 xxHash64(F_UNITID));
```



**分片类型：**

- random 随机分片：写入数据会被随机分发到分布式集群中的某个节点上
- constant 固定分片：写入数据会被分发到固定一个节点上
- column value 分片：按照某一列的值进行 hash 分片
- 自定义表达式分片：指定任意合法表达式，根据表达式被计算后的值进行 hash 分片



## [函数](https://clickhouse.com/docs/zh/sql-reference/functions/)



## 聚合函数

### 聚合函数列表

标准聚合函数:

- [count](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/count)
- [min](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/min)
- [max](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/max)
- [sum](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/sum)
- [avg](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/avg)
- [any](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/any)
- [stddevPop](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/stddevpop)
- [stddevSamp](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/stddevsamp)
- [varPop](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/varpop)
- [varSamp](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/varsamp)
- [covarPop](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/covarpop)
- [covarSamp](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/covarsamp)

ClickHouse 特有的聚合函数:

- [anyHeavy](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/anyheavy)
- [anyLast](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/anylast)
- [argMin](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/argmin)
- [argMax](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/argmax)
- [avgWeighted](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/avgweighted)
- [topK](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/topk)
- [topKWeighted](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/topkweighted)
- [groupArray](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/grouparray)
- [groupUniqArray](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/groupuniqarray)
- [groupArrayInsertAt](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/grouparrayinsertat)
- [groupArrayMovingAvg](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/grouparraymovingavg)
- [groupArrayMovingSum](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/grouparraymovingsum)
- [groupBitAnd](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/groupbitand)
- [groupBitOr](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/groupbitor)
- [groupBitXor](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/groupbitxor)
- [groupBitmap](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/groupbitmap)
- [groupBitmapAnd](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/groupbitmapand)
- [groupBitmapOr](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/groupbitmapor)
- [groupBitmapXor](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/groupbitmapxor)
- [sumWithOverflow](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/sumwithoverflow)
- [sumMap](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/summap)
- [minMap](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/minmap)
- [maxMap](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/maxmap)
- [skewSamp](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/skewsamp)
- [skewPop](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/skewpop)
- [kurtSamp](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/kurtsamp)
- [kurtPop](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/kurtpop)
- [uniq](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/uniq)
- [uniqExact](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/uniqexact)
- [uniqCombined](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/uniqcombined)
- [uniqCombined64](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/uniqcombined64)
- [uniqHLL12](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/uniqhll12)
- [quantile](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/quantile)
- [quantiles](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/quantiles)
- [quantileExact](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/quantileexact)
- [quantileExactLow](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/quantileexact#quantileexactlow)
- [quantileExactHigh](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/quantileexact#quantileexacthigh)
- [quantileExactWeighted](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/quantileexactweighted)
- [quantileTiming](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/quantiletiming)
- [quantileTimingWeighted](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/quantiletimingweighted)
- [quantileDeterministic](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/quantiledeterministic)
- [quantileTDigest](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/quantiletdigest)
- [quantileTDigestWeighted](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/quantiletdigestweighted)
- [simpleLinearRegression](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/simplelinearregression)
- [stochasticLinearRegression](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/stochasticlinearregression)
- [stochasticLogisticRegression](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/stochasticlogisticregression)
- [categoricalInformationValue](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/reference/categoricalinformationvalue)



### [聚合函数组合器](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/combinators)



### [参数聚合函数](https://clickhouse.com/docs/zh/sql-reference/aggregate-functions/parametric-functions)



## [表函数](https://clickhouse.com/docs/zh/sql-reference/table-functions/)



## [字典](https://clickhouse.com/docs/zh/sql-reference/dictionaries/)



## [数据类型](https://clickhouse.com/docs/zh/sql-reference/data-types/)



## ANSI 兼容性

下表列出了ClickHouse能够使用，但与ANSI SQL规定有差异的查询特性。

| 功能ID  | 功能名称              | 差异                                                         |
| ------- | --------------------- | ------------------------------------------------------------ |
| E011    | 数值型数据类型        | 带小数点的数字被视为近似值 (`Float64`）而不是精确值 (`Decimal`) |
| E051-05 | SELECT 的列可以重命名 | 字段重命名的作用范围不限于进行重命名的SELECT子查询（参考[表达式别名](https://clickhouse.com/docs/zh/sql-reference/syntax/#notes-on-usage)） |
| E141-01 | NOT NULL（非空）约束  | ClickHouse表中每一列默认为`NOT NULL`                         |
| E011-04 | 算术运算符            | ClickHouse在运算时会进行溢出，而不是四舍五入。此外会根据自定义规则修改结果数据类型（参考[溢出检查](https://clickhouse.com/docs/zh/sql-reference/data-types/decimal/#yi-chu-jian-cha)） |



## [操作符](https://clickhouse.com/docs/zh/sql-reference/operators/)
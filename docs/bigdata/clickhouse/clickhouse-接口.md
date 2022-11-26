ClickHouse 的底层访问接口支持 TCP 和 HTTP 两种协议

## TCP 

**TCP 原生接口的协议拥有更好的性能**，其默认端口为 9000，主要用于集群间的**内部通信及 CLI 客户端**。

> 原生 ClickHouse 协议还没有正式的规范



## HTTP

> INSERT 必须通过 POST 方法来插入数据

HTTP 协议比原生接口受到更多的限制，但拥有更好的兼容性，允许在任何编程语言的任何平台上使用 ClickHouse，可以通过 REST 服务的形式被广泛用于 JAVA、Python 等编程语言的客户端，其默认端口为 8123

- 健康检查

  ```bash
  #返回一个字符串 «Ok.»
  curl 'http://localhost:8123/'
  ```

- 运行状况检查

  ```bash
  #使用 GET /ping 请求
  ```

- 检查复制集的延迟

  ```bash
  #使用 GET /replicas_status 请求
  ```

- Web UI 

  http://localhost:8123/play

  

## 接口

在大多数情况下，建议使用适当的工具或库，而不是直接与它们交互。Yandex 官方支持的项目有:

- [命令行客户端](https://clickhouse.com/docs/zh/interfaces/cli)
- [输入输出格式](https://clickhouse.com/docs/zh/interfaces/formats#formatschema)
  - CSV
  - JSON
  - Avro
  - Parquet
  - XXX
- [MySQL 接口](https://clickhouse.com/docs/zh/interfaces/mysql)
  - 不支持 prepared queries
  - 某些数据类型以字符串形式发送
- [JDBC 驱动](https://clickhouse.com/docs/zh/interfaces/jdbc)
- [ODBC 驱动](https://clickhouse.com/docs/zh/interfaces/odbc)
- [C++ 客户端](https://clickhouse.com/docs/zh/interfaces/cpp)



### JDBC 驱动

https://github.com/ClickHouse/clickhouse-jdbc/tree/master



#### Configuration

**Driver Class**: `com.clickhouse.jdbc.ClickHouseDriver`

> `ru.yandex.clickhouse.ClickHouseDriver` has been deprecated and everything under `ru.yandex.clickhouse` will be removed in 0.3.3



**URL Syntax**: `jdbc:(ch|clickhouse)[:<protocol>]://endpoint1[,endpoint2,...][/<database>][?param1=value1&m2=value2][#tag1,tag2,...]`, for examples:

- `jdbc:ch://localhost` is same as `jdbc:clickhouse:http://localhost:8123`
- `jdbc:ch:https://localhost` is same as `jdbc:clickhouse:http://localhost:8443?ssl=true&sslmode=STRICT`
- `jdbc:ch:grpc://localhost` is same as `jdbc:clickhouse:grpc://localhost:9100`

> **endpoint:**  [protocol://]host[:port][/database][?parameters][#tags]
> **protocol:**  (grpc|grpcs|http|https|tcp|tcps)



**load balancing**

```sql
String connString = "jdbc:ch://server1,server2,server3/database?load_balancing_policy=random&health_check_interval=5000&failover=2";
ClickHouseDataSource ds = new ClickHouseDataSource(connString);
ClickHouseConnection conn = ds.getConnection("default", "");
```



**Connection Properties**:

| Property                 | Default | Description                                                  |
| ------------------------ | ------- | ------------------------------------------------------------ |
| continueBatchOnError     | `false` | Whether to continue batch processing when error occurred     |
| createDatabaseIfNotExist | `false` | Whether to create database if it does not exist              |
| custom_http_headers      |         | comma separated custom http headers, for example: `User-Agent=client1,X-Gateway-Id=123` |
| custom_http_params       |         | comma separated custom http query parameters, for example: `extremes=0,max_result_rows=100` |
| nullAsDefault            | `0`     | `0` - treat null value as is and throw exception when inserting null into non-nullable column; `1` - treat null value as is and disable null-check for inserting; `2` - replace null to default value of corresponding data type for both query and insert |
| jdbcCompliance           | `true`  | Whether to support standard synchronous UPDATE/DELETE and fake transaction |
| typeMappings             |         | Customize mapping between ClickHouse data type and Java class, which will affect result of both [getColumnType()](https://docs.oracle.com/javase/8/docs/api/java/sql/ResultSetMetaData.html#getColumnType-int-) and [getObject(Class)](https://docs.oracle.com/javase/8/docs/api/java/sql/ResultSet.html#getObject-java.lang.String-java.lang.Class-). For example: `UInt128=java.lang.String,UInt256=java.lang.String` |
| wrapperObject            | `false` | Whether [getObject()](https://docs.oracle.com/javase/8/docs/api/java/sql/ResultSet.html#getObject-int-) should return java.sql.Array / java.sql.Struct for Array / Tuple. |



#### [Build with Maven](https://github.com/ClickHouse/clickhouse-jdbc/tree/master#build-with-maven)

Use `mvn -DskipITs clean verify` to compile and generate packages if you're using JDK 8. 



#### Features

|                   |                                                              |           |                                                              |
| ----------------- | ------------------------------------------------------------ | --------- | ------------------------------------------------------------ |
| Category          | Feature                                                      | Supported | Remark                                                       |
| API               | [JDBC](https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/) | ✅         |                                                              |
|                   | [R2DBC](https://r2dbc.io/)                                   | ❌         | will be supported in 0.3.3                                   |
|                   | [GraphQL](https://graphql.org/)                              | ❌         |                                                              |
| Protocol          | [HTTP](https://clickhouse.com/docs/en/interfaces/http/)      | ✅         | recommended, defaults to `java.net.HttpURLConnection` and can be changed to `java.net.http.HttpClient`(less stable) |
|                   | [gRPC](https://clickhouse.com/docs/en/interfaces/grpc/)      | ✅         | still experimental, works with 22.3+, known to has [issue](https://github.com/ClickHouse/ClickHouse/issues/28671#issuecomment-1087049993) when using LZ4 compression |
|                   | [TCP/Native](https://clickhouse.com/docs/en/interfaces/tcp/) | ✅         | `clickhouse-cli-client`(wrapper of ClickHouse native command-line client) was added in 0.3.2-patch10, `clickhouse-tcp-client` will be available in 0.3.3 |
|                   | [Local/File](https://clickhouse.com/docs/en/operations/utilities/clickhouse-local/) | ❌         | `clickhouse-cli-client` will be enhanced to support `clickhouse-local` |
| Compatibility     | Server < 20.7                                                | ❌         | use 0.3.1-patch(or 0.2.6 if you're stuck with JDK 7)         |
|                   | Server >= 20.7                                               | ✅         | use 0.3.2 or above. All [active releases](https://github.com/ClickHouse/ClickHouse/pulls?q=is%3Aopen+is%3Apr+label%3Arelease) are supported. |
| Compression       | [gzip](https://www.gzip.org/)                                | ✅         |                                                              |
|                   | [lz4](https://lz4.github.io/lz4/)                            | ✅         | default                                                      |
|                   | [zstd](https://facebook.github.io/zstd/)                     | ❌         |                                                              |
| Data Format       | RowBinary                                                    | ✅         | `RowBinaryWithNamesAndTypes` for query and `RowBinary` for insertion |
|                   | TabSeparated                                                 | ✅         | Does not support as many data types as RowBinary             |
| Data Type         | AggregatedFunction                                           | ❌         | limited to `groupBitmap`                                     |
|                   | Array(*)                                                     | ✅         |                                                              |
|                   | Bool                                                         | ✅         |                                                              |
|                   | Date*                                                        | ✅         |                                                              |
|                   | DateTime*                                                    | ✅         |                                                              |
|                   | Decimal*                                                     | ✅         | `SET output_format_decimal_trailing_zeros=1` in 21.9+ for consistency |
|                   | Enum*                                                        | ✅         | can be treated as both string and integer                    |
|                   | Geo Types                                                    | ✅         | Point, Ring, Polygon, and MultiPolygon                       |
|                   | Int*, UInt*                                                  | ✅         | UInt64 is mapped to `long`                                   |
|                   | IPv*                                                         | ✅         |                                                              |
|                   | Map(*)                                                       | ✅         |                                                              |
|                   | Nested(*)                                                    | ✅         |                                                              |
|                   | Object('JSON')                                               | ✅         | supported since 0.3.2-patch8                                 |
|                   | SimpleAggregateFunction                                      | ✅         |                                                              |
|                   | *String                                                      | ✅         |                                                              |
|                   | Tuple(*)                                                     | ✅         |                                                              |
|                   | UUID                                                         | ✅         |                                                              |
| High Availability | Load Balancing                                               | ✅         | supported since 0.3.2-patch10                                |
|                   | Failover                                                     | ✅         | supported since 0.3.2-patch10                                |
| Transaction       | Transaction                                                  | ✅         | supported since 0.3.2-patch11, use ClickHouse 22.7+ for native implicit transaction support |
|                   | Savepoint                                                    | ❌         |                                                              |
|                   | XAConnection                                                 | ❌         |                                                              |




> 本文来自[https://www.begtut.com/mysql/mysql-find-duplicate-values.html](https://www.begtut.com/mysql/mysql-find-duplicate-values.html)，郎涯进行简单排版与补充



### 查找重复数据

仅当列的组合重复时，行才被视为重复，因此我们 `AND` 在 `HAVING` 子句中使用了运算符

 ```sql
 SELECT 
     col1, COUNT(col1),
     col2, COUNT(col2),
     ...
  
 FROM
     table_name
 GROUP BY 
     col1, 
     col2, ...
 HAVING 
        (COUNT(col1) > 1) AND 
        (COUNT(col2) > 1) AND 
        ... 
 ```



### 删除重复数据

MySQL 为您提供了 `DELETE JOIN` 可用于快速删除重复行的语句。

以下语句删除重复行并保留最高 ID：

```sql
DELETE t1 FROM contacts_test t1
        INNER JOIN
    contacts_test t2 
WHERE
    t1.id < t2.id AND t1.email = t2.email; 
```



### UUID

在 MySQL 中，UUID 值是一个 128 位数字，表示为五个十六进制数字的 utf8 字符串，格式如下：

```sql
aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee 
```

要生成 UUID 值，请使用以下 `UUID()` 函数：

```sql
UUID() 
```



将 UUID 用作主键具有以下优点：

- 表，数据库甚至服务器之间的 UUID 值是唯一的，它们允许您合并来自不同数据库的行或跨服务器分发数据库
- UUID 值不会公开有关数据的信息，因此在URL中使用它们更安全。例如，如果 ID 为10的客户通过`http://www.example.com/customers/10/`URL 访问其帐户，则很容易猜到有客户11,12等，这可能是攻击的目标
- 可以在避免往返数据库服务器的任何地方生成 UUID 值。它还简化了应用程序中的逻辑。例如，要将数据插入父表和子表，您必须先插入父表，获取生成的 ID，然后将数据插入子表。通过使用 UUID，您可以预先生成父表的主键值，并在事务中同时将行插入父表和子表



除了优点，UUID 值也有一些缺点：

- 存储 UUID 值（16字节）比整数（4字节）或甚至大整数（8字节）占用更多存储空间
- 调试似乎更难，想象一下表达`WHERE id = 'df3b7cb7-6a95-11e7-8846-b05adad3f0ae'` 而不是`WHERE id = 10`
- 使用 UUID 值可能会导致性能问题，因为它们的**大小**和**没有被排序**



### 表复制

将数据从现有表复制到新表的完整命令如下：

```sql
CREATE TABLE IF NOT EXISTS new_table 
SELECT col1, col2, col3 
FROM
    existing_table
WHERE
    conditions; 
```

请注意，上面的语句只是复制表及其数据。它不会复制与表关联的其他数据库对象，如[索引](https://www.begtut.com/mysql/mysql-index/mysql-create-index/)，[主键约束](https://www.begtut.com/mysql/mysql-primary-key.html)，[外键约束](https://www.begtut.com/mysql/mysql-foreign-key.html)， [触发器](https://www.begtut.com/mysql/mysql-triggers.html)等。



要从一个表以及表的所有从属对象复制数据，请使用以下语句：

```sql
CREATE TABLE IF NOT EXISTS new_table LIKE existing_table;
 
INSERT new_table SELECT * FROM existing_table; 
```



### [复制库](https://www.begtut.com/mysql/mysql-copy-database.html)



### 表的存储引擎

```sql
SHOW TABLE STATUS LIKE 'offices'; 
#or
SHOW CREATE TABLE offices; 
```



### 查询第 N 高记录

MySQL 为我们提供了 [LIMIT子句](https://www.begtut.com/mysql/mysql-limit.html)，子句约束返回结果集中的行数。您可以将以上查询重写为以下查询：

```sql
SELECT 
    *
FROM
    table_name
ORDER BY column_name DESC
LIMIT n - 1, 1; 
```

查询返回 n-1 行后的第一行，因此您获得第n 个最高记录。



### 重置自增值

您可以使用`ALTER TABLE`语句重置自动增量值。`ALTER TABLE` 重置自动增量值的语句的语法如下：

```sql
ALTER TABLE table_name AUTO_INCREMENT = value; 
```

[TRUNCATE TABLE](https://www.begtut.com/mysql/mysql-truncate-table.html) 语句从表中删除所有数据并重置自动递增值为零

以下说明了 `TRUNCATE TABLE` 语句的语法：

```sql
TRUNCATE TABLE table_name; 
```



### 注释

MySQL支持三种注释样式：

从 `-- ` 一行到最后一行。双破折号注释样式在第二个破折号后至少需要空格或控制字符（空格，制表符，换行符等）

```sql
SELECT * FROM users; -- This is a comment 
```

请注意，标准 SQL 在第二个破折号后不需要空格。MySQL 使用空格来避免某些 SQL 构造的问题，例如：

```sql
SELECT 10--1; 
```

语句返回 11 .如果 MySQL 没有使用空格，它将返回 10。



从 `#` 一行到最后一行

```sql
SELECT 
    lastName, firstName
FROM
    employees
WHERE
    reportsTo = 1002; # get subordinates of Diane 
```



C 风格的评论 `/**/` 可以跨越多行。您可以使用此注释样式来记录 SQL 代码块

```sql
/*
    Get sales rep employees
    that reports to Anthony
*/ 
SELECT 
    lastName, firstName
FROM
    employees
WHERE
    reportsTo = 1143
        AND jobTitle = 'Sales Rep'; 
```



### 可执行的注释

MySQL 提供可执行注释以支持不同数据库之间的可移植性。这些注释允许您嵌入仅在 MySQL 中执行但不在其他数据库中执行的 SQL 代码。

以下说明了可执行注释语法：

```sql
/*! MySQL-specific code */ 
```

例如，以下语句使用可执行注释：

```sql
SELECT 1 /*! +1 */ 
```

语句返回2而不是1.但是，如果在其他数据库系统中执行它，它将返回1。

如果要从特定版本的 MySQL 执行注释，请使用以下语法：

```sql
/*!##### MySQL-specific code */ 
```

字符串 '#####' 表示可以执行注释的 MySQL 的最低版本。第一个＃是主要版本，例如5或8.第二个2号（##）是次要版本。最后2个是补丁级别。

例如，以下注释仅在 MySQL 5.1.10 或更高版本中可执行：

```sql
CREATE TABLE t1 (
    k INT AUTO_INCREMENT,
    KEY (k)
)  /*!50110 KEY_BLOCK_SIZE=1024; */ 
```



### [EXPLAIN](https://www.begtut.com/mysql/mysql-explain.html#explain_partitions)



### 比较两个表

在数据迁移中，我们经常必须比较两个表以标识一个表中的一条记录，而另一表中没有相应的记录。

例如，我们有一个新数据库，其架构与旧数据库不同。我们的任务是将所有数据从旧数据库迁移到新数据库，并验证数据是否已正确迁移。

要检查数据，我们必须比较两个表，一个在新数据库中，一个在旧数据库中，并识别不匹配的记录。

假设我们有两个表：`t1` 和 `t2`。以下步骤比较两个表并标识不匹配的记录：

首先，使用 UNION 语句合并两个表中的行；仅包括需要比较的列，返回的结果集用于比较。

```sql
SELECT t1.pk, t1.c1
FROM t1
UNION ALL
SELECT t2.pk, t2.c1
FROM t2 
```

其次，根据需要比较的主键和列将记录分组。如果需要比较的列中的值相同，则`COUNT(*)`返回2，否则 `COUNT(*)`返回1。

请参阅以下查询：

```sql
SELECT pk, c1
FROM
 (
   SELECT t1.pk, t1.c1
   FROM t1
   UNION ALL
   SELECT t2.pk, t2.c1
   FROM t2
)  t
GROUP BY pk, c1
HAVING COUNT(*) = 1
ORDER BY pk 
```

如果比较中涉及的列中的值相同，则不返回任何行。



### [数据分层](https://www.begtut.com/mysql/mysql-adjacency-list-tree.html)



### Like 优化

要提高 Mysql 的查询效率最有效的办法是让所有的查询走索引字段，但是在 Mysql 中 Like 关键字只有对前缀查询(`"keyword%"`) 走索引。

```sql
select * from table where name like "keyword%"
```

我们常常需要模糊查询（`"%keyword%"`）或后缀查询(`"%keyword"`)。

解决办法的思路是想办法让模糊查询和后缀查询都能走索引就可以达到目的。

后缀查询解决方案：使用新建字段反转索引然后关键字段反转变成前缀查询：

```sql
select * from table where rename like "drowyey%"
```

目前无法使用模糊查询，如果量大可以全文索引。

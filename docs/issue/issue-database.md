## 索引

### 前缀索引

f_id 为 `varchar` 类型，取其前 8 位作为索引，从而减少索引对磁盘空间的占用

```sql
CREATE INDEX uidx_ums_file_f_id
    ON ums_file
        (
         f_id(8)
            );
```

**注意：**

如果字段值的前缀很多相同，如都是以 `qianzhuixxxx` 开头，那么建立的前缀索引将失效，非但不能提高性能，反而导致性能聚减。所以前缀索引比较类似 `GUID` 这种前缀差异很大的字段值类型。



## 字段

### text 字符长度

以 utf8 编码计算的话

- `LANGTEXT`

    4294967295/3=1431655765个汉字，14亿

    存储空间占用：4294967295/1024/1024/1024=4G的数据；

- `MEDIUMTEXT`

    16777215/3=5592405个汉字，560万

    存储空间占用：16777215/1024/1024=16M的数据；

- `TEXT`

    65535/3=21845个汉字，约20000

    存储空间占用：65535/1024=64K的数据；



## 其他

### 违反检查约束条件

例如执行语句时，报错

```sql
SQL 错误 [2290] [23000]: ORA-02290: 违反检查约束条件 (GL_OSP65.SYS_C00101490)
Error position: line: 3
```

**解决方案：**

首先要确定约束在哪一字段上，使用 sql：，(这里要注意 TABELNAME 必须是大写,表示业务表名称)

```sql
select * from user_constraints where table_name ='TABLENAME'
```

也可以通过客户端查看该表的约束条件

![image-20220617145540064](https://img-note.langyastudio.com/202206171455101.png?x-oss-process=style/watermark)

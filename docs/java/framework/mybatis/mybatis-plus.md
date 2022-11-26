- 只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑

- 只需简单配置，即可快速进行单表 CRUD 操作，从而节省大量时间

- 代码生成、物理分页、性能分析等功能一应俱全



### 依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.2</version>
</dependency>
```



### 配置

```yml
mybatis-plus:
  global-config:
    db-config:
      #表名前缀
      table-prefix: a_
      #逻辑删除
      logic-not-delete-value: null
      logic-delete-value: now()
      logic-delete-field: delete_time
  configuration:
    use-deprecated-executor: false
```



### 使用

https://mybatis.plus/guide

https://mybatis.plus/guide/faq.html



**注解**

@TableId

- 描述：主键注解

| 属性  |  类型  | 必须指定 |   默认值    |    描述    |
| :---: | :----: | :------: | :---------: | :--------: |
| value | String |    否    |     ""      | 主键字段名 |
| type  |  Enum  |    否    | IdType.NONE |  主键类型  |

@IdType

|     值      |                             描述                             |
| :---------: | :----------------------------------------------------------: |
|    AUTO     |                         数据库ID自增                         |
|    NONE     | 无状态,该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT) |
|    INPUT    |                    insert前自行set主键值                     |
|  ASSIGN_ID  | 分配ID(主键类型为Number(Long和Integer)或String)(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextId`(默认实现类为`DefaultIdentifierGenerator`雪花算法) |
| ASSIGN_UUID | 分配UUID,主键类型为String(since 3.3.0),使用接口`IdentifierGenerator`的方法`nextUUID`(默认default方法) |



@TableField

- 描述：字段注解(非主键)

fill - FieldFill

|      值       |         描述         |
| :-----------: | :------------------: |
|    DEFAULT    |      默认不处理      |
|    INSERT     |    插入时填充字段    |
|    UPDATE     |    更新时填充字段    |
| INSERT_UPDATE | 插入和更新时填充字段 |



@TableLogic

- 描述：表字段逻辑处理注解（逻辑删除）

|  属性  |  类型  | 必须指定 | 默认值 |     描述     |
| :----: | :----: | :------: | :----: | :----------: |
| value  | String |    否    |   ""   | 逻辑未删除值 |
| delval | String |    否    |   ""   |  逻辑删除值  |



@Version

- 描述：乐观锁注解、标记 `@Verison` 在字段上
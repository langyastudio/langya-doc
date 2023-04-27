MongoDB 是一个功能丰富的 NoSQL 文档数据库。它既**不关联任何数据记录**，又**无固定的数据模式**。同时，它提供高插入能力、动态性、以及灵活的数据管理功能。



## 入门

### 应用场景

- 游戏场景，使用 MongoDB 存储游戏用户信息，用户的装备、积分等直接以内嵌文档的形式存储，方便查询、更新

- 物流场景，使用 MongoDB 存储**订单信息**，订单状态在运送过程中会不断更新，以 MongoDB 内嵌数组的形式来存储，一次查询就能将订单所有的变更读取出来

- 社交场景，使用 MongoDB 存储存储用户信息，以及用户发表的朋友圈信息，通过**地理位置**索引实现附近的人、地点等功能

- 物联网场景，使用 MongoDB 存储所有接入的智能设备信息，以及设备汇报的**日志信息**，并对这些信息进行多维度的分析

- 视频直播，使用 MongoDB 存储用户信息、礼物信息等



### 安装

#### windows

下载地址：https://www.mongodb.com/download-center/community

下载到本地后，直接安装即可。

再双击 `mongo.exe` 可以运行 MongoDB 自带客户端，操作 MongoDB

![image-20220119133956309](https://img-note.langyastudio.com/202201191339436.png?x-oss-process=style/watermark)



#### Linux

来源：[docker mongodb](https://hub.docker.com/_/mongo)

采用 docker-composer 安装，新建 docker-compose-env.yml 文件如下：

```yml
version: '3.5'

services:
  mongo:
    image: mongo:4.4.11
    container_name: mongo
    # 远程访问
    command: [ "--bind_ip_all" ]
    #授权账号、密码
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: mongo
    restart: always
    networks:
      edu-net:
        ipv4_address: 172.28.0.60
    volumes:
      - /mnt/volume/mongo/data:/data/db
      - /mnt/volume/mongo/log:/var/log/mongodb
    ports:
      - 27017:27017


#docker network ls
#The actual name is volume_edu-net
networks:
  edu-net:
    driver: bridge
    driver_opts:
      com.docker.network.enable_ipv6: "false"
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
```

此时运行 `docker-compose -f docker-compose-env.yml up -d`  完成 mongodb 在 Linux 系统的安装



### 客户端

- Robo 3T（免费，以前叫 Robomongo ）

  下载地址：https://robomongo.org/download

  下载完成后解压，双击 `robo3t.exe` 即可使用

- Navicat 15 (收费)

  用户名/密码为 docker 部署文件设置的 root/mongo（如果安装时没设置，可以不输入）

  ![image-20220119134827936](https://img-note.langyastudio.com/202201191348021.png?x-oss-process=style/watermark)



### 相关概念

> MongoDB 是非关系型数据库当中最像关系型数据库的，所以我们通过它与关系型数据库的对比，来了解下它的概念 

| SQL 概念    | MongoDB 概念 | 解释/说明                               |
| ----------- | ------------ | --------------------------------------- |
| database    | database     | 数据库                                  |
| table       | collection   | 数据库表/集合                           |
| row         | document     | 数据记录行/文档                         |
| column      | field        | 数据字段/域                             |
| index       | index        | 索引                                    |
| primary key | primary key  | 主键，MongoDB 自动将 _id 字段设置为主键 |



### 常见操作

#### 数据库操作

- 创建数据库，使用 `use` 命令去创建数据库，**当插入第一条数据时会创建数据库**，例如创建一个 `test` 数据库

```sql
> use test
switched to db test

> db.article.insert({name:"MongoDB 教程"})
WriteResult({ "nInserted" : 1 })
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
```

- 删除数据库，使用 db 对象中的 `dropDatabase()` 方法来删除

```sql
> db.dropDatabase()
{ "dropped" : "test", "ok" : 1 }
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```



#### 集合操作

- 创建集合，使用 db 对象中的 `createCollection()` 方法来创建集合，例如创建一个 `article` 集合

```sql
> use test
switched to db test
> db.createCollection("article")
{ "ok" : 1 }
> show collections
article
```

- 删除集合，使用 collection 对象的 `drop()` 方法来删除集合，例如删除一个 `article` 集合

```sql
> db.article.drop()
true
> show collections
```



#### 文档操作

> 上面的数据库和集合操作是在 MongoDB 的客户端中进行的，下面的文档操作都是在 Robomongo 中进行的

##### 插入文档

- MongoDB 通过 collection 对象的 `insert()` 方法向集合中插入文档，语法如下

```sql
db.collection.insert(document)
```

- 使用 collection 对象的 `insert()` 方法来插入文档，例如插入一个`article`文档

```sql
db.article.insert({title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: 'Andy',
    url: 'https://www.mongodb.com/',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})
```

- 使用 collection 对象的 `find()` 方法可以获取文档，例如获取所有的 `article` 文档

```sql
db.article.find({})
{
    "_id" : ObjectId("5e9943661379a112845e4056"),
    "title" : "MongoDB 教程",
    "description" : "MongoDB 是一个 Nosql 数据库",
    "by" : "Andy",
    "url" : "https://www.mongodb.com/",
    "tags" : [ 
        "mongodb", 
        "database", 
        "NoSQL"
    ],
    "likes" : 100.0
}
```



##### 更新文档

- MongoDB 通过 collection 对象的 `update()` 来更新集合中的文档，语法如下；

```sql
db.collection.update(
   <query>,
   <update>,
   {
     multi: <boolean>
   }
)
# query：修改的查询条件，类似于SQL中的WHERE部分
# update：更新属性的操作符，类似与SQL中的SET部分
# multi：设置为true时会更新所有符合条件的文档，默认为false只更新找到的第一条
```

- 将 title 为 `MongoDB 教程` 的所有文档的 title 修改为 `MongoDB`

```sql
db.article.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}},{multi:true})
```

- 除了 `update()` 方法以外，`save()` 方法可以用来替换已有文档，语法如下

```sql
db.collection.save(document)
```

- 这次我们将 ObjectId 为 `5e9943661379a112845e4056 `的文档的 title 改为 `MongoDB 教程`

```sql
db.article.save({
    "_id" : ObjectId("5e9943661379a112845e4056"),
    "title" : "MongoDB 教程",
    "description" : "MongoDB 是一个 Nosql 数据库",
    "by" : "Andy",
    "url" : "https://www.mongodb.com/",
    "tags" : [ 
        "mongodb", 
        "database", 
        "NoSQL"
    ],
    "likes" : 100.0
})
```



##### 删除文档

- MongoDB 通过 collection 对象的 `remove()` 方法来删除集合中的文档，语法如下

```sql
db.collection.remove(
   <query>,
   {
     justOne: <boolean>
   }
)
# query：删除的查询条件，类似于SQL中的WHERE部分
# justOne：设置为true只删除一条记录，默认为false删除所有记录
```

- 删除 title 为 `MongoDB 教程` 的所有文档

```sql
db.article.remove({'title':'MongoDB 教程'})
```



##### 查询文档

- MongoDB 通过 collection 对象的 `find()` 方法来查询文档，语法如下

```sql
db.collection.find(query, projection)
# query：查询条件，类似于SQL中的WHERE部分
# projection：可选，使用投影操作符指定返回的键
```

- 查询 `article` 集合中的所有文档

```sql
db.article.find()
/* 1 */
{
    "_id" : ObjectId("5e994dcb1379a112845e4057"),
    "title" : "MongoDB 教程",
    "description" : "MongoDB 是一个 Nosql 数据库",
    "by" : "Andy",
    "url" : "https://www.mongodb.com/",
    "tags" : [ 
        "mongodb", 
        "database", 
        "NoSQL"
    ],
    "likes" : 50.0
}

/* 2 */
{
    "_id" : ObjectId("5e994df51379a112845e4058"),
    "title" : "Elasticsearch 教程",
    "description" : "Elasticsearch 是一个搜索引擎",
    "by" : "Ruby",
    "url" : "https://www.elastic.co/cn/",
    "tags" : [ 
        "elasticearch", 
        "database", 
        "NoSQL"
    ],
    "likes" : 100.0
}

/* 3 */
{
    "_id" : ObjectId("5e994e111379a112845e4059"),
    "title" : "Redis 教程",
    "description" : "Redis 是一个key-value数据库",
    "by" : "Andy",
    "url" : "https://redis.io/",
    "tags" : [ 
        "redis", 
        "database", 
        "NoSQL"
    ],
    "likes" : 150.0
}
```

- MongoDB 中的条件操作符，通过与 SQL 语句的对比来了解下

| 操作       | 格式                     | SQL中的类似语句                |
| ---------- | ------------------------ | ------------------------------ |
| 等于       | `{<key>:<value>}`        | `where title = 'MongoDB 教程'` |
| 小于       | `{<key>:{$lt:<value>}}`  | `where likes < 50`             |
| 小于或等于 | `{<key>:{$lte:<value>}}` | `where likes <= 50`            |
| 大于       | `{<key>:{$gt:<value>}}`  | `where likes > 50`             |
| 大于或等于 | `{<key>:{$gte:<value>}}` | `where likes >= 50`            |
| 不等于     | `{<key>:{$ne:<value>}}`  | `where likes != 50`            |

- 条件查询，查询 title 为 `MongoDB 教程` 的所有文档

```sql
db.article.find({'title':'MongoDB 教程'})
```

- 条件查询，查询 likes 大于 50 的所有文档

```sql
db.article.find({'likes':{$gt:50}})
```

- AND 条件可以通过在 `find()` 方法传入多个键，以逗号隔开来实现，例如查询title为 `MongoDB 教程` 并且 by 为 `Andy` 的所有文档

```sql
db.article.find({'title':'MongoDB 教程','by':'Andy'})
```

- OR 条件可以通过使用 `$or` 操作符实现，例如查询 title 为 `Redis 教程` 或 `MongoDB 教程` 的所有文档

```sql
db.article.find({$or:[{"title":"Redis 教程"},{"title": "MongoDB 教程"}]})
```

- AND 和 OR 条件的联合使用，例如查询 likes 大于50，并且 title 为`Redis 教程 `或者 `MongoDB 教程` 的所有文档

```sql
db.article.find({"likes": {$gt:50}, $or: [{"title": "Redis 教程"},{"title": "MongoDB 教程"}]})
```



#### 其他操作

##### Limit 与 Skip 操作

- 读取指定数量的文档，可以使用 `limit()` 方法，语法如下

```sql
db.collection.find().limit(NUMBER)
```

- 只查询 article 集合中的 2 条数据；

```sql
db.article.find().limit(2)
```

- 跳过指定数量的文档来读取，可以使用 `skip()` 方法，语法如下

```sql
db.collection.find().limit(NUMBER).skip(NUMBER)
```

- 从第二条开始，查询 article 集合中的2条数据；

```sql
db.article.find().limit(2).skip(1)
```



##### 排序

- 在 MongoDB 中使用 `sort()` 方法对数据进行排序，`sort()` 方法通过参数来指定排序的字段，并使用 1 和 -1 来指定排序方式，1 为升序，-1 为降序

```sql
db.collection.find().sort({KEY:1})
```

- 按 article 集合中文档的likes字段降序排列

```sql
db.article.find().sort({likes:-1})
```



##### 索引

- 索引通常能够极大的提高查询的效率，如果没有索引，MongoDB 在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录
- MongoDB 使用 `createIndex() `方法来创建索引，语法如下

```sql
db.collection.createIndex(keys, options)
# background：建索引过程会阻塞其它数据库操作，设置为true表示后台创建，默认为false
# unique：设置为true表示创建唯一索引
# name：指定索引名称，如果没有指定会自动生成
```

- 给 title 和 description 字段创建索引，1 表示升序索引，-1 表示降序索引，指定以后台方式创建

```sql
db.article.createIndex({"title":1,"description":-1}, {background: true})
```

- 查看 article 集合中已经创建的索引

```sql
db.article.getIndexes()
/* 1 */
[
    {
        "v" : 2,
        "key" : {
            "_id" : 1
        },
        "name" : "_id_",
        "ns" : "test.article"
    },
    {
        "v" : 2,
        "key" : {
            "title" : 1.0,
            "description" : -1.0
        },
        "name" : "title_1_description_-1",
        "ns" : "test.article",
        "background" : true
    }
]
```



##### 聚合

- MongoDB 中的聚合使用 `aggregate()` 方法，类似于 SQL 中的 group by 语句，语法如下

```sql
db.collection.aggregate(AGGREGATE_OPERATION)
```

- 聚合中常用操作符如下

| 操作符 | 描述       |
| ------ | ---------- |
| $sum   | 计算总和   |
| $avg   | 计算平均值 |
| $min   | 计算最小值 |
| $max   | 计算最大值 |

- 根据 by 字段聚合文档并计算文档数量，类似与 SQL 中的 count() 函数

```sql
db.article.aggregate([{$group : {_id : "$by", sum_count : {$sum : 1}}}])
/* 1 */
{
    "_id" : "Andy",
    "sum_count" : 2.0
}

/* 2 */
{
    "_id" : "Ruby",
    "sum_count" : 1.0
}
```

- 根据 by 字段聚合文档并计算 likes 字段的平均值，类似与 SQL 中的 avg() 语句

```sql
db.article.aggregate([{$group : {_id : "$by", avg_likes : {$avg : "$likes"}}}])
/* 1 */
{
    "_id" : "Andy",
    "avg_likes" : 100.0
}

/* 2 */
{
    "_id" : "Ruby",
    "avg_likes" : 100.0
}
```



##### 正则表达式

- MongoDB 使用 `$regex` 操作符来设置匹配字符串的正则表达式，可以用来模糊查询，类似于 SQL 中的 like 操作
- 例如查询 title 中包含 `教程` 的文档

```sql
db.article.find({title:{$regex:"教程"}})
```

- 不区分大小写的模糊查询，使用 `$options` 操作符

```sql
db.article.find({title:{$regex:"elasticsearch",$options:"$i"}})
```



## 整合 spring

Spring Data Mongodb 是 Spring 提供的一种以 Spring Data 风格来操作数据存储的方式，它可以避免编写大量的样板代码。

**源码地址：**

[https://github.com/langyastudio/langya-tech/tree/master/spring-boot/spring-boot-mongodb](https://github.com/langyastudio/langya-tech/tree/master/spring-boot/spring-boot-mongodb)



### pom.xml 依赖

```xml
<!---mongodb-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

```yml
server:
  port: 8080
  
spring:
  data:
    mongodb:
      host: 192.168.123.22
      port: 27017
      #具有用户凭据的集合的数据库的名称/账号/密码
      #没有时，可以不指定
      authentication-database: admin
      username: root
      password: mongo
      #存储的数据库
	  database: edu
```

**注意：**如果 mongodb 配置了账号密码，需要指定用户凭据集合的数据库的名称 `authentication-database`



### Document 数据记录对象

> 文档对象的 ID 域添加 @Id 注解，需要检索的字段添加 @Indexed 注解

```java
@Data
@Document
public class MemberReadHistory
{
    @Id
    private String id;

    @Indexed
    private Long memberId;

    private String memberNickname;
    private String memberIcon;

    @Indexed
    private Long productId;

    private String productName;
    private String productPic;
    private String productSubTitle;
    private String productPrice;
    private Date   createTime;
}
```



### MongoRepository 操作数据

> 继承 MongoRepository 接口，这样就拥有了一些基本的 Mongodb 数据操作方法，同时定义了一个衍生查询方法

```java
public interface MemberReadHistoryRepository extends MongoRepository<MemberReadHistory,String> {
    /**
     * 根据会员id按时间倒序获取浏览记录
     * @param memberId 会员id
     */
    List<MemberReadHistory> findByMemberIdOrderByCreateTimeDesc(Long memberId);
}
```



### Service 逻辑实现

```java
/**
 * 浏览记录管理
 */
public interface MemberReadHistoryService
{
    /**
     * 生成浏览记录
     */
    int create(MemberReadHistory memberReadHistory);

    /**
     * 批量删除浏览记录
     */
    int delete(List<String> ids);

    /**
     * 获取用户浏览历史记录
     */
    Page<MemberReadHistory> list(Long memberId, Integer pageNum, Integer pageSize);
}
```

```java
@Service
public class MemberReadHistoryServiceImpl implements MemberReadHistoryService
{
    @Autowired
    private MemberReadHistoryRepository memberReadHistoryRepository;

    @Override
    public int create(MemberReadHistory memberReadHistory)
    {
        memberReadHistory.setId(null);
        memberReadHistory.setCreateTime(new Date());
        memberReadHistoryRepository.save(memberReadHistory);

        return 1;
    }

    @Override
    public int delete(List<String> ids)
    {
        List<MemberReadHistory> deleteList = new ArrayList<>();
        for (String id : ids)
        {
            MemberReadHistory memberReadHistory = new MemberReadHistory();
            memberReadHistory.setId(id);
            deleteList.add(memberReadHistory);
        }
        memberReadHistoryRepository.deleteAll(deleteList);

        return ids.size();
    }

    @Override
    public Page<MemberReadHistory> list(Long memberId, Integer pageNum, Integer pageSize)
    {
        Pageable pageable = PageRequest.of(pageNum-1, pageSize);

        return memberReadHistoryRepository.findByMemberIdOrderByCreateTimeDesc(memberId, pageable);
    }
}
```



### Controller 接口定义

```java
@RestController
@RequestMapping("/api/member/readHistory")
@Validated
public class ApiController
{
    @Autowired
    private MemberReadHistoryService memberReadHistoryService;

    @PostMapping(value = "/create")
    public ResultInfo create(@RequestBody MemberReadHistory memberReadHistory)
    {
        int count = memberReadHistoryService.create(memberReadHistory);
        if (count > 0)
        {
            return ResultInfo.data(count);
        }
        else
        {
            return ResultInfo.data(EC.ERROR_DATA_SAVE_FAILURE);
        }
    }

    @PostMapping(value = "/delete")
    public ResultInfo delete(@RequestParam("ids") List<String> ids)
    {
        int count = memberReadHistoryService.delete(ids);
        if (count > 0)
        {
            return ResultInfo.data(count);
        }
        else
        {
            return ResultInfo.data(EC.ERROR_DATA_DELETE_FAILURE);
        }
    }

    @GetMapping(value = "/list")
    public ResultInfo list(Long memberId,
                           @RequestParam(value = "pageNum", defaultValue = "1") Integer pageNum,
                           @RequestParam(value = "pageSize", defaultValue = "5") Integer pageSize)
    {
        Page<MemberReadHistory> memberReadHistoryList = memberReadHistoryService.list(memberId, pageNum, pageSize);
        return ResultInfo.data(memberReadHistoryList);
    }
}
```



### 接口测试

**添加浏览记录**

采用 PostMan 工具发送 POST 请求 http://localhost:8080/api/member/readHistory/create 接口，传入 Body 的 JSON 数据：

```
{
    "memberId": 1,
    "memberNickname": "郎涯",
    "memberIcon": "图像地址",
    "productId": "59",
    "productName": "xiaomi9A",
    "productPic": "9A 图像地址",
    "productSubTitle": "详细描述信息",
    "productPrice": 600
}
```

![image-20220120114224079](https://img-note.langyastudio.com/202201201142265.png?x-oss-process=style/watermark)

**查询记录**

采用 PostMan 工具发送 GET 请求 http://localhost:8080/api/member/readHistory/list?memberId=1 接口

![image-20220120115801744](https://img-note.langyastudio.com/202201201158850.png?x-oss-process=style/watermark)



## 参考

[MongoDB 快速入门，掌握这些刚刚好！](http://www.macrozheng.com/#/reference/mongodb_start)

[MongoDB 都有哪些使用的业务场景](https://cloud.tencent.com/developer/article/1583307)

[万字详解，吃透 MongoDB！](https://mp.weixin.qq.com/s/T_oxRwDp8W-OgLwb9AbdCw)

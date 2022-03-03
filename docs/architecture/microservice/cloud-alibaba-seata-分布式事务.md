## 分布式事务

### 单体应用

单体应用中，一个业务操作需要调用三个模块完成，此时数据的一致性由本地事务来保证。

![img](https://img-note.langyastudio.com/202201251038178.png?x-oss-process=style/watermark)



### 微服务应用

随着业务需求的变化，单体应用被拆分成微服务应用，原来的三个模块被拆分成三个独立的应用，分别使用独立的数据源，业务操作需要调用三个服务来完成。此时每个服务内部的数据一致性由本地事务来保证，但是全局的数据一致性问题没法保证。

![img](https://img-note.langyastudio.com/202201251038919.png?x-oss-process=style/watermark)



在微服务架构中由于全局数据一致性没法保证产生的问题就是分布式事务问题。简单来说，一次业务操作需要操作多个数据源或需要进行远程调用，就会产生分布式事务问题。



## Seata 简介

Seata 是一款开源的 **分布式事务解决方案**，致力于提供高性能和简单易用的分布式事务服务。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。

[Seata](https://github.com/seata/seata) 以高效并且对业务 0 侵入的方式，解决 微服务 场景下面临的分布式事务问题。

![image](https://img-note.langyastudio.com/202201251038877.png?x-oss-process=style/watermark)



**协议分布式事务处理过程的三个组件：**

- TC (Transaction Coordinator) - 事务协调者

  维护全局和分支事务的状态，驱动全局事务提交或回滚

- TM (Transaction Manager) - 事务管理器

  定义全局事务的范围：开始全局事务、提交或回滚全局事务

- RM (Resource Manager) - 资源管理器

  管理分支事务处理的资源，与 TC 交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚



**一个典型的分布式事务过程：**

- TM 向 TC 申请开启一个**全局事务**，全局事务创建成功并生成一个全局唯一的 XID
- XID 在微服务调用链路的上下文中传播
- RM 向 TC 注册**分支事务**，将其纳入 XID 对应全局事务的管辖
- TM 向 TC 发起针对 XID 的全局提交或回滚决议
- TC 调度 XID 下管辖的全部分支事务完成提交或回滚请求

![img](https://img-note.langyastudio.com/202201251039246.png?x-oss-process=style/watermark)



## Spring Cloud 支持

- 通过 Spring MVC 提供服务的服务提供者，在收到 header 中含有 Seata 信息的 HTTP 请求时，可以自动还原 Seata 上下文

- 支持服务调用者通过 RestTemplate 调用时，自动传递 Seata 上下文

- 支持服务调用者通过 FeignClient 调用时，**自动传递 Seata 上下文**

- 支持 SeataClient 和 Hystrix 同时使用的场景

- 支持 SeataClient 和 Sentinel 同时使用的场景



## Seata 安装

### docker-compose 

- 使用 Nacos 作为注册中心

- 准备 `registry.conf` 文件

  注册 seata 服务到 nacos 注册中心；指定 seata 配置的方式为 file ，即文件类型


```yml
registry {
  type = "nacos"

  nacos {
  # seata 服务注册在 nacos 上的别名，客户端通过该别名调用服务
    application = "seata-server"
  # 指定注册至nacos注册中心的分组名
    group = "SEATA_GROUP"
    
  # 请根据实际生产环境配置nacos服务的ip和端口
    serverAddr = "192.168.123.22:8848"
  # nacos上指定的namespace
    namespace = ""
  # 指定注册至nacos注册中心的集群名  
    cluster = "default"
    username = "nacos"
    password = "nacos"
  }
}

config {
  type = "file"

  file {
    name="file:/root/seata-config/file.conf"
  }
}
```

- 准备 `file.conf` 文件

  配置存储模式为 db，并指定数据库的 host、user、password 等，[更多配置](https://github.com/seata/seata/blob/develop/script/config-center/config.txt)

```properties
# 存储模式
store.mode=db

store.db.datasource=druid
store.db.dbType=mysql

# 需要根据 mysql 的版本调整 driverClassName
# mysql8 及以上版本对应的 driver：com.mysql.cj.jdbc.Driver
# mysql8 以下版本的 driver：com.mysql.jdbc.Driver
store.db.driverClassName=com.mysql.cj.jdbc.Driver

# 注意根据生产实际情况调整参数 host 和 port
store.db.url="jdbc:mysql://192.168.123.22:3306/seata-server?useUnicode=true&characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&allowPublicKeyRetrieval=true"

# 数据库用户名
store.db.user=test

# 用户名密码
store.db.password=testMysql
```

`store.mode` 为 db 模式需要在数据库 `seata-server` 创建对应的表结构，[[建表脚本\]](https://github.com/seata/seata/tree/develop/script/server/db)

创建一个 seata-server 数据库，建表 sql 位于 `doc/seata/db_store.sql` 中，以 mysql 为例：

```sql
-- The script used when storeMode is 'db' 
-- the table to store GlobalSession data
  CREATE TABLE IF NOT EXISTS `global_table`
  (
      `xid`                       VARCHAR(128) NOT NULL,
      `transaction_id`            BIGINT,
      `status`                    TINYINT      NOT NULL,
      `application_id`            VARCHAR(32),
      `transaction_service_group` VARCHAR(32),
      `transaction_name`          VARCHAR(128),
      `timeout`                   INT,
      `begin_time`                BIGINT,
      `application_data`          VARCHAR(2000),
      `gmt_create`                DATETIME,
      `gmt_modified`              DATETIME,
      PRIMARY KEY (`xid`),
      KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
      KEY `idx_transaction_id` (`transaction_id`)
  ) ENGINE = InnoDB
    DEFAULT CHARSET = utf8;
  
  -- the table to store BranchSession data
  CREATE TABLE IF NOT EXISTS `branch_table`
  (
      `branch_id`         BIGINT       NOT NULL,
      `xid`               VARCHAR(128) NOT NULL,
      `transaction_id`    BIGINT,
      `resource_group_id` VARCHAR(32),
      `resource_id`       VARCHAR(256),
      `branch_type`       VARCHAR(8),
      `status`            TINYINT,
      `client_id`         VARCHAR(64),
      `application_data`  VARCHAR(2000),
      `gmt_create`        DATETIME(6),
      `gmt_modified`      DATETIME(6),
      PRIMARY KEY (`branch_id`),
      KEY `idx_xid` (`xid`)
  ) ENGINE = InnoDB
    DEFAULT CHARSET = utf8;
  
  -- the table to store lock data
  CREATE TABLE IF NOT EXISTS `lock_table`
  (
      `row_key`        VARCHAR(128) NOT NULL,
      `xid`            VARCHAR(128),
      `transaction_id` BIGINT,
      `branch_id`      BIGINT       NOT NULL,
      `resource_id`    VARCHAR(256),
      `table_name`     VARCHAR(32),
      `pk`             VARCHAR(36),
      `status`         TINYINT      NOT NULL DEFAULT '0' COMMENT '0:locked ,1:rollbacking',
      `gmt_create`     DATETIME,
      `gmt_modified`   DATETIME,
      PRIMARY KEY (`row_key`),
      KEY `idx_status` (`status`),
      KEY `idx_branch_id` (`branch_id`)
  ) ENGINE = InnoDB
    DEFAULT CHARSET = utf8;
  
  CREATE TABLE IF NOT EXISTS `distributed_lock`
  (
      `lock_key`       CHAR(20) NOT NULL,
      `lock_value`     VARCHAR(20) NOT NULL,
      `expire`         BIGINT,
      primary key (`lock_key`)
  ) ENGINE = InnoDB
    DEFAULT CHARSET = utf8mb4;
  
  INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('AsyncCommitting', ' ', 0);
  INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('RetryCommitting', ' ', 0);
  INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('RetryRollbacking', ' ', 0);
  INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('TxTimeoutCheck', ' ', 0);
```

- 准备 docker-compose.yaml 文件

将 `doc/seata/conf/registry.conf 与 file.conf` 放到 `/mnt/volume/seata/config` 文件夹中。nacos、seata 容器配置如下：

```yml
version: '3.5'

services:
  nacos-registry:
    image: nacos/nacos-server:2.0.3
    container_name: nacos-registry
    #command:
    restart: always
    #https://nacos.io/zh-cn/docs/quick-start-docker.html
    environment:
      - "MODE=standalone"
    ports:
      - 8848:8848

  # http://seata.io/zh-cn/docs/ops/deploy-by-docker-compose.html
  # chmod 777 /mnt/volume/seata
  seata:
    image: seataio/seata-server:1.4.2
    container_name: seata
    environment:
      # 指定seata服务启动端口
      - SEATA_PORT=8091
      # 注册到nacos上的ip。客户端将通过该ip访问seata服务
      - SEATA_IP=192.168.123.22
      - SEATA_CONFIG_NAME=file:/root/seata-config/registry
    restart: always
    volumes:
      # 因为registry.conf中是nacos配置中心，只需要把doc/seata/conf/registry.conf与file.conf放到/mnt/volume/seata/config文件夹中
      - /mnt/volume/seata/conf:/root/seata-config
      - /mnt/volume/seata/logs:/root/logs
    ports:
      - "8091:8091"
```

- 启动容器

运行 `docker-compose -f docker-compose-env.yml up -d` 启动 nacos、seata 容器

可以看到 seata-server 成功注册到nacos注册中心

![image-20220125103300989](https://img-note.langyastudio.com/202201251033205.png?x-oss-process=style/watermark)



### 环境变量

seata-server 支持以下环境变量：

- SEATA_IP

可选, 指定 seata-server 启动的 IP, 该 IP 用于向注册中心注册时使用

- SEATA_PORT

可选, 指定 seata-server 启动的端口, 默认为 `8091`

- STORE_MODE

可选, 指定 seata-server 的事务日志存储方式, 支持 `db`, `file`, `redis` (Seata-Server 1.3及以上版本支持), 默认是 `file`

- SERVER_NODE

可选, 用于指定 seata-server 节点 ID, 如 `1`,`2`,`3`...,  默认为 `根据ip生成`

- SEATA_ENV

可选, 指定 seata-server 运行环境, 如 `dev`, `test` 等, 服务启动时会使用 `registry-dev.conf` 这样的配置

- SEATA_CONFIG_NAME

可选, 指定配置文件位置, 如 `file:/root/registry`, 将会加载 `/root/registry.conf` 作为配置文件，如果需要同时指定 `file.conf` 文件，需要将 `registry.conf`的`config.file.name` 的值改为类似 `file:/root/file.conf`：



## 项目实战

源码：[https://github.com/langyastudio/langya-tech/tree/master/spring-cloud](https://github.com/langyastudio/langya-tech/tree/master/spring-cloud)

### 数据库准备

- 业务数据库

  - seat-order：存储订单的数据库
  - seat-storage：存储库存的数据库
  - seat-account：存储账户信息的数据库

- 日志回滚表

  每个业务数据库都需要添加日志回滚表

  Seata AT 模式需要使用日志回滚表 undo_log 表

完整 SQL 如下：

```sql
-- seat-order：存储订单的数据库
create database `seat-order`;
use `seat-order`;

CREATE TABLE `seat-order`.`order_tbl`
(
    `id`             int(11) NOT NULL AUTO_INCREMENT,
    `user_id`        varchar(255) DEFAULT NULL COMMENT '用户id',
    `commodity_code` varchar(255) DEFAULT NULL COMMENT '商品编码',
    `count`          int(11)      DEFAULT 0 COMMENT '数量',
    `money`          int(11)      DEFAULT 0 COMMENT '金额',
    `status`         int(1)       DEFAULT NULL COMMENT '订单状态：0：创建中；1：已完结',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- Seata AT 模式需要使用日志回滚表 undo_log 表
-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
CREATE TABLE `seat-order`.`undo_log`
(
    `id`            bigint(20)   NOT NULL AUTO_INCREMENT,
    `branch_id`     bigint(20)   NOT NULL,
    `xid`           varchar(100) NOT NULL,
    `context`       varchar(128) NOT NULL,
    `rollback_info` longblob     NOT NULL,
    `log_status`    int(11)      NOT NULL,
    `log_created`   datetime     NOT NULL,
    `log_modified`  datetime     NOT NULL,
    `ext`           varchar(100) DEFAULT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8;


-- seat-storage：存储库存的数据库
create database `seat-storage`;
use `seat-storage`;

CREATE TABLE `seat-storage`.`storage_tbl`
(
    `id`             int(11) NOT NULL AUTO_INCREMENT,
    `commodity_code` varchar(255) DEFAULT NULL COMMENT '商品编码',
    `count`          int(11)      DEFAULT 0 COMMENT '数量',
    PRIMARY KEY (`id`),
    UNIQUE KEY (`commodity_code`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
INSERT INTO `seat-storage`.`storage_tbl` (`id`, `commodity_code`, `count`)
VALUES (1, 'C00321', 100);

-- Seata AT 模式需要使用日志回滚表 undo_log 表
-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
CREATE TABLE `seat-storage`.`undo_log`
(
    `id`            bigint(20)   NOT NULL AUTO_INCREMENT,
    `branch_id`     bigint(20)   NOT NULL,
    `xid`           varchar(100) NOT NULL,
    `context`       varchar(128) NOT NULL,
    `rollback_info` longblob     NOT NULL,
    `log_status`    int(11)      NOT NULL,
    `log_created`   datetime     NOT NULL,
    `log_modified`  datetime     NOT NULL,
    `ext`           varchar(100) DEFAULT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8;


-- seat-account：存储账户信息的数据库
create database `seat-account`;
use `seat-account`;

CREATE TABLE `seat-account`.`account_tbl`
(
    `id`      int(11) NOT NULL AUTO_INCREMENT,
    `user_id` varchar(255) DEFAULT NULL COMMENT '用户id',
    `money`   int(11)      DEFAULT 0 COMMENT '金额',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
INSERT INTO `seat-account`.`account_tbl` (`id`, `user_id`, `money`)
VALUES (1, 'U100001', 1000);

-- Seata AT 模式需要使用日志回滚表 undo_log 表
-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
CREATE TABLE `seat-account`.`undo_log`
(
    `id`            bigint(20)   NOT NULL AUTO_INCREMENT,
    `branch_id`     bigint(20)   NOT NULL,
    `xid`           varchar(100) NOT NULL,
    `context`       varchar(128) NOT NULL,
    `rollback_info` longblob     NOT NULL,
    `log_status`    int(11)      NOT NULL,
    `log_created`   datetime     NOT NULL,
    `log_modified`  datetime     NOT NULL,
    `ext`           varchar(100) DEFAULT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8;
```



完整数据库示意图：

![image-20220125104919942](https://img-note.langyastudio.com/202201251049023.png?x-oss-process=style/watermark)



### 客户端配置

对 seata-order-service、seata-storage-service 和 seata-account-service 三个 seata 的客户端进行配置，它们配置大致相同，下面以 seata-order-service 的配置为例

- 修改 application.yml 文件

  - 数据库连接url、账号、密码

  - 自定义事务组的名称
  - nacos 注册配置

```yaml
spring:
  application:
    name: order-service

  datasource:
    name: '"orderDataSource"'
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.123.22:3306/seat-order?useSSL=false&serverTimezone=UTC
    username: test
    password: testMysql
    druid:
      initial-size: 2
      max-active: 20
      min-idle: 2

  cloud:
    nacos:
      discovery:
        server-addr: 192.168.123.22:8848
        username: naocs
        password: nacos

    alibaba:
      seata:
        #自定义事务组名称需要与 seata-server 中的对应
        tx-service-group: business-service

seata:
  enabled: true
  service:
    disable-global-transaction: false
    grouplist:
      default: 192.168.123.22:8091
    vgroup-mapping:
      #事务分组名=集群名称
      #集群名需要与Seata-server注册到Nacos的cluster保持一致
      business-service: default

  # if use registry center
  registry:
    nacos:
      cluster: default
      server-addr: 192.168.123.22
      username: nacos
      password: nacos
    type: nacos
```

- 修改全局事务处理

  使用 `@GlobalTransactional` 注解开启分布式事务，如

  ```java
  @GlobalTransactional(timeoutMills = 300000, name = "spring-cloud-business-tx")
  ```



### 事务问题

这里会创建三个服务，一个订单服务，一个库存服务，一个账户服务。该操作跨越三个数据库，有两次远程调用，很明显会有分布式事务问题。

- 当用户下单时，会在订单服务中创建一个订单
- 然后通过远程调用库存服务来扣减下单商品的库存
- 再通过远程调用账户服务来扣减用户账户里面的余额
- 最后在订单服务中修改订单状态为已完成



### 事务演示

> 通过 @GlobalTransactional 参数，实现所有服务数据要么执行、要么不执行

如果分布式事务生效的话， 那么以下等式应该成立：


- 用户原始金额(1000) = 用户现存的金额  +  货物单价 (2) * 订单数量 * 每单的货物数量(2)

- 货物的初始数量(100) = 货物的现存数量 + 订单数量 * 每单的货物数量(2)

  

在本示例中，模拟了一个用户购买货物的场景，seata-storage-service 负责扣减库存数量，seata-order-service 负责保存订单，seata-account-service 负责扣减用户账户余额。

为了演示样例，在 seata-account-service 中使用 random.nextBoolean() 的方式来随机抛出异常,模拟了在服务调用时随机发生异常的场景。

- 运行 seata-order-service、seata-storage-service、seata-account-service、seata-business-service 四个服务

- 调用接口进行下单操作后查看数据库：http://localhost:18081/seata/feign

  由于 account 服务抛出异常，此时可以发现下单后数据库数据并没有任何改变

- 在 seata-account-service 中屏蔽 `throw new RuntimeException` 异常后，继续接口

  ```java
  @GetMapping(value = "/account", produces = "application/json")
  public String account(String userId, int money)
  {
      --------
      if (random.nextBoolean())
      {
          //throw new RuntimeException("this is a mock Exception");
      }
  	-----   
  }
  ```

​	   正常执行，发现数据库中的数据发生变化



## 参考

[Seata 官方文档](http://seata.io/zh-cn/docs/overview/what-is-seata.html)

[使用 Seata 彻底解决 Spring Cloud 中的分布式事务问题！](http://www.macrozheng.com/#/cloud/seata?id=使用seata彻底解决spring-cloud中的分布式事务问题！)
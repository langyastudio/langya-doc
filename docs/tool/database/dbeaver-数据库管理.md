> 本文来自JavaGuide，郎涯进行简单排版与补充

## DBeaver 概览

DBeaver 是一个基于 Java 开发 ，并且支持几乎所有的数据库产品的开源数据库管理工具。

DBeaver 社区版不光支持关系型数据库比如MySQL、PostgreSQL、MariaDB、SQLite、Oracle、Db2、SQL Server，还比如 SQLite、H2 这些内嵌数据库。还支持常见的全文搜索引擎比如 Elasticsearch 和 Solr、大数据相关的工具比如 Hive和 Spark。

![DBeaver 支持的数据库概览](https://img-note.langyastudio.com/202206061535137.jpeg?x-oss-process=style/watermark)

甚至说，DBeaver 的商业版本还支持各种 NoSQL  数据库

![DBeaver 的商业版本还支持各种 NoSQL  数据库](https://img-note.langyastudio.com/202206061535754.png?x-oss-process=style/watermark)



## 使用

**DBeaver 虽然小巧，但是功能还是十分强大的。基本的表设计、SQL 执行、ER 图、数据导入导出等等常用功能都不在话下。**



### 下载安装

官方网提供的下载地址：https://dbeaver.io/download/ ，你可以根据自己的操作系统选择合适的版本进行下载安装。

比较简单，这里就不演示了。



### 连接数据库

选择自己想要的连接的数据库，然后点击下一步即可（第一次连接可能需要下载相关驱动）

这里以 MySQL 为例

![image-20200803082419586](https://img-note.langyastudio.com/202206061539363.png?x-oss-process=style/watermark)

输入数据库的地址、用户名和密码等信息，然后点击完成即可连接

点击完成之前，你可以先通过左下方的测试连接来看一下数据库是否可以被成功连接上

![](https://img-note.langyastudio.com/202206061539489.png?x-oss-process=style/watermark)



### 新建数据库

右键-> 新建数据库

![](https://img-note.langyastudio.com/202206061539014.png?x-oss-process=style/watermark)



### 数据库表相关操作

#### 新建表

![](https://img-note.langyastudio.com/202206061539765.png?x-oss-process=style/watermark)

#### 新建列

![](https://img-note.langyastudio.com/202206061539706.png?x-oss-process=style/watermark)



#### 创建约束（主键、唯一键）

![](https://img-note.langyastudio.com/202206061539804.png?x-oss-process=style/watermark)

![](https://img-note.langyastudio.com/202206061539654.png?x-oss-process=style/watermark)



#### 插入数据

我们通过 SQL 编辑器插入数据：

![](https://img-note.langyastudio.com/202206061539895.png?x-oss-process=style/watermark)

```java
INSERT into user(id,name,phone,password) values ('A00001','guide哥','181631312315','123456'); 
INSERT into user(id,name,phone,password) values ('A00002','guide哥2','181631312313','123456');
INSERT into user(id,name,phone,password) values ('A00003','guide哥3','181631312312','123456');
```



### 导入/导出

在对应的数据库实例上右击，选择 **工具 -> 转储数据库/执行脚本**，实现导出数据库与执行 SQL 脚本的需求



## 总结

总的来说，简单体验之后感觉还是很不错的，占用内存也确实比 DataGrip 确实要小很多。

各位小伙伴可以自行体验一下。毕竟免费并且开源，还是很香的！




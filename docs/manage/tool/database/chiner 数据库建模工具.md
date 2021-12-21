> 本文来自JavaGuide，郎涯进行简单排版与补充



CHINER 的前身是 PDMan，CHINER 是 CHINESE Entity Relation 的缩写，翻译过来就是国产实体关系图工具。**可以直接导入 PowerDesigner 文件、PDMan 文件**，还可以直接从数据库或者 DDL 语句直接导入。

![](https://img-note.langyastudio.com/202111171118717.png?x-oss-process=style/watermark)

CHINER 的技术栈：React+Electron+Java 。

* Gitee 地址：https://gitee.com/robergroup/chiner 

* 操作手册： https://www.yuque.com/chiner/docs/manual 

  

### 下载安装

CHINER 提供了 **Windows** 、**Mac** 、**Linux** 下的一键安装包，我们直接下载即可。

> 下载地址：https://gitee.com/robergroup/chiner/releases

需要注意的是：如果你当前使用的 Chrome 浏览器的话，无法直接点击链接下载。你可以更换浏览器下载或者右键链接选择链接存储为...。

![](https://img-note.langyastudio.com/202111171118982.png?x-oss-process=style/watermark)



打开软件之后，界面如下图所示

![](https://img-note.langyastudio.com/202111171123710.png?x-oss-process=style/watermark)



我这里以电商项目参考模板来演示 CHINER 的基本操作。

### 模块化管理

电商项目比较复杂，我们可以将其拆分为一个一个独立的模块（表分组），每个模块下有数据表，视图，关系图，数据字典。像这个电商项目就创建了 3 个模块：消费端、商家端、平台端。

![](https://img-note.langyastudio.com/202111171130400.png?x-oss-process=style/watermark)

不过，对于一些比较简单的项目比如博客系统、企业管理系统直接使用简单模式即可。



### 数据库表管理

右键数据表即可创建新的数据库表，点击指定的数据库表即可对指定的数据库表进行设计

![](https://img-note.langyastudio.com/202111171130833.png?x-oss-process=style/watermark)



并且，数据表字段可以直接关联数据字典。

![](https://img-note.langyastudio.com/202111171138096.png?x-oss-process=style/watermark)



如果需要创建视图的话，直接右键视图即可。视图是从一个或多个表导出的虚拟的表，其内容由查询定义。具有普通表的结构，但是不实现数据存储。

![](https://img-note.langyastudio.com/202111171133423.png?x-oss-process=style/watermark)

数据库视图可以方便我们进行查询。不过，数据库视图会影响数据库性能，通常不建议使用。



### 关系图

我平时在项目中比较常见的 **ER 关联关系图** ，可以使用 CHINER 进行手动维护。

如果你需要添加新的数据库表到关系图的话，直接拖拽指定的数据库表到右边的关系图展示界面即可。另外，表与表之间的关联也需要你手动对相关联的字段进行连接。

![](https://img-note.langyastudio.com/202111171133980.png?x-oss-process=style/watermark)

手动进行维护，说实话还是比较麻烦的，也比较容易出错。



像 [Navicat Data Modeler](https://www.navicat.com.cn/products/navicat-data-modeler) 在这方面就强多了，它可以自动生成 ER 图。

![](https://img-note.langyastudio.com/202111171133611.png?x-oss-process=style/watermark)



### 数据库表代码模板

支持直接生成对应表的 SQL 代码（支持 MySQL、Oracle、SQL Server、PostgreSQL 等数据库）并且还提供了 Java 和 C# 的 JavaBean。

![](https://img-note.langyastudio.com/202111171133817.png?x-oss-process=style/watermark)



### 导出数据库表

你可以选择导出 DDL、Word 文档、数据字典 SQL、当前关系图的图片。

![](https://img-note.langyastudio.com/202111171133015.png?x-oss-process=style/watermark)



### 数据库逆向

你还可以连接数据库，逆向解析数据库。

![](https://img-note.langyastudio.com/202111171133461.png?x-oss-process=style/watermark)



数据库连接成功之后，我们点击右上角的菜单 `导入—> 从数据库导入` 即可。

![](https://img-note.langyastudio.com/202111171133116.png?x-oss-process=style/watermark)

SQLite 是一个进程内的库，实际操作时直接访问其存储文件。实现了自给自足的、无服务器的、零配置的、事务性的 SQL 数据库引擎。

它的数据库就是一个文件，由于 SQLite 本身是 C 写的，而且体积很小，所以经常被集成到各种应用程序中，甚至在 iOS 和 Android 的 App 中都可以集成。



#### SQLite 存储类

![在这里插入图片描述](https://img-note.langyastudio.com/20210708093725.png?x-oss-process=style/watermark)



SQLite 支持列的亲和类型概念。任何列仍然可以存储任何类型的数据，当数据插入时，该字段的数据将会优先采用亲缘类型作为该值的存储方式。
![在这里插入图片描述](https://img-note.langyastudio.com/20210708093734.png?x-oss-process=style/watermark)



#### SQLite 注释

没法像 MySQL 那样增加 comment 注释，但可以通过 `--` 的方式增加 DDL 注释
例如：

```sql
CREATE TABLE s_tests (
	id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
	title varchar (128) NOT NULL DEFAULT '' COLLATE NOCASE, 	-- 标题
	content text NOT NULL DEFAULT '' COLLATE NOCASE, 	-- 内容
	description varchar (512) NOT NULL DEFAULT '' COLLATE NOCASE, 	-- 简介
	img_path varchar (128) NOT NULL DEFAULT '' COLLATE NOCASE, 	-- 图像全路径
	update_time datetime NOT NULL, 	-- 更新时间
	delete_time datetime DEFAULT NULL, 	-- 删除标记
	create_time datetime NOT NULL 	-- 创建时间
);
```
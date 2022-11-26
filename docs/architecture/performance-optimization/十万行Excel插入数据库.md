> https://mp.weixin.qq.com/s/teHAbU-1UpXdTJW_vx_z3Q

提升 Excel 导入速度的方法：

- 使用更快的 Excel 读取框架(推荐使用阿里 EasyExcel)
- 对于需要与数据库交互的校验、按照业务逻辑适当的使用缓存。用空间换时间
- 使用 values(),(),() 拼接长 SQL 一次插入多行数据 ，如 1000 条（innodb_buffer_pool_size 因为过长的 SQL 在写操作的时候由于超过内存阈值，可能会发生了磁盘交换）
- 使用多线程插入数据，利用掉网络 IO 等待时间(推荐使用并行流 parallelStream，简单易用)
- 避免在循环中打印无用的日志
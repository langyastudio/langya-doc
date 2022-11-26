[没内鬼，来点干货！SQL优化和诊断](https://juejin.cn/post/6844904135964229646)

[那些年我们一起优化的SQL](https://mp.weixin.qq.com/s/sPO-6ULwIfUexLY3V4acBg)

[聊聊sql优化的15个小技巧 ](https://cloud.tencent.com/developer/article/1899907)

[后端程序员必备：SQL高性能优化指南！35+条优化建议立马GET! ](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247488618&idx=1&sn=e70a31865b5eadcb151f439004a4dd72&chksm=cea25ba1f9d5d2b795222ba90e0326618d649e858ec23e9c7360f90fbfc23a7786c33bff9556&token=1647609083&lang=zh_CN#rd)

[面试题：在日常工作中怎么做MySQL优化的？](https://mp.weixin.qq.com/s/AuDUJs35dBVLenSuT4RCWQ)

[为什么大家都说SELECT * 效率低](https://blog.csdn.net/qq_39390545/article/details/106766965)

[MySQL 性能优化神器 Explain 使用分析](https://segmentfault.com/a/1190000008131735)

[慢查询 MySQL 定位优化技巧](https://mp.weixin.qq.com/s/77pCwiyfOrZn-zsv7cFaWA)



### count(*) 为什么会慢

> https://mp.weixin.qq.com/s/5037NgbeJd69CAo_3snmrA

**innodb 引擎**的数据表则会**查全表数据**，选择**体积最小的索引树**，然后通过遍历叶子节点的个数挨个加起来，这样也能得到全表数据。当数据表行数变大后，**单次 count 就需要扫描大量的数据**，因此很可能就会出现超时报错。



#### **性能排序**

```mysql
count(*) ≈ count(1) > count(主键id) > count(普通索引列) > count(未加索引列)
```



#### **允许粗略估计行数**

希望知道数据库里还有多少短信是堆积在那没发的，具体是 1k 还是 2k 其实都是差不多量级，等到了百万以上，具体数值已经不重要了，我们知道它现在堆积得很离谱，就够了。因此这个场景，其实是允许使用**比较粗略**的估计的。

**explain命令** 有个 **rows**，会用来**估计**接下来执行这条 sql 需要扫描和检查多少行。它是通过采样的方式计算出来的，虽然会有一定的偏差，但它能反映一定的数量级。

![图片](https://img-note.langyastudio.com/202208310946048.png?x-oss-process=style/watermark)

有些语言的orm里可能没有专门的explain语法，但是肯定有执行raw sql的功能，你**可以把 explain 语句当做 raw sql传入，从返回的结果里将 rows 那一列读出来使用。**

一般情况下，explain 的 sql 如果能走索引，那会比不走索引的情况更准 。单个字段的索引会比多个字段组成的复合索引要准。索引区分度越高，rows 的值也会越准。



#### 要求行数准确

可以建个新表，里面专门放表行数的信息。

- 如果对**实时性要求比较高**的话，可以将更新行数的 sql 放入到对应事务里，这样既能满足事务隔离性，还能快速读取到行数信息

- 如果对**实时性要求不高**，接受一小时或者一天的更新频率，那既可以自己写脚本遍历全表后更新行数信息。也可以将通过监听 binlog 将数据导入 hive，需要数据时直接通过 hive 计算得出。

  举个例子，比如上面的短信表，可以**按 id 排序**，每次取出 1w 条数据，**记下这一批里最大的 id，然后下次从最大id 开始再拿 1w 条数据出来，不断循环。**

  对于未发送的短信，就只需要在捞出的那 1w 条数据里，筛选出 state=0 的条数。

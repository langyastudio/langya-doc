## Redis 事务遇上 @Transactional 有大坑

> https://mp.weixin.qq.com/s/vwx99ER-9qiH9nfwtFW1zQ

![图片](https://img-note.langyastudio.com/202211180824720.png?x-oss-process=style/watermark)

RedisTemplete 开启了 Redis 事务支持后，在 `@Transactional` 中执行的 Redis 命令**也会被认为是在 Redis 事务中执行的**，要执行的递增命令会被放到队列中，不会立即返回执行后的结果，返回的是一个 null，需要等待事务提交时，队列中的命令才会顺序执行，最后 Redis 数据库的键值才会递增。



### 方案一

每次 Redis 的事务操作完成后，关闭 Redis 事务支持，然后再执行 `@Transactional` 中的 Redis 命令（**有弊端**）

![图片](https://img-note.langyastudio.com/202211180827370.png?x-oss-process=style/watermark)



**但是这种写法有个弊端**，如果在执行 Redis 事务期间，在 @Transactional 注解的方法里面执行 Redis 命令，则还是会造成返回结果为 null。

![图片](https://img-note.langyastudio.com/202211180828239.png?x-oss-process=style/watermark)



### 方案二

创建两个 StringRedisTemplate，一个专门用来执行 Redis 事务，一个用来执行普通的 Redis 命令

弄两个 `RedisTemplate` Bean，一个是用来执行 Redis 事务的，一个是用来执行普通 Redis 命令的（不支持事务）。不同的地方引入不同的 Bean 就可以了。

![图片](https://img-note.langyastudio.com/202211180828057.png?x-oss-process=style/watermark)

接下来在测试的 Service 类中注入两个不同的 `StringRedisTemplate` 实例，代码如下所示：

![图片](https://img-note.langyastudio.com/202211180829435.png?x-oss-process=style/watermark)


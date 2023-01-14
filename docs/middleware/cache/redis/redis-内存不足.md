## 错误日志
通过 redis.log 可以看到错误日志如下：`Cannot allocate memory`
```log
15602:M 30 Dec 2022 17:39:09.988 * RDB memory usage when created 19775.56 Mb
15602:M 30 Dec 2022 17:39:44.766 # Done loading RDB, keys loaded: 529954, keys expired: 26.
15602:M 30 Dec 2022 17:39:44.766 * DB loaded from disk: 34.780 seconds
15602:M 30 Dec 2022 17:39:44.766 * Ready to accept connections
15602:M 30 Dec 2022 17:41:40.133 * 10000 changes in 60 seconds. Saving...
15602:M 30 Dec 2022 17:41:40.184 # Can't save in background: fork: Cannot allocate memory
15602:M 30 Dec 2022 17:41:46.015 * 10000 changes in 60 seconds. Saving...
15602:M 30 Dec 2022 17:41:46.070 # Can't save in background: fork: Cannot allocate memory
15602:M 30 Dec 2022 17:41:52.019 * 10000 changes in 60 seconds. Saving...
```



## 内存淘汰机制

> 务必为希望到期的键设置 [TTL](https://redis.io/commands/ttl)

### **过期数据的删除策略**

如果假设你设置了一批 key 只能存活 1 分钟，那么 1 分钟后，redis 是怎么对这批 key 进行删除的呢？

常用的过期数据的删除策略就两个（重要！自己造缓存轮子的时候需要格外考虑的东西）：

- **惰性删除** ：只会在取出 key 的时候才对数据进行过期检查。这样对 CPU 最友好，但是可能会造成太多过期 key 没有被删除。
- **定期删除** ： 每隔一段时间抽取一批 key 执行删除过期 key 操作。并且，redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对 CPU 时间的影响。

定期删除对内存更加友好，惰性删除对 CPU 更加友好。两者各有千秋，所以 redis 采用的是 **定期删除+惰性/懒汉式删除** 。

但是，仅仅通过给 key 设置过期时间还是有问题的。**因为还是可能存在定期删除和惰性删除漏掉了很多过期 key 的情况。**这样就导致大量过期 key 堆积在内存里，然后就 **Out of memory** 了。

怎么解决这个问题呢？答案就是：**redis 内存淘汰机制。**



### **redis 内存淘汰机制**

> 使用 `config get maxmemory-policy` 命令，来查看当前 `Redis` 的内存淘汰策略。

**`Redis`** **默认使用的是** **`noeviction`** **类型的内存淘汰机制**，它表示当运行内存超过最大设置内存时，不淘汰任何数据，但新增操作会报错！！！！

redis 提供 6 种数据淘汰策略：

- **volatile-lru（least recently used）**：从已设置过期时间的数据集（server.db[i].expires）中挑选最近**最少使用**的数据淘汰
- **volatile-ttl**：从已设置过期时间的数据集（server.db[i].expires）中挑选**将要过期**的数据淘汰
- **volatile-random**：从已设置过期时间的数据集（server.db[i].expires）中**任意选择**数据淘汰
- **allkeys-lru（least recently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key
- **allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰
- **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。**这个应该没人使用吧！**

4.0 版本后增加以下两种：

- **volatile-lfu（least frequently used）**：从已设置过期时间的数据集（server.db[i].expires）中挑选最不经常使用的数据淘汰
- **allkeys-lfu（least frequently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的 key



### 如何配置

修改 `redis.conf` 文件，进行内存淘汰策略的配置

![在这里插入图片描述](https://img-note.langyastudio.com/202301031923158.png?x-oss-process=style/watermark)



## 内存碎片率

### 如何查看内存碎片率

使用 `info memory` 命令即可查看 Redis 内存相关的信息。下图中每个参数具体的含义，Redis 官方文档有详细的介绍：https://redis.io/commands/INFO 。

![img](https://img-note.langyastudio.com/202301031815485.png?x-oss-process=style/watermark)

Redis 内存碎片率的计算公式：`mem_fragmentation_ratio` （内存碎片率）= `used_memory_rss` (操作系统实际分配给 Redis 的物理内存空间大小) / `used_memory` (Redis 内存分配器为了存储数据实际申请使用的内存空间大小)

也就是说，`mem_fragmentation_ratio` （内存碎片率）的值越大代表内存碎片率越严重。

一定不要误认为 `used_memory_rss` 减去 `used_memory` 值就是内存碎片的大小！！！这不仅包括内存碎片，还包括其他进程开销，以及共享库、堆栈等的开销。

很多小伙伴可能要问了：“多大的内存碎片率才是需要清理呢？”。

通常情况下，我们认为 `mem_fragmentation_ratio > 1.5` 的话才需要清理内存碎片。 `mem_fragmentation_ratio > 1.5` 意味着你使用 Redis 存储实际大小 2G 的数据需要使用大于 3G 的内存。

如果想要快速查看内存碎片率的话，你还可以通过下面这个命令：

```Java
redis-cli -p 6379 info | grep mem_fragmentation_ratio
```

另外，内存碎片率可能存在小于 1 的情况。这种情况我在日常使用中还没有遇到过，感兴趣的小伙伴可以看看这篇文章 [故障分析 | Redis 内存碎片率太低该怎么办？- 爱可生开源社区](https://mp.weixin.qq.com/s/drlDvp7bfq5jt2M5pTqJCw) 。



### **如何清理 Redis 内存碎片**

Redis4.0-RC3 版本以后自带了内存整理，可以避免内存碎片率过大的问题。

直接通过 `config set` 命令将 `activedefrag` 配置项设置为 `yes` 即可。

```Bash
config set activedefrag yes
```

具体什么时候清理需要通过下面两个参数控制：

```Bash
# 内存碎片占用空间达到 500mb 的时候开始清理
config set active-defrag-ignore-bytes 500mb

# 内存碎片率大于 1.5 的时候开始清理
config set active-defrag-threshold-lower 50
```

通过 Redis 自动内存碎片清理机制可能会对 Redis 的性能产生影响，我们可以通过下面两个参数来减少对 Redis 性能的影响：

```Bash
# 内存碎片清理所占用 CPU 时间的比例不低于 20%
config set active-defrag-cycle-min 20

# 内存碎片清理所占用 CPU 时间的比例不高于 50%
config set active-defrag-cycle-max 50
```

另外，重启节点可以做到内存碎片重新整理。如果你采用的是高可用架构的 Redis 集群的话，你可以将碎片率过高的主节点转换为从节点，以便进行安全重启。



## 限制 maxmemory

> 当有多个 slave 连上达到内存上限的实例时，master 为同步 slave 的输出缓冲区所需内存不计算在使用内存中。这样当驱逐 key 时，就不会因网络问题 / 重新同步事件触发驱逐 key 的循环，反过来 slaves 的输出缓冲区充满了 key 被驱逐的 DEL 命令，这将触发删除更多的 key，直到这个数据库完全被清空为止

如果你需要附加多个 slave，建议你设置一个稍小 `maxmemory` 限制，这样系统就会有空闲的内存作为 slave 的输出缓存区。**建议该值设置为系统内存的 80%**，如 10G 内存，设置该值为 8G。

修改 `redis.conf` 文件，限制 Redis 可使用的**最大内存**

![image-20230103182552674](https://img-note.langyastudio.com/202301031825759.png?x-oss-process=style/watermark)



## 扩充实例容量

如果以上措施处理后内存仍然不足，此时需要扩充实例的容量。
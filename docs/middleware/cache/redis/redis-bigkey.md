简单来说，如果一个 key 对应的 value 所**占用的内存比较大**，那这个 key 就可以看作是 bigkey。具体多大才算大呢？有一个不是特别精确的参考标准：string 类型的 value 超过 10 kb，复合类型的 value 包含的元素超过 5000 个（对于复合类型的 value 来说，不一定包含的元素越多，占用的内存就越多）。

- 单个简单的 key 存储的 value 很大

- hash， set，zset，list 中存储过多的元素（以万为单位）

- 一个集群存储了上亿的 key，Key 本身过多也带来了更多的空间占用



### bigkey 有什么危害

除了会消耗更多的内存空间，bigkey 对性能也会有比较大的影响。



### 如何发现 bigkey

**使用 Redis 自带的 `--bigkeys` 参数来查找。**

```bash
# redis-cli -p 6379 --bigkeys

# Scanning the entire keyspace to find biggest keys as well as
# average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).

[00.00%] Biggest string found so far '"ballcat:oauth:refresh_auth:f6cdb384-9a9d-4f2f-af01-dc3f28057c20"' with 4437 bytes
[00.00%] Biggest list   found so far '"my-list"' with 17 items

-------- summary -------

Sampled 5 keys in the keyspace!
Total key length in bytes is 264 (avg len 52.80)

Biggest   list found '"my-list"' has 17 items
Biggest string found '"ballcat:oauth:refresh_auth:f6cdb384-9a9d-4f2f-af01-dc3f28057c20"' has 4437 bytes

1 lists with 17 items (20.00% of keys, avg size 17.00)
0 hashs with 0 fields (00.00% of keys, avg size 0.00)
4 strings with 4831 bytes (80.00% of keys, avg size 1207.75)
0 streams with 0 entries (00.00% of keys, avg size 0.00)
0 sets with 0 members (00.00% of keys, avg size 0.00)
0 zsets with 0 members (00.00% of keys, avg size 0.00
```

从这个命令的运行结果，我们可以看出：这个命令会扫描(Scan) Redis 中的所有 key ，会对 Redis 的性能有一点影响。并且，这种方式只能找出每种数据结构 top 1 bigkey（占用内存最大的 string 数据类型，包含元素最多的复合数据类型）。



**分析 RDB 文件**

通过分析 RDB 文件来找出 big key。这种方案的前提是你的 Redis 采用的是 RDB 持久化。

网上有现成的代码/工具可以直接拿来使用：

- [redis-rdb-tools](https://github.com/sripathikrishnan/redis-rdb-tools) ：Python 语言写的用来分析 Redis 的 RDB 快照文件用的工具
- [rdb_bigkeys](https://github.com/weiyanwei412/rdb_bigkeys) : Go 语言写的用来分析 Redis 的 RDB 快照文件用的工具，性能更好



### 如何解决

> 来自：https://mp.weixin.qq.com/s/UeJ4xo2cRQmXRopgGUbe7Q

#### value 占用空间过大

- 该对象需要每次都整存整取

可以尝试将对象**分拆成几个 key-value**， 使用 multiGet 获取值，这样分拆的意义在于分拆单次操作的压力，将操作压力平摊到多个 redis 实例中，降低对单个 redis 的 IO 影响 

   

- 该对象每次只需要存取部分数据

可以像第一种做法一样，分拆成几个 key-value，也可以将这个存储在一个 **hash 中，每个 field 代表一个具体的属性**，

使用 hget,hmget 来获取部分的 value，使用 hset，hmset 来更新部分属性   

 

#### value 存储元素过多   

以 hash 为例，原先的正常存取流程是  hget(hashKey, field) ; hset(hashKey, field, value)

现在，**固定一个桶的数量**，比如 10000， 每次存取的时候，先在本地计算 field 的 hash 值，模除 10000， 确定了该 field落在哪个 key 上。

newHashKey  =  hashKey + ( set, zset, list 也可以类似上述做法）

 

#### key 数量过多

如果 key 的个数过多会带来更多的内存空间占用

   i：key 本身的占用（每个 key 都会有一个 Category 前缀）

   ii：集群模式中，服务端需要建立一些 slot2key 的映射关系，这其中的指针占用在 key 多的情况下也是浪费巨大空间

这两个方面在 **key 个数上亿**的时候消耗内存十分明显（Redis 3.2 及以下版本均存在这个问题，4.0 有优化）；

所以减少 key 的个数可以减少内存消耗，可以参考的方案是转 Hash 结构存储，即原先是直接使用 Redis String 的结构存储，现在将多个 key 存储在一个 Hash 结构中，具体场景参考如下：



- key 本身就有很强的相关性

比如多个 key 代表一个对象，每个 key 是对象的一个属性，这种可直接按照特定对象的特征来设置一个新 Key——Hash 结构， 原先的 key 则作为这个新 Hash 的 field。

**举例说明：** 

原先存储的三个key 

user.zhangsan-id = 123; 

user.zhangsan-age = 18; 

user.zhangsan-country = china;   

这三个 key 本身就具有很强的相关特性，转成 Hash 存储就像这样 key = user.zhangsan

field:id = 123; 

field:age = 18; 

field:country = china;

即 redis 中**存储一个key** ：user.zhangsan， 他有三个 field， 每个 field + key 就对应原先的一个 key。

   

- key 本身没有相关性

预估一下总量，采取和上述第二种场景类似的方案，**预分一个固定的桶数量**

比如现在预估 key 的总数为 2亿，按照一个 hash 存储 100个 field 来算，需要 2亿 / 100 = 200W 个桶 (200W 个 key 占用的空间很少，2 亿可能有将近 20G )

现在按照 200W 固定桶分就是先计算出桶的序号 hash(123456789)  % 200W ， 这里最好保证这个 hash 算法的值是个正数，否则需要调整下模除的规则；

> 注意两个地方：1，hash 取模对负数的处理； 2，预分桶的时候， 一个hash 中存储的值最好不要超过 512 ，100 左右较为合适



#### 大 Bitmap 拆分

使用 bitmap 或布隆过滤器的场景，往往是数据量极大的情况，在这种情况下，Bitmap 和布隆过滤器使用空间也比较大，比如用于公司 userid 匹配的布隆过滤器，就需要 512MB 的大小，这对 redis 来说是绝对的大 value 了。

这种场景下，我们就需要对其进行拆分，拆分为足够小的 Bitmap，比如将 512MB 的大 Bitmap 拆分为 1024 个 512KB 的 Bitmap。

不过拆分的时候需要注意，要将每个 key 落在一个 Bitmap 上。有些业务只是把 Bitmap 拆开， 但还是当做一个整体的 bitmap 看， 所以一个 key 还是落在多个 Bitmap 上，这样就有可能导致一个 key 请求需要查询多个节点、多个 Bitmap。如下图，被请求的值被 hash 到多个 Bitmap 上，也就是 redis 的多个 key 上，这些 key 还有可能在不同节点上，这样拆分显然大大降低了查询的效率。

![图片](https://img-note.langyastudio.com/202211051728777.png?x-oss-process=style/watermark)

因此我们所要做的是把所有拆分后的 Bitmap 当作独立的 bitmap，然后通过 hash 将不同的 key 分配给不同的 bitmap上，而不是把所有的小 Bitmap 当作一个整体。这样做后每次请求都只要取 redis 中一个 key 即可。

![图片](https://img-note.langyastudio.com/202211051728771.png?x-oss-process=style/watermark)



有同学可能会问，通过这样拆分后，相当于 Bitmap 变小了，会不会增加布隆过滤器的误判率？实际上是不会的，布隆过滤器的误判率是哈希函数个数k，集合元素个数 n，以及 Bitmap 大小 m 所决定的，其约等于![图片](https://img-note.langyastudio.com/202211051728161.png?x-oss-process=style/watermark)。

因此如果我们在第一步，也就是在**分配 key 给不同 Bitmap 时，能够尽可能均匀的拆分**，那么 n／m 的值几乎是一样的，误判率也就不会改变。

同时，客户端也提供便利的 api （>=2.3.4版本）， setBits/ getBits 用于一次操作同一个 key 的多个 bit 值 。

建议 ：k 取 13 个， 单个 bloomfilter 控制在 512KB 以下
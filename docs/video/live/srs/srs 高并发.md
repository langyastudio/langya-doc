如何支持更多的人推流？如何支持更多的人观看？这本质上就是系统的水平扩展能力。



### 源站集群-更多推流

在SRS的角色中，Edge主要解决播放或下行的扩展能力，而Origin则是解决上行或推流的扩展能力，比如需要推1万路流。



#### Vhost方案

如下图所示：

![image-20201127175229546](https://img-note.langyastudio.com/20201127175229.png?x-oss-process=style/watermark)

> 此时需要终端指定不同的vhost



#### 源站集群方案

基于Origin Cluster源站集群扩展源站，如下图所示：

![img](https://img-note.langyastudio.com/20201127174104.webp?x-oss-process=style/watermark)

> 两个Origin服务器之间会互相查询流，若Edge请求的流不在本源站上，会将Edge定向到有流的Origin

由于源站是流的最终所在地，所以他本质上是有状态的，两个源站并不是完全等价的。而边缘可以认为是合并回源的代理，两个Edge是没有差别的，它们并没有存储流的信息，都是通过源站获取流。

因此，**推流的扩展能力，比播放的扩展能力，对系统的挑战是更大的**。



**通过Edge推流**

不建议直接推流到Origin源，而是通过Edge推流到Origin

![](https://img-note.langyastudio.com/20201127143308.png?x-oss-process=style/watermark)



#### 编写源站origin配置文件

The config for origin 19350:

```shell
# conf/origin.conf
listen              19350;
max_connections     1000;

daemon              off;
pid                 objs/origin.cluster.serverA.pid;
srs_log_file        ./objs/origin.cluster.serverA.log;

http_api {
    enabled         on;
    listen          9090;
}

vhost __defaultVhost__ {
    cluster {
        # local: It's an origin server, serve streams itself.
        # remote: It's an edge server, fetch or push stream to origin server.
        mode            local;

        # 源站集群只支持RTMP协议
        # 如果需要HTTP-FLV，可以加一个Edge将RTMP转成HTTP-FLV
        origin_cluster      on;

        # For origin (mode local) cluster, the co-worker's HTTP APIs.
        # This origin will connect to co-workers and communicate with them.
        # https://github.com/ossrs/srs/wiki/v3_CN_OriginCluster
        coworkers           127.0.0.1:9091;
    }
}
```



The config for origin 19351:

```shell
# conf/origin.conf
listen              19351;
max_connections     1000;

daemon              off;
pid                 objs/origin.cluster.serverB.pid;
srs_log_file        ./objs/origin.cluster.serverB.log;

http_api {
    enabled         on;
    listen          9091;
}

vhost __defaultVhost__ {
    cluster {
        # local: It's an origin server, serve streams itself.
        # remote: It's an edge server, fetch or push stream to origin server.
        mode            local;

        # 源站集群只支持RTMP协议
        # 如果需要HTTP-FLV，可以加一个Edge将RTMP转成HTTP-FLV
        origin_cluster      on;

        # For origin (mode local) cluster, the co-worker's HTTP APIs.
        # This origin will connect to co-workers and communicate with them.
        # https://github.com/ossrs/srs/wiki/v3_CN_OriginCluster
        coworkers           127.0.0.1:9090;
    }
}
```

- mode

     集群的模式，对于源站集群，值应该是local。

- origin_cluster:

    是否开启源站集群。

- coworkers:

    源站集群中的其他源站的HTTP API地址

如果流不在本源站，会通过HTTP API查询其他源站是否有流。如果流在其他源站，则返回RTMP 302重定向请求到该源站。如果所有源站都没有流则返回错误。

>  特别注意的是，如果流还没有开始推，那么服务器会返回失败，这点和流没有在源站集群的行为不同。
>
>  当源站独立工作时，会等待流推上来；当源站在源站集群中时，因为流可能不会推到本源站，所以等待流推上来没有意义。



### 边缘集群-更多播放

在监控或者一对一聊天场景中，一个流会被少量的播放器消费，一对一就是一个播放器消费，监控可能更特殊些可能没有人消费只有在某些时候才会被消费，比如`GB28181`使用SIP协议，在有播放器消费流时才邀请摄像头把流送上来。

在直播场景中，一个流会被非常多的播放器消费，比如一个球赛、国庆活动、一个电商大V的直播，直播对于播放的扩展能力是核心诉求，这也是`CDN`解决的关键问题之一。SRS也可以构建这样的能力，区别在于`CDN`的带宽成本会更低，而SRS一般部署在BGP云机房，网络稳定性更高但成本更高。



#### 方案

将源站部署在阿里云杭州ECS上，主播从上海使用OBS推流，杭州我们需要支持4K播放，北京我们需要支持8K播放，我们就可以在杭州和北京部署SRS边缘服务器，如下图所示：

![img](https://img-note.langyastudio.com/20210104140401.png?x-oss-process=style/watermark)

![image-20201127175141026](https://img-note.langyastudio.com/20201127175141.png?x-oss-process=style/watermark)

> 一个SRS源站，最多可以支持7K个边缘节点。如果7K个节点不够，还可以使用多级边缘服务器，完成无数个节点的扩展能力



#### 多CPU

SRS使用单进程单线程模型，即SRS只能使用一个CPU。

解决方案：

- 降低机器的CPU个数，比如申请ECS时只申请2个CPU的，这样就不会有多个CPU的烦恼了

- 使用K8S和Docker虚拟化资源，使用Docker跑SRS，这样每个SRS最多用100%，但可以跑多个Docker

- 使用SRS的多进程和集群方案，下面详细讲这种：

    通过多个SRS进程，将多核能力利用起来

![img](https://img-note.langyastudio.com/20201127173734.png?x-oss-process=style/watermark)

> 这种部署结构只能扩展源站的播放能力，因为新增的是Edge服务器，流最终还是要回源到Origin服务器



#### 编写边缘Edge配置文件

在源站集群后面挂一系列的Edge服务器，参考[这里](https://github.com/ossrs/srs/issues/464#issuecomment-366169962)，Edge服务器可以转换协议，支持RTMP和HTTP-FLV，同时支持源站origin故障时自动切换，不中断客户端。

We can also start a edge server, which will follow the RTMP 302

可以配置多个edge，The config for edge 29350:

```nginx
listen              29350;
max_connections     1000;

daemon              off;
pid                 objs/edge.serverA.pid;
srs_log_file        ./objs/edge.serverA.log;

http_server {
    enabled         on;
    listen          8081;
    dir             ./objs/nginx/html;
}

vhost __defaultVhost__ {
    cluster {
        mode            remote;
        origin          127.0.0.1:19350 127.0.0.1:19351;
    }
    
    http_remux {
        enabled     on;
        mount       [vhost]/[app]/[stream].flv;
        hstrs       on;
    }
}
```

> The edge will try to fetch stream from 19350, then it'll be redirected to 19351.
>
> 可以配合[Vhost](https://github.com/ossrs/srs/wiki/v3_CN_RtmpUrlVhost)做更多简易的逻辑处理



### 大规模集群

![image-20201127175123916](https://img-note.langyastudio.com/20201127175124.png?x-oss-process=style/watermark)

- 以上扩展能力，可以组合使用，比如源站可以是单个SRS，也可以用一个Origin和多个Edge组成小集群源站，再让Edge使用Reuse Port对外就是一个IP和端口。

- Edge拉流可以支持多个协议，对于扩展同样是适用的，比如WebRTC也可以使用Edge扩展播放的能力，同样GB28181推流后播放的协议和源站架构无关。但目前WebRTC推流和源站集群的能力还在开发中。

- 一般来说，Edge就是为了扩展播放的能力，但推流也可以走Edge这是为了让推流的地址更简单，而不用关注Origin的部署结构。一般来说，源站Origin是为了扩展收流的能力，但对于WebRTC这种结构，可能没有固定的Origin和Edge，它可能需要的是一种切换角色的能力。

    

### 注意事项

这里列出来一些不建议的做法：

- 不建议Forward，Forward并不是扩展集群的能力，而是把流复制一份给别的系统，一般在系统迁移，或者需要两个系统有同样的流时，才需要用Forward。
- 不建议在边缘切HLS和DVR，因为边缘没有流，所以无法切片HLS，只能在有播放RTMP或FLV时，才会去把流拉到边缘。所以如果边缘做HLS切片，会发现播放HLS前要播放RTMP，这是很奇怪的做法。
- 不建议把所有业务放一台服务器，比如有些流是指需要出HLS，有些流只需要DVR，有些流只需要FLV，那么这些流就应该分成不同的Vhost，送到不同的源站处理，这样可以避免互相干扰。比如HLS和DVR需要写磁盘，可能会导致IO负载高，可能会影响到FLV流。
- 不建议把业务做到SRS中，比如无人播放时停止推流，那么不应该让SRS断开连接，而应该业务系统观察到无人播放时，通知推流停止推流。这样可以让SRS集中在流媒体处理，而不是因为业务代码Crash。
- 不建议Edge前面挂F5或Proxy，Edge本身就是Proxy，再挂个Proxy意欲何为呢，流量一点都没有节省。有些可能是为了让客户端访问的IP保持一致，那么可以用DNS域名方式，这样客户端看到的都是一个DNS名称，会解析成不同的Edge的IP。  

> https://mp.weixin.qq.com/s/pd9YQS0WR3hSuHybkm1F7Q
>
> https://www.processon.com/view/link/5e3f5581e4b0a3daae80ecef


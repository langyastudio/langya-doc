## 低延时模式

### RTMP延时特点

**延迟较低：**

比起YY的那种UDP私有协议，RTMP算延迟大的（延迟在1-3秒），比起HTTP流的延时（一般在10秒以上）RTMP算低延时。一般的直播应用，只要不是电话类对话的那种要求，RTMP延迟是可以接受的。在一般的视频会议（参考SRS的视频会议延时）应用中，RTMP延时也能接受，原因是别人在说话的时候我们一般在听，实际上1秒延时没有关系，我们也要思考（话说有些人的CPU处理速度还没有这么快）。

**有累积延迟：**

技术一定要知道弱点，RTMP有个弱点就是累积误差，原因是RTMP基于TCP不会丢包。所以当网络状态差时，服务器会将包缓存起来，导致累积的延迟；待网络状况好了，就一起发给客户端。这个的对策就是，当客户端的缓冲区很大，就断开重连。当然SRS也提供配置。



### 延时影响因素

- GOP

    SRS可以关闭GOP的cache来避免这个影响

- 服务器性能太低

    服务器来不及发送数据

- 播放端

    譬如flash客户端的NetStream.bufferTime设置为10秒，那么延迟至少10秒以上

- 推流端

    GOP、Profile、Tune等编码参数

>  SRS集群（边缘）不会增加延迟



### 编写配置文件

配置SRS为低延时模式，可以将RTMP延迟降低到0.8-3秒：

```shell
# conf/realtime.conf
listen              1935;
max_connections     1000;
vhost __defaultVhost__ {
    tcp_nodelay     on;
    min_latency     on;

    play {
        gop_cache       off;
        queue_length    10;
        mw_latency      100;
    }

    publish {
        mr off;
    }
}
```

> **低延时模式影响性能**，更多参考如下：
>
> https://github.com/ossrs/srs/wiki/v3_CN_LowLatency



可以将HLS延迟降低到3-5秒：

```shell
listen              1935;
max_connections     1000;
vhost __defaultVhost__ {
    hls {
        enabled            on;
        hls_path           ./objs/nginx/html;
        hls_fragment       0.2;
        hls_window         2;
        hls_wait_keyframe  off;
    }
}
```

> HLS还可考虑使用FLV协议替换



### 编码参数设置

`GOP = 1；Profile = baseline； Tune = zerolatency` (主要是GOP编码参数)，例如OBS的编码设置如下：

![image-20201126174856200](https://img-note.langyastudio.com/20201126174856.png?x-oss-process=style/watermark)



测试结果，RTMP为1秒、m3u8为3秒：

![image-20201126181308096](https://img-note.langyastudio.com/20201126181308.png?x-oss-process=style/watermark)

![image-20201126181013211](https://img-note.langyastudio.com/20201126181013.png?x-oss-process=style/watermark)



## reload

SRS 配置支持 Reload，即在不中断服务的前提下替换应用配置文件并生效

### 配置方式

修改配置文件，相关配置项如下：

```shell
#do not support reload.
daemon off;
# Whether auto reload by watching the config file by inotify.
inotify_auto_reload on;
```

> 如果服务器支持使用inotify_auto_reload，则配置文件替换更新后直接生效，无需调用命令行进行手动reload
>
> 此时如果srs使用srs.conf配置文件，则替换该文件即可



### 应用场景

- 配置快速生效

    不用重启服务，修改配置后，只需要执行`killall -1 srs`或`/etc/init.d/srs reload`即可生效

- 不中断服务

    

### 不支持的情况

- deamon

    是否后台启动，开启后导致reload失效

- mode

    修改vhost的模式，即vhost是源站还是边缘

### HLS 服务

HLS 的主要优势是：

- 性能高：和HTTP一样
- 穿墙：和HTTP一样
- 原生支持很好：IOS上支持完美，Android 3 以上支持。PC/HTML5 or flash上现在也有各种as插件支持HLS

HLS的主要劣势是：

- **实时性差**：基本上HLS的延迟在10秒以上
- 文件碎片：若分发HLS，码流低，切片较小时，小文件分发不是很友好。特别是一些对存储比较敏感的情况，譬如源站的存储，嵌入式的SD卡



#### 编写配置文件hls.conf

```shell
# 安装后，存在默认的配置文件/usr/local/srs/conf/hls.conf
listen              1935;
max_connections     1000;
daemon              off;
srs_log_tank        console;

#SRS内置的HTTP服务器分发HLS切片
#也可以使用Nginx等Web服务器分发
http_server {
    enabled         on;
    listen          8080;
    dir             ./objs/nginx/html;
    crossdomain     on;
}

vhost __defaultVhost__ {
    hls {
        enabled         on;
        hls_path        ./objs/nginx/html;
        hls_fragment    10;
        hls_window      60;
    }
}
```

> 详细配置：[HLS分发: HLS流程](https://github.com/ossrs/srs/wiki/v3_CN_DeliveryHLS)



#### 启动SRS

```shell
#cd到/usr/local/srs目录
./objs/srs -c conf/hls.conf
```



#### 推流

使用 [OBS](https://obsproject.com/) 推流，**H264 + AAC** 进行视音频编码，推流地址如下：

```
rtmp://192.168.123.22/live/livestream
```



#### 播流

[srs 播放器](http://ossrs.net/srs.release/trunk/research/players/srs_player.html)

```
http://192.168.123.22:8080/live/livestream.m3u8
```

---



### FLV 服务

SRS所指的HTTP FLV流，是严格意义上的直播流，有RTMP的所有特征，譬如集群、**低延迟**、热备、GOP cache等，主要的优势在于：

- 互联网流媒体实时领域，还是RTMP。HTTP-FLV和RTMP的延迟一样，因此可以满足延迟的要求。

- 穿墙：很多防火墙会墙掉RTMP，但是不会墙HTTP，因此HTTP FLV出现奇怪问题的概率很小。

- 调度：RTMP也有个302，可惜是播放器as中支持的，HTTP FLV流就支持302方便CDN纠正DNS的错误。

- 容错：SRS的HTTP FLV回源时可以回多个，和RTMP一样，可以支持多级热备。

- 通用：Flash可以播RTMP，也可以播HTTP FLV。自己做的APP，也都能支持。主流播放器也都支持http flv的播放。

- 简单：FLV是最简单的流媒体封装，HTTP是最广泛的协议，这两个到一起维护性很高，比RTMP简单多了。



#### 编写配置文件http.flv.live.conf

```shell
#安装后，存在默认的配置文件/usr/local/srs/conf/http.flv.live.conf
listen              1935;
max_connections     1000;
daemon              off;
srs_log_tank        console;

http_server {
    enabled         on;
    listen          8080;
    dir             ./objs/nginx/html;
}

vhost __defaultVhost__ {
    http_remux {
        enabled     on;
        mount       [vhost]/[app]/[stream].flv;
        hstrs       on;
    }
}
```



#### 启动SRS

```shell
#cd到/usr/local/srs目录
./objs/srs -c conf/http.flv.live.conf
```



#### 播流

[srs 播放器](http://ossrs.net/srs.release/trunk/research/players/srs_player.html)

```
http://192.168.123.22:8080/live/livestream.flv
```


### HTTP回调

[HttpCallBack](https://github.com/ossrs/srs/wiki/v3_CN_HTTPCallback) 

- 事件：发生该事件时，回调指定的HTTP地址
- HTTP地址：可以支持多个，以空格分隔，SRS会依次回调这些接口
- 数据传输：SRS将数据POST到HTTP接口



#### 修改配置文件

配置文件需要开启http_hooks：

```shell
listen              1935;
max_connections     1000;

vhost __defaultVhost__ {

	http_hooks {
        enabled on;
        on_connect http://127.0.0.1:8631/api/hook/index;
        on_close http://127.0.0.1:8631/api/hook/index;
        on_publish http://127.0.0.1:8631/api/hook/index;
        on_unpublish http://127.0.0.1:8631/api/hook/index;
        on_play http://127.0.0.1:8631/api/hook/index;
        on_stop http://127.0.0.1:8631/api/hook/index;
        on_dvr http://127.0.0.1:8631/api/hook/index; 
        on_hls_notify http://127.0.0.1:8631/api/hook/index;
    }

}
```

#### 回调事件POST的数据

| 事件          | 数据                                                         | 说明                                                         |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| on_connect    | {"action": "on_connect","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","tcUrl": "rtmp://video.test.com/live?key=d2fa801d08e3f90ed1e1670e6e52651a","pageUrl": "http://www.test.com/live.html"} | 当客户端连接到指定的vhost和app                               |
| on_close      | {"action": "on_close","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","send_bytes": 10240, "recv_bytes": 10240} | 当客户端关闭连接，或者SRS主动关闭连接                        |
| on_publish    | {"action": "on_publish","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","tcUrl" => "rtmp://video.test.com/live?token=xxx&salt=yyy","stream": "livestream", "param":"?token=xxx&salt=yyy"} | 当客户端发布流时，譬如flash/FMLE方式推流到服务器             |
| on_unpublish  | {"action": "on_unpublish","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","stream": "livestream", "param":"?token=xxx&salt=yyy"} | 当客户端停止发布流                                           |
| on_play       | {"action": "on_play","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","stream": "livestream", "param":"?token=xxx&salt=yyy","pageUrl": "http://www.test.com/live.html"} | 当客户端开始播放流                                           |
| on_stop       | {"action": "on_stop","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","stream": "livestream", "param":"?token=xxx&salt=yyy"} | 当客户端停止播放时。备注：停止播放可能不会关闭连接，还能再继续播放。 |
| on_dvr        | {"action": "on_dvr","client_id": 1985,"ip": "192.168.1.10", "vhost": "video.test.com", "app": "live","stream": "livestream", "param":"?token=xxx&salt=yyy","cwd": "/usr/local/srs","file": "./objs/nginx/html/live/livestream.1420254068776.flv"} | 当DVR录制关闭一个文件                                        |
| on_hls_notify | HTTP GET and use the variable following :app、stream、param、ts_url | used to push file to cdn network                             |



#### 推流授权

回调的事件很多，这里介绍常用的推送RTMP流的场景：`on_publish` - 发布RTMP流时，需要安全校验

**推流地址：**

`rtmp://192.168.123.22/live?key=d33c3468&token=5ip0xBNk/livestream`

**HTTP回调：**

打印的POST数据如下：

```
(
  'action' => 'on_publish',
  'client_id' => 15694,
  'ip' => '192.168.123.100',
  'vhost' => '__defaultVhost__',
  'app' => 'live',
  'tcUrl' => 'rtmp://192.168.123.22/live?key=d33c3468&token=5ip0xBNk',
  'stream' => 'livestream',
  'param' => '?key=bd990b32&token=8PJTEelH',
)
```

此时解析 `param` 或 `tcUrl` 参数获取推流时传递的`key`、`token`等参数即可根据自己的业务逻辑进行授权判断是否合法。

如果合法，回调的接口需要返回 HTTP Code 200并且response内容为整数错误码（0表示成功），其他错误码会断开客户端连接。



### 统计

[SRS API](https://github.com/ossrs/srs/wiki/v3_CN_HTTPApi)

SRS的HTTP接口遵循最简单原则，主要包括：

- 只提供json数据格式接口，要求请求和响应的数据全都是json，并支持跨域
- [srs-console](https://github.com/ossrs/srs-console) 可访问SRS的API，提供管理后台
- 发生错误时，支持HTTP错误码，或者json中的code错误码

#### 修改配置文件

配置文件需要开启http-api：

```shell
# the config for srs to delivery RTMP
# @see https://github.com/ossrs/srs/wiki/v1_CN_SampleRTMP
# @see full.conf for detail config.

listen              1935;
max_connections     1000;

#stats
#the http api, for instance, /api/v1/summaries will show these data.
stats {
    # the index of device ip
    network         0;
    # ignore the device of /proc/diskstats if not configed.
    disk            sda sdb xvda xvdb;
}

#http api
#api of srs
http_api {
    enabled         on;
    listen          1985;
    crossdomain     on;
}

vhost __defaultVhost__ {
	
}
```

#### 主要接口

地址是：`http://192.168.123.22:1985/api/v1`，主要包含的子api有：

| API               | Example                   | Description                                                  |
| ----------------- | ------------------------- | ------------------------------------------------------------ |
| server            | 4481                      | 服务器标识                                                   |
| versions          | /api/v1/versions          | 获取服务器版本信息                                           |
| summaries         | /api/v1/summaries         | 获取服务器的摘要信息                                         |
| rusages           | /api/v1/rusages           | 获取服务器资源使用信息                                       |
| self_proc_stats   | /api/v1/self_proc_stats   | 获取服务器进程信息                                           |
| system_proc_stats | /api/v1/system_proc_stats | 获取服务器所有进程情况                                       |
| meminfos          | /api/v1/meminfos          | 获取服务器内存使用情况                                       |
| authors           | /api/v1/authors           | 获取作者、版权和License信息                                  |
| features          | /api/v1/features          | 获取系统支持的功能列表                                       |
| requests          | /api/v1/requests          | 获取请求的信息，即当前发起的请求的详细信息                   |
| vhosts            | /api/v1/vhosts            | 获取服务器上的vhosts信息                                     |
| streams           | /api/v1/streams           | 获取服务器的streams信息                                      |
| clients           | /api/v1/clients           | 获取服务器的clients信息(默认start为0，count为10，即查询头10个clients)。 |
| configs           | /api/v1/configs           | CUID配置，RAW API                                            |

追加API参数即可访问接口，例如`http://192.168.123.22:1985/api/v1/summaries`



#### 管理控制台

[srs-console](https://github.com/ossrs/srs-console) 可访问SRS的API，提供管理后台服务。填写服务器IP与API端口（基于配置文件）点击【连接到SRS】，可以进行在线管理SRS监控、视频流、客户端等

![image-20201126164520848](https://img-note.langyastudio.com/20201126164520.png?x-oss-process=style/watermark)
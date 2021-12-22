## HLS CDN

**提高 CDN  命中率**

http_hooks 下增加：

    # when srs reap a ts file of hls, call this hook,
    # used to push file to cdn network, by get the ts file from cdn network.
    # so we use HTTP GET and use the variable following:
    #       [app], replace with the app.
    #       [stream], replace with the stream.
    #       [param], replace with the param.
    #       [ts_url], replace with the ts url.
    # ignore any return data of server.
    # @remark random select a url to report, not report all.
    on_hls_notify   http://your-cdn/[app]/[ts_url][param]; 

hls 下增加：

```
# the max size to notify hls,
# to read max bytes from ts of specified cdn network,
# @remark only used when on_hls_notify is config.
# default: 64
hls_nb_notify   1024;
```



## 阿里云直播CDN

使用域名推流到 srs，再 forward 到阿里云直播中心

### 背景
- 在ECS服务器自建了SRS流媒体服务
  域名为 srs.xxx.com
- 在阿里云CDN直播中心创建了推流域名
  域名为 cloud-push.xxx.com
- 业务需求
  将本地流推送到SRS流媒体服务（srs.xxx.com），再由SRS流媒体服务器forward到阿里云CDN（cloud-push.xxx.com），从而实现大规模播放直播的需求
- 版本
  使用 3.0.156 版本测试



https://github.com/ossrs/srs/issues/2130

https://github.com/ossrs/srs/issues/1342

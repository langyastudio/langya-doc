## SRS

### SRS 转发 CDN 失败

基于阿里云 ECS 自建的 srs 流媒体服务器 forward 流到阿里云 CDN 失败

解决方案

- 使用 IP 地址推流

> 可能是 srs 的bug



### 推流失败/丢包率严重

> rtmp publish timeout 5000ms, nb_msgs=857

![image-20210118172916140](https://img-note.langyastudio.com/20210118172916.png?x-oss-process=style/watermark)

**原因**

推流超时

publish timeout means your rtmp publish client NOT send ANY new rtmp msg in 5 second
srs receive RTMP msg timeout and close the session

**解决**

客户端网络问题 —— 通过重启路由器、交换机、设备网络配置等方式解决



### 拉流失败

> parse message : grow buffer : read bytes : timeout 15000 ms

![image-20210118173918484](https://img-note.langyastudio.com/20210118173918.png?x-oss-process=style/watermark)

**原因**

网络断流



### rtmp busy

```
[2021-02-01 13:38:07.353][Error][9][80476][11] serve error code=1028 : service cycle : rtmp: stream service : rtmp: stream /live/xsdgz-gzb-cide is busy
```

**原因**

流被占用



**解决**

- 同一个流被多个客户端推送
- 摄像机组播等影响



## 转码

### 转码后的 m3u8 文件播放无声音

**问题**

录制的 ts 文件列表合成为 mp4，再转码为 m3u8 时，web 端播放无声音/VLC 播放画面静止

**原因**

原始录制的 ts 文件存在时间戳中断、缺数据等问题，导致合并的 mp4 文件的帧率异常

![img](https://img-note.langyastudio.com/20210316175116.png?x-oss-process=style/watermark)

**解决**

使用参数 `-r 25`，强制统一帧率，使得输出视频的帧率不过高



### 多视频流

使用参数 `-map 0 `，使得所有的流都参与到转换中去



### iphone 黑屏 或 画面卡顿

视频的每个 idr 都带有 sps、pps 信息，是否都完整带有
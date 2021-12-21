## srs 与主流流媒体服务器对比

目前主流的流媒体服务器主要有 nginx-rtmp、crtmpd、wowza、red5、adobe fms等。



### **支持的网络协议对比**

协议是服务器的基础，协议决定了关键应用场景，譬如毫秒级别延时只能用udp，秒级别延迟用RTMP，十秒级别可以用HLS。

| Feature     | SRS        | NGINX  | CRTMPD | AMS    | WOWZA  |
| ----------- | ---------- | ------ | ------ | ------ | ------ |
| RTMP        | Stable     | Stable | Stable | Stable | Stable |
| HLS         | Stable     | Stable | X      | Stable | Stable |
| HTTP FLV    | Stable     | X      | X      | X      | X      |
| HLS(aonly)  | Stable     | X      | X      | Stable | Stable |
| HDS         | Experiment | X      | X      | Stable | Stable |
| MPEG-DASH   | Experiment | X      | X      | X      | X      |
| HTTP Server | Stable     | Stable | X      | X      | Stable |



### **性能对比**

单进程的SRS支持7k以上的下行并发，如果码率是1Mbps，妥妥的7Gbps的下行流量。

| Feature         | SRS                                                         | NGINX  | CRTMPD | AMS  | WOWZA |
| --------------- | ----------------------------------------------------------- | ------ | ------ | ---- | ----- |
| Concurrency     | 7.5k                                                        | 3k     | 2k     | 2k   | 3k    |
| MultipleProcess | [Stable](https://github.com/ossrs/srs/wiki/v3_CN_REUSEPORT) | Stable | X      | X    | X     |
| RTMP Latency    | 0.1s                                                        | 3s     | 3s     | 3s   | 3s    |
| HLS Latency     | 10s                                                         | 30s    | X      | 30s  | 30s   |



### **提供服务对比**

| Feature        | SRS    | NGINX  | CRTMPD | AMS    | WOWZA  |
| -------------- | ------ | ------ | ------ | ------ | ------ |
| DVR            | Stable | Stable | X      | X      | Stable |
| DVR API        | Stable | Stable | X      | X      | X      |
| DVR MP4        | Stable | X      | X      | X      | X      |
| EXEC           | Stable | Stable | X      | X      | X      |
| Transcode      | Stable | X      | X      | X      | Stable |
| HTTP API       | Stable | Stable | X      | X      | Stable |
| HTTP RAW API   | Stable | X      | X      | X      | X      |
| HTTP hooks     | Stable | X      | X      | X      | X      |
| GopCache       | Stable | X      | X      | Stable | X      |
| Security       | Stable | Stable | X      | X      | Stable |
| Token Traverse | Stable | X      | X      | Stable | X      |



### **集群对比**

| Feature     | SRS    | NGINX | CRTMPD | AMS    | WOWZA  |
| ----------- | ------ | ----- | ------ | ------ | ------ |
| RTMP Edge   | Stable | X     | X      | Stable | X      |
| RTMP Backup | Stable | X     | X      | X      | X      |
| VHOST       | Stable | X     | X      | Stable | Stable |
| Reload      | Stable | X     | X      | X      | X      |
| Forward     | Stable | X     | X      | X      | X      |
| ATC         | Stable | X     | X      | X      | X      |
| Docker      | Stable | X     | X      | X      | X      |



**Stream Caster**

| Feature       | SRS        | NGINX | CRTMPD | AMS  | WOWZA  |
| ------------- | ---------- | ----- | ------ | ---- | ------ |
| Ingest        | Stable     | X     | X      | X    | X      |
| Push MPEGTS   | Experiment | X     | X      | X    | Stable |
| Push RTSP     | Experiment | X     | Stable | X    | Stable |
| Push HTTP FLV | Experiment | X     | X      | X    | X      |



**系统调试**

| Feature      | SRS    | NGINX | CRTMPD | AMS  | WOWZA |
| ------------ | ------ | ----- | ------ | ---- | ----- |
| BW check     | Stable | X     | X      | X    | X     |
| Tracable Log | Stable | X     | X      | X    | X     |



**其他**

| Feature        | SRS    | NGINX  | CRTMPD | AMS  | WOWZA |
| -------------- | ------ | ------ | ------ | ---- | ----- |
| ARM/MIPS       | Stable | Stable | X      | X    | X     |
| Client Library | Stable | X      | X      | X    | X     |



**文档**

| Feature     | SRS    | NGINX   | CRTMPD | AMS  | WOWZA  |
| ----------- | ------ | ------- | ------ | ---- | ------ |
| Demos       | Stable | X       | X      | X    | X      |
| WIKI(EN+CN) | Stable | EN only | X      | X    | Stable |



----

SRS优势的基础在于基础架构，采用自实现的[ST](https://github.com/ossrs/state-threads)轻量线程（epoll+send）。主要优势体现在以下几点：

- **简化**

- **更高性能**

- 配置简单

- 支持Reload

    不影响在线用户，想怎么改都行

- 快速重启

    SRS重启以毫秒计算

- 可追溯日志

    记录完整日志，都有错误码，而且有client id，可以查询到某个客户端的整个信息

- 支持热备

    SRS边缘可以回多个源站，一个挂了切另外一个

- url格式简单

    把rtmp换成http，后面加上.m3u8就是HLS，多么简单！

- 支持转码

    SRS使用ffmpeg做了支持

- 支持录制

- **开源代码**

    

例如HLS热备的简化 — 使用standby方案替代完全热备：

- 编码器输出相同的流给异地机房的两个流媒体服务器，两个服务器之间保持通信。
- 两个服务器一个时刻只有一个输出HLS切片，另外一个standby。
- 当另外一个服务器挂掉时，standby的立刻开始切片。
- 两个服务器的切片都输出到一个存储。



> https://github.com/winlinvip/srs/tree/3.0release#performance



## 安装与部署rtmp服务

SRS主要运行在Linux系统上，譬如Centos和Ubuntu，包括x86、x86-64、ARM和MIPS。其他的OS可以使用 [srs-docker](https://github.com/ossrs/srs-docker)开发和运行，比如macOS、Windows等。

SRS可以在一台服务器上运行集群，或者在多台服务器上也可以运行集群。SRS是单进程模型，不支持多进程；您可以使用 [go-oryx ](https://github.com/ossrs/go-oryx)支持多进程。



### 安装

#### 测试环境

CentOS 8 x64

基于安装包安装



#### 下载与安装

SRS发布版本提供安装包的下载，访问 [**ossrs.net**](http://ossrs.net/srs.release/releases/download.html/) 下载安装包，如：`SRS-CentOS7-x86_64-3.0.153.zip`

![image-20201125151534988](https://img-note.langyastudio.com/20201125151535.png?x-oss-process=style/watermark)

 

上传到Linux系统并执行安装：

````shell
#SRS-CentOS7-x86_64-3.0.153.zip文件名替换为实际的文件名
unzip SRS-CentOS7-x86_64-3.0.153.zip
cd ./SRS-CentOS7-x86_64-3.0.153/
./INSTALL 
````

![image-20201125152129573](https://img-note.langyastudio.com/20201125152129.png?x-oss-process=style/watermark)

>若您需要自己编译SRS，请参考[编译SRS](https://github.com/ossrs/srs/wiki/v3_CN_Build)



### RTMP部署

```shell
#安装的目录
#/usr/local/srs
#默认的可用配置文件
#/usr/local/srs/conf/srs.conf
```



#### 编写配置文件rtmp.conf

```shell
# 安装后，存在默认的配置文件/usr/local/srs/conf/rtmp.conf
listen              1935;
max_connections     1000;
vhost __defaultVhost__ {
}
```



#### 启动SRS

```shell
#cd到/usr/local/srs目录
./objs/srs -c conf/rtmp.conf
```



#### 推流

使用 [OBS](https://obsproject.com/) 推流，安装软件后依次点击【设置 - 推流】并填写相关信息，例如

```
#服务器的地址
rtmp://192.168.123.22/live
#串流密钥
livestream
```

![image-20201125154920501](https://img-note.langyastudio.com/20201125154920.png?x-oss-process=style/watermark)

再设置【场景-来源】添加视音频或图像等采集源，点击【开始推流】，此时OBS的右下角会显示推流成功的状态如下：

![](https://img-note.langyastudio.com/20201125155428.png?x-oss-process=style/watermark)



#### 播流

[FLV播放器](http://bilibili.github.io/flv.js/demo/)

[HLS播放器](https://hls-js.netlify.app/demo/)

[RTMP播放器](http://ossrs.net/srs.release/trunk/research/players/srs_player.html?vhost=__defaultVhost__&autostart=true&server=192.168.1.170&app=live&stream=livestream&port=1935)

填写对应协议的播放地址，例如`rtmp://192.168.123.22/live/livestream`，点击【播放视频】效果如下：

![image-20201125155719767](https://img-note.langyastudio.com/20201125155719.png?x-oss-process=style/watermark)



#### 常见问题

有时候启动没有问题，但是就是看不了，原因是防火墙和selinux开着。

可以用下面的方法关掉防火墙：

```shell
# disable the firewall
sudo /etc/init.d/iptables stop
sudo /sbin/chkconfig iptables off
```

selinux也需要disable，运行命令`getenforce`，若不是Disabled，执行下面的步骤：

```shell
#编辑配置文件
sudo vi /etc/sysconfig/selinux
#把SELINUX的值改为disabled
SELINUX=disabled
#重启系统
sudo init 6
```
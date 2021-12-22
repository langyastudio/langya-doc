### HLS 是什么
`HTTP Live Streaming`（缩写是 HLS）是一个由苹果公司提出的基于 HTTP 的流媒体网络传输协议。​是苹果公司QuickTime X 和 iPhone 软件系统的一部分。 `它的工作原理是把整个流分成一个个小的基于 HTTP 的文件来下载，每次只下载一些`。当媒体流正在播放时，客户端可以选择从许多不同的备用源中以不同的速率下载同样的资源，允许流媒体会话适应不同的数据速率。

在开始一个流媒体会话时，客户端会下载一个包含元数据的 extended M3U (m3u8) playlist 文件，用于寻找可用的媒体流。HLS 只请求基本的 HTTP 报文，与实时传输协议（RTP) 不同，HLS 可以穿过任何允许 HTTP 数据通过的防火墙或者代理服务器。​它也很容易使用内容分发网络来传输媒体流。



目前直播主流协议有 RTMP、HLS、FLV等

- `RTMP` 指 Adobe 的 RTMP(Realtime Message Protocol)，广泛应用于低延时直播，也是编码器和服务器对接的实际标准协议，在 PC（Flash）上有最佳观看体验和最佳稳定性。

- `HLS` 指 Apple 的 HLS(Http Live Streaming)，本身就是 Live（直播）的，不过 Vod（点播）也能支持。HLS 是 Apple平台的标准流媒体协议，和 RTMP 在 PC 上一样支持得天衣无缝。
![这里写图片描述](https://img-note.langyastudio.com/20201113144827.jpeg?x-oss-process=style/watermark)



### HLS 主要的应用场景

- **跨平台**

    PC 主要的直播方案是 RTMP，也有一些库能播放 HLS，譬如 jwplayer，基于 osmf 的 hls 插件也一大堆。所以实际上如果选一种协议能跨 PC/Android/IOS，那就是 HLS。

- **IOS 上苛刻的稳定性要求**

    IOS 上最稳定的当然是 HLS，稳定性不差于 RTMP 在 PC-flash 上的表现。

- **友好的 CDN 分发方式**

    目前 CDN 对于 RTMP 也是基本协议，但是 HLS 分发的基础是 HTTP，所以 CDN 的接入和分发会比 RTMP 更加完善。能在各种 CDN 之间切换，RTMP虽然也能只是可能需要对接测试。

- **简单**

    HLS 作为流媒体协议非常简单，apple 支持得也很完善。Android 对 HLS 的支持也会越来越完善。至于 DASH/HDS，好像没有什么特别的理由，就像  linux 已经大行其道而且开放，其他的系统很难再广泛应用。

> 总之，SRS 支持 HLS 主要是作为输出的分发协议，直播以 RTMP+HLS 分发，满足各种应用场景，点播以 HLS 为主。



### HLS 协议详解

HLS 是提供一个 m3u8 地址，Apple 的 Safari 浏览器直接就能打开 m3u8 地址，譬如：
```url
http://demo.srs.com/live/livestream.m3u8
```
Android 不能直接打开，需要使用 html5 的 video 标签，然后在浏览器中打开这个页面即可，譬如：
```html
<!-- livestream.html -->
<video width="640" height="360"
        autoplay controls autobuffer 
        src="http://demo.srs.com/live/livestream.m3u8"
        type="application/vnd.apple.mpegurl">
</video>
```



#### HLS 协议规定

- 视频的封装格式是 TS。

- 视频的编码格式为 H264, 音频编码格式为 MP3、AAC 或者 AC-3。

- 除了 TS 视频文件本身，还定义了用来控制播放的 m3u8 文件（文本文件）。



#### HLS 协议说明

HLS 的 m3u8，是一个 ts 的列表，也就是告诉浏览器可以播放这些 ts 文件，譬如：
```ini
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-ALLOW-CACHE:YES
#EXT-X-TARGETDURATION:13
#EXT-X-MEDIA-SEQUENCE:430
#EXT-X-PLAYLIST-TYPE:VOD
#EXTINF:11.800
news-430.ts
#EXTINF:10.120
news-431.ts
#EXT-X-DISCONTINUITY
#EXTINF:11.952
news-430.ts
#EXTINF:12.640
news-431.ts
#EXTINF:11.160
news-432.ts
#EXT-X-DISCONTINUITY
#EXTINF:11.751
news-430.ts
#EXTINF:2.040
news-431.ts
#EXT-X-ENDLIST
```
- `EXTM3U`
每个M3U文件第一行必须是这个tag，提供标示作用

- `EXT-X-VERSION`
用以标示协议版本。这里是3， 那么这里用的就是 HLS 协议第三个版本，此标签只能有0或1个，不写代表使用版本1

- `EXT-X-TARGETDURATION`
所有切片的最大时长，有些Apple设备这个参数不正确会无法播放。

- `EXT-X-MEDIA-SEQUENCE`
切片的开始序号。每一个切片都有唯一的序号，相邻之间序号+1。这个编号会继续增长，保证流的连续性。

- `EXTINF`
ts 切片的实际时长。duration : 媒体持续时间
```
#EXTINF <duration>,<title>
```
- `EXT-X-PLAYLIST-TYPE`
类型，vod 表示点播。

- `EXT-X-ENDLIST`
文件结束符号。表示不再向播放列表文件添加媒体文件。

> [Apple HTTP Live Streaming 官方说明](https://developer.apple.com/library/archive/technotes/tn2288/_index.html)